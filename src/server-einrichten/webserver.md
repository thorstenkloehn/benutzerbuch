# Webserver

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

MediaWiki, die Grundlage der Wissenssammlung dieses Buchs, besteht aus vielen PHP-Dateien (siehe [Laufzeitumgebung](./laufzeitumgebung.md)). Damit ein Besucher diese Seiten im Browser sehen kann, fehlt noch ein Programm dazwischen: der **Webserver**. Er nimmt die Anfrage aus dem Internet entgegen, holt die passende Seite und schickt sie zurück.

Dieses Kapitel erklärt, was ein Webserver ist, stellt die vier bekanntesten vor und zeigt für die drei, die man wirklich selbst betreibt, wie man sie auf einem Server mit **Ubuntu 26.04 LTS** (siehe [Betriebssystem](./betriebssystem.md)) einrichtet und mit einem kostenlosen SSL-Zertifikat auf verschlüsselte Verbindungen (HTTPS) umstellt. Alle Befehle werden in der Textkonsole des Servers eingegeben. Das vorangestellte `sudo` bedeutet: mit Verwaltungsrechten ausführen.

## Was ein Webserver ist

Man kann sich einen Webserver wie den Empfang in einem großen Bürohaus vorstellen. Ein Besucher kommt herein und nennt sein Anliegen ("Ich möchte die Startseite von example.de sehen"). Der Empfang schaut nach, wer zuständig ist, holt die Auskunft und gibt sie heraus. Der Besucher muss nie selbst durch das Gebäude laufen.

Genau das macht ein Webserver mit Anfragen aus dem Internet. Er beherrscht dabei zwei Aufgaben:

- **Feste Dateien ausliefern.** Bilder, Stylesheets, heruntergeladene Dokumente oder eine einfache HTML-Seite liegen als Datei auf der Festplatte. Der Webserver schickt sie unverändert an den Browser.
- **Anfragen weiterreichen.** Seiten, die erst erzeugt werden müssen – jede MediaWiki-Seite gehört dazu –, gibt der Webserver an das zuständige Programm im Hintergrund weiter, bei MediaWiki an PHP-FPM. Er wartet auf dessen Antwort und leitet sie an den Browser zurück. In dieser Rolle heißt der Webserver auch **Reverse Proxy**.

Dazu kommen Aufgaben, die man an einer zentralen Stelle bündeln möchte: die Verschlüsselung der Verbindung (HTTPS), das Umleiten von alten auf neue Adressen, das Sperren unerwünschter Zugriffe und das Zusammenfassen mehrerer Programme unter einer Adresse. Auch der Zwischenspeicher aus dem Kapitel [Containerisierung von Software](./containerisierung.md) sitzt oft hinter demselben Webserver.

## Die vier bekanntesten Webserver

### NGINX

NGINX (gesprochen "Engine-X") wurde entwickelt, um sehr viele gleichzeitige Verbindungen mit wenig Arbeitsspeicher zu bewältigen. Es liefert feste Dateien schnell aus und ist als Reverse Proxy vor anderen Programmen weit verbreitet. Die offizielle MediaWiki-Anleitung und ein Großteil der Beispiele im Internet gehen von NGINX aus. Die Einstellungen stehen in kurzen Textblöcken; man beschreibt darin, welche Adresse zu welchem Verzeichnis oder welchem Hintergrunddienst gehört.

Für die Wissenssammlung dieses Buchs ist NGINX die naheliegende Wahl: am meisten Anleitungen, sparsam im Verbrauch, gut mit PHP-FPM zu verbinden.

### Apache HTTP Server

Der Apache HTTP Server ist der älteste der vier und lange Zeit der Standard im Web gewesen. Viele fertige Anleitungen und Auslieferungspakete – auch die "einfache" MediaWiki-Installation – sind auf Apache zugeschnitten. Seine Stärke ist die Erweiterbarkeit über Zusatzmodule und die Möglichkeit, Einstellungen pro Verzeichnis in einer Datei namens `.htaccess` abzulegen, ohne den ganzen Server neu zu laden.

Diese Bequemlichkeit hat einen Preis: Apache verbraucht bei vielen gleichzeitigen Besuchern mehr Arbeitsspeicher als NGINX. Für eine kleine Wissenssammlung fällt das kaum ins Gewicht. Wer bereits Apache-Erfahrung hat, kann bei Apache bleiben.

### Caddy

Caddy ist der jüngste der vier und verfolgt ein klares Ziel: möglichst wenig Einrichtung. Sein größter Unterschied zu den anderen ist die **automatische Verschlüsselung**. Caddy besorgt sich das SSL-Zertifikat beim ersten Start selbst und erneuert es von allein – ein zusätzliches Werkzeug wie Certbot entfällt vollständig. Die Konfigurationsdatei ("Caddyfile") ist oft nur wenige Zeilen lang.

Caddy ist eine gute Wahl für alle, die den Server möglichst einfach halten wollen und keine besonderen Ansprüche an die Feinabstimmung haben. Es gibt weniger fertige Anleitungen als für NGINX, dafür braucht man auch weniger davon.

### Pingora

Pingora fällt aus der Reihe. Es ist **kein fertiger Webserver, den man installiert und startet**, sondern ein Baukasten in der Programmiersprache Rust, mit dem Entwickler eigene Netzwerkdienste bauen. Die Firma Cloudflare hat Pingora zunächst für den Eigenbedarf geschrieben – als Ersatz für NGINX in ihrem weltweiten Netz – und den Baukasten später quelloffen veröffentlicht.

Für den Aufbau dieses Buchs spielt Pingora keine Rolle. Es gibt kein `apt`-Paket dafür; man bräuchte die vollständige Rust-Entwicklungsausstattung und müsste den Dienst selbst programmieren. Pingora ist hier nur genannt, weil der Name in Vergleichen und Nachrichten auftaucht. Wer einmal ein fertiges Programm betreibt, das auf Pingora aufbaut, merkt davon im Betrieb nichts Besonderes.

### Kurzvergleich

| Webserver | Art | Verschlüsselung | Für diesen Aufbau |
| --- | --- | --- | --- |
| NGINX | fertiger Server | mit Certbot | empfohlen, die meisten Anleitungen |
| Apache HTTP Server | fertiger Server | mit Certbot | geeignet, etwas mehr Verbrauch |
| Caddy | fertiger Server | eingebaut, automatisch | geeignet, am wenigsten Einrichtung |
| Pingora | Programmbaukasten (Rust) | selbst zu bauen | nicht relevant, nur zum Einordnen |

## Installation auf Ubuntu 26.04

Vor der Installation lohnt ein Blick auf die Ports: Ein Webserver belegt Port 80 (unverschlüsselt) und Port 443 (verschlüsselt). Läuft dort schon ein anderer Webserver, muss dieser vorher gestoppt werden – zwei gleichzeitig geht nicht.

### NGINX

```bash
sudo apt update
sudo apt install nginx

# Zeigt die installierte Version zur Kontrolle an
nginx -v
```

Nach der Installation läuft NGINX bereits und zeigt unter der IP-Adresse des Servers eine Willkommensseite. Diese gehört zur mitgelieferten Beispielkonfiguration. Bevor eine eigene Seite eingerichtet wird, entfernt man sie:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

Die eigenen Einstellungen kommen als neue Datei nach `/etc/nginx/sites-available/` und werden mit einem Verweis ("Symlink") nach `/etc/nginx/sites-enabled/` aktiv geschaltet. Nach jeder Änderung prüft `sudo nginx -t` die Datei auf Fehler, `sudo systemctl reload nginx` übernimmt sie.

### Apache HTTP Server

```bash
sudo apt update
sudo apt install apache2

# Zeigt die installierte Version zur Kontrolle an
apache2 -v
```

Auch Apache startet sofort mit einer Standardseite. Für den Betrieb hinter PHP-FPM und für die spätere Verschlüsselung werden einige Zusatzmodule eingeschaltet:

```bash
sudo a2enmod proxy_fcgi setenvif rewrite headers ssl
sudo systemctl restart apache2
```

Die Standard-Beispielseite schaltet man mit `sudo a2dissite 000-default` ab. Eigene Konfigurationsdateien liegen unter `/etc/apache2/sites-available/` und werden mit `sudo a2ensite <name>` aktiviert. `sudo apachectl configtest` prüft auf Fehler, `sudo systemctl reload apache2` übernimmt Änderungen.

### Caddy

Caddy liegt nicht in den Paketquellen von Ubuntu. Es wird über die offizielle Paketquelle des Herstellers eingerichtet, mitsamt einem digitalen Schlüssel, mit dem Ubuntu die Echtheit der Pakete prüft:

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl

curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' \
  | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg

curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' \
  | sudo tee /etc/apt/sources.list.d/caddy-stable.list

sudo apt update
sudo apt install caddy
```

Danach läuft Caddy als Hintergrunddienst. Die gesamte Konfiguration steht in der Datei `/etc/caddy/Caddyfile`. Für MediaWiki genügen wenige Zeilen: der Domainname, das Wurzelverzeichnis und die Weitergabe von PHP-Dateien an PHP-FPM. Nach dem Ändern der Datei übernimmt `sudo systemctl reload caddy` die neuen Einstellungen. Um Zertifikate muss man sich nicht kümmern – dazu unten mehr.

## Verschlüsselung mit HTTPS

Eine Website sollte heute ausschließlich über verschlüsselte Verbindungen erreichbar sein. Sichtbar wird das am Kürzel **`https://`** und am Schloss-Symbol im Browser. Die Verschlüsselung schützt nicht nur Passwörter, sondern verhindert auch, dass unterwegs jemand den Inhalt der Seiten mitliest oder verändert.

Grundlage ist ein **SSL-Zertifikat**. Es ist eine kleine, digital unterschriebene Datei, die bestätigt: "Dieser Server gehört wirklich zu dieser Domain." Ausgestellt wird es von einer **Zertifizierungsstelle**. Die gemeinnützige Stelle **Let's Encrypt** stellt solche Zertifikate kostenlos aus; sie sind allerdings nur 90 Tage gültig und müssen darum automatisch erneuert werden.

Das Werkzeug, das Zertifikate bei Let's Encrypt beantragt, einbaut und rechtzeitig erneuert, heißt **Certbot**. Ausnahme ist Caddy: Es bringt diese Aufgabe schon mit.

### Voraussetzung: die Domain zeigt auf den Server

Bevor ein Zertifikat beantragt werden kann, muss die Domain (z. B. `meine-domain.de`) beim DNS-Anbieter auf die IP-Adresse des Servers zeigen – über einen sogenannten **A-Record** für IPv4 und, falls vorhanden, einen **AAAA-Record** für IPv6. Let's Encrypt prüft bei der Ausstellung, ob die Domain tatsächlich zu diesem Server führt.

### Certbot installieren

Die Certbot-Entwickler empfehlen die Installation über das Paketformat **Snap**, weil diese Fassung immer aktuell bleibt:

```bash
sudo apt install snapd
sudo snap install core
sudo snap install --classic certbot

# Certbot als normalen Befehl "certbot" verfügbar machen
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### NGINX und Apache: Zertifikat in einem Schritt

Für den Normalfall – eine feste Domain, deren A-Record auf den Server zeigt – erledigt Certbot alles allein. Es beantragt das Zertifikat, trägt es in die Serverkonfiguration ein und richtet die Umleitung von `http://` auf `https://` ein:

```bash
# Für NGINX
sudo certbot --nginx -d meine-domain.de -d www.meine-domain.de

# Für Apache
sudo certbot --apache -d meine-domain.de -d www.meine-domain.de
```

Beim ersten Aufruf fragt Certbot nach einer E-Mail-Adresse für Ablaufwarnungen und nach der Zustimmung zu den Nutzungsbedingungen. Die automatische Erneuerung richtet Snap im Hintergrund selbst ein; `sudo certbot renew --dry-run` prüft, ob sie funktioniert.

### Wildcard-Zertifikat über einen DNS-Eintrag

Soll ein Zertifikat für **alle Unterdomains** auf einmal gelten (`*.meine-domain.de`), reicht der einfache Weg nicht. Let's Encrypt verlangt dann einen Nachweis über einen DNS-Eintrag. Dieser Weg funktioniert für jeden Webserver gleich, weil er das Zertifikat nur beschafft (`certonly`) und nicht selbst einbaut:

```bash
sudo certbot certonly --manual --preferred-challenges dns \
  -d '*.meine-domain.de' -d meine-domain.de
```

Certbot zeigt daraufhin eine zufällige Zeichenkette an und wartet. Diese Zeichenkette wird beim DNS-Anbieter als Eintrag hinterlegt:

1. Beim DNS-Anbieter anmelden und die Verwaltungsoberfläche für die Domain öffnen.
2. Den Bereich für DNS-Einträge suchen und einen neuen Eintrag anlegen:
   - **Typ:** TXT
   - **Name/Host:** `_acme-challenge` (manche Oberflächen erwarten den vollständigen Namen `_acme-challenge.meine-domain.de`)
   - **Wert:** die von Certbot angezeigte Zeichenkette
3. Den Eintrag speichern. Die Änderung braucht einige Minuten, bis sie weltweit sichtbar ist. Ob sie schon greift, lässt sich mit dem Befehl `dig` oder auf einer Prüfseite wie whatsmydns.net kontrollieren.
4. Erst dann bei Certbot die Eingabetaste drücken.

| Name/Host | Typ | Wert |
| --- | --- | --- |
| `_acme-challenge.meine-domain.de` | TXT | (zufällige Zeichenkette von Certbot) |

> **Hinweis:** Jeder neue Antrag erzeugt eine neue Zeichenkette. Der TXT-Eintrag muss bei jeder Verlängerung von Hand angepasst werden – oder man verwendet ein Certbot-Zusatzmodul, das den Eintrag über die Schnittstelle des DNS-Anbieters selbst setzt.

Nach erfolgreicher Prüfung legt Certbot die Dateien ab und nennt die Pfade, zum Beispiel:

```
Certificate is saved at: /etc/letsencrypt/live/meine-domain.de/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/meine-domain.de/privkey.pem
```

`fullchain.pem` enthält das Zertifikat samt der Zwischenzertifikate, `privkey.pem` den geheimen Schlüssel. Diese beiden Pfade bleiben auch bei einer Erneuerung gleich: Certbot überschreibt die Dateien, der Ordnername unter `/etc/letsencrypt/live/` ändert sich nicht. Deshalb kann man sie fest in die Konfiguration von NGINX oder Apache eintragen.

### Caddy: nichts zu tun

Bei Caddy genügt es, im Caddyfile den Domainnamen anzugeben. Beim ersten Start holt Caddy das Zertifikat selbst und erneuert es fortan ohne weiteres Zutun. Ein Wildcard-Zertifikat ist die einzige Ausnahme: Dafür braucht auch Caddy ein Zusatzmodul für den jeweiligen DNS-Anbieter.

## Für dieses Buch

Für die beschriebene Wissenssammlung ist **NGINX** die empfohlene Wahl: Es gibt die meisten Anleitungen – auch die offizielle von MediaWiki –, es verbraucht wenig, und das Zusammenspiel mit PHP-FPM ist gut dokumentiert. Wer die Einrichtung so knapp wie möglich halten will und auf Feinabstimmung verzichten kann, ist mit **Caddy** gut bedient, allein schon wegen der automatischen Verschlüsselung. **Apache** ist eine solide Wahl für alle, die damit schon vertraut sind. **Pingora** kommt für diesen Aufbau nicht in Frage.

Ein Grundsatz gilt für alle: HTTPS ist Pflicht, nicht Kür. Das Zertifikat von Let's Encrypt ist kostenlos, und die Erneuerung läuft nach der Einrichtung von selbst. Wer Docker einsetzt (siehe [Containerisierung von Software](./containerisierung.md)), sollte zusätzlich beachten, dass Docker die Firewall umgehen kann; dort veröffentlicht man die Programme nur nach innen und stellt den Webserver davor.

## Fazit

Ein Webserver nimmt Anfragen aus dem Internet entgegen, liefert feste Dateien selbst aus und reicht alles Übrige an das zuständige Programm weiter – bei MediaWiki an PHP-FPM. **NGINX** ist für diesen Aufbau die beste Wahl, **Caddy** die einfachste, **Apache** die vertraute; **Pingora** ist ein Programmbaukasten und kein fertiger Server. Auf Ubuntu 26.04 wird NGINX und Apache mit einem einzigen `apt`-Befehl eingerichtet, Caddy über die Paketquelle des Herstellers. Die verschlüsselte Verbindung besorgt bei NGINX und Apache das Werkzeug **Certbot** mit einem kostenlosen Zertifikat von **Let's Encrypt** – für eine feste Domain in einem Schritt, für alle Unterdomains über einen DNS-Eintrag. Caddy erledigt diesen Teil von allein.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
