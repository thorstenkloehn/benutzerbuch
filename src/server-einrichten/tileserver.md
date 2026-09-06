# Tileserver

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Das Kapitel [Datenbank](./datenbank.md) zeigt in einem optionalen Abschnitt, wie sich das freie Kartenmaterial von **OpenStreetMap** mit der Geo-Erweiterung **PostGIS** in PostgreSQL laden lässt. Danach liegen Straßen, Gebäude und Punkte als Tabellen in der Datenbank. Was noch fehlt, ist der Schritt von diesen Tabellen zu einer Landkarte, die ein Besucher im Browser sehen, verschieben und heranzoomen kann. Diesen Schritt erledigt ein **Tileserver**.

Dieses Kapitel erklärt, was ein Tileserver ist und worin sich **Vektorkacheln** und **Rasterkacheln** unterscheiden. Danach stellt es die bekanntesten quelloffenen Tileserver vor und richtet zwei davon auf einem Server mit **Ubuntu 26.04 LTS** (siehe [Betriebssystem](./betriebssystem.md)) ein: **Martin** für Vektorkacheln und die Kombination **renderd + mod_tile** für das klassische Kartenbild als PNG. Alle Befehle werden in der Textkonsole des Servers eingegeben. Das vorangestellte `sudo` bedeutet: mit Verwaltungsrechten ausführen.

## Was ein Tileserver ist

Eine Weltkarte in voller Auflösung wäre ein Bild von vielen Millionen Bildpunkten Kantenlänge – viel zu groß, um es an einen Browser zu schicken. Deshalb wird die Karte in kleine, quadratische Stücke zerschnitten, meist 256 Bildpunkte breit und hoch. Diese Stücke heißen **Kacheln** (englisch „tiles"). Der Browser lädt immer nur die paar Dutzend Kacheln, die gerade auf den Bildschirm passen.

Damit das Heranzoomen funktioniert, gibt es die Karte in mehreren **Zoomstufen**. Stufe 0 ist die ganze Welt in einer einzigen Kachel. Bei jeder weiteren Stufe wird jede Kachel in vier neue geteilt, die Karte also doppelt so genau. Stufe 19 zeigt einzelne Häuser. Jede Kachel hat eine feste Adresse aus drei Zahlen: Zoomstufe, Spalte und Zeile – im Web meist als `.../{z}/{x}/{y}.png` geschrieben. Wenn man in einer Online-Karte den Ausschnitt verschiebt, fordert der Browser einfach die Kacheln mit den neuen Nummern nach. Diese Art der beweglichen Karte nennt man **Slippy Map**.

Ein **Tileserver** ist das Programm, das auf solche Kachelanfragen antwortet. Es nimmt die drei Zahlen entgegen, sucht die passenden Geodaten aus der Datenbank, formt daraus eine Kachel und schickt sie zurück.

## Vektorkacheln oder Rasterkacheln

Es gibt zwei grundsätzlich verschiedene Sorten von Kacheln.

**Rasterkacheln** sind fertige Bilder, meist im Format PNG. Der Server zeichnet die Karte – Farben, Linienstärken, Beschriftungen – vollständig vorab und legt das Ergebnis als Bilddatei ab. Der Browser muss die Kachel nur noch anzeigen. Das funktioniert in jedem Browser und mit einer sehr einfachen Karten-Bibliothek. Der Preis: Das Aussehen der Karte steckt fest im Bild. Für eine andere Farbe, eine andere Schrift oder eine gedrehte Ansicht muss der Server alle Kacheln neu zeichnen. Und für jede Zoomstufe, jeden Bildschirm und jede Auflösung müssen eigene Bilder vorgehalten werden, was viel Speicherplatz kostet.

**Vektorkacheln** enthalten keine Bilder, sondern die nackten Geometrien mitsamt ihren Eigenschaften: „hier verläuft eine Straße vom Typ Wohnstraße mit Namen Hauptstraße", „hier liegt eine Fläche vom Typ Park". Gezeichnet wird erst im Browser, von einer modernen Karten-Bibliothek wie **MapLibre GL**. Das hat mehrere Vorteile: Die Kacheln sind klein, das Kartenbild ist bei jeder Bildschirmauflösung gestochen scharf, die Karte lässt sich stufenlos zoomen und drehen, und das gesamte Aussehen lässt sich ändern, ohne eine einzige Kachel neu zu erzeugen. Der Nachteil: Der Browser muss mehr Arbeit leisten, und man braucht eine aktuelle Karten-Bibliothek mit JavaScript.

| | Rasterkacheln | Vektorkacheln |
| --- | --- | --- |
| Inhalt | fertiges PNG-Bild | Geometrien und Eigenschaften |
| Gezeichnet wird | auf dem Server, vorab | im Browser, sofort |
| Aussehen ändern | alle Kacheln neu zeichnen | nur Stildatei austauschen |
| Speicherbedarf | hoch | niedrig |
| Browser-Anforderung | gering | moderne Karten-Bibliothek |

Für einen neuen Kartendienst sind Vektorkacheln heute der Normalfall. Rasterkacheln bleiben sinnvoll, wenn das vertraute OpenStreetMap-Kartenbild eins zu eins gebraucht wird oder wenn die Karte in sehr alten Browsern oder in Druckerzeugnissen funktionieren muss.

## Quelloffene Tileserver im Überblick

Alle folgenden Programme sind quelloffen und können ihre Kacheln direkt aus einer PostgreSQL-Datenbank mit PostGIS erzeugen.

| Programm | Sprache | Kacheln | Kurzcharakter |
| --- | --- | --- | --- |
| Martin | Rust | Vektor | erkennt Tabellen von selbst, kaum Konfiguration, sehr schnell |
| Tegola | Go | Vektor | Ebenen einzeln in einer Textdatei beschrieben, eingebauter Zwischenspeicher |
| t-rex | Rust | Vektor | ähnlich wie Tegola, erzeugt auch Kachelpakete zum Verteilen |
| renderd + mod_tile | C | Raster | das Gespann hinter dem OpenStreetMap-Kartenbild, rendert mit Mapnik |

- **Martin** ist Teil des MapLibre-Projekts. Es verbindet sich mit der Datenbank und veröffentlicht jede Tabelle mit einer Geometrie-Spalte automatisch als Vektorkachel-Quelle. Für einen einfachen Dienst braucht es keine Konfigurationsdatei.
- **Tegola** verlangt eine Konfigurationsdatei im TOML-Format, in der jede Kartenebene mit ihrer Tabelle und ihrer SQL-Abfrage steht. Das ist mehr Arbeit, gibt aber genaue Kontrolle darüber, was in welcher Zoomstufe erscheint. Ein Zwischenspeicher für die fertigen Kacheln ist eingebaut.
- **t-rex** ähnelt Tegola in Aufbau und Zweck. Es kann Kacheln zusätzlich als eine einzige Paketdatei (**MBTiles**) ausgeben, die sich leicht auf andere Server kopieren lässt.
- **renderd + mod_tile** ist kein Vektor-, sondern ein Raster-Gespann. **Mapnik** ist die Bibliothek, die das eigentliche Kartenbild zeichnet; **renderd** ist ein Hintergrunddienst, der Zeichenaufträge in eine Warteschlange stellt und fertige Kacheln auf der Festplatte ablegt; **mod_tile** ist ein Modul für den Apache-Webserver, das Kacheln ausliefert und fehlende bei renderd nachbestellt. Dieses Gespann steckt hinter der Karte auf openstreetmap.org und ist unter [switch2osm.org](https://switch2osm.org/) ausführlich dokumentiert.

## Empfehlung für dieses Buch

Für einen neuen Kartendienst auf Basis der OpenStreetMap-Daten aus dem Kapitel [Datenbank](./datenbank.md) ist **Martin** die naheliegende Wahl: Die Datenbank mit PostGIS ist ohnehin vorhanden, die Einrichtung ist kurz, und Vektorkacheln lassen sich später ohne Neuberechnung umgestalten.

Wer das exakt gleiche Kartenbild wie openstreetmap.org als PNG braucht – etwa als Hintergrund für eine bestehende Anwendung, die nur Rasterkacheln versteht –, richtet stattdessen **renderd + mod_tile** ein. Beide Wege sind unten beschrieben; nötig ist nur einer.

## Voraussetzungen

- Ein Server mit Ubuntu 26.04 und Zugang über SSH.
- **PostgreSQL mit PostGIS** ist eingerichtet und enthält einen OpenStreetMap-Ausschnitt (siehe [Datenbank](./datenbank.md), Abschnitt „Geodaten aus OpenStreetMap laden"). PostGIS muss mindestens in Version 3.1 vorliegen; die Paketquelle aus dem Datenbank-Kapitel erfüllt das.
- Für den öffentlichen Zugang ein Webserver mit einer Domain und einem SSL-Zertifikat (siehe [Webserver](./webserver.md)).

## Vektorkacheln mit Martin einrichten

### Schritt 1: Martin installieren

Das Projekt bietet ein fertiges Debian-Paket auf seiner Veröffentlichungsseite an. Es enthält nur eine einzige Programmdatei ohne weitere Abhängigkeiten.

```bash
cd /tmp
curl -L -O https://github.com/maplibre/martin/releases/latest/download/debian-x86_64.deb
sudo apt install -y ./debian-x86_64.deb

# Zur Kontrolle
martin --version
```

### Schritt 2: Verbindung zur Datenbank prüfen

Martin bekommt die Zugangsdaten zur Datenbank als **Verbindungszeichenkette** übergeben. Sie hat die Form `postgresql://BENUTZER:PASSWORT@localhost/DATENBANK`. Für die Datenbank `wiki` aus dem Datenbank-Kapitel, in die der OpenStreetMap-Ausschnitt geladen wurde, lautet ein kurzer Test:

```bash
martin postgresql://wiki:DATENBANKPASSWORT@localhost/wiki
```

Beim Start durchsucht Martin die Datenbank und meldet in der Konsole, welche Tabellen es als Kachel-Quellen gefunden hat – bei einem osm2pgsql-Import sind das `planet_osm_point`, `planet_osm_line`, `planet_osm_polygon` und `planet_osm_roads`. Der Dienst hört danach auf `0.0.0.0:3000`. Mit `Strg`+`C` wird er wieder beendet.

### Schritt 3: Martin als Dienst einrichten

Für den Dauerbetrieb soll Martin als Hintergrunddienst laufen, nur lokal erreichbar sein und die Zugangsdaten nicht in der Prozessliste zeigen. Dazu die Zugangsdaten in eine geschützte Umgebungsdatei schreiben:

```bash
sudo install -d -m 750 /etc/martin
echo 'DATABASE_URL=postgresql://wiki:DATENBANKPASSWORT@localhost/wiki' \
  | sudo tee /etc/martin/martin.env
sudo chmod 640 /etc/martin/martin.env
```

Dann die Dienstdatei `/etc/systemd/system/martin.service` anlegen:

```ini
[Unit]
Description=Martin Vektorkachel-Server
After=network.target postgresql.service

[Service]
EnvironmentFile=/etc/martin/martin.env
ExecStart=/usr/bin/martin --listen-addresses 127.0.0.1:3000
Restart=on-failure
DynamicUser=yes

[Install]
WantedBy=multi-user.target
```

`--listen-addresses 127.0.0.1:3000` sorgt dafür, dass Martin nur vom Server selbst erreichbar ist; nach außen kommt später der Webserver davor. Starten und für den automatischen Start nach einem Neustart vormerken:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now martin
sudo systemctl status martin
```

### Schritt 4: Testen

Martin stellt unter `/catalog` ein Verzeichnis aller Kachel-Quellen bereit:

```bash
curl http://127.0.0.1:3000/catalog
```

Für jede Quelle gibt es unter `/QUELLE` eine Beschreibungsdatei (**TileJSON**) und darunter die Kacheln selbst unter `/QUELLE/{z}/{x}/{y}`. Mehrere Tabellen lassen sich zu einer gemeinsamen Quelle verbinden, indem man ihre Namen mit Komma aneinanderreiht:

```bash
curl "http://127.0.0.1:3000/planet_osm_polygon,planet_osm_line,planet_osm_point"
```

### Schritt 5: Den Webserver davorstellen

Der Zugang aus dem Internet läuft über den Webserver, der die Verbindung verschlüsselt und die Anfragen an Martin weiterreicht (siehe [Webserver](./webserver.md)). Ein Block für NGINX unter `/etc/nginx/sites-available/karten`:

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;
    server_name karten.meine-domain.de;

    ssl_certificate     /etc/letsencrypt/live/karten.meine-domain.de/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/karten.meine-domain.de/privkey.pem;

    # Fertige Kacheln einen Tag lang zwischenspeichern
    proxy_cache_path /var/cache/nginx/karten levels=1:2
        keys_zone=karten:10m max_size=2g inactive=1d;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_cache karten;
        proxy_cache_valid 200 1d;
        add_header X-Cache-Status $upstream_cache_status;
    }
}
```

Der Zwischenspeicher (`proxy_cache`) ist bei Martin wichtig: Anders als das Raster-Gespann legt Martin fertige Kacheln nicht selbst auf der Festplatte ab, sondern erzeugt sie bei jeder Anfrage neu. Der Webserver hält die Antworten dann für kurze Zeit vor, sodass beliebte Kacheln nicht bei jedem Aufruf die Datenbank belasten.

### Schritt 6: Die Karte im Browser anzeigen

Zum Anzeigen dient **MapLibre GL JS**, eine quelloffene Karten-Bibliothek. Eine einfache HTML-Seite, die die Umrisse aus den Vektorkacheln zeichnet:

```html
<!doctype html>
<html lang="de">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link href="https://cdnjs.cloudflare.com/ajax/libs/maplibre-gl/5.8.0/maplibre-gl.min.css" rel="stylesheet">
  <script src="https://cdnjs.cloudflare.com/ajax/libs/maplibre-gl/5.8.0/maplibre-gl.min.js"></script>
  <style> body { margin: 0 } #map { height: 100vh } </style>
</head>
<body>
  <div id="map"></div>
  <script>
    const map = new maplibregl.Map({
      container: "map",
      center: [10.24, 53.67],
      zoom: 13,
      style: {
        version: 8,
        sources: {
          osm: {
            type: "vector",
            url: "https://karten.meine-domain.de/planet_osm_polygon,planet_osm_line,planet_osm_point"
          }
        },
        layers: [
          { id: "hintergrund", type: "background", paint: { "background-color": "#f5f5f3" } },
          { id: "flaechen", type: "fill", source: "osm", "source-layer": "planet_osm_polygon",
            paint: { "fill-color": "#e0e0d8" } },
          { id: "linien", type: "line", source: "osm", "source-layer": "planet_osm_line",
            paint: { "line-color": "#b0b0a8" } }
        ]
      }
    });
  </script>
</body>
</html>
```

Das ist bewusst schlicht. Ein vollständiges, hübsches Kartenbild entsteht über eine ausgearbeitete **Stildatei**; fertige quelloffene Stile für OpenStreetMap-Vektorkacheln gibt es beim Projekt OpenMapTiles und bei VersaTiles.

## Rasterkacheln mit renderd und mod_tile einrichten

Dieser Weg erzeugt das vertraute OpenStreetMap-Kartenbild als PNG. Er ist aufwendiger als Martin, weil das Kartenaussehen aus einem umfangreichen Stilprojekt übersetzt werden muss.

### Was zusammenspielt

- **osm2pgsql** liest die OpenStreetMap-Daten in die Datenbank (siehe [Datenbank](./datenbank.md)). Für dieses Kartenbild braucht der Import ein bestimmtes Tabellenschema.
- **openstreetmap-carto** ist das quelloffene Stilprojekt, das festlegt, wie die Karte aussieht – welche Farben, welche Schriften, ab welcher Zoomstufe welche Beschriftung.
- **Mapnik** ist die Bibliothek, die anhand dieses Stils aus den Datenbanktabellen das Bild zeichnet.
- **renderd** ist der Hintergrunddienst, der Zeichenaufträge sammelt, der Reihe nach abarbeitet und fertige Kacheln unter `/var/cache/renderd/tiles` ablegt.
- **mod_tile** ist ein Apache-Modul. Es liefert vorhandene Kacheln sofort aus und meldet fehlende oder veraltete an renderd zur Nachproduktion.

Weil dieses Gespann Apache voraussetzt, läuft es am einfachsten auf einem eigenen Server oder – wenn NGINX bereits Port 80 und 443 belegt – hinter NGINX auf einem anderen Port.

### Schritt 1: Pakete installieren

Ubuntu bringt alle Bestandteile in seinen Paketquellen mit.

```bash
sudo apt update
sudo apt install -y \
  apache2 libapache2-mod-tile renderd \
  mapnik-utils python3-mapnik python3-psycopg2 python3-yaml \
  gdal-bin node-carto \
  osm2pgsql git unzip
```

### Schritt 2: Datenbank und Daten vorbereiten

Für das Kartenbild wird ein eigener Datenbankbenutzer und eine eigene Datenbank `gis` angelegt (die Trennung von der Wiki-Datenbank hält die Karten-Tabellen übersichtlich):

```bash
sudo -u postgres createuser _renderd
sudo -u postgres createdb -E UTF8 -O _renderd gis
sudo -u postgres psql -d gis -c "CREATE EXTENSION IF NOT EXISTS postgis;"
sudo -u postgres psql -d gis -c "CREATE EXTENSION IF NOT EXISTS hstore;"
```

Dann das Stilprojekt holen, denn der Import braucht die darin enthaltenen Regeldateien:

```bash
mkdir -p ~/src && cd ~/src
git clone https://github.com/gravitystorm/openstreetmap-carto.git
cd openstreetmap-carto
```

Jetzt den OpenStreetMap-Ausschnitt importieren – im Beispiel wieder der Ausschnitt aus dem Datenbank-Kapitel. Der Import verwendet die Regeldateien des Stilprojekts, damit die Tabellen genau so heißen und aufgebaut sind, wie der Stil sie erwartet:

```bash
osm2pgsql -d gis --create --slim -G --hstore \
  --tag-transform-script ~/src/openstreetmap-carto/openstreetmap-carto.lua \
  -S ~/src/openstreetmap-carto/openstreetmap-carto.style \
  ~/ahrensburg.pbf
```

Anschließend die vom Stil erwarteten Zusatzdaten (Küstenlinien, Landflächen) und Schriften laden sowie die zusätzlichen Datenbank-Indizes anlegen:

```bash
cd ~/src/openstreetmap-carto
scripts/get-external-data.py
./get-fonts.sh
sudo -u postgres psql -d gis -f indexes.sql
```

### Schritt 3: Den Kartenstil übersetzen

Das Stilprojekt beschreibt die Karte in vielen kurzen Regeldateien. Mapnik erwartet daraus eine einzige große XML-Datei. Das Werkzeug `carto` fasst die Regeln zu dieser XML-Datei zusammen:

```bash
cd ~/src/openstreetmap-carto
carto project.mml > mapnik.xml
```

Die entstandene Datei `mapnik.xml` ist der Bauplan, nach dem renderd später zeichnet.

### Schritt 4: renderd einstellen

Die Einstellungen von renderd stehen in `/etc/renderd.conf`. Wichtig sind der Pfad zur eben erzeugten `mapnik.xml`, die Adresse, unter der die Kacheln erreichbar sein sollen, und der Ablageort:

```bash
sudo nano /etc/renderd.conf
```

Im Abschnitt für die Karte (er heißt oft `[default]`) diese Werte setzen:

```ini
[default]
URI=/tiles/
XML=/home/thorsten/src/openstreetmap-carto/mapnik.xml
HOST=localhost
TILEDIR=/var/cache/renderd/tiles
MAXZOOM=20
```

Danach den Dienst starten:

```bash
sudo systemctl enable --now renderd
sudo systemctl status renderd
```

### Schritt 5: Apache mit mod_tile einstellen

Das Paket `libapache2-mod-tile` legt eine Beispielkonfiguration an, die nur noch aktiviert werden muss. Sie verweist auf denselben Ablageort und dieselbe Adresse wie renderd:

```apache
LoadModule tile_module /usr/lib/apache2/modules/mod_tile.so

<IfModule tile_module>
    ModTileTileDir /var/cache/renderd/tiles
    ModTileRenderdSocketName /run/renderd/renderd.sock
    AddTileConfig /tiles/ default
</IfModule>
```

Aktivieren und Apache neu laden:

```bash
sudo a2enmod tile
sudo a2enconf renderd
sudo systemctl reload apache2
```

### Schritt 6: Testen

Eine einzelne Kachel abrufen – die Zahlen stehen für Zoomstufe 0, Spalte 0, Zeile 0, also die ganze Welt:

```bash
curl -o /tmp/test.png http://localhost/tiles/0/0/0.png
```

Kommt eine PNG-Datei zurück, arbeitet das Gespann. Zum Anzeigen im Browser genügt hier die schlanke Bibliothek **Leaflet**:

```html
<div id="map" style="height: 100vh"></div>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.js"></script>
<script>
  const map = L.map("map").setView([53.67, 10.24], 13);
  L.tileLayer("https://karten.meine-domain.de/tiles/{z}/{x}/{y}.png", {
    maxZoom: 20,
    attribution: "Karte: OpenStreetMap-Mitwirkende"
  }).addTo(map);
</script>
```

### Schritt 7: Kacheln im Voraus erzeugen

Standardmäßig zeichnet renderd eine Kachel erst, wenn sie zum ersten Mal angefragt wird – der erste Besucher eines Gebiets wartet dann kurz. Für kleine, oft genutzte Ausschnitte lohnt es sich, die Kacheln der unteren Zoomstufen vorab zu erzeugen:

```bash
render_list -m default -a -z 0 -Z 14 --num-threads 4
```

`-z 0 -Z 14` legt den Bereich der Zoomstufen fest, `--num-threads` die Anzahl paralleler Zeichenvorgänge. Höhere Zoomstufen für die ganze Welt vorab zu erzeugen ist nicht praktikabel – dort wächst die Zahl der Kacheln ins Unermessliche.

### Daten aktuell halten

OpenStreetMap ändert sich ständig. Um den lokalen Bestand nachzuführen, gibt es das Werkzeug `osm2pgsql-replication`: Es lädt regelmäßig die Änderungen seit dem letzten Stand und schreibt sie in die Datenbank. Betroffene Kacheln werden dabei als „veraltet" vermerkt und beim nächsten Abruf neu gezeichnet. Für einen kleinen Ausschnitt genügt oft auch ein wöchentlicher Neu-Import.

## Für dieses Buch

Der Kartendienst ist eine **optionale Ergänzung** zum Web Stack, kein Pflichtbestandteil. Er baut auf der Datenbank mit PostGIS auf, die im Kapitel [Datenbank](./datenbank.md) eingerichtet wird.

Empfohlen wird **Martin**: Es erzeugt Vektorkacheln direkt aus den vorhandenen OpenStreetMap-Tabellen, läuft als schlanker Hintergrunddienst nur lokal und bekommt über den Webserver aus dem Kapitel [Webserver](./webserver.md) einen verschlüsselten, zwischengespeicherten Zugang. Das Aussehen der Karte steckt in einer austauschbaren Stildatei im Browser und lässt sich jederzeit ändern.

Das Gespann **renderd + mod_tile** ist die richtige Wahl, wenn genau das bekannte OpenStreetMap-Kartenbild als PNG gebraucht wird. Es ist aufwendiger einzurichten, verlangt Apache und benötigt mehr Speicherplatz für die fertigen Kacheln, liefert dafür aber ohne JavaScript in jedem Browser ein vollständiges Kartenbild.

## Fazit

Ein Tileserver macht aus Geodaten in der Datenbank eine bewegliche Karte, indem er sie in kleine, quadratisch nummerierte Kacheln zerlegt. **Rasterkacheln** sind fertige Bilder – einfach anzuzeigen, aber unflexibel und speicherhungrig. **Vektorkacheln** enthalten die Geometrien und werden erst im Browser gezeichnet – klein, scharf und jederzeit umgestaltbar. Für einen neuen Dienst auf Basis der OpenStreetMap-Daten dieses Buchs ist **Martin** die einfachste Lösung: Debian-Paket installieren, Verbindungszeichenkette zur Datenbank angeben, als Dienst hinter den Webserver stellen. Wer das klassische OpenStreetMap-Kartenbild als PNG braucht, richtet **renderd + mod_tile** mit Mapnik und dem Stilprojekt openstreetmap-carto ein.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
