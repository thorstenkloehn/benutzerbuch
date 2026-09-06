# XWiki einrichten

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Das Kapitel [Wissenssystem](./wissensystem.md) grenzt die Auswahl auf zwei Programme ein: **MediaWiki** und **XWiki**. XWiki ist die passende Wahl, wenn die Datenbank fest auf **PostgreSQL** steht (siehe [Datenbank](../server-einrichten/datenbank.md)) oder wenn die Sammlung neben Fließtext auch feste Datenfelder führen soll – etwa eine Liste von Geräten mit Hersteller, Baujahr und Standort.

Dieses Kapitel zeigt Schritt für Schritt, wie XWiki auf einem Server mit **Ubuntu 26.04 LTS** (siehe [Betriebssystem](../server-einrichten/betriebssystem.md)) eingerichtet wird: von der Paketquelle über die Datenbank und die Ersteinrichtung bis zum öffentlichen Zugang über einen Webserver. Alle Befehle werden in der Textkonsole des Servers eingegeben. Das vorangestellte `sudo` bedeutet: mit Verwaltungsrechten ausführen.

## Was bei XWiki zusammenspielt

XWiki ist in der Programmiersprache **Java** geschrieben. Ein Java-Webprogramm läuft nicht allein, sondern braucht einen **Anwendungsserver** – ein Programm, das das Webprogramm lädt, am Laufen hält und Anfragen an es weiterreicht. Für XWiki ist das **Apache Tomcat**. Man kann sich Tomcat wie einen Motorraum vorstellen: XWiki ist der Motor, Tomcat das Gehäuse mit Halterungen, Anschlüssen und Anlasser drumherum.

Vier Teile arbeiten also zusammen:

- **Java** – die Laufzeitumgebung, die den Programmcode ausführt (siehe [Laufzeitumgebung](../server-einrichten/laufzeitumgebung.md)).
- **Tomcat** – der Anwendungsserver, der XWiki hält und auf einem Netzwerk-Port lauscht.
- **PostgreSQL** – die Datenbank, in der alle Seiten, Versionen und Einstellungen liegen (siehe [Datenbank](../server-einrichten/datenbank.md)).
- **Ein Webserver** davor – nimmt die Anfragen aus dem Internet entgegen, verschlüsselt die Verbindung (HTTPS) und reicht sie an Tomcat weiter (siehe [Webserver](../server-einrichten/webserver.md)). In diesem Buch ist das **NGINX**.

Die XWiki-Pakete des Projekts bringen Java und Tomcat als Abhängigkeiten mit; man muss sie nicht getrennt installieren. PostgreSQL und den Webserver richtet man wie in den genannten Kapiteln beschrieben ein.

## Voraussetzungen

Bevor es losgeht, sollte Folgendes vorhanden sein:

- Ein Server mit Ubuntu 26.04 und mindestens **2 GB freiem Arbeitsspeicher** allein für XWiki. Weniger führt im Betrieb zu Abstürzen.
- **PostgreSQL** ist eingerichtet (siehe [Datenbank](../server-einrichten/datenbank.md)). Falls die Datenbank für XWiki noch fehlt, legt der Installationsassistent sie in Schritt 3 selbst an.
- Eine **Domain**, die auf die IP-Adresse des Servers zeigt, zum Beispiel `wiki.meine-domain.de`. Sie wird für das SSL-Zertifikat und den Webserver gebraucht.
- **NGINX** ist installiert (siehe [Webserver](../server-einrichten/webserver.md)).

## Schritt 1: Die Paketquelle von XWiki eintragen

XWiki wird nicht aus den normalen Ubuntu-Paketquellen installiert, sondern aus einer eigenen Paketquelle, die das XWiki-Projekt selbst betreibt. Damit `apt` diese Quelle nutzen kann, sind zwei Dateien nötig: ein digitaler Schlüssel, mit dem Ubuntu die Echtheit der Pakete prüft, und eine Datei mit der Adresse der Quelle.

```bash
sudo wget https://maven.xwiki.org/xwiki-keyring.gpg -O /usr/share/keyrings/xwiki-keyring.gpg
sudo wget "https://maven.xwiki.org/stable/xwiki-stable.list" -O /etc/apt/sources.list.d/xwiki-stable.list
```

Die Quelle `stable` enthält alle fertigen Versionen ohne Vorabausgaben. Wer lieber eine Version mit besonders langer Pflegezusage möchte, ersetzt in beiden Zeilen `stable` durch `lts` (englisch für „Long Term Support", also Langzeitunterstützung).

Danach die Paketliste neu einlesen, damit die neue Quelle bekannt wird:

```bash
sudo apt update
```

## Schritt 2: Nach verfügbaren Versionen sehen

Ein kurzer Blick zeigt, welche XWiki-Pakete jetzt zur Auswahl stehen:

```bash
apt-cache search xwiki
```

Und welche Version das Paket für PostgreSQL mitbringt:

```bash
apt-cache policy xwiki-tomcat10-pgsql
```

Das Paket `xwiki-tomcat10-pgsql` ist ein **Sammelpaket**: Es zieht XWiki selbst, den Anwendungsserver Tomcat 10, die Java-Laufzeit und die Anbindung an PostgreSQL in einem Rutsch nach.

## Schritt 3: XWiki installieren

```bash
sudo apt install xwiki-tomcat10-pgsql
```

Während der Installation stellt der Assistent einige Fragen zur Datenbank:

- Ob die Datenbank automatisch eingerichtet werden soll – hier **Ja** wählen.
- Den Namen der Datenbank und des Datenbankbenutzers (Vorschlag: beides `xwiki`).
- Ein Passwort für diesen Datenbankbenutzer. Es wird gleich zweimal abgefragt und danach automatisch in die XWiki-Konfiguration eingetragen.

Sind die Angaben gemacht, legt der Assistent die leere Datenbank an, verbindet XWiki damit und startet Tomcat. Für die nächsten Schritte wird Tomcat zunächst wieder angehalten:

```bash
sudo systemctl stop tomcat10
```

Nach der Installation liegen die wichtigsten Dateien an diesen Stellen:

| Datei oder Ordner | Inhalt |
| --- | --- |
| `/etc/xwiki/` | Konfiguration von XWiki (unter anderem die Datenbankverbindung) |
| `/etc/tomcat10/server.xml` | Einstellungen des Anwendungsservers, zum Beispiel der Port |
| `/etc/default/tomcat10` | Starteinstellungen für Tomcat, zum Beispiel der Arbeitsspeicher |
| `/var/lib/xwiki/` | Daten von XWiki, etwa hochgeladene Dateien |

## Schritt 4: Arbeitsspeicher für Tomcat festlegen

Java teilt sich seinen Arbeitsspeicher nicht nach Bedarf zu, sondern bekommt eine feste Obergrenze vorgegeben. Ist sie zu niedrig, bricht XWiki unter Last ab. Für eine kleine bis mittlere Sammlung sind 2 GB ein guter Startwert. Die Grenze wird in der Datei `/etc/default/tomcat10` gesetzt:

```bash
sudo nano /etc/default/tomcat10
```

Dort die Zeile mit `JAVA_OPTS` suchen oder ergänzen, sodass sie so aussieht:

```
JAVA_OPTS="-Xms512m -Xmx2048m -XX:+UseG1GC"
```

- `-Xms512m` ist der Speicher, den Java gleich beim Start belegt.
- `-Xmx2048m` ist die Obergrenze – hier 2048 MB, also 2 GB. Auf einem größeren Server darf dieser Wert höher liegen, aber nie an die Grenze des gesamten Arbeitsspeichers reichen; das Betriebssystem und PostgreSQL brauchen ebenfalls Platz.
- `-XX:+UseG1GC` wählt ein Aufräumverfahren, das für Programme mit vielen gleichzeitigen Zugriffen gut geeignet ist.

## Schritt 5: Tomcat nur auf den eigenen Server hören lassen

Von Haus aus lauscht Tomcat auf Port 8080 und nimmt Verbindungen aus dem ganzen Netz an. Beides wird geändert: Der Port wird auf **9000** gelegt (damit er nicht mit anderen Diensten kollidiert), und Tomcat soll nur noch auf `127.0.0.1` hören – das ist der Rechner selbst, von außen nicht erreichbar.

Der Grund für die zweite Einschränkung: Beim allerersten Aufruf zeigt XWiki einen Einrichtungsassistenten („Distribution Wizard"), der das Administratorkonto anlegt. Solange dieser Assistent nicht durchlaufen ist, darf die Seite auf keinen Fall öffentlich erreichbar sein. Statt den Port kurz in der Firewall zu öffnen, bleibt er komplett auf dem Server und wird für die Ersteinrichtung über einen verschlüsselten Tunnel erreicht (Schritt 6).

```bash
sudo nano /etc/tomcat10/server.xml
```

In der Datei den folgenden Abschnitt suchen:

```xml
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

Und ihn so ändern – der Port wird zu `9000`, und das neue Attribut `address` bindet den Dienst an den eigenen Rechner:

```xml
<Connector address="127.0.0.1" port="9000" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

Die Bindung an `127.0.0.1` bleibt dauerhaft bestehen. NGINX läuft auf demselben Server und erreicht Tomcat über diese lokale Adresse; ein Zugang von außen über Port 9000 ist im normalen Betrieb nie nötig.

## Schritt 6: Die Ersteinrichtung über einen SSH-Tunnel

Jetzt Tomcat starten:

```bash
sudo systemctl start tomcat10
```

XWiki läuft nun, ist aber nur vom Server selbst erreichbar. Um den Einrichtungsassistenten trotzdem im Browser des eigenen Rechners zu öffnen, wird ein **SSH-Tunnel** aufgebaut. Der folgende Befehl wird auf dem **eigenen Rechner** eingegeben, nicht auf dem Server:

```bash
ssh -L 9000:127.0.0.1:9000 admin@SERVER-IP
```

Der Teil `-L 9000:127.0.0.1:9000` heißt „Local Forwarding", auf Deutsch etwa „örtliche Weiterleitung". Er öffnet auf dem eigenen Rechner den Port 9000 und verbindet ihn durch die verschlüsselte SSH-Verbindung mit Port 9000 auf dem Server. Solange dieses SSH-Fenster offen ist, landet jeder Aufruf von `http://localhost:9000/` im Browser des eigenen Rechners bei Tomcat auf dem Server.

Nun im Browser `http://localhost:9000/` aufrufen. Der Einrichtungsassistent erscheint. Er führt durch drei Punkte:

1. Ein Passwort für das Administratorkonto festlegen.
2. Die Grundausstattung („Flavor") installieren – die Standardauswahl übernehmen.
3. Am Ende bestätigen, dass die Einrichtung abgeschlossen ist.

Danach das SSH-Fenster schließen; der Tunnel wird nicht mehr gebraucht.

## Schritt 7: NGINX als Reverse Proxy einrichten

Damit die Wissenssammlung öffentlich und verschlüsselt erreichbar ist, kommt NGINX davor. Es nimmt die Anfragen auf Port 443 (HTTPS) entgegen und reicht sie an Tomcat auf Port 9000 weiter. Diese Rolle heißt **Reverse Proxy** (siehe [Webserver](../server-einrichten/webserver.md)).

Zuerst das SSL-Zertifikat besorgen. Wie das mit **Certbot** und **Let's Encrypt** geht, steht ausführlich im Kapitel [Webserver](../server-einrichten/webserver.md); für eine feste Domain genügt:

```bash
sudo certbot certonly --nginx -d wiki.meine-domain.de
```

Dann die Konfigurationsdatei `/etc/nginx/sites-available/xwiki` anlegen:

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;
    server_name wiki.meine-domain.de;

    ssl_certificate     /etc/letsencrypt/live/wiki.meine-domain.de/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/wiki.meine-domain.de/privkey.pem;

    # Große Anhänge zulassen
    client_max_body_size 100m;

    location / {
        proxy_pass http://127.0.0.1:9000/;
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;
    listen [::]:80;
    server_name wiki.meine-domain.de;
    # Alle unverschlüsselten Aufrufe auf HTTPS umleiten
    return 301 https://$host$request_uri;
}
```

Die vier `proxy_set_header`-Zeilen geben Tomcat weiter, wer die Anfrage ursprünglich gestellt hat und dass sie über HTTPS kam. Ohne sie baut XWiki falsche Links und hält jeden Besucher für den Server selbst.

Die Datei aktiv schalten, prüfen und übernehmen:

```bash
sudo ln -s /etc/nginx/sites-available/xwiki /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Jetzt ist die Wissenssammlung unter `https://wiki.meine-domain.de/` erreichbar.

## Schritt 8: Port 9000 in der Firewall sperren

Die Bindung an `127.0.0.1` aus Schritt 5 schützt Port 9000 bereits auf Netzwerkebene. Als zweite, unabhängige Absicherung wird der Port zusätzlich in der Firewall gesperrt – für den Fall, dass die Bindung später versehentlich wieder geöffnet wird. Ubuntu bringt dafür **UFW** mit („Uncomplicated Firewall", also unkomplizierte Firewall):

```bash
sudo ufw allow "Nginx Full"
sudo ufw deny 9000/tcp
sudo ufw enable
sudo ufw status verbose
```

- `Nginx Full` öffnet die Ports 80 und 443 für den Webserver.
- `deny 9000/tcp` schließt Port 9000 ausdrücklich. Bei der UFW-Standardeinstellung (alles Eingehende ist ohnehin gesperrt) ist diese Regel technisch überflüssig, macht die Absicht aber in `ufw status` sichtbar.

Für spätere Wartungszugriffe direkt auf Port 9000 – etwa zur Fehlersuche ohne NGINX – eignet sich weiterhin der SSH-Tunnel aus Schritt 6, ohne dass ein Port geöffnet werden muss.

## Noch strikter: ein Unix-Socket statt eines Ports

Statt eines Netzwerk-Ports kann Tomcat auch über eine **Socket-Datei** ansprechbar sein – einen sogenannten Unix-Socket. Das ist ein besonderer Eintrag im Dateisystem, über den zwei Programme auf demselben Rechner miteinander reden, ganz ohne Netzwerk. Der Vorteil: Es gibt keinen Port, der aus Versehen geöffnet werden könnte, und die Zugriffsrechte regelt das Dateisystem.

**In Tomcat** wird dazu in `/etc/tomcat10/server.xml` der `<Connector>` ohne Port, aber mit einem Pfad zur Socket-Datei angelegt:

```xml
<Connector protocol="HTTP/1.1"
           unixDomainSocketPath="/var/lib/tomcat10/xwiki.sock"
           unixDomainSocketPathPermissions="rw-rw----"
           connectionTimeout="20000" />
```

Damit NGINX (das als Benutzer `www-data` läuft) die Socket-Datei benutzen darf, wird dieser Benutzer in die Gruppe `tomcat` aufgenommen:

```bash
sudo usermod -aG tomcat www-data
sudo systemctl restart tomcat10 nginx
```

**In NGINX** verweist `proxy_pass` dann auf die Socket-Datei statt auf eine Adresse:

```nginx
    location / {
        proxy_pass http://unix:/var/lib/tomcat10/xwiki.sock:/;
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
```

Auch die Ersteinrichtung lässt sich über einen Tunnel zu dieser Socket-Datei erledigen. SSH kann einen lokalen Port mit einer entfernten Socket-Datei verbinden. Der Befehl auf dem eigenen Rechner lautet dann:

```bash
ssh -L 9000:/var/lib/tomcat10/xwiki.sock admin@SERVER-IP
```

Der Unterschied zu Schritt 6: Hinter dem lokalen Port `9000` steht diesmal kein entfernter Port, sondern der Pfad `/var/lib/tomcat10/xwiki.sock` auf dem Server. Im Browser wird weiterhin `http://localhost:9000/` aufgerufen; die Anfrage läuft durch den Tunnel und endet an der Socket-Datei, an der Tomcat lauscht.

Der Socket-Weg ist etwas aufwendiger einzurichten und bei der Fehlersuche unhandlicher (man kann nicht eben mit einem Browser auf einen Port schauen). Wer es einfach halten will, bleibt bei Port 9000 auf `127.0.0.1` aus Schritt 5. Wer jede Möglichkeit eines offenen Ports ausschließen möchte, nimmt den Socket.

## Für dieses Buch

XWiki bringt eine eigene Volltextsuche mit (auf Basis von Apache Solr); der separate Suchdienst **Meilisearch** oder **Typesense** aus dem Kapitel [Datenbank](../server-einrichten/datenbank.md) ist auf MediaWiki gemünzt und für XWiki nicht nötig. Die **Bedeutungssuche** mit `pgvector` bleibt möglich, ist bei XWiki aber ein eigenes Zusatzprojekt und kein fertiger Baustein.

Empfohlen wird der Betrieb **direkt auf dem Server** (ohne Container, siehe [Containerisierung von Software](../server-einrichten/containerisierung.md)): Java, Tomcat, XWiki und PostgreSQL laufen nebeneinander, NGINX steht davor. Für den Zugang von außen genügt Port 9000 auf `127.0.0.1`; der Unix-Socket ist die strengere, aber aufwendigere Alternative.

Zwei Dinge gehören von Anfang an zum Betrieb dazu: eine ausreichend hohe Speichergrenze für Tomcat (Schritt 4) und eine regelmäßige, automatische Sicherung der PostgreSQL-Datenbank an einen zweiten Ort (`pg_dump`, siehe [Datenbank](../server-einrichten/datenbank.md)).

## Fazit

XWiki besteht aus vier zusammenspielenden Teilen: der Java-Laufzeit, dem Anwendungsserver Tomcat, der Datenbank PostgreSQL und einem Webserver davor. Die Installation läuft über die eigene Paketquelle des XWiki-Projekts; das Sammelpaket `xwiki-tomcat10-pgsql` bringt alles Nötige mit und fragt bei der Einrichtung die Datenbankdaten ab. Tomcat wird auf Port 9000 gelegt und an `127.0.0.1` gebunden, sodass die unfertige Installation nicht öffentlich erreichbar ist; die einmalige Kontoanlage im Einrichtungsassistenten geschieht über einen SSH-Tunnel. Anschließend stellt NGINX als Reverse Proxy die verschlüsselte, öffentliche Adresse bereit, und die Firewall sperrt Port 9000 zusätzlich ab. Wer den Port ganz vermeiden will, lässt Tomcat über eine Unix-Socket-Datei sprechen.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
