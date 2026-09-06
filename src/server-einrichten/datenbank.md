# Datenbank

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Im Kapitel [Inhalts-Software selbst betreiben](../grundlagen/inhalt-software-verwalten.md) taucht die Datenbank als einer der Bausteine auf: Sie ist der Ort, an dem alle Texte, alle früheren Versionen und alle Einstellungen der Wissenssammlung dauerhaft liegen. MediaWiki selbst speichert nichts – es zeigt Seiten an und nimmt Änderungen entgegen, aber ablegen tut es sie in der Datenbank. Fällt die Datenbank aus, ist das Wiki leer.

Dieses Kapitel erklärt, was eine Datenbank ist, was es bedeutet, dass sie „robust" sein soll, und welche Programme dafür in Frage kommen. Danach zeigt es Schritt für Schritt, wie **PostgreSQL** auf einem Server mit **Ubuntu 26.04 LTS** (siehe [Betriebssystem](./betriebssystem.md)) eingerichtet wird, und ergänzt die Suchdienste **Meilisearch** und **Typesense** sowie die Erweiterungen für die Suche nach Bedeutung. Alle Befehle werden in der Textkonsole des Servers eingegeben. Das vorangestellte `sudo` bedeutet: mit Verwaltungsrechten ausführen.

## Was eine Datenbank ist

Eine Datenbank ist ein gut sortiertes Lager mit Katalog. Man kann Dinge hineingeben, ohne sich zu merken, in welchem Regal sie landen, und man bekommt sie auf eine kurze Anfrage hin sofort wieder heraus – auch dann, wenn Millionen Dinge im Lager liegen. Das Programm, das dieses Lager verwaltet, heißt **Datenbankmanagementsystem**; im Alltag sagt man einfach „die Datenbank".

Die hier betrachteten Datenbanken sind **relational**. Das heißt: Die Daten stehen in Tabellen, ähnlich wie in einer Tabellenkalkulation. Eine Tabelle für die Seiten, eine für die Versionen, eine für die Benutzerkonten. Zwischen den Tabellen gibt es Verweise – eine Version gehört zu einer Seite, eine Änderung zu einem Konto. Abgefragt werden die Tabellen in einer eigenen Sprache namens **SQL** (gesprochen „Es-Ku-El" oder „Sieckwel"). MediaWiki setzt diese Abfragen selbst zusammen; man bekommt SQL im Normalbetrieb nie zu sehen.

Zwei Eigenschaften machen eine Datenbank zu mehr als einer Sammlung von Dateien:

- **Sie hält Ordnung, auch bei gleichzeitigen Zugriffen.** Wenn zwei Personen dieselbe Seite im selben Moment speichern, sorgt die Datenbank dafür, dass kein Mischmasch entsteht, sondern ein klares Vorher und Nachher.
- **Sie lässt nichts halb Fertiges zurück.** Eine Änderung wird entweder ganz gespeichert oder gar nicht. Stürzt der Server mitten im Schreiben ab, ist danach kein zerrissener Datensatz da. Fachleute nennen diese Eigenschaft **ACID**.

## Was „robust" bei einer Datenbank bedeutet

Die Ausgangsfrage lautete: Welche Datenbank ist quelloffen, industrietauglich, sehr belastbar, verwaltet eine Million Texteinträge mit verschiedenen Inhaltstypen – und geht dabei nicht kaputt? Diese Anforderungen lassen sich einzeln beantworten.

- **Eine Million Einträge sind wenig.** Für eine ausgewachsene Datenbank ist das eine kleine bis mittlere Größe. Die deutschsprachige Wikipedia hat ein Vielfaches davon. Entscheidend für die Geschwindigkeit ist nicht die Menge, sondern ob die Datenbank die passenden **Indizes** hat – ein Index ist ein zusätzliches Verzeichnis, das das Suchen abkürzt, so wie das Stichwortverzeichnis am Ende eines Buchs.
- **Verschiedene Inhaltstypen** – Artikel, Bilder, Kategorien, Verweise, Kommentare – bildet man über das **Datenmodell** ab, also über den Zuschnitt der Tabellen. Jede der hier genannten Datenbanken kann beliebig viele solcher Typen verwalten. Die Arbeit steckt im sauberen Entwurf, nicht in der Wahl des Programms.
- **„Geht nicht kaputt"** heißt zweierlei: Die Daten bleiben auch bei einem Absturz unversehrt (das leistet ACID), und das Programm wird über viele Jahre gepflegt, sodass Sicherheitslücken geschlossen werden und der Umstieg auf neue Versionen geregelt ist. Genau das ist mit **Reifegrad** gemeint.
- **„Auf dem Metall"** (englisch „bare metal") bedeutet: Die Datenbank läuft direkt auf dem Server, ohne Zwischenschicht wie einen Container (siehe [Containerisierung von Software](./containerisierung.md)). Das ist der Weg, den dieses Buch beschreibt – wenige bewegliche Teile, volle Geschwindigkeit.
- **Benchmarks** – vergleichende Messungen unter gleichen Bedingungen – gibt es für Datenbanken reichlich. Sie sind mit Vorsicht zu lesen; dazu weiter unten mehr.

## Datenbanken und Suchdienste im Überblick

Die genannten Programme lösen unterschiedliche Aufgaben. Es hilft, sie in drei Gruppen zu ordnen.

| Programm | Gruppe | Aufgabe |
| --- | --- | --- |
| PostgreSQL | relationale Datenbank | speichert alle Inhalte dauerhaft und geordnet |
| MariaDB | relationale Datenbank | dasselbe; MediaWikis Standard-Datenbank |
| Meilisearch | Suchdienst | schnelle Wortsuche über die Texte, verzeiht Tippfehler |
| Typesense | Suchdienst | dasselbe wie Meilisearch, anderer Anbieter |
| pgvector, pg_trgm | PostgreSQL-Erweiterungen | Suche nach Bedeutung bzw. nach ähnlicher Schreibweise |
| Qdrant | Vektordatenbank | eigenständige Datenbank nur für die Suche nach Bedeutung |

**Die relationale Datenbank ist das Fundament.** Sie muss vorhanden sein, sonst läuft MediaWiki nicht. Alles Übrige ist Ergänzung, die die Suche verbessert.

### PostgreSQL oder MariaDB

Beide sind quelloffen, seit Jahrzehnten im Einsatz, in großen Unternehmen bewährt und werden auf absehbare Zeit gepflegt. Beide verkraften die beschriebene Wissenssammlung mühelos.

- **MariaDB** ist eine Abspaltung von MySQL und die Datenbank, die MediaWiki von Haus aus erwartet. Die meisten MediaWiki-Anleitungen gehen von ihr aus. Sie gilt als einfach einzurichten.
- **PostgreSQL** ist besonders streng bei der Datenkorrektheit und lässt sich um zusätzliche Fähigkeiten erweitern. Für dieses Buch ist das der Ausschlag: Die Suche nach Bedeutung (**pgvector**) und die Ähnlichkeitssuche (**pg_trgm**) sind solche Erweiterungen. Mit PostgreSQL wird aus der ohnehin nötigen Datenbank zugleich eine Vektordatenbank – ein zweites Programm dafür entfällt.

Dieses Kapitel richtet **PostgreSQL** ein. Wer MariaDB bevorzugt, folgt der offiziellen MediaWiki-Anleitung; die übrigen Kapitel bleiben davon unberührt.

### Suchdienst: Meilisearch oder Typesense

Die eingebaute Suche von MediaWiki wird bei vielen Seiten langsam und ungenau. Ein eigener Suchdienst führt ein separates Wörterverzeichnis und liefert Treffer schon während des Tippens. **Meilisearch** und **Typesense** sind zwei sehr ähnliche Programme dieser Art: schlank, schnell, quelloffen, für moderne Suchfelder auf Webseiten gemacht. Meilisearch ist etwas weiter verbreitet und einfacher einzurichten; Typesense ist beim Umgang mit sehr großen Datenmengen etwas flexibler. Für die Wissenssammlung dieses Buchs genügt eines von beiden.

### Suche nach Bedeutung: pgvector oder Qdrant

Manchmal steht das gesuchte Wort nicht im Text – man sucht „Auto reparieren", die Seite heißt „Wagen instand setzen". Dafür wird jeder Text in eine lange Zahlenreihe umgerechnet, die seine Bedeutung beschreibt (fachlich: ein **Vektor** oder **Embedding**). Ähnliche Texte ergeben ähnliche Zahlenreihen. Diese Zahlenreihen müssen gespeichert und schnell verglichen werden.

- **pgvector** ist eine Erweiterung von PostgreSQL. Die Zahlenreihen liegen dann in derselben Datenbank wie die Texte. Kein zusätzliches Programm, keine zusätzliche Sicherung.
- **Qdrant** ist eine eigenständige Vektordatenbank. Sie ist spezialisiert und bei sehr vielen Vektoren schneller, bringt aber ein weiteres Programm ins Spiel, das eingerichtet, überwacht und gesichert werden will.

Für den beschriebenen Umfang reicht **pgvector**. Qdrant lohnt sich erst, wenn die Bedeutungssuche zum Kern einer eigenen Anwendung wird.

## PostgreSQL auf Ubuntu einrichten

Ubuntu 26.04 bringt PostgreSQL 18 bereits in seinen eigenen Paketquellen mit. Die folgende Anleitung fügt zusätzlich die **offizielle Paketquelle des PostgreSQL-Projekts** hinzu. Das hat zwei Gründe: Man bekommt die neuesten Fehlerkorrekturen früher, und die Erweiterungspakete wie `postgresql-18-pgvector` liegen dort verlässlich bereit.

### Schritt 1: Die Paketquelle eintragen

```bash
sudo apt install -y curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail \
  https://www.postgresql.org/media/keys/ACCC4CF8.asc

. /etc/os-release
sudo sh -c "echo 'deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $VERSION_CODENAME-pgdg main' > /etc/apt/sources.list.d/pgdg.list"

sudo apt update
```

Die erste Zeile lädt einen digitalen Schlüssel herunter, mit dem Ubuntu prüft, dass die Pakete echt sind. Die mittlere Zeile schreibt die Adresse der Paketquelle in eine Datei; `$VERSION_CODENAME` setzt dabei automatisch den Namen der Ubuntu-Ausgabe ein.

### Schritt 2: PostgreSQL installieren

```bash
sudo apt install -y postgresql-18
```

Nach der Installation läuft PostgreSQL bereits als Hintergrunddienst und startet nach jedem Server-Neustart von selbst wieder. Zur Kontrolle:

```bash
sudo systemctl status postgresql
psql --version
```

### Schritt 3: Benutzer und Datenbank anlegen

PostgreSQL legt bei der Installation einen Systembenutzer `postgres` an, der die Datenbank verwaltet. Über ihn wird ein eigener Datenbankbenutzer für das Wiki angelegt und eine leere Datenbank, die ihm gehört. Die Namen (`wiki`) sind frei wählbar; sie müssen später in der MediaWiki-Konfiguration genau so eingetragen werden.

```bash
# Datenbankbenutzer anlegen, dabei nach einem Passwort fragen
sudo -u postgres createuser --pwprompt wiki

# Datenbank mit UTF-8-Zeichensatz anlegen, Eigentümer ist "wiki"
sudo -u postgres createdb -E UTF8 -O wiki wiki
```

`UTF8` ist der Zeichensatz, der alle Buchstaben und Zeichen der Welt kennt – wichtig für Umlaute, Anführungszeichen und fremdsprachige Inhalte.

### Schritt 4: Erweiterungen einschalten

Erweiterungen werden als Paket installiert und danach in der jeweiligen Datenbank mit einem SQL-Befehl aktiviert.

```bash
# Pakete installieren
sudo apt install -y postgresql-18-pgvector

# Erweiterungen in der Datenbank "wiki" einschalten
sudo -u postgres psql -d wiki -c "CREATE EXTENSION IF NOT EXISTS vector;"
sudo -u postgres psql -d wiki -c "CREATE EXTENSION IF NOT EXISTS pg_trgm;"
sudo -u postgres psql -d wiki -c "CREATE EXTENSION IF NOT EXISTS hstore;"
```

- **vector** (aus dem Paket `pgvector`) speichert und vergleicht die Zahlenreihen für die Bedeutungssuche.
- **pg_trgm** findet ähnlich geschriebene Wörter. Es zerlegt jedes Wort in Dreier-Gruppen von Buchstaben und vergleicht diese – so wird „Müller" auch bei der Eingabe „Mueller" gefunden.
- **hstore** erlaubt es, zu einem Datensatz beliebige zusätzliche Schlüssel-Wert-Paare zu speichern, ohne die Tabelle umzubauen. Einige MediaWiki-Erweiterungen und Zusatzwerkzeuge setzen das voraus.

### Schritt 5: Zugriff absichern

Standardmäßig nimmt PostgreSQL nur Verbindungen vom selben Rechner an. Das ist die sichere Voreinstellung und für dieses Buch richtig: MediaWiki läuft auf demselben Server wie die Datenbank. Die Datenbank sollte **nicht** aus dem Internet erreichbar sein. Ob das eingehalten ist, zeigt:

```bash
sudo ss -tlnp | grep 5432
```

Erscheint dort `127.0.0.1:5432` (und gegebenenfalls `[::1]:5432`), hört die Datenbank nur lokal. Erscheint `0.0.0.0:5432`, ist sie nach außen offen – dann in der Datei `/etc/postgresql/18/main/postgresql.conf` die Zeile `listen_addresses` auf `'localhost'` setzen und `sudo systemctl restart postgresql` ausführen.

## Optionales Beispiel: Geodaten aus OpenStreetMap laden

Dieser Abschnitt ist kein Pflichtteil des Wiki-Aufbaus. Er zeigt an einem konkreten Fall, wie sich PostgreSQL mit der **Geo-Erweiterung PostGIS** um fremde Datenbestände erweitern lässt – hier das freie Kartenmaterial von OpenStreetMap für einen kleinen Ausschnitt (das Beispiel nimmt die Stadt Ahrensburg).

Zuerst die Geo-Erweiterung installieren und in der Datenbank einschalten:

```bash
sudo apt install -y postgis postgresql-18-postgis-3
sudo -u postgres psql -d wiki -c "CREATE EXTENSION IF NOT EXISTS postgis;"
```

**PostGIS** lehrt PostgreSQL den Umgang mit Orten, Linien und Flächen: Entfernungen berechnen, prüfen, ob ein Punkt in einem Gebiet liegt, Umkreissuchen. Ohne PostGIS kennt die Datenbank nur Zahlen und Text, keine Landkarte.

Dann die Kartendaten herunterladen, auf den gewünschten Ausschnitt zuschneiden und in die Datenbank einlesen:

```bash
cd $HOME
wget https://download.geofabrik.de/europe/germany/schleswig-holstein-latest.osm.pbf
sudo apt install -y osmosis osm2pgsql

# Auf einen rechteckigen Ausschnitt zuschneiden (Längen- und Breitengrade)
osmosis --read-pbf file=schleswig-holstein-latest.osm.pbf \
  --bounding-box left=10.1141 right=10.3716 top=53.7136 bottom=53.6249 \
  --write-pbf file=ahrensburg.pbf

# Den Ausschnitt in die Datenbank "wiki" schreiben
osm2pgsql -d wiki -H localhost -U wiki --create -G --hstore -W ahrensburg.pbf
```

- **`schleswig-holstein-latest.osm.pbf`** ist die komprimierte Kartendatei für das gesamte Bundesland, bezogen vom Anbieter Geofabrik.
- **`osmosis`** schneidet daraus mit `--bounding-box` ein Rechteck heraus; die vier Werte sind die Grenzen in Längen- und Breitengraden.
- **`osm2pgsql`** liest den Ausschnitt in die Datenbank. `--create` legt die nötigen Tabellen an, `-G` erzeugt einfache Geometrien, `--hstore` übernimmt alle zusätzlichen Angaben (Straßennamen, Öffnungszeiten und so weiter) in ein hstore-Feld, `-W` fragt nach dem Datenbankpasswort.

Danach stehen die Straßen, Gebäude und Punkte des Ausschnitts als Tabellen in der Datenbank und lassen sich mit SQL abfragen.

## Suchdienst einrichten

### Meilisearch

Meilisearch wird über die Paketquelle des Anbieters installiert:

```bash
echo "deb [trusted=yes] https://apt.fury.io/meilisearch/ /" \
  | sudo tee /etc/apt/sources.list.d/meilisearch.list
sudo apt update
sudo apt install -y meilisearch-http
```

Meilisearch braucht einen **Hauptschlüssel** (englisch „master key"), ohne den jeder mit Zugang zum Server die Suchdaten lesen und ändern könnte. Einen zufälligen Schlüssel erzeugen und zusammen mit den übrigen Einstellungen in einer Konfigurationsdatei ablegen:

```bash
openssl rand -base64 32
```

Datei `/etc/meilisearch.toml` anlegen (der Wert bei `master_key` ist der eben erzeugte Schlüssel):

```toml
env = "production"
master_key = "HIER_DEN_ERZEUGTEN_SCHLUESSEL_EINSETZEN"
http_addr = "127.0.0.1:7700"
db_path = "/var/lib/meilisearch/data"
```

Die Adresse `127.0.0.1` sorgt dafür, dass Meilisearch nur vom selben Server aus erreichbar ist. Anschließend als Hintergrunddienst einrichten. Dazu die Datei `/etc/systemd/system/meilisearch.service` anlegen:

```ini
[Unit]
Description=Meilisearch
After=network.target

[Service]
ExecStart=/usr/bin/meilisearch --config-file-path /etc/meilisearch.toml
Restart=on-failure
DynamicUser=yes
StateDirectory=meilisearch

[Install]
WantedBy=multi-user.target
```

Starten und für den automatischen Start nach einem Neustart vormerken:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now meilisearch
```

### Typesense

Typesense wird als fertiges Paket von der Seite des Anbieters heruntergeladen und installiert:

```bash
curl -O https://dl.typesense.org/releases/29.0/typesense-server-29.0-amd64.deb
sudo apt install -y ./typesense-server-29.0-amd64.deb
```

Bei der Installation fragt das Paket nach einem **API-Schlüssel** (dieselbe Rolle wie Meilisearchs Hauptschlüssel) und legt selbst einen Hintergrunddienst an. Die Einstellungen stehen danach in `/etc/typesense/typesense-server.ini`. Dort sollte `api-address = 127.0.0.1` eingetragen sein, damit der Dienst nur lokal erreichbar ist. Nach einer Änderung:

```bash
sudo systemctl restart typesense-server
sudo systemctl enable typesense-server
```

## Zur Geschwindigkeit: Was Benchmarks sagen – und was nicht

Für PostgreSQL und MariaDB gibt es zahllose Vergleichsmessungen, und je nach Testaufbau gewinnt mal die eine, mal die andere. Für den Alltag einer Wissenssammlung ist das ohne Bedeutung. Drei Punkte zählen mehr als jede Messung:

- **Die richtigen Indizes.** Eine Abfrage ohne passenden Index durchsucht die ganze Tabelle; mit Index springt sie sofort zum Ziel. Der Unterschied ist oft der Faktor Tausend. MediaWiki bringt seine Indizes mit; bei eigenen Zusatzprogrammen muss man selbst daran denken.
- **Nur laden, was gebraucht wird.** Ein Programm, das immer alle Einträge holt und dann wegwirft, was es nicht braucht, wird mit jeder Zeile langsamer. Gute Programme holen seitenweise.
- **Ein Zwischenspeicher.** Wird eine fertige Antwort für kurze Zeit vorgehalten (etwa mit Redis oder direkt im Webserver, siehe [Webserver](./webserver.md)), spielt die Geschwindigkeit der Datenbank für diese Anfrage keine Rolle mehr.

Kurz: Die Wahl zwischen zwei ausgereiften Datenbanken entscheidet über die Geschwindigkeit weit weniger als der Umgang mit ihnen.

## Für dieses Buch

Die Wissenssammlung braucht genau eine relationale Datenbank. Dieses Buch nimmt **PostgreSQL**, weil sich damit die Bedeutungssuche (`pgvector`) und die Ähnlichkeitssuche (`pg_trgm`) ohne ein weiteres Programm abdecken lassen. Wer nur MediaWiki betreiben und der offiziellen Anleitung folgen will, kann ebenso gut **MariaDB** einsetzen.

Als Suchdienst genügt **Meilisearch** oder **Typesense** – eines von beiden, nicht beide. Für die Bedeutungssuche bleibt es bei **pgvector**; **Qdrant** ist für diesen Umfang zu viel.

Die Datenbank läuft direkt auf dem Server und ist nur von dort erreichbar, nicht aus dem Internet. Gesichert werden muss vor allem ihr Inhalt: ein regelmäßiger, automatischer Export (`pg_dump`) an einen zweiten Ort gehört zu jedem Datenbankbetrieb dazu.

## Fazit

Eine Datenbank legt die Inhalte der Wissenssammlung geordnet ab und gibt sie schnell wieder heraus, ohne bei gleichzeitigen Zugriffen oder einem Absturz Schaden zu nehmen. **PostgreSQL** und **MariaDB** sind beide quelloffen, ausgereift und für eine Million Einträge weit mehr als ausreichend; dieses Buch wählt PostgreSQL wegen seiner Erweiterungen. Auf Ubuntu 26.04 wird PostgreSQL 18 über die offizielle Paketquelle eingerichtet, danach werden Benutzer, Datenbank und die Erweiterungen `vector`, `pg_trgm` und `hstore` angelegt. **Meilisearch** oder **Typesense** ergänzen die schnelle Wortsuche, **pgvector** die Suche nach Bedeutung. Ob die Sammlung schnell bleibt, entscheidet nicht die Wahl der Datenbank, sondern die richtigen Indizes, sparsames Laden und ein Zwischenspeicher.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
</content>
</invoke>
