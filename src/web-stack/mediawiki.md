# MediaWiki einrichten

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Das Kapitel [Wissenssystem](./wissensystem.md) grenzt die Auswahl auf zwei Programme ein: **MediaWiki** und **XWiki**. MediaWiki ist die passende Wahl, wenn die Sammlung im Kern ein verlinktes Nachschlagewerk nach dem Vorbild der Wikipedia sein soll: viele Artikel, die aufeinander verweisen und in Kategorien stehen. MediaWiki hat das mit Abstand größte Umfeld an Erweiterungen, Anleitungen und erfahrenen Betreibern und ist seit 2002 ohne Unterbrechung in Entwicklung.

Dieses Kapitel zeigt Schritt für Schritt, wie MediaWiki auf einem Server mit **Ubuntu 26.04 LTS** (siehe [Betriebssystem](../server-einrichten/betriebssystem.md)) eingerichtet wird: von der Laufzeitumgebung über die Datenbank und die Ersteinrichtung bis zum öffentlichen Zugang über einen Webserver. Alle Befehle werden in der Textkonsole des Servers eingegeben. Das vorangestellte `sudo` bedeutet: mit Verwaltungsrechten ausführen.

## Was bei MediaWiki zusammenspielt

MediaWiki ist eine Sammlung von Dateien in der Programmiersprache **PHP**. PHP-Code läuft nicht allein, sondern braucht ein Programm auf dem Server, das ihn ausführt – die Laufzeitumgebung (siehe [Laufzeitumgebung](../server-einrichten/laufzeitumgebung.md)). MediaWiki selbst speichert nichts; es zeigt Seiten an und nimmt Änderungen entgegen, ablegen tut es sie in einer Datenbank.

Vier Teile arbeiten zusammen:

- **PHP-FPM** – die Laufzeitumgebung, die den MediaWiki-Code ausführt. „FPM" steht für „FastCGI Process Manager": PHP läuft als Hintergrunddienst, an den der Webserver die Anfragen weiterreicht (siehe [Laufzeitumgebung](../server-einrichten/laufzeitumgebung.md)).
- **PostgreSQL** – die Datenbank, in der alle Seiten, alle früheren Versionen und alle Einstellungen liegen (siehe [Datenbank](../server-einrichten/datenbank.md)). Fällt sie aus, ist das Wiki leer.
- **Ein Webserver** davor – nimmt die Anfragen aus dem Internet entgegen, verschlüsselt die Verbindung (HTTPS) und reicht die Seiten an PHP-FPM weiter (siehe [Webserver](../server-einrichten/webserver.md)). In diesem Buch ist das **NGINX**.
- **MediaWiki** selbst – der PHP-Code, der aus den Daten in der Datenbank die fertigen Seiten baut.

## Ein Hinweis zur Datenbank vorweg

MediaWiki wird von seinen Entwicklern zusammen mit **MariaDB** oder **MySQL** empfohlen. Die Unterstützung für **PostgreSQL** ist vorhanden und wird gepflegt, gilt aber ausdrücklich als zweitrangig: Sie wird von Freiwilligen betreut, weniger getestet, und einzelne Erweiterungen setzen MySQL voraus. Das Kapitel [Wissenssystem](./wissensystem.md) geht darauf genauer ein.

Dieses Kapitel richtet MediaWiki trotzdem auf **PostgreSQL** ein, weil das ganze Buch auf PostgreSQL aufbaut (unter anderem für die Bedeutungssuche mit `pgvector`, siehe [Datenbank](../server-einrichten/datenbank.md)). Wer nur MediaWiki betreiben und keine PostgreSQL-Erweiterungen braucht, folgt besser der offiziellen Anleitung mit MariaDB; die übrigen Kapitel dieses Buchs ändern sich dadurch kaum.

## Welche Version

MediaWiki bringt etwa alle sechs Monate eine neue Ausgabe heraus. Jede vierte davon – ungefähr alle zwei Jahre – ist eine Ausgabe mit langer Pflegezusage („LTS", englisch für „Long Term Support"). Eine LTS-Ausgabe bekommt drei Jahre lang Sicherheitskorrekturen. Für einen Server, der ruhig laufen soll, ist eine LTS-Ausgabe die richtige Wahl.

Zum Zeitpunkt dieses Kapitels ist **1.46** die neueste reguläre Ausgabe und **1.43** die aktuelle LTS-Ausgabe (gepflegt bis Ende 2027). Die nächste LTS-Ausgabe, **1.47**, wird für Ende 2026 erwartet.

Wichtig ist der Zusammenhang mit PHP: MediaWiki 1.43 läuft nur mit **PHP 8.1 bis 8.3**, nicht mit neueren PHP-Fassungen. Ubuntu 26.04 liefert aber **PHP 8.5** mit (siehe [Laufzeitumgebung](../server-einrichten/laufzeitumgebung.md)). Damit passt 1.43 nicht ohne Weiteres zu diesem Server. Erst **MediaWiki 1.46** unterstützt PHP bis 8.5.

Daraus ergibt sich für dieses Buch: die reguläre Ausgabe **1.46** einsetzen und auf **1.47 LTS** wechseln, sobald sie erschienen ist. Die folgenden Befehle nennen `1.46` an den Stellen, an denen die Versionsnummer vorkommt; für eine spätere Ausgabe wird sie entsprechend ersetzt.

## Voraussetzungen

Bevor es losgeht, sollte Folgendes vorhanden sein:

- Ein Server mit Ubuntu 26.04 und Zugang über SSH.
- **PostgreSQL** ist eingerichtet (siehe [Datenbank](../server-einrichten/datenbank.md)). Die leere Datenbank für MediaWiki wird in Schritt 2 angelegt.
- Eine **Domain**, die auf die IP-Adresse des Servers zeigt, zum Beispiel `wiki.meine-domain.de`. Sie wird für das SSL-Zertifikat und den Webserver gebraucht.
- **NGINX** ist installiert (siehe [Webserver](../server-einrichten/webserver.md)).

## Schritt 1: PHP-FPM und die benötigten Erweiterungen installieren

MediaWiki braucht PHP-FPM und eine Reihe von PHP-Erweiterungen. Jede Erweiterung erledigt eine Teilaufgabe: `php-pgsql` verbindet PHP mit PostgreSQL, `php-mbstring` behandelt Texte mit Umlauten korrekt, `php-gd` verkleinert hochgeladene Bilder, `php-intl` sorgt für die richtige Sortierung fremdsprachiger Namen.

```bash
sudo apt update
sudo apt install php-fpm php-cli \
  php-pgsql php-mbstring php-xml php-curl php-gd php-intl php-zip \
  php-apcu php-openssl

# Zeigt die installierte Version zur Kontrolle an
php -v
```

`php-apcu` ist ein Zwischenspeicher im Arbeitsspeicher; MediaWiki wird damit spürbar schneller. `php-openssl` wird von MediaWiki 1.43 an vorausgesetzt.

Nach der Installation läuft PHP-FPM bereits als Hintergrunddienst. Die Socket-Datei, über die NGINX es später erreicht, liegt bei PHP 8.5 unter `/run/php/php8.5-fpm.sock`. Der genaue Name lässt sich prüfen mit:

```bash
ls /run/php/
```

Zwei Einstellungen in PHP sollten für MediaWiki angehoben werden: die Obergrenze für den Arbeitsspeicher eines einzelnen Aufrufs und die maximale Größe eines Datei-Uploads. Beides steht in der Datei `/etc/php/8.5/fpm/php.ini`:

```bash
sudo nano /etc/php/8.5/fpm/php.ini
```

Dort diese Werte suchen und anpassen:

```
memory_limit = 256M
upload_max_filesize = 100M
post_max_size = 100M
```

Danach PHP-FPM neu starten, damit die Änderungen greifen:

```bash
sudo systemctl restart php8.5-fpm
```

## Schritt 2: Datenbank und Datenbankbenutzer in PostgreSQL anlegen

PostgreSQL legt bei der Installation einen Systembenutzer `postgres` an, der die Datenbank verwaltet. Über ihn werden ein eigener Datenbankbenutzer für das Wiki und eine leere Datenbank angelegt, die ihm gehört. Die Namen (hier `wiki`) sind frei wählbar, müssen aber später in der MediaWiki-Konfiguration genau so eingetragen werden.

```bash
# Datenbankbenutzer anlegen, dabei nach einem Passwort fragen
sudo -u postgres createuser --pwprompt wiki

# Datenbank mit UTF-8-Zeichensatz anlegen, Eigentümer ist "wiki"
sudo -u postgres createdb -E UTF8 -O wiki wiki
```

`UTF8` ist der Zeichensatz, der alle Buchstaben und Zeichen der Welt kennt – wichtig für Umlaute, Anführungszeichen und fremdsprachige Inhalte.

Die Tabellen legt der MediaWiki-Installationsassistent in Schritt 5 selbst an. Ein eigenes Datenbankschema muss nicht von Hand eingerichtet werden.

## Schritt 3: MediaWiki herunterladen

Am einfachsten ist der Weg über das fertige Archiv („Tarball"), das das MediaWiki-Projekt für jede Ausgabe bereitstellt. Es enthält alles, was für den Betrieb nötig ist: den Kern, die Standard-Oberflächen („Skins") und die mitgelieferten PHP-Bausteine.

```bash
cd /var/www
sudo wget https://releases.wikimedia.org/mediawiki/1.46/mediawiki-1.46.0.tar.gz
sudo wget https://releases.wikimedia.org/mediawiki/1.46/mediawiki-1.46.0.tar.gz.sig
```

Vor dem Entpacken lohnt ein Blick auf die Prüfsumme, damit sichergestellt ist, dass die Datei unterwegs nicht verändert wurde. Das MediaWiki-Projekt veröffentlicht dafür digitale Signaturen; die Anleitung dazu steht auf der Download-Seite von MediaWiki. Danach entpacken und den Ordner auf einen kurzen Namen bringen:

```bash
sudo tar -xzf mediawiki-1.46.0.tar.gz
sudo mv mediawiki-1.46.0 mediawiki
sudo rm mediawiki-1.46.0.tar.gz mediawiki-1.46.0.tar.gz.sig
```

MediaWiki liegt jetzt unter `/var/www/mediawiki`.

### Alternative: über Git

Wer die Ausgabe später mit einem einzigen Befehl aktualisieren möchte, kann MediaWiki stattdessen aus dem Quelltext-Verwaltungssystem **Git** holen. Dieser Weg hat mehr Schritte, weil Skins und die PHP-Bausteine getrennt nachgeladen werden müssen:

```bash
cd /var/www
sudo git clone --branch REL1_46 https://gerrit.wikimedia.org/r/mediawiki/core.git mediawiki
cd mediawiki
sudo git submodule update --init --recursive

# Composer holt die benötigten PHP-Bausteine in den Ordner vendor/
sudo composer update --no-dev
```

`REL1_46` ist der Zweig der Ausgabe 1.46; er bekommt weiterhin Sicherheitskorrekturen. Ein späteres `sudo git pull` in diesem Ordner holt sie. Das Werkzeug **Composer** wird im Kapitel [Laufzeitumgebung](../server-einrichten/laufzeitumgebung.md) eingerichtet.

## Schritt 4: Dateirechte setzen

Der Webserver und PHP-FPM laufen unter dem Benutzer `www-data`. Damit MediaWiki hochgeladene Bilder ablegen und den Zwischenspeicher schreiben kann, muss dieser Benutzer Schreibrechte auf die entsprechenden Ordner haben. Der übrige Code sollte ihm nur zum Lesen gehören.

```bash
# Alles gehört www-data, Lese- und Ausführungsrechte für die Gruppe
sudo chown -R www-data:www-data /var/www/mediawiki

# Schreibbar nur die Ordner, die MediaWiki wirklich beschreibt
sudo find /var/www/mediawiki -type d -exec chmod 755 {} \;
sudo find /var/www/mediawiki -type f -exec chmod 644 {} \;
sudo chmod -R 775 /var/www/mediawiki/images
```

Der Ordner `cache/` wird erst nach der Einrichtung gebraucht; MediaWiki legt ihn bei Bedarf an.

## Schritt 5: MediaWiki einrichten

Bei der Einrichtung wird die Datei `LocalSettings.php` erzeugt. Sie enthält alle Einstellungen des Wikis, darunter das Datenbankpasswort. Es gibt zwei Wege dorthin.

### Weg A: über die Textkonsole (empfohlen)

MediaWiki bringt ein Einrichtungsprogramm mit, das ohne Browser auskommt. Es fragt nichts nach, sondern nimmt alle Angaben als Befehlszeile entgegen und schreibt am Ende die fertige `LocalSettings.php`:

```bash
cd /var/www/mediawiki
sudo -u www-data php maintenance/run.php install \
  --dbtype postgres \
  --dbserver localhost \
  --dbname wiki \
  --dbuser wiki \
  --dbpass 'DATENBANKPASSWORT' \
  --installdbuser wiki \
  --installdbpass 'DATENBANKPASSWORT' \
  --server "https://wiki.meine-domain.de" \
  --scriptpath "" \
  --lang de \
  --pass 'ADMINPASSWORT' \
  "Wissenssammlung" "Admin"
```

- `--dbtype postgres` wählt PostgreSQL als Datenbank.
- `--dbname`, `--dbuser`, `--dbpass` sind die Angaben aus Schritt 2.
- `--server` ist die spätere öffentliche Adresse, `--scriptpath ""` legt fest, dass das Wiki direkt unter dieser Adresse liegt (nicht in einem Unterordner).
- Die beiden letzten Werte sind der Name des Wikis und der Name des Administratorkontos; `--pass` ist dessen Passwort.

Das Programm legt die Tabellen in der Datenbank an und schreibt `LocalSettings.php` in den aktuellen Ordner. Danach die Datei absichern, damit nur `www-data` sie lesen kann:

```bash
sudo chown www-data:www-data /var/www/mediawiki/LocalSettings.php
sudo chmod 600 /var/www/mediawiki/LocalSettings.php
```

### Weg B: über den Assistenten im Browser

MediaWiki hat auch einen grafischen Einrichtungsassistenten unter der Adresse `/mw-config/`. Er darf während der Einrichtung **nicht** öffentlich erreichbar sein, denn solange keine `LocalSettings.php` vorhanden ist, kann jeder Besucher das Wiki einrichten.

Der einfachste Schutz ist, die Einrichtung über einen verschlüsselten SSH-Tunnel zu erledigen, genau wie im Kapitel [XWiki einrichten](./xwiki.md) beschrieben. NGINX aus Schritt 6 muss dafür schon stehen, darf aber testweise nur auf `127.0.0.1` hören. Der Assistent führt durch dieselben Fragen wie Weg A und bietet am Ende die fertige `LocalSettings.php` zum Herunterladen an. Diese Datei wird dann auf den Server kopiert:

```bash
# Auf dem eigenen Rechner ausführen
scp ~/Downloads/LocalSettings.php admin@wiki.meine-domain.de:/tmp/LocalSettings.php

# Auf dem Server ausführen
sudo mv /tmp/LocalSettings.php /var/www/mediawiki/LocalSettings.php
sudo chown www-data:www-data /var/www/mediawiki/LocalSettings.php
sudo chmod 600 /var/www/mediawiki/LocalSettings.php
```

Weg A ist kürzer und sicherer, weil dabei zu keinem Zeitpunkt eine ungeschützte Seite im Netz steht.

## Schritt 6: NGINX als Webserver einrichten

Damit das Wiki öffentlich und verschlüsselt erreichbar ist, kommt NGINX davor. Es liefert die festen Dateien (Bilder, Stylesheets) selbst aus und reicht alle PHP-Aufrufe an PHP-FPM weiter (siehe [Webserver](../server-einrichten/webserver.md)).

Zuerst das SSL-Zertifikat besorgen. Wie das mit **Certbot** und **Let's Encrypt** geht, steht ausführlich im Kapitel [Webserver](../server-einrichten/webserver.md); für eine feste Domain genügt:

```bash
sudo certbot certonly --nginx -d wiki.meine-domain.de
```

Dann die Konfigurationsdatei `/etc/nginx/sites-available/mediawiki` anlegen:

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;
    server_name wiki.meine-domain.de;

    ssl_certificate     /etc/letsencrypt/live/wiki.meine-domain.de/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/wiki.meine-domain.de/privkey.pem;

    root /var/www/mediawiki;
    index index.php;

    # Große Datei-Uploads zulassen (muss zu php.ini aus Schritt 1 passen)
    client_max_body_size 100m;

    # Kurze Adressen: /Seitenname wird an index.php übergeben
    location / {
        try_files $uri $uri/ @mediawiki;
    }
    location @mediawiki {
        rewrite ^/(.*)$ /index.php?title=$1&$args;
    }

    # PHP-Dateien an PHP-FPM weiterreichen
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.5-fpm.sock;
    }

    # Interne Ordner und versteckte Dateien sperren
    location ^~ /cache/            { deny all; }
    location ^~ /images/deleted/   { deny all; }
    location ^~ /maintenance/      { deny all; }
    location ~ /\.(ht|git|svn)     { deny all; }
    location = /LocalSettings.php  { deny all; }
}

server {
    listen 80;
    listen [::]:80;
    server_name wiki.meine-domain.de;
    # Alle unverschlüsselten Aufrufe auf HTTPS umleiten
    return 301 https://$host$request_uri;
}
```

Die beiden `location`-Blöcke für die kurzen Adressen sind der Kern: Fragt der Browser eine echte Datei an (ein Bild, ein Stylesheet), liefert NGINX sie direkt. Fragt er einen Seitennamen an (`/Sonnenblume`), gibt es diese Datei nicht – dann greift `@mediawiki` und leitet die Anfrage intern auf `index.php?title=Sonnenblume` um. MediaWiki erzeugt daraufhin die Seite.

Die `deny all`-Blöcke schützen Ordner, die nie direkt aus dem Browser aufgerufen werden sollen – vor allem `LocalSettings.php` mit dem Datenbankpasswort.

Die Datei aktiv schalten, prüfen und übernehmen:

```bash
sudo ln -s /etc/nginx/sites-available/mediawiki /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Damit MediaWiki die kurzen Adressen auch selbst so schreibt, gehören zwei Zeilen in die `LocalSettings.php` (Weg A trägt `$wgScriptPath` bereits ein):

```php
$wgScriptPath = "";
$wgArticlePath = "/$1";
$wgUsePathInfo = false;
```

Mit `$wgArticlePath = "/$1"` liegen die Artikel direkt unter der Domain: `https://wiki.meine-domain.de/Sonnenblume`. Der einzige Haken dabei: Ein Artikel darf nicht so heißen wie eine echte Datei im Wiki-Ordner (etwa „index.php"). Wer das ausschließen will, stellt den Artikeln ein festes Kürzel voran – dann lautet die Zeile `$wgArticlePath = "/wiki/$1";` und im NGINX-Block wird aus `location /` ein `location /wiki/`.

Jetzt ist das Wiki unter `https://wiki.meine-domain.de/` erreichbar.

## Schritt 7: LocalSettings.php ergänzen

Ein paar Einstellungen lohnen sich von Anfang an. Sie werden am Ende der `LocalSettings.php` eingetragen:

```php
# Hochladen von Bildern und Dateien erlauben
$wgEnableUploads = true;

# Zwischenspeicher im Arbeitsspeicher nutzen (aus php-apcu, Schritt 1)
$wgMainCacheType = CACHE_ACCEL;

# Hintergrundaufgaben nicht bei jedem Seitenaufruf abarbeiten,
# sondern über einen eigenen Zeitplan (Schritt 8)
$wgJobRunRate = 0;
```

Nach jeder Änderung an `LocalSettings.php` empfiehlt sich ein Blick auf die Wartungsseite `Special:Version` im Wiki – sie zeigt, ob alles fehlerfrei geladen wird.

## Schritt 8: Wartungsaufgaben über einen Zeitplan

MediaWiki hat Aufgaben, die im Hintergrund laufen: Verweise auf umbenannte Seiten nachziehen, Vorschaubilder erzeugen, E-Mails verschicken. Wird `$wgJobRunRate = 0` gesetzt (Schritt 7), muss ein Zeitplan diese Aufgaben regelmäßig anstoßen. Dafür wird ein Eintrag in der Zeitplan-Tabelle von `www-data` angelegt:

```bash
sudo -u www-data crontab -e
```

Und dort diese Zeile einfügen – sie arbeitet einmal pro Minute anstehende Aufgaben ab:

```
* * * * * /usr/bin/php /var/www/mediawiki/maintenance/run.php runJobs --maxjobs 20 > /dev/null 2>&1
```

Nach einem Versionswechsel (etwa von 1.46 auf 1.47) muss außerdem einmalig die Datenbank an die neue Ausgabe angepasst werden:

```bash
cd /var/www/mediawiki
sudo -u www-data php maintenance/run.php update
```

## Sicherung und Wiederherstellung

Zu einem Wiki-Betrieb gehört eine regelmäßige, automatische Sicherung an einen zweiten Ort. Es gibt zwei Arten von Sicherung, die sich ergänzen.

### Die vollständige Sicherung: pg_dump und der Bilder-Ordner

Der verlässlichste Weg ist ein vollständiger Export der PostgreSQL-Datenbank zusammen mit dem Ordner `images/`. Der Datenbank-Export enthält wirklich alles: Seiten, Versionen, Benutzerkonten, Einstellungen. Der Ordner `images/` enthält die hochgeladenen Dateien, die nicht in der Datenbank liegen.

```bash
# Datenbank exportieren
sudo -u postgres pg_dump wiki > /home/thorsten/backup/wiki-$(date +%F).sql

# Hochgeladene Dateien sichern
sudo tar -czf /home/thorsten/backup/wiki-images-$(date +%F).tar.gz \
  -C /var/www/mediawiki images
```

Beide Dateien gehören anschließend an einen anderen Ort – auf einen zweiten Server, in einen Objektspeicher oder in ein privates Git-Repository. Wie sich ein solcher Export über einen Zeitplan automatisieren lässt, steht im Kapitel [Datenbank](../server-einrichten/datenbank.md).

### Die inhaltliche Sicherung: der XML-Export

MediaWiki kann zusätzlich alle Seiten samt Versionsgeschichte als eine einzige XML-Datei ausgeben. Diese Datei enthält **nur die Texte** – keine Benutzerkonten, keine hochgeladenen Bilder, keine Einstellungen. Sie ist dafür gut lesbar, versionierbar und lässt sich in jede andere MediaWiki-Installation einspielen.

```bash
cd /var/www/mediawiki
sudo -u www-data php maintenance/run.php dumpBackup --full > /home/thorsten/backup/seiten.xml
```

Betreibt man mehrere Wikis auf demselben Server (jedes in einem eigenen Ordner unter `/var/www/`), wird dieser Befehl für jedes Wiki einzeln ausgeführt.

### Wiederherstellen

Eine vollständige Sicherung wird zurückgespielt, indem die Datenbank neu angelegt und der `pg_dump`-Export eingelesen wird:

```bash
sudo -u postgres dropdb wiki
sudo -u postgres createdb -E UTF8 -O wiki wiki
sudo -u postgres psql -d wiki -f /home/thorsten/backup/wiki-2026-09-06.sql
```

Anschließend den Bilder-Ordner aus dem `tar`-Archiv an seinen Platz zurücklegen.

Einen reinen XML-Export spielt man dagegen in eine bereits eingerichtete Installation ein. Danach müssen einige Verzeichnisse neu aufgebaut werden, die sich aus den Seiten ableiten:

```bash
cd /var/www/mediawiki
sudo -u www-data php maintenance/run.php importDump < /home/thorsten/backup/seiten.xml
sudo -u www-data php maintenance/run.php rebuildrecentchanges
sudo -u www-data php maintenance/run.php initSiteStats
sudo -u www-data php maintenance/run.php refreshLinks
```

- `importDump` liest die Seiten und ihre Versionen ein.
- `rebuildrecentchanges` baut die Liste der letzten Änderungen neu auf.
- `initSiteStats` zählt Seiten und Bearbeitungen neu.
- `refreshLinks` erneuert die Verweise zwischen den Seiten und die Kategorien.

## Für dieses Buch

MediaWiki läuft **direkt auf dem Server** (ohne Container, siehe [Containerisierung von Software](../server-einrichten/containerisierung.md)): PHP-FPM, PostgreSQL und NGINX nebeneinander, MediaWiki als Ordner unter `/var/www/`. Empfohlen wird die Einrichtung über die Textkonsole (Weg A in Schritt 5), weil dabei nie eine ungeschützte Einrichtungsseite im Netz steht.

Zwei Dinge gehören von Anfang an dazu: ein Zeitplan-Eintrag für die Hintergrundaufgaben (Schritt 8) und eine regelmäßige, automatische Sicherung – ein `pg_dump` der Datenbank plus der Ordner `images/`, an einen zweiten Ort.

Die schnelle Wortsuche über einen eigenen Suchdienst (**Meilisearch** oder **Typesense**) und die Bedeutungssuche mit **pgvector** sind eigene Bausteine; ihre Einrichtung und die Anbindung an MediaWiki über die passende Erweiterung sind im Kapitel [Datenbank](../server-einrichten/datenbank.md) beschrieben.

## Fazit

MediaWiki besteht aus vier zusammenspielenden Teilen: der Laufzeitumgebung PHP-FPM, der Datenbank PostgreSQL, einem Webserver davor und dem MediaWiki-Code selbst. Die Einrichtung läuft in klaren Schritten: PHP und seine Erweiterungen installieren, in PostgreSQL eine leere Datenbank anlegen, MediaWiki als fertiges Archiv herunterladen, die Dateirechte setzen, mit dem Konsolen-Programm `install` die `LocalSettings.php` erzeugen und zuletzt NGINX als verschlüsselten Zugang davorstellen. Weil Ubuntu 26.04 PHP 8.5 mitbringt, ist die Ausgabe **1.46** die richtige Wahl, bis **1.47 LTS** erscheint. PostgreSQL wird von MediaWiki nur zweitrangig unterstützt – wer die PostgreSQL-Erweiterungen dieses Buchs nicht braucht, kann MediaWiki ebenso gut mit MariaDB betreiben.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
