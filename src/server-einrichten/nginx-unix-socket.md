# Unix-Socket bei NGINX

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Der Webserver **NGINX** (siehe [Webserver](./webserver.md)) sitzt zwischen dem Internet und den Programmen, die eine Seite erst erzeugen müssen – bei MediaWiki ist das PHP-FPM, bei XWiki der Anwendungsserver Tomcat. An zwei Stellen muss dabei eine Verbindung zustande kommen:

1. **vorne:** Ein Besucher aus dem Internet erreicht NGINX.
2. **hinten:** NGINX erreicht das Programm im Hintergrund und holt sich von dort die fertige Seite.

Beide Verbindungen können über einen **Netzwerk-Port** laufen oder über eine **Socket-Datei** – einen sogenannten Unix-Socket. Dieses Kapitel zeigt, wann sich der Unix-Socket lohnt und wie man ihn in der NGINX-Konfiguration einträgt: hinten als Weg zu PHP-FPM oder Tomcat, vorne als eigener Lauschposten und, als dritte Möglichkeit, als dauerhafte Brücke zwischen einem Port und einer Socket-Datei.

Alle Befehle werden in der Textkonsole des Servers eingegeben. Ein vorangestelltes `sudo` bedeutet: mit Verwaltungsrechten ausführen. Nach jeder Änderung an der Konfiguration prüft `sudo nginx -t` die Dateien auf Fehler, `sudo systemctl reload nginx` übernimmt sie.

## Was ein Unix-Socket ist

Ein **Netzwerk-Port** ist der bekannte Weg: Ein Programm lauscht auf einer Nummer wie 9000, ein anderes verbindet sich mit `127.0.0.1:9000`. Das funktioniert auch über Rechnergrenzen hinweg.

Ein **Unix-Socket** ist der zweite Weg. Statt einer Portnummer gibt es eine Datei im Dateisystem, zum Beispiel `/run/php/php8.5-fpm.sock`. Ein Programm lauscht an dieser Datei, ein anderes verbindet sich mit ihr. Beide müssen dafür auf demselben Rechner laufen. Zwei Eigenschaften machen den Unix-Socket für Server interessant:

- **Kein offener Port.** Eine Socket-Datei ist grundsätzlich nur auf demselben Rechner erreichbar. Sie kann nicht aus Versehen für das ganze Netz geöffnet werden.
- **Zugriff über Dateirechte.** Wer die Socket-Datei benutzen darf, regeln dieselben Lese- und Schreibrechte wie bei jeder anderen Datei.

Eine ausführliche Erklärung – auch dazu, wie man eine Socket-Datei zum Testen kurz als Port erreichbar macht – steht im Kapitel [Unix-Socket](../entwicklungs-rechner/unix-socket.md).

## Die zwei Verbindungen von NGINX

In der NGINX-Konfiguration tauchen die beiden Verbindungen an unterschiedlichen Stellen auf.

| Stelle | Aufgabe | über einen Port | über einen Unix-Socket |
| --- | --- | --- | --- |
| `listen` | wo NGINX auf Besucher wartet | `listen 80;` | `listen unix:/run/nginx-intern.sock;` |
| `fastcgi_pass` | Weg zu PHP-FPM | `fastcgi_pass 127.0.0.1:9000;` | `fastcgi_pass unix:/run/php/php8.5-fpm.sock;` |
| `proxy_pass` | Weg zu einem anderen Programm (z. B. Tomcat) | `proxy_pass http://127.0.0.1:9000/;` | `proxy_pass http://unix:/var/lib/tomcat10/xwiki.sock:/;` |

Der häufigste Fall ist die hintere Verbindung über `fastcgi_pass` oder `proxy_pass`. Die vordere Verbindung über `listen unix:` braucht man selten und nur in besonderen Aufbauten.

## Hinten: NGINX erreicht das Hintergrundprogramm über einen Socket

### PHP-FPM

PHP-FPM legt seine Socket-Datei bei einer Standardinstallation von selbst an, bei PHP 8.5 unter `/run/php/php8.5-fpm.sock`. Diese Datei gehört dem Benutzer `www-data`, und unter genau diesem Benutzer läuft auch NGINX. Der Zugriff ist damit ohne weitere Einstellung möglich. In der Server-Konfiguration steht dann statt einer Adresse der Pfad:

```nginx
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.5-fpm.sock;
    }
```

Das Kapitel [MediaWiki einrichten](../web-stack/mediawiki.md) verwendet genau diesen Weg. Ein Port für PHP-FPM wird dabei nirgends geöffnet.

### Ein anderes Programm über den Reverse Proxy

Für Programme, die selbst HTTP sprechen – etwa Tomcat hinter XWiki –, benutzt NGINX `proxy_pass`. Auch hier kann statt der Adresse `http://127.0.0.1:9000/` eine Socket-Datei stehen. Die Schreibweise ist etwas ungewohnt: Nach `http://unix:` folgt der Pfad, dann ein Doppelpunkt und der Pfad innerhalb der Anwendung.

```nginx
    location / {
        proxy_pass http://unix:/var/lib/tomcat10/xwiki.sock:/;
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
```

Anders als bei PHP-FPM gehört diese Socket-Datei nicht `www-data`, sondern dem Benutzer des jeweiligen Dienstes. Damit NGINX sie benutzen darf, wird `www-data` in dessen Gruppe aufgenommen – im XWiki-Beispiel ist das die Gruppe `tomcat`:

```bash
sudo usermod -aG tomcat www-data
sudo systemctl restart tomcat10 nginx
```

Der genaue Weg für XWiki steht im Kapitel [XWiki einrichten](../web-stack/xwiki.md).

### Warum überhaupt ein Socket hinten?

Der Geschwindigkeitsunterschied zwischen Port und Socket ist auf einem einzelnen Server winzig und im Alltag nicht zu bemerken. Der eigentliche Gewinn liegt woanders:

- Es wird **kein Port** belegt, der versehentlich nach außen geöffnet werden könnte. Man muss ihn auch nicht in der Firewall ausdrücklich sperren.
- Der Zugriff hängt an **Dateirechten**. Nur wer in der richtigen Gruppe ist, kommt an den Dienst.

Für einen Aufbau, bei dem Webserver und Hintergrundprogramm ohnehin auf demselben Rechner liegen, ist der Unix-Socket deshalb die sauberere Wahl.

## Vorne: NGINX wartet an einem Socket auf Besucher

Statt `listen 80;` kann ein NGINX-Server auch an einer Socket-Datei lauschen:

```nginx
server {
    listen unix:/run/nginx-intern.sock;
    # ...
}
```

Das ergibt nur Sinn, wenn vor diesem NGINX noch etwas anderes steht, das die Anfragen über die Socket-Datei hereinreicht – zum Beispiel ein zweiter NGINX, ein Zwischenspeicher oder ein Anwendungsserver auf demselben Rechner. Für den normalen Betrieb, bei dem NGINX direkt die Verbindungen aus dem Internet annimmt, bleibt es bei `listen 80;` und `listen 443 ssl;`, denn aus dem Netz ist eine Socket-Datei nicht erreichbar.

## Die dauerhafte Brücke: der `stream`-Baustein

Manchmal soll ein Werkzeug, das nur Ports kennt, dauerhaft an eine Socket-Datei kommen – oder umgekehrt. Für einen kurzen Test erledigt das der Befehl `socat` (siehe [Unix-Socket](../entwicklungs-rechner/unix-socket.md)). Läuft NGINX auf dem Server ohnehin, kann es diese Brücke dauerhaft übernehmen. Zuständig ist der `stream`-Baustein. Er arbeitet nicht mit HTTP-Seiten, sondern reicht rohe Verbindungen unverändert weiter.

Der `stream`-Block steht auf der obersten Ebene der Datei `/etc/nginx/nginx.conf`, außerhalb des `http`-Blocks:

```nginx
stream {
    server {
        listen 127.0.0.1:9000;
        proxy_pass unix:/var/lib/tomcat10/xwiki.sock;
    }
}
```

Solange NGINX läuft, nimmt es auf Port 9000 – nur auf dem Rechner selbst, wegen `127.0.0.1` – Verbindungen an und leitet sie an die Socket-Datei weiter. Die umgekehrte Richtung geht genauso: `listen unix:/run/meine.sock;` und `proxy_pass 127.0.0.1:9000;`.

Der `stream`-Baustein gehört bei Ubuntu zum Standardpaket `nginx` und muss nicht nachinstalliert werden. Anders als ein `socat`-Befehl im Terminal übersteht diese Brücke einen Neustart des Servers.

## Fehlersuche

Wenn NGINX das Hintergrundprogramm nicht erreicht, meldet der Browser **502 Bad Gateway**. Den genauen Grund nennt das Fehlerprotokoll:

```bash
sudo tail -n 20 /var/log/nginx/error.log
```

- **`connect() to unix:/… failed (13: Permission denied)`**: Der Benutzer `www-data` darf die Socket-Datei nicht benutzen. Die Rechte zeigt `ls -l /run/php/php8.5-fpm.sock` (bzw. der jeweilige Pfad). Meist fehlt `www-data` in der richtigen Gruppe; nach `sudo usermod -aG <gruppe> www-data` muss NGINX neu gestartet werden (`sudo systemctl restart nginx`), ein bloßes `reload` reicht für die neue Gruppenzugehörigkeit nicht.
- **`connect() to unix:/… failed (2: No such file or directory)`**: Der Pfad stimmt nicht, oder der Dienst läuft nicht und hat die Socket-Datei deshalb nicht angelegt. Prüfen mit `systemctl status php8.5-fpm` bzw. `systemctl status tomcat10`.
- **`invalid URL prefix in …`** beim `nginx -t`: Bei `proxy_pass` auf einen Socket fehlt das `http://unix:` am Anfang oder der Doppelpunkt vor dem Pfad innerhalb der Anwendung.
- **`unknown directive "stream"`**: Der `stream`-Block wurde in eine Server-Datei unter `sites-available/` geschrieben. Er gehört auf die oberste Ebene von `/etc/nginx/nginx.conf`.

## Für dieses Buch

Für die Wissenssammlung dieses Buchs erreicht NGINX **PHP-FPM über dessen Socket-Datei** `/run/php/php8.5-fpm.sock` – das ist die Standardeinstellung und braucht keine weitere Anpassung (siehe [MediaWiki einrichten](../web-stack/mediawiki.md)). Wer XWiki betreibt, kann Tomcat auf dieselbe Weise über einen Unix-Socket ansprechen und spart sich damit den offenen Port 9000 (siehe [XWiki einrichten](../web-stack/xwiki.md)).

Die vordere Seite bleibt bei Ports: `listen 80;` und `listen 443 ssl;`. Der `stream`-Baustein ist nur dann nötig, wenn eine Socket-Datei auf dem Server dauerhaft auch als Port bereitstehen soll; für den einmaligen Test genügt `socat`.

## Fazit

NGINX verbindet sich an zwei Stellen: vorne mit dem Besucher, hinten mit dem Programm, das die Seite erzeugt. Die hintere Verbindung läuft am besten über einen **Unix-Socket** – bei PHP-FPM mit `fastcgi_pass unix:/run/php/php8.5-fpm.sock;`, bei einem HTTP-Programm mit `proxy_pass http://unix:/pfad/zur.sock:/;`. Der Vorteil ist nicht die Geschwindigkeit, sondern dass kein Port offensteht und der Zugriff an Dateirechten hängt; deshalb muss `www-data` oft in die Gruppe des Dienstes aufgenommen werden. Die vordere Verbindung bleibt bei Ports. Soll eine Socket-Datei dauerhaft als Port erreichbar sein, übernimmt der **`stream`-Baustein** von NGINX diese Brücke – zuverlässiger als ein `socat`-Befehl im Terminal.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
