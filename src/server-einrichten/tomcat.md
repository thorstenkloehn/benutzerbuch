# Tomcat 11 einrichten

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Manche Webprogramme sind in der Programmiersprache **Java** geschrieben. Ein Java-Webprogramm läuft nicht für sich allein – es braucht ein Trägerprogramm, das es lädt, am Laufen hält und die Anfragen aus dem Netz an es weiterreicht. Dieses Trägerprogramm heißt **Anwendungsserver**, und der verbreitetste ist **Apache Tomcat**.

Im Kapitel [XWiki einrichten](../web-stack/xwiki.md) taucht Tomcat schon auf, dort aber versteckt: Die XWiki-Pakete bringen ihn als Beigabe mit. Dieses Kapitel zeigt Tomcat für sich – wie man **Tomcat 11** auf einem Server mit **Ubuntu 26.04 LTS** (siehe [Betriebssystem](./betriebssystem.md)) einrichtet, auf zwei Wegen: von Hand aus dem offiziellen Download und aus den Paketquellen von Ubuntu. Danach wird Tomcat über einen **Unix-Socket** an den Webserver NGINX angebunden, am Beispiel der Adresse `https://tomcat11.de`.

Alle Befehle werden in der Textkonsole des Servers eingegeben. Das vorangestellte `sudo` bedeutet: mit Verwaltungsrechten ausführen.

## Was Tomcat ist

Ein fertiges Java-Webprogramm wird meist als eine einzige Datei mit der Endung `.war` ausgeliefert – kurz für „Web Application Archive", also ein gepacktes Webprogramm. Diese Datei kann man nicht einfach starten. Sie muss in einen **Servlet-Container** gelegt werden: ein Programm, das sich um alles Drumherum kümmert – Netzwerk, Verbindungen, mehrere Anfragen gleichzeitig, Neustart – und das Webprogramm nur noch mit den fertig aufbereiteten Anfragen versorgt.

Man kann sich Tomcat wie einen Motorraum vorstellen: Das Java-Webprogramm ist der Motor, Tomcat das Gehäuse mit Halterungen, Anschlüssen und Anlasser darum herum.

Tomcat setzt eine Reihe von Java-Standards um – für Servlets, für JavaServer Pages (JSP) und für WebSockets. Welche Fassung dieser Standards ein Tomcat beherrscht, hängt an seiner Hauptversion:

| Tomcat-Version | Standard-Bündel | Mindest-Java | Einordnung |
| --- | --- | --- | --- |
| 9.0 | Java EE 8 | Java 8 | alt, nur noch für ältere Programme |
| 10.1 | Jakarta EE 10 | Java 11 | noch verbreitet |
| **11.0** | **Jakarta EE 11** | **Java 17** | aktuelle Wahl für neue Aufbauten |

Für einen neuen Server ist **Tomcat 11** die richtige Wahl. Es braucht mindestens Java 17; die in diesem Buch verwendete Standard-Java-Version von Ubuntu 26.04 (OpenJDK 25, siehe [Laufzeitumgebung](./laufzeitumgebung.md)) passt. Zum Zeitpunkt dieses Kapitels ist 11.0.25 die neueste Ausgabe.

Ein Wechsel von Java EE zu **Jakarta EE** hat übrigens einen praktischen Haken: Programme für Tomcat 9 und älter laufen **nicht** ohne Anpassung auf Tomcat 10 oder 11. Wer ein bestehendes Programm übernimmt, muss prüfen, für welche Tomcat-Reihe es gebaut ist.

## Zwei Wege zur Installation

Es gibt zwei Wege, Tomcat 11 auf den Server zu bringen:

- **Von Hand** aus dem offiziellen Download unter <https://tomcat.apache.org/download-11.cgi>. Man bekommt immer die neueste Version und schreibt den Startdienst selbst. Dieser Weg gibt volle Kontrolle und eignet sich, wenn eine ganz bestimmte oder die allerneueste Version nötig ist oder mehrere Tomcats nebeneinander laufen sollen.
- **Aus den Ubuntu-Paketquellen** mit `apt install tomcat11`. Ubuntu 26.04 bringt Tomcat 11 mit. Der Weg ist schneller, und Sicherheitsaktualisierungen kommen automatisch über `apt`. Dafür ist die Version an die von Ubuntu ausgelieferte Fassung gebunden.

Die folgenden Abschnitte beschreiben zuerst den Weg von Hand ausführlich (**Weg A**), dann kurz den Paket-Weg (**Weg B**). Die Anbindung an NGINX danach gilt für beide; die Pfade unterscheiden sich nur an wenigen Stellen, jeweils angegeben.

## Weg A: Tomcat 11 aus dem offiziellen Download

### Schritt 1: Java installieren

Tomcat 11 braucht eine Java-Laufzeitumgebung ab Version 17. Für den reinen Betrieb genügt die kopflose Fassung ohne grafische Bestandteile:

```bash
sudo apt update
sudo apt install openjdk-25-jre-headless

# Zur Kontrolle die installierte Version anzeigen
java -version
```

Mehr zu Java steht im Kapitel [Laufzeitumgebung](./laufzeitumgebung.md).

### Schritt 2: Einen eigenen Benutzer anlegen

Tomcat soll **nicht** mit Verwaltungsrechten laufen. Sonst hätte ein Fehler im Java-Programm sofort Zugriff auf den ganzen Server. Deshalb bekommt Tomcat einen eigenen Benutzer, der sich nicht anmelden kann und dessen Zuhause das spätere Programmverzeichnis ist:

```bash
sudo useradd --system --home-dir /opt/tomcat11 --shell /usr/sbin/nologin tomcat
```

- `--system` legt ein Dienstkonto an, kein persönliches.
- `--shell /usr/sbin/nologin` verhindert das Anmelden mit diesem Benutzer.

### Schritt 3: Tomcat herunterladen und entpacken

Die aktuelle Versionsnummer steht auf der Download-Seite <https://tomcat.apache.org/download-11.cgi>. Hier wird sie einmal in eine Variable geschrieben, damit die folgenden Befehle unverändert bleiben:

```bash
cd /tmp
VERSION=11.0.25

# Das gepackte Programm herunterladen
wget "https://dlcdn.apache.org/tomcat/tomcat-11/v${VERSION}/bin/apache-tomcat-${VERSION}.tar.gz"

# Die Prüfsumme dazu herunterladen
wget "https://downloads.apache.org/tomcat/tomcat-11/v${VERSION}/bin/apache-tomcat-${VERSION}.tar.gz.sha512"

# Die heruntergeladene Datei gegen die Prüfsumme halten – muss "OK" anzeigen
sha512sum -c "apache-tomcat-${VERSION}.tar.gz.sha512"
```

Die **Prüfsumme** ist ein Fingerabdruck der Datei. Stimmt der berechnete Fingerabdruck mit dem veröffentlichten überein, ist die Datei unterwegs nicht verändert worden. Zeigt der Befehl etwas anderes als `OK`, wird die Datei nicht benutzt, sondern neu geladen.

> **Hinweis:** Ältere Tomcat-Versionen werden von `dlcdn.apache.org` irgendwann entfernt und wandern nach `https://archive.apache.org/dist/tomcat/`. Wer eine ältere Version braucht, ändert die Adresse entsprechend.

Jetzt das Zielverzeichnis anlegen, es dem Benutzer `tomcat` übergeben und die Rechte setzen:

```bash
# Zielverzeichnis anlegen (falls noch nicht vorhanden)
sudo mkdir -p /opt/tomcat11

# Eigentümer auf Benutzer und Gruppe "tomcat" setzen
sudo chown tomcat:tomcat /opt/tomcat11

# Rechte setzen (rwxr-x---): Besitzer darf alles, die Gruppe darf lesen und
# in das Verzeichnis wechseln, alle anderen nichts
sudo chmod 750 /opt/tomcat11
```

Das Recht der Gruppe, in das Verzeichnis zu wechseln (das `x` in `r-x`), ist kein Beiwerk: Ohne dieses Recht erreicht NGINX später die Socket-Datei in diesem Verzeichnis nicht, und die Anbindung schlägt fehl (siehe Abschnitt „Tomcat über einen Unix-Socket an NGINX anbinden").

Dann das Archiv hineinentpacken:

```bash
sudo tar xzf "apache-tomcat-${VERSION}.tar.gz" -C /opt/tomcat11 --strip-components=1
```

`--strip-components=1` lässt die oberste Ordnerebene aus dem Archiv weg, sodass die Dateien direkt in `/opt/tomcat11` liegen und nicht in `/opt/tomcat11/apache-tomcat-11.0.25`.

Das Entpacken lief als `root`, deshalb gehören die entpackten Dateien zunächst `root`. Anschließend gehört wieder alles dem Benutzer `tomcat`:

```bash
sudo chown -R tomcat:tomcat /opt/tomcat11
```

Nach dem Entpacken liegen die wichtigsten Verzeichnisse an diesen Stellen:

| Verzeichnis | Inhalt |
| --- | --- |
| `/opt/tomcat11/bin/` | Start- und Stopp-Skripte |
| `/opt/tomcat11/conf/` | Konfiguration, vor allem `server.xml` |
| `/opt/tomcat11/webapps/` | die Webprogramme (`.war`-Dateien) |
| `/opt/tomcat11/logs/` | Protokolldateien, u. a. `catalina.out` |
| `/opt/tomcat11/lib/` | von Tomcat mitgebrachte Java-Bausteine |

### Schritt 4: Tomcat als Dienst einrichten

Damit Tomcat beim Serverstart automatisch hochfährt und sich bequem starten und stoppen lässt, wird er als **systemd-Dienst** eingetragen. Zuerst den genauen Pfad zur Java-Installation herausfinden:

```bash
sudo update-alternatives --list java
```

Die Ausgabe endet auf `/bin/java`, zum Beispiel `/usr/lib/jvm/java-25-openjdk-amd64/bin/java`. Der Teil davor – hier `/usr/lib/jvm/java-25-openjdk-amd64` – ist gleich als `JAVA_HOME` einzutragen.

Nun die Dienstdatei anlegen:

```bash
sudo nano /etc/systemd/system/tomcat.service
```

Mit diesem Inhalt (den `JAVA_HOME`-Pfad gegebenenfalls anpassen):

```ini
[Unit]
Description=Apache Tomcat 11
After=network.target

[Service]
Type=simple
User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/java-25-openjdk-amd64"
Environment="CATALINA_HOME=/opt/tomcat11"
Environment="CATALINA_BASE=/opt/tomcat11"
Environment="CATALINA_PID=/opt/tomcat11/temp/tomcat.pid"
Environment="CATALINA_OPTS=-Xms512m -Xmx1024m -XX:+UseG1GC"

ExecStart=/opt/tomcat11/bin/catalina.sh run
ExecStop=/opt/tomcat11/bin/catalina.sh stop

Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Die Zeile `CATALINA_OPTS` legt den Arbeitsspeicher fest, den sich Java nehmen darf:

- `-Xms512m` ist der Speicher, der gleich beim Start belegt wird.
- `-Xmx1024m` ist die Obergrenze – hier 1024 MB, also 1 GB. Auf einem größeren Server darf dieser Wert höher liegen, aber nie an die Grenze des gesamten Arbeitsspeichers reichen; das Betriebssystem und die Datenbank brauchen ebenfalls Platz.
- `-XX:+UseG1GC` wählt ein Aufräumverfahren, das für Programme mit vielen gleichzeitigen Zugriffen gut geeignet ist.

Dann den Dienst bekannt machen, dauerhaft einschalten und starten:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now tomcat
sudo systemctl status tomcat
```

`status` sollte `active (running)` zeigen.

### Schritt 5: Kurzer Test

```bash
curl http://localhost:8080/
```

Wenn HTML zurückkommt, läuft Tomcat. In diesem Zustand lauscht er allerdings auf Port 8080 und nimmt Verbindungen aus dem ganzen Netz an – das wird bei der Anbindung an NGINX weiter unten geändert.

## Weg B: Tomcat 11 aus den Ubuntu-Paketquellen

Wer sich den Aufbau von Hand sparen will, nimmt das fertige Paket. Zuerst nachsehen, welche Version Ubuntu anbietet:

```bash
sudo apt update
apt-cache policy tomcat11
```

Dann installieren:

```bash
sudo apt install tomcat11
```

Das Paket zieht Java als Abhängigkeit mit und richtet den Dienst gleich ein. Er heißt `tomcat11` und läuft ebenfalls unter dem Benutzer `tomcat`:

```bash
systemctl status tomcat11
```

Die Pfade sind beim Paket anders als beim Weg von Hand. Die genaue Liste zeigt `dpkg -L tomcat11`; die wichtigsten sind:

| Datei oder Verzeichnis | Inhalt |
| --- | --- |
| `/var/lib/tomcat11/conf/server.xml` | Konfiguration, u. a. der Port |
| `/var/lib/tomcat11/webapps/` | die Webprogramme |
| `/var/lib/tomcat11/logs/` | Protokolldateien |
| `tomcat11.service` | der systemd-Dienst |

Den Arbeitsspeicher stellt man beim Paket über eine Ergänzungsdatei des Dienstes ein:

```bash
sudo systemctl edit tomcat11
```

Dort im vorgesehenen Bereich eintragen:

```ini
[Service]
Environment="CATALINA_OPTS=-Xms512m -Xmx1024m -XX:+UseG1GC"
```

Danach neu starten:

```bash
sudo systemctl restart tomcat11
```

**Für den Rest des Kapitels gilt:** Wer den Paket-Weg gewählt hat, ersetzt in den folgenden Befehlen und Dateien `/opt/tomcat11/conf` durch `/var/lib/tomcat11/conf`, den Dienstnamen `tomcat` durch `tomcat11` und den Socket-Pfad `/opt/tomcat11/tomcat11.sock` durch `/var/lib/tomcat11/tomcat11.sock`.

## Tomcat über einen Unix-Socket an NGINX anbinden

Von Haus aus öffnet Tomcat den Port 8080 für das ganze Netz. Für einen öffentlichen Server ist das die falsche Einstellung: Davor gehört ein Webserver, der die Verbindung verschlüsselt (HTTPS) und die Anfragen an Tomcat weiterreicht. Diese Rolle heißt **Reverse Proxy**; in diesem Buch übernimmt sie **NGINX** (siehe [Webserver](./webserver.md)).

Für die Verbindung zwischen NGINX und Tomcat gibt es zwei Möglichkeiten: über einen Port nur auf dem eigenen Rechner (`127.0.0.1:8080`) oder – sauberer – über eine **Socket-Datei**, den sogenannten Unix-Socket. Dann gibt es überhaupt keinen Port, der aus Versehen geöffnet werden könnte, und der Zugriff hängt allein an Dateirechten. Was ein Unix-Socket ist, erklären die Kapitel [Unix-Socket](../entwicklungs-rechner/unix-socket.md) und [Unix-Socket bei NGINX](./nginx-unix-socket.md) ausführlich. Tomcat 11 kann von sich aus an einer Socket-Datei lauschen; dieser Abschnitt richtet genau das ein.

### Schritt 1: Den Connector auf einen Socket umstellen

In der Datei `/opt/tomcat11/conf/server.xml` steht der Abschnitt, der festlegt, wo Tomcat auf Anfragen wartet. Er heißt `<Connector>` und sieht in etwa so aus:

```xml
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443"
           maxParameterCount="1000" />
```

Diesen Abschnitt so ersetzen:

```xml
<Connector protocol="org.apache.coyote.http11.Http11NioProtocol"
           unixDomainSocketPath="/opt/tomcat11/tomcat11.sock"
           unixDomainSocketPathPermissions="rw-rw----"
           connectionTimeout="20000"
           maxParameterCount="1000" />
```

Was sich ändert:

- Es gibt **keinen `port` mehr**. Stattdessen nennt `unixDomainSocketPath` den Pfad zur Socket-Datei. Tomcat legt sie beim Start an und entfernt sie beim Stoppen wieder.
- `protocol="org.apache.coyote.http11.Http11NioProtocol"` wählt ausdrücklich die Verarbeitungsart, die Socket-Dateien unterstützt.
- `unixDomainSocketPathPermissions="rw-rw----"` setzt die Dateirechte: Besitzer und Gruppe dürfen lesen und schreiben, alle anderen nichts. Der Besitzer ist der Benutzer `tomcat`.

Die Socket-Datei liegt hier bewusst im Verzeichnis `/opt/tomcat11`, das dem Benutzer `tomcat` gehört und einen Neustart übersteht. Damit NGINX die Datei darin erreicht, muss die Gruppe `tomcat` in das Verzeichnis wechseln dürfen – das leisten die Rechte `750` aus Schritt 3 von Weg A. Beim Paket-Weg ist `/var/lib/tomcat11` bereits passend gesetzt.

> **Hinweis:** Stürzt Tomcat ab, bleibt die Socket-Datei manchmal liegen. Beim nächsten Start meldet das Protokoll dann, die Datei sei schon vorhanden, und Tomcat startet nicht. In dem Fall die Datei von Hand entfernen: `sudo rm /opt/tomcat11/tomcat11.sock`, dann `sudo systemctl start tomcat`.

### Schritt 2: NGINX darf die Socket-Datei benutzen

NGINX läuft unter dem Benutzer `www-data`. Die Socket-Datei gehört `tomcat:tomcat` mit den Rechten `rw-rw----`. Damit `www-data` sie benutzen darf, wird dieser Benutzer in die Gruppe `tomcat` aufgenommen:

```bash
sudo usermod -aG tomcat www-data
sudo systemctl restart tomcat nginx
```

Ein bloßes `reload` genügt für die neue Gruppenzugehörigkeit nicht; NGINX muss vollständig neu starten.

### Schritt 3: Tomcat die echte Besucheradresse mitteilen

Hinter einem Reverse Proxy sieht Tomcat jede Anfrage so, als käme sie vom Server selbst und über unverschlüsseltes HTTP. Dann baut das Java-Programm falsche Links und hält jeden Besucher für den Server. Ein zusätzlicher Baustein in `server.xml` behebt das. Innerhalb des `<Host>`-Abschnitts – kurz vor dessen Ende `</Host>` – wird eingefügt:

```xml
<Valve className="org.apache.catalina.valves.RemoteIpValve"
       remoteIpHeader="X-Forwarded-For"
       protocolHeader="X-Forwarded-Proto" />
```

Dieser Baustein liest die beiden Zusatzangaben aus, die NGINX gleich mitschickt (`X-Forwarded-For` für die Besucheradresse, `X-Forwarded-Proto` für „war verschlüsselt"), und stellt sie dem Java-Programm als die wahren Werte hin.

Danach Tomcat neu starten:

```bash
sudo systemctl restart tomcat
```

### Schritt 4: NGINX als Reverse Proxy für `tomcat11.de`

Zuerst das SSL-Zertifikat besorgen. Wie das mit **Certbot** und **Let's Encrypt** geht, steht ausführlich im Kapitel [Webserver](./webserver.md); für eine feste Domain genügt:

```bash
sudo certbot certonly --nginx -d tomcat11.de -d www.tomcat11.de
```

Dann die Konfigurationsdatei `/etc/nginx/sites-available/tomcat11.de` anlegen:

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;
    server_name tomcat11.de www.tomcat11.de;

    ssl_certificate     /etc/letsencrypt/live/tomcat11.de/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/tomcat11.de/privkey.pem;

    # Größere Uploads zulassen
    client_max_body_size 50m;

    location / {
        proxy_pass http://unix:/opt/tomcat11/tomcat11.sock:/;
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;
    listen [::]:80;
    server_name tomcat11.de www.tomcat11.de;
    # Alle unverschlüsselten Aufrufe auf HTTPS umleiten
    return 301 https://$host$request_uri;
}
```

Die Zeile `proxy_pass http://unix:/opt/tomcat11/tomcat11.sock:/;` ist der Weg zur Socket-Datei. Die Schreibweise ist etwas ungewohnt: Nach `http://unix:` folgt der Pfad zur Socket-Datei, dann ein Doppelpunkt und der Pfad innerhalb der Anwendung (`/`). Die vier `proxy_set_header`-Zeilen geben Tomcat weiter, wer die Anfrage gestellt hat und dass sie über HTTPS kam – sie gehören zum Baustein aus Schritt 3.

Die Datei aktiv schalten, prüfen und übernehmen:

```bash
sudo ln -s /etc/nginx/sites-available/tomcat11.de /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Jetzt ist Tomcat unter `https://tomcat11.de/` erreichbar – ohne dass irgendwo Port 8080 offensteht.

### Schritt 5: Firewall

Weil kein Port für Tomcat geöffnet wurde, ist nichts zusätzlich zu sperren. Es genügt, dem Webserver die Ports 80 und 443 zu erlauben:

```bash
sudo ufw allow "Nginx Full"
sudo ufw enable
sudo ufw status verbose
```

Wer statt des Sockets doch den Weg über `127.0.0.1:8080` gewählt hat, sperrt Port 8080 zusätzlich ausdrücklich: `sudo ufw deny 8080/tcp`.

## Eine Anwendung einspielen

Ein Java-Webprogramm wird als `.war`-Datei in das Verzeichnis `webapps` gelegt. Der Dateiname bestimmt die Adresse: `ROOT.war` wird zur Startseite (`https://tomcat11.de/`), `laden.war` läge unter `https://tomcat11.de/laden/`.

```bash
sudo cp meineanwendung.war /opt/tomcat11/webapps/ROOT.war
sudo chown tomcat:tomcat /opt/tomcat11/webapps/ROOT.war
```

Tomcat bemerkt die neue Datei, entpackt sie von selbst und startet das Programm. Ob das klappt, zeigt das Protokoll:

```bash
sudo journalctl -u tomcat -f
```

Zusätzlich schreibt Tomcat nach `/opt/tomcat11/logs/catalina.out`.

Der offizielle Download bringt einige Beispielprogramme mit (`docs`, `examples`, `manager`, `host-manager`). Auf einem öffentlichen Server haben sie nichts zu suchen und werden entfernt:

```bash
sudo rm -rf /opt/tomcat11/webapps/{docs,examples,manager,host-manager}
```

## Fehlersuche

- **502 Bad Gateway** im Browser: NGINX erreicht Tomcat nicht. Grund im Protokoll suchen: `sudo tail -n 20 /var/log/nginx/error.log`.
  - `Permission denied (13)`: Der Benutzer `www-data` ist nicht in der Gruppe `tomcat`, oder NGINX wurde nach `usermod` nur neu geladen statt neu gestartet. `sudo systemctl restart nginx`.
  - `No such file or directory (2)`: Tomcat läuft nicht, oder der Pfad in `unixDomainSocketPath` und `proxy_pass` stimmt nicht überein. `systemctl status tomcat` prüfen.
- **Tomcat startet nicht**, das Protokoll nennt die Socket-Datei als schon vorhanden: übrig gebliebene Datei nach einem Absturz. `sudo rm /opt/tomcat11/tomcat11.sock`, dann starten.
- **Die Socket-Datei direkt testen**, ohne NGINX:

  ```bash
  sudo -u www-data curl --unix-socket /opt/tomcat11/tomcat11.sock http://localhost/
  ```

  Kommt hier HTML zurück, liegt ein etwaiger Fehler in der NGINX-Konfiguration, nicht bei Tomcat.
- **`java -version`** muss 17 oder höher anzeigen. Ältere Java-Versionen starten Tomcat 11 nicht.
- **`OutOfMemoryError`** in `catalina.out`: Die Speichergrenze `-Xmx` ist zu niedrig. In der Dienstdatei erhöhen und `sudo systemctl daemon-reload && sudo systemctl restart tomcat`.

## Für dieses Buch

Die Wissenssammlung dieses Buchs läuft auf MediaWiki und braucht **kein** Tomcat. Tomcat wird nur zum Thema, wenn ein Java-Programm dazukommt:

- **XWiki** als Wissenssystem: Dann folgt man dem Kapitel [XWiki einrichten](../web-stack/xwiki.md), das ein eigenes Sammelpaket mit Tomcat 10 verwendet und die Schritte dort im Zusammenhang beschreibt.
- **Ein selbst geschriebenes Java-Programm**, etwa auf Basis von Spring Boot (siehe [Webframework](./webframework.md)): Dafür ist Tomcat 11 von Hand in `/opt/tomcat11` der saubere Weg – eigener Benutzer, Anbindung an NGINX über einen Unix-Socket, kein offener Port.

Zwei Dinge gehören von Anfang an zum Betrieb: eine passende Speichergrenze für Java (`-Xmx`) und regelmäßige Sicherheitsaktualisierungen für Java und Tomcat. Beim Weg von Hand heißt das, die Tomcat-Version selbst im Blick zu behalten und bei einer neuen Ausgabe das Verzeichnis auszutauschen.

## Fazit

Tomcat ist der Anwendungsserver, der ein Java-Webprogramm trägt. **Tomcat 11** setzt Jakarta EE 11 um und braucht mindestens Java 17. Es lässt sich von Hand aus dem offiziellen Download einrichten – mit eigenem Benutzer, eigenem systemd-Dienst und selbst gesetzter Speichergrenze – oder mit `apt install tomcat11` aus den Paketquellen von Ubuntu 26.04. Davor gehört NGINX als Reverse Proxy für HTTPS. Die Verbindung dahin läuft am besten über einen **Unix-Socket**: In `server.xml` bekommt der `<Connector>` statt eines Ports den Eintrag `unixDomainSocketPath`, NGINX erreicht die Socket-Datei über `proxy_pass http://unix:/opt/tomcat11/tomcat11.sock:/;`, und der Benutzer `www-data` wird dafür in die Gruppe `tomcat` aufgenommen. So steht für Tomcat kein Port offen, und der Zugriff hängt allein an Dateirechten.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
