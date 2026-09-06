# Git Hosting

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Der Quelltext dieses Buchs liegt in einem **Git-Verzeichnis** (siehe [Docs-as-Code](../entwicklungs-rechner/docs-as-code.md)). Git merkt sich jede Änderung: wer sie gemacht hat, wann und warum. Solange dieses Verzeichnis nur auf dem eigenen Rechner liegt, ist es aber schlecht geschützt und lässt sich nicht mit anderen teilen. Dafür braucht es eine zentrale Stelle im Netz, bei der alle Beteiligten ihre Änderungen abliefern und die Änderungen der anderen abholen. Diese Stelle heißt **Git-Hosting**.

Bekannt ist vor allem **GitHub**. Das ist ein Dienst einer Firma; die Verzeichnisse liegen auf deren Servern. Man kann so einen Dienst aber auch selbst betreiben (siehe [Inhalts-Software selbst betreiben](../grundlagen/inhalt-software-verwalten.md)). Dieses Kapitel zeigt, wie man dafür **Forgejo** auf einem Server mit **Ubuntu 26.04 LTS** (siehe [Betriebssystem](./betriebssystem.md)) einrichtet und über einen **Unix-Socket** an den Webserver NGINX anbindet – am Beispiel der Adresse `https://git.meine-domain.de`.

Alle Befehle werden in der Textkonsole des Servers eingegeben. Ein vorangestelltes `sudo` bedeutet: mit Verwaltungsrechten ausführen.

## Was Git-Hosting ist

**Git** allein ist nur ein Programm auf der Festplatte. Es verwaltet die Versionsgeschichte eines Verzeichnisses. Damit mehrere Leute am selben Projekt arbeiten können, muss es eine gemeinsame Ablage geben, die alle erreichen. Diese gemeinsame Ablage nennt Git ein **entferntes Verzeichnis** (auf Englisch *remote*).

Ein Git-Hosting-Dienst stellt solche entfernten Verzeichnisse bereit und packt eine Weboberfläche darum herum. Damit kann man:

- **Verzeichnisse anlegen und verwalten** – öffentlich sichtbar oder nur für angemeldete Personen.
- **Den Quelltext im Browser durchblättern**, ohne ihn erst herunterzuladen.
- **Änderungswünsche besprechen.** Wer etwas beitragen möchte, reicht seine Änderungen als Vorschlag ein (auf Englisch *pull request*). Andere sehen den Unterschied Zeile für Zeile, schreiben Anmerkungen und übernehmen den Vorschlag erst, wenn er passt.
- **Aufgaben und Fehler sammeln** – als Liste von Einträgen (auf Englisch *issues*), die man einzelnen Personen zuordnen und abhaken kann.
- **Automatische Abläufe starten**, sobald neue Änderungen ankommen – zum Beispiel das Buch bauen und veröffentlichen (siehe [Webseiten und Blogs](../entwicklungs-rechner/docs-as-code-webseiten.md)).

Der eigentliche Datenaustausch läuft über zwei Wege: **HTTPS** (dieselbe verschlüsselte Verbindung wie beim Surfen) oder **SSH** (eine verschlüsselte Verbindung mit Schlüsseldatei statt Passwort). Die Weboberfläche selbst ist immer über HTTPS erreichbar.

## Was Forgejo ist

**Forgejo** (gesprochen etwa „for-dsche-jo", vom Esperanto-Wort *forĝejo* – die Schmiede) ist ein Programm, mit dem man einen eigenen Git-Hosting-Dienst betreibt. Es ist **Freie Software** (siehe [Freies Wissen](../grundlagen/freies-wissen.md)) unter der Lizenz GPL Version 3 und wird von **Codeberg e. V.** getragen, einem gemeinnützigen Verein aus Berlin.

### Woher Forgejo kommt

Forgejo ist eine **Abspaltung von Gitea** (auf Englisch *fork*): eine Kopie des Quelltexts, die von da an einen eigenen Weg geht. Gitea wiederum war selbst eine Abspaltung eines älteren Programms namens Gogs.

Der Grund für die Abspaltung war ein Streit um die Kontrolle über das Projekt. Ende 2022 übertrug der Hauptentwickler von Gitea die Namensrechte und die Leitung an eine gewinnorientierte Firma, ohne die Gemeinschaft der Mitwirkenden vorher zu fragen. Ein Teil dieser Gemeinschaft wollte das Projekt lieber in den Händen eines gemeinnützigen Vereins sehen und gründete daraufhin Forgejo. Anfangs übernahm Forgejo noch alle Änderungen von Gitea; seit Anfang 2024 sind die beiden Programme vollständig getrennt und entwickeln sich auseinander.

Für den Betrieb heißt das: Forgejo wird von einem Verein nach demokratischen Regeln geführt, nicht von einer einzelnen Firma. Wer von Gitea zu Forgejo wechselt, kann seine Daten in aller Regel unverändert übernehmen, weil beide noch dieselbe Grundstruktur haben.

### Was Forgejo kann

Forgejo ist in der Programmiersprache **Go** geschrieben (siehe [Programmiersprachen](../entwicklungs-rechner/programmiersprachen.md)). Ein Go-Programm wird zu einer einzigen ausführbaren Datei übersetzt, die ohne weitere Bestandteile läuft. Deshalb besteht die Installation im Kern nur darin, diese eine Datei auf den Server zu legen.

Der Funktionsumfang ist groß:

| Bereich | Was Forgejo bietet |
| --- | --- |
| Verzeichnisse | anlegen, durchblättern, Rechte je Person oder Gruppe vergeben |
| Zusammenarbeit | Änderungsvorschläge, Anmerkungen Zeile für Zeile, Aufgabenlisten |
| Automatische Abläufe | „Forgejo Actions" – Aufgaben, die bei jeder Änderung anlaufen |
| Pakete | eine Ablage für fertige Programmbausteine (z. B. für Java, JavaScript, Container) |
| Wiki | zu jedem Verzeichnis eine kleine Sammlung von Erklärseiten |
| Anmeldung | eigene Konten oder Anbindung an einen zentralen Verzeichnisdienst |

Ein längerfristiges Ziel des Projekts ist die **Föderation**: Verschieden betriebene Forgejo-Server sollen sich untereinander verständigen können, sodass jemand auf Server A einen Änderungsvorschlag für ein Verzeichnis auf Server B einreichen kann, ohne dort ein Konto zu haben. Dieser Teil ist noch in Arbeit.

### Forgejo im Vergleich

| | Forgejo (selbst betrieben) | GitHub | GitLab (selbst betrieben) |
| --- | --- | --- | --- |
| Betreiber | man selbst | Firma (Microsoft) | man selbst |
| Lizenz | Freie Software (GPL 3) | geschlossen | Kern frei, Zusätze geschlossen |
| Ressourcen | sehr sparsam | – | vergleichsweise hoch |
| Einrichtung | eine Programmdatei plus Datenbank | entfällt (fertiger Dienst) | umfangreich |

GitLab bietet mehr fertige Funktionen, verlangt aber deutlich mehr Arbeitsspeicher und Pflege. Für eine kleine Redaktion, die vor allem ein sicheres, gemeinsames Zuhause für ihre Texte und ihren Quelltext braucht, ist Forgejo die passende Größe.

## Voraussetzungen

- Ein Server mit **Ubuntu 26.04 LTS** und einem Webserver davor. Dieses Kapitel geht von **NGINX** aus (siehe [Webserver](./webserver.md)).
- Eine **Domain**, deren DNS-Eintrag auf den Server zeigt – hier `git.meine-domain.de`.
- Eine **Datenbank**. Forgejo kann eine einzelne Datei (SQLite) verwenden oder eine richtige Datenbank (PostgreSQL, MariaDB). Für wenige Nutzer genügt SQLite und spart die Einrichtung. Wer mit stärkerem Andrang rechnet, nimmt PostgreSQL (siehe [Datenbank](./datenbank.md)). Dieses Kapitel zeigt den Weg mit SQLite und nennt die Stellen, an denen PostgreSQL abweicht.
- **Arbeitsspeicher:** Forgejo selbst kommt mit wenigen Hundert MB aus. Zusammen mit Betriebssystem und Webserver reicht ein kleiner Server mit 1 bis 2 GB.

## Forgejo installieren

### Schritt 1: Git auf dem Server bereitstellen

Forgejo ruft im Hintergrund das Programm `git` auf. Für größere Dateien kommt `git-lfs` dazu:

```bash
sudo apt update
sudo apt install git git-lfs
```

### Schritt 2: Die Programmdatei herunterladen und prüfen

Die aktuelle Versionsnummer steht auf der Seite <https://forgejo.org/download/>. Sie wird hier einmal in eine Variable geschrieben, damit die folgenden Befehle unverändert bleiben:

```bash
cd /tmp
VERSION=16.0.3

# Die Programmdatei für 64-Bit-PCs herunterladen
wget "https://code.forgejo.org/forgejo/forgejo/releases/download/v${VERSION}/forgejo-${VERSION}-linux-amd64"

# Die zugehörige Unterschrift herunterladen
wget "https://code.forgejo.org/forgejo/forgejo/releases/download/v${VERSION}/forgejo-${VERSION}-linux-amd64.asc"

# Den öffentlichen Schlüssel des Forgejo-Projekts holen
gpg --keyserver keys.openpgp.org --recv EB114F5E6C0DC2BCDD183550A4B61A2DC5923710

# Die Datei gegen die Unterschrift prüfen – die Ausgabe muss "Good signature" enthalten
gpg --verify "forgejo-${VERSION}-linux-amd64.asc" "forgejo-${VERSION}-linux-amd64"
```

Die **digitale Unterschrift** ist ein Nachweis, dass die Datei wirklich vom Forgejo-Projekt stammt und unterwegs nicht verändert wurde. Meldet `gpg` etwas anderes als eine gültige Unterschrift, wird die Datei nicht benutzt, sondern neu geladen. Der Hinweis „This key is not certified with a trusted signature" ist dabei normal – er besagt nur, dass man dem Schlüssel noch nicht ausdrücklich vertraut hat.

Danach die geprüfte Datei an ihren Platz legen:

```bash
sudo cp "forgejo-${VERSION}-linux-amd64" /usr/local/bin/forgejo
sudo chmod 755 /usr/local/bin/forgejo
```

### Schritt 3: Einen eigenen Benutzer anlegen

Forgejo soll **nicht** mit Verwaltungsrechten laufen. Sonst hätte ein Fehler im Programm sofort Zugriff auf den ganzen Server. Deshalb bekommt es einen eigenen Benutzer namens `git`:

```bash
sudo adduser --system --shell /bin/bash --gecos 'Git Versionsverwaltung' \
  --group --disabled-password --home /home/git git
```

- `--system` legt ein Dienstkonto an, kein persönliches.
- `--disabled-password` verhindert das Anmelden mit einem Passwort.
- `--shell /bin/bash` wird hier gebraucht, weil Forgejo den Zugang über SSH später über diesen Benutzer abwickelt.

### Schritt 4: Verzeichnisse anlegen

Forgejo trennt seine Dateien in zwei Bereiche: die **Daten** (Verzeichnisse, Datenbank, Anhänge) unter `/var/lib/forgejo` und die **Konfiguration** unter `/etc/forgejo`.

```bash
sudo mkdir /var/lib/forgejo
sudo chown git:git /var/lib/forgejo
sudo chmod 750 /var/lib/forgejo

sudo mkdir /etc/forgejo
sudo chown root:git /etc/forgejo
sudo chmod 770 /etc/forgejo
```

Das Konfigurationsverzeichnis gehört `root`, die Gruppe `git` darf hineinschreiben. Das ist nötig, weil die Ersteinrichtung im Browser die Konfigurationsdatei anlegt. Danach werden die Rechte wieder enger gesetzt (siehe Schritt 8).

### Schritt 5: Eine erste Konfiguration schreiben

Forgejo liest beim Start die Datei `/etc/forgejo/app.ini`. Für den ersten Start genügt ein kleines Gerüst; den Rest ergänzt die Ersteinrichtung im Browser. Die Datei wird als Benutzer `git` angelegt:

```bash
sudo -u git nano /etc/forgejo/app.ini
```

Mit diesem Inhalt:

```ini
APP_NAME = Git der Redaktion
RUN_USER = git
RUN_MODE = prod

[server]
PROTOCOL  = http+unix
HTTP_ADDR = /run/forgejo/forgejo.sock
UNIX_SOCKET_PERMISSION = 660
DOMAIN    = git.meine-domain.de
ROOT_URL  = https://git.meine-domain.de/
SSH_DOMAIN = git.meine-domain.de

[database]
DB_TYPE = sqlite3
PATH    = /var/lib/forgejo/data/forgejo.db
```

Was die wichtigsten Zeilen bedeuten:

- `PROTOCOL = http+unix` und `HTTP_ADDR = …` sagen Forgejo, dass es **nicht** auf einem Netzwerk-Port lauschen soll, sondern an einer **Socket-Datei**. Was das ist und warum es sich lohnt, erklären die Kapitel [Unix-Socket](../entwicklungs-rechner/unix-socket.md) und [Unix-Socket bei NGINX](./nginx-unix-socket.md). Kurz gesagt: So steht kein Port offen, der aus Versehen ins Netz geöffnet werden könnte, und der Zugriff hängt allein an Dateirechten.
- `ROOT_URL` ist die Adresse, unter der Besucher Forgejo erreichen. Sie muss stimmen, sonst baut Forgejo falsche Links.
- Für **PostgreSQL** statt SQLite lautet der `[database]`-Abschnitt anders (`DB_TYPE = postgres`, dazu `HOST`, `NAME`, `USER`, `PASSWD`); die Werte stammen aus dem Kapitel [Datenbank](./datenbank.md).

### Schritt 6: Den Dienst einrichten

Damit Forgejo beim Serverstart automatisch hochfährt, wird es als **systemd-Dienst** eingetragen. Das Projekt liefert eine fertige Dienstdatei mit:

```bash
sudo wget -O /etc/systemd/system/forgejo.service \
  https://code.forgejo.org/forgejo/forgejo/raw/branch/forgejo/contrib/systemd/forgejo.service
```

Diese Datei setzt bereits die richtigen Pfade (`WorkingDirectory=/var/lib/forgejo`, Konfiguration unter `/etc/forgejo/app.ini`). Eine Ergänzung ist nötig, damit das Verzeichnis für die Socket-Datei bei jedem Start angelegt wird:

```bash
sudo systemctl edit forgejo
```

Im vorgesehenen Bereich eintragen:

```ini
[Service]
RuntimeDirectory = forgejo
RuntimeDirectoryMode = 0750
```

`RuntimeDirectory = forgejo` sorgt dafür, dass systemd beim Start des Dienstes das Verzeichnis `/run/forgejo` anlegt – dem Benutzer `git` gehörend – und beim Stoppen wieder entfernt. Genau dort legt Forgejo dann seine Socket-Datei ab.

Dann den Dienst bekannt machen, einschalten und starten:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now forgejo
sudo systemctl status forgejo
```

`status` sollte `active (running)` zeigen. Zur Kontrolle, dass die Socket-Datei da ist:

```bash
ls -l /run/forgejo/forgejo.sock
```

### Schritt 7: NGINX als Reverse Proxy

Der Webserver **NGINX** nimmt die Verbindungen aus dem Internet an, verschlüsselt sie (HTTPS) und reicht sie an die Socket-Datei weiter. Diese Rolle heißt **Reverse Proxy** (siehe [Webserver](./webserver.md)).

Zuerst darf der Benutzer `www-data`, unter dem NGINX läuft, die Socket-Datei benutzen. Die Datei gehört `git:git` mit den Rechten `rw-rw----`. Also wird `www-data` in die Gruppe `git` aufgenommen:

```bash
sudo usermod -aG git www-data
sudo systemctl restart forgejo nginx
```

Ein bloßes `reload` genügt für die neue Gruppenzugehörigkeit nicht; NGINX muss vollständig neu starten.

Dann das SSL-Zertifikat besorgen. Wie das mit **Certbot** und **Let's Encrypt** genau geht, steht im Kapitel [Webserver](./webserver.md); für eine feste Domain genügt:

```bash
sudo certbot certonly --nginx -d git.meine-domain.de
```

Jetzt die Konfigurationsdatei `/etc/nginx/sites-available/git.meine-domain.de` anlegen:

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;
    server_name git.meine-domain.de;

    ssl_certificate     /etc/letsencrypt/live/git.meine-domain.de/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/git.meine-domain.de/privkey.pem;

    # Git-Verzeichnisse und Anhänge können groß werden
    client_max_body_size 512m;

    # Kodierte Schrägstriche in Adressen nicht zusammenfassen
    merge_slashes off;

    location / {
        proxy_pass http://unix:/run/forgejo/forgejo.sock:/;
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Für die Live-Aktualisierung mancher Seiten
        proxy_set_header Upgrade    $http_upgrade;
        proxy_set_header Connection $http_connection;
    }
}

server {
    listen 80;
    listen [::]:80;
    server_name git.meine-domain.de;
    # Alle unverschlüsselten Aufrufe auf HTTPS umleiten
    return 301 https://$host$request_uri;
}
```

Die Zeile `proxy_pass http://unix:/run/forgejo/forgejo.sock:/;` ist der Weg zur Socket-Datei. Die Schreibweise ist etwas ungewohnt: Nach `http://unix:` folgt der Pfad zur Socket-Datei, dann ein Doppelpunkt und der Pfad innerhalb der Anwendung (`/`).

Die Datei aktiv schalten, prüfen und übernehmen:

```bash
sudo ln -s /etc/nginx/sites-available/git.meine-domain.de /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Weil kein Port für Forgejo geöffnet wurde, ist an der Firewall nichts zusätzlich zu sperren. Es genügt, dem Webserver die Ports 80 und 443 zu erlauben (siehe [Webserver](./webserver.md)).

### Schritt 8: Die Ersteinrichtung im Browser

Jetzt `https://git.meine-domain.de/` im Browser aufrufen. Beim ersten Mal zeigt Forgejo eine **Einrichtungsseite**. Die meisten Felder sind aus der `app.ini` schon vorbelegt. Wichtig sind:

1. **Datenbank:** unverändert lassen (SQLite bzw. die PostgreSQL-Werte).
2. **Verwaltungskonto erstellen:** ganz unten Benutzername, Passwort und E-Mail-Adresse für das erste Konto eintragen. Dieses Konto hat volle Rechte. Diesen Schritt nicht überspringen – sonst wird das erste selbst registrierte Konto zum Verwalter.
3. **Weitere Einstellungen** (aufklappbar): Hier lässt sich „Selbstregistrierung deaktivieren" ankreuzen, damit sich nicht Fremde ein Konto anlegen können.

Nach einem Klick auf „Forgejo installieren" schreibt das Programm die vollständige `app.ini` und startet neu. Danach die Rechte an der Datei wieder eng setzen:

```bash
sudo chmod 640 /etc/forgejo/app.ini
sudo chmod 750 /etc/forgejo
```

Die Einrichtung ist damit abgeschlossen. Die Anmeldung erfolgt oben rechts mit dem eben angelegten Verwaltungskonto.

### Schritt 9: Zugang über SSH (empfohlen)

Für das tägliche Arbeiten ist der Weg über **SSH** bequemer als über HTTPS, weil man kein Passwort und keinen Zugangs-Token eingeben muss. Forgejo nutzt dafür den vorhandenen SSH-Dienst des Servers und den Benutzer `git`.

In der Weboberfläche hinterlegt jede Person unter „Einstellungen → SSH-Schlüssel" ihren **öffentlichen Schlüssel** (die Datei `~/.ssh/id_ed25519.pub` auf dem eigenen Rechner, siehe [Allgemeine Einstellungen](../entwicklungs-rechner/allgemeine-einstellungen.md)). Von da an funktioniert das Abliefern und Abholen ohne weitere Eingabe. Die Adresse eines Verzeichnisses sieht dann so aus:

```
git@git.meine-domain.de:redaktion/benutzerhandbuch.git
```

## Aktualisieren

Forgejo besteht aus einer einzigen Programmdatei. Eine Aktualisierung heißt: neue Datei herunterladen, prüfen, austauschen, Dienst neu starten.

```bash
cd /tmp
VERSION=16.0.4   # neue Versionsnummer von forgejo.org/download

wget "https://code.forgejo.org/forgejo/forgejo/releases/download/v${VERSION}/forgejo-${VERSION}-linux-amd64"
wget "https://code.forgejo.org/forgejo/forgejo/releases/download/v${VERSION}/forgejo-${VERSION}-linux-amd64.asc"
gpg --verify "forgejo-${VERSION}-linux-amd64.asc" "forgejo-${VERSION}-linux-amd64"

sudo systemctl stop forgejo
sudo cp "forgejo-${VERSION}-linux-amd64" /usr/local/bin/forgejo
sudo chmod 755 /usr/local/bin/forgejo
sudo systemctl start forgejo
```

Vor einer Aktualisierung eine **Sicherung** anlegen. Forgejo bringt dafür einen Befehl mit, der Datenbank, Verzeichnisse und Konfiguration in eine einzige Datei packt:

```bash
sudo -u git forgejo dump -c /etc/forgejo/app.ini --work-path /var/lib/forgejo
```

Forgejo bleibt bei kleineren Versionssprüngen (etwa von 16.0 auf 16.1) verträglich. Vor einem großen Sprung (etwa von 15 auf 16) lohnt ein Blick in die Ankündigung auf <https://forgejo.org/releases/>, weil dort seltene Handgriffe vermerkt sind.

## Fehlersuche

Wenn der Browser **502 Bad Gateway** meldet, erreicht NGINX das Forgejo-Programm nicht. Den Grund nennt das Fehlerprotokoll:

```bash
sudo tail -n 20 /var/log/nginx/error.log
```

- **`connect() to unix:/run/forgejo/forgejo.sock failed (13: Permission denied)`**: Der Benutzer `www-data` ist nicht in der Gruppe `git`, oder NGINX wurde nach `usermod` nur neu geladen statt neu gestartet. `sudo systemctl restart nginx`.
- **`connect() to unix:/run/forgejo/forgejo.sock failed (2: No such file or directory)`**: Forgejo läuft nicht, oder die Ergänzung `RuntimeDirectory` fehlt. Prüfen mit `systemctl status forgejo` und `ls -l /run/forgejo/`.
- **`invalid URL prefix`** beim `nginx -t`: Bei `proxy_pass` auf einen Socket fehlt das `http://unix:` am Anfang oder der Doppelpunkt vor dem Pfad innerhalb der Anwendung.

Weitere Anhaltspunkte liefert das Protokoll von Forgejo selbst:

```bash
sudo journalctl -u forgejo -f
```

- **Falsche Links, Weiterleitungen ins Leere:** `ROOT_URL` in `/etc/forgejo/app.ini` stimmt nicht mit der echten Adresse überein. Nach der Änderung `sudo systemctl restart forgejo`.
- **Der Zugang über SSH fragt nach einem Passwort:** Der öffentliche Schlüssel ist in der Weboberfläche nicht (oder falsch) hinterlegt, oder der Benutzer `git` hat keine Anmelde-Shell (`--shell /bin/bash` in Schritt 3).

## Für dieses Buch

Der Text dieses Buchs und der Aufbau der Kapitel gehören in ein Git-Verzeichnis, das die Redaktion gemeinsam pflegt. **Forgejo** ist dafür die richtige Größe: eine Programmdatei, eine Datenbank, ein Webserver davor – mehr braucht es nicht. Die Weboberfläche bündelt an einer Stelle, was sonst über E-Mails und Dateiordner verstreut wäre: Änderungsvorschläge, Anmerkungen, eine Liste offener Aufgaben.

Zwei Dinge gehören von Anfang an zum Betrieb: **regelmäßige Sicherungen** mit `forgejo dump` und **zeitnahe Aktualisierungen**, sobald eine neue Version erscheint. Weil Forgejo nur aus einer Datei besteht, ist beides schnell erledigt.

Wer die automatische Veröffentlichung des Buchs einrichten will (siehe [Webseiten und Blogs](../entwicklungs-rechner/docs-as-code-webseiten.md)), nutzt dafür **Forgejo Actions**: Bei jeder Änderung am Hauptzweig baut ein kurzer Ablauf das Buch und legt das Ergebnis auf dem Webserver ab.

## Fazit

**Git-Hosting** ist die zentrale Stelle im Netz, an der alle Beteiligten ihre Änderungen an einem Git-Verzeichnis abliefern und abholen. **Forgejo** ist ein freies Programm dafür, getragen von einem gemeinnützigen Verein, entstanden als Abspaltung von Gitea. Es besteht aus einer einzigen ausführbaren Datei, braucht wenig Arbeitsspeicher und bietet dennoch Änderungsvorschläge, Aufgabenlisten, Pakete, ein Wiki und automatische Abläufe.

Die Einrichtung auf Ubuntu 26.04: die geprüfte Programmdatei nach `/usr/local/bin/forgejo` legen, einen eigenen Benutzer `git` anlegen, die Verzeichnisse `/var/lib/forgejo` und `/etc/forgejo` einrichten, eine kurze `app.ini` schreiben und den mitgelieferten systemd-Dienst starten. Die Verbindung zu **NGINX** läuft über einen **Unix-Socket**: In `app.ini` steht `PROTOCOL = http+unix` mit einem Pfad unter `/run/forgejo`, NGINX erreicht ihn über `proxy_pass http://unix:/run/forgejo/forgejo.sock:/;`, und `www-data` wird dafür in die Gruppe `git` aufgenommen. So steht für Forgejo kein Port offen, und der Zugriff hängt allein an Dateirechten.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
