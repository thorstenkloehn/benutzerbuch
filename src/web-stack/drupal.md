# Drupal einrichten

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Das Kapitel [Content-Management-System](./cms.md) grenzt die Auswahl für einen redaktionellen Webauftritt mit der Festlegung auf **PostgreSQL** auf zwei Systeme ein und empfiehlt für den Regelfall **Drupal**: PostgreSQL wird offiziell und gleichwertig unterstützt, strukturierte Inhaltstypen sind eingebaut, und das System ist seit 2001 ausgereift. Drupal läuft zudem auf derselben PHP-Laufzeitumgebung, die [MediaWiki](./mediawiki.md) für die Wissenssammlung ohnehin braucht.

Dieses Kapitel zeigt Schritt für Schritt, wie Drupal auf einem Server mit **Ubuntu 26.04 LTS** (siehe [Betriebssystem](../server-einrichten/betriebssystem.md)) eingerichtet wird: von der Laufzeitumgebung über die Datenbank und die Installation ohne Browser bis zum öffentlichen, verschlüsselten Zugang über einen Webserver. Alle Befehle werden in der Textkonsole des Servers eingegeben. Das vorangestellte `sudo` bedeutet: mit Verwaltungsrechten ausführen.

Die Beispiele gehen davon aus, dass neben der Wissenssammlung ein zweiter, redaktioneller Auftritt entstehen soll – etwa ein Portal oder ein Nachrichtenbereich – und dieser unter einer eigenen Unterdomain wie `portal.meine-domain.de` erreichbar ist.

## Was bei Drupal zusammenspielt

Drupal ist eine Sammlung von Dateien in der Programmiersprache **PHP**. PHP-Code läuft nicht allein, sondern braucht ein Programm auf dem Server, das ihn ausführt – die Laufzeitumgebung (siehe [Laufzeitumgebung](../server-einrichten/laufzeitumgebung.md)). Drupal selbst speichert nichts dauerhaft; es baut Seiten zusammen und legt alle Inhalte in einer Datenbank ab.

Fünf Teile arbeiten zusammen:

- **PHP-FPM** – die Laufzeitumgebung, die den Drupal-Code ausführt. „FPM" steht für „FastCGI Process Manager": PHP läuft als Hintergrunddienst, an den der Webserver die Anfragen weiterreicht (siehe [Laufzeitumgebung](../server-einrichten/laufzeitumgebung.md)).
- **Composer** – das Paketwerkzeug von PHP. Drupal wird nicht als fertiges Archiv heruntergeladen, sondern von Composer aus vielen Einzelbausteinen zusammengestellt. Composer hält diese Bausteine später auch aktuell.
- **PostgreSQL** – die Datenbank, in der alle Seiten, alle früheren Fassungen, alle Benutzerkonten und die gesamte Konfiguration liegen (siehe [Datenbank](../server-einrichten/datenbank.md)). Fällt sie aus, ist die Seite leer.
- **Ein Webserver** davor – nimmt die Anfragen aus dem Internet entgegen, verschlüsselt die Verbindung (HTTPS) und reicht die Seiten an PHP-FPM weiter (siehe [Webserver](../server-einrichten/webserver.md)). In diesem Buch ist das **NGINX**.
- **Drush** – die Kommandozeilen-Shell für Drupal. Damit wird Drupal ohne Browser eingerichtet, aktualisiert und gesichert. „Drush" ist die Kurzform von „Drupal Shell".

## Welche Version

Drupal bringt etwa jedes halbe Jahr eine neue Nebenausgabe heraus und alle zwei Jahre eine neue Hauptausgabe. Die aktuelle Hauptausgabe ist **Drupal 11** (erschienen im August 2024); zum Zeitpunkt dieses Kapitels ist **11.4** die neueste Nebenausgabe.

Wie bei MediaWiki gibt es einen Zusammenhang mit der PHP-Fassung, den man kennen sollte:

- **Drupal 11** verlangt mindestens PHP 8.3; als beste Wahl gilt PHP 8.4. Die Fassung **11.4** läuft auch mit dem **PHP 8.5**, das Ubuntu 26.04 mitbringt (siehe [Laufzeitumgebung](../server-einrichten/laufzeitumgebung.md)) – alle Bestandteile von Drupal 11.4 sind für PHP 8.5 vorbereitet.
- **Drupal 12** wird PHP 8.5 und **PostgreSQL 18** zur Pflicht machen – genau die Fassungen, die auf dem hier beschriebenen Server ohnehin laufen. Drupal 12 ist ab Mitte September 2026 als Testfassung verfügbar; die fertige Ausgabe wird zum Jahresende 2026 erwartet. Drupal 11 wird danach noch bis mindestens Mitte 2028 mit Sicherheitskorrekturen versorgt.

Daraus ergibt sich für dieses Buch: mit der aktuellen Ausgabe **Drupal 11** starten und auf **Drupal 12** wechseln, sobald diese fertig erschienen ist. Der Umstieg ist dann klein, weil Server, PHP und Datenbank die Anforderungen von Drupal 12 bereits erfüllen. Wer strikt auf der offiziell empfohlenen PHP-Fassung bleiben möchte, richtet zusätzlich PHP 8.4 über die Paketquelle `packages.sury.org` ein und lässt Drupal über einen eigenen PHP-FPM-Pool auf dieser Fassung laufen (Schritt 6).

Bei der Datenbank gibt es keinen Konflikt: Drupal 11 verlangt **PostgreSQL 16 oder neuer**, und das Kapitel [Datenbank](../server-einrichten/datenbank.md) richtet ohnehin PostgreSQL 18 ein.

## Voraussetzungen

Bevor es losgeht, sollte Folgendes vorhanden sein:

- Ein Server mit Ubuntu 26.04 und Zugang über SSH, mit mindestens **1 GB freiem Arbeitsspeicher** für Drupal.
- **PostgreSQL** ist eingerichtet (siehe [Datenbank](../server-einrichten/datenbank.md)). Die leere Datenbank für Drupal wird in Schritt 3 angelegt.
- **Composer** ist eingerichtet (siehe [Laufzeitumgebung](../server-einrichten/laufzeitumgebung.md), Abschnitt „Composer").
- Eine **Domain oder Unterdomain**, die auf die IP-Adresse des Servers zeigt, zum Beispiel `portal.meine-domain.de`. Sie wird für das SSL-Zertifikat und den Webserver gebraucht.
- **NGINX** ist installiert (siehe [Webserver](../server-einrichten/webserver.md)).

## Schritt 1: PHP-FPM und die benötigten Erweiterungen installieren

Drupal braucht PHP-FPM und eine Reihe von PHP-Erweiterungen. Jede Erweiterung erledigt eine Teilaufgabe: `php-pgsql` verbindet PHP mit PostgreSQL, `php-mbstring` behandelt Texte mit Umlauten korrekt, `php-gd` verkleinert hochgeladene Bilder, `php-intl` sorgt für die richtige Sortierung fremdsprachiger Namen, `php-xml` und `php-curl` werden für den Datenaustausch gebraucht.

```bash
sudo apt update
sudo apt install php-fpm php-cli \
  php-pgsql php-mbstring php-xml php-curl php-gd php-intl php-zip \
  php-apcu php-opcache \
  git unzip

# Zeigt die installierte Version zur Kontrolle an
php -v
```

`php-apcu` und `php-opcache` sind Zwischenspeicher im Arbeitsspeicher; Drupal wird damit spürbar schneller. `git` und `unzip` braucht Composer, um die Bausteine herunterzuladen.

Zwei Einstellungen in PHP sollten für Drupal angehoben werden: die Obergrenze für den Arbeitsspeicher eines einzelnen Aufrufs und die maximale Größe eines Datei-Uploads. Beides steht in der Datei `/etc/php/8.5/fpm/php.ini`:

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

PostgreSQL legt bei der Installation einen Systembenutzer `postgres` an, der die Datenbank verwaltet. Über ihn werden ein eigener Datenbankbenutzer für Drupal und eine leere Datenbank angelegt, die ihm gehört. Die Namen (hier `drupal`) sind frei wählbar, müssen aber später bei der Installation genau so angegeben werden.

```bash
# Datenbankbenutzer anlegen, dabei nach einem Passwort fragen
sudo -u postgres createuser --pwprompt drupal

# Datenbank mit UTF-8-Zeichensatz anlegen, Eigentümer ist "drupal"
sudo -u postgres createdb -E UTF8 -O drupal drupal
```

`UTF8` ist der Zeichensatz, der alle Buchstaben und Zeichen der Welt kennt – wichtig für Umlaute, Anführungszeichen und fremdsprachige Inhalte.

Drupal setzt bei PostgreSQL zwingend die Erweiterung **`pg_trgm`** voraus. Sie findet ähnlich geschriebene Wörter, indem sie jedes Wort in Dreier-Gruppen von Buchstaben zerlegt und diese vergleicht – dieselbe Erweiterung, die das Kapitel [Datenbank](../server-einrichten/datenbank.md) für die Ähnlichkeitssuche ohnehin nutzt. Der Drupal-Installationsassistent würde versuchen, sie selbst einzuschalten; sicherer ist es, das vorab als Verwalter zu erledigen:

```bash
sudo -u postgres psql -d drupal -c "CREATE EXTENSION IF NOT EXISTS pg_trgm;"
```

Die Tabellen legt die Installation in Schritt 5 selbst an; ein eigenes Datenbankschema muss nicht von Hand eingerichtet werden.

## Schritt 3: Drupal mit Composer zusammenstellen

Drupal wird aus dem offiziellen Projektgerüst `drupal/recommended-project` erzeugt. Es bringt Drupal Core und die sinnvollen Standard-Abhängigkeiten mit. Damit Composer nicht mit Verwaltungsrechten läuft (davon rät Composer ausdrücklich ab), gehört das Projektverzeichnis zunächst dem angemeldeten Benutzer:

```bash
sudo mkdir -p /var/www/drupal
sudo chown "$USER":"$USER" /var/www/drupal

# Projekt in das vorhandene, leere Verzeichnis erzeugen
composer create-project drupal/recommended-project /var/www/drupal

cd /var/www/drupal

# Drush als Kommandozeilen-Shell ergänzen
composer require drush/drush
```

Nach diesem Schritt liegt der eigentliche Webinhalt im Unterordner `web/` – dieser Ordner wird später das Wurzelverzeichnis des Webservers. Die Bausteine liegen in `vendor/`, der Drush-Befehl unter `vendor/bin/drush`.

## Schritt 4: Dateirechte setzen

Der Webserver und PHP-FPM laufen unter dem Benutzer `www-data`. Dieser Benutzer soll den Drupal-Code lesen dürfen, aber nur zwei Stellen beschreiben: den Ordner für hochgeladene Dateien und – während der Installation – den Ordner, in den die zentrale Einstellungsdatei geschrieben wird.

```bash
# Alles gehört www-data, mit Lese- und Ausführungsrechten für die Gruppe
sudo chown -R www-data:www-data /var/www/drupal
sudo find /var/www/drupal -type d -exec chmod 755 {} \;
sudo find /var/www/drupal -type f -exec chmod 644 {} \;

# Der Ordner für hochgeladene Dateien muss beschreibbar sein
sudo mkdir -p /var/www/drupal/web/sites/default/files
sudo chmod -R 775 /var/www/drupal/web/sites/default/files

# Verzeichnis für die exportierte Konfiguration (außerhalb von web/)
sudo mkdir -p /var/www/drupal/config/sync
sudo chown -R www-data:www-data /var/www/drupal/config
```

## Schritt 5: Drupal ohne Browser installieren

Anders als bei [XWiki](./xwiki.md) oder MediaWikis grafischem Assistenten braucht Drupal für die Ersteinrichtung keinen Zugang über den Browser und keinen SSH-Tunnel. Der Befehl `drush site:install` erledigt die komplette Einrichtung nichtinteraktiv auf der Kommandozeile: Er legt die Tabellen an, schreibt die Einstellungsdatei und richtet das Administratorkonto ein.

Damit Drush die Einstellungsdatei `settings.php` schreiben kann, muss ihr Ordner kurz beschreibbar sein:

```bash
sudo chmod 775 /var/www/drupal/web/sites/default

cd /var/www/drupal
sudo -u www-data php vendor/bin/drush site:install standard \
  --db-url='pgsql://drupal:DATENBANKPASSWORT@localhost/drupal' \
  --site-name="Portal Ahrensburg" \
  --account-name=admin \
  --account-pass='ADMINPASSWORT' \
  --yes
```

- `standard` ist die mitgelieferte Grundausstattung mit den üblichen Funktionen.
- `--db-url` enthält alle Angaben aus Schritt 2: Benutzer, Passwort, Server (`localhost`) und Datenbankname. `pgsql` wählt PostgreSQL.
- `--account-name` und `--account-pass` sind Name und Passwort des ersten Administratorkontos.

Danach die Rechte an der Einstellungsdatei wieder einschränken, sodass `www-data` sie nur noch lesen kann:

```bash
sudo chmod 644 /var/www/drupal/web/sites/default/settings.php
sudo chmod 755 /var/www/drupal/web/sites/default
```

Zwei Zeilen gehören noch an das Ende von `/var/www/drupal/web/sites/default/settings.php`. Die erste legt fest, unter welchem Domainnamen die Seite antworten darf (das schützt vor gefälschten Anfragen). Die zweite legt das Verzeichnis für die exportierte Konfiguration fest:

```php
$settings['trusted_host_patterns'] = ['^portal\.meine\-domain\.de$'];
$settings['config_sync_directory'] = '../config/sync';
```

## Schritt 6: Ein eigener PHP-FPM-Pool für Drupal

Auf dem Server läuft mit MediaWiki bereits eine zweite PHP-Anwendung. Damit beide sich nicht gegenseitig den Arbeitsspeicher wegnehmen und sich getrennt einstellen lassen, bekommt Drupal einen **eigenen PHP-FPM-Pool** mit einer eigenen Socket-Datei. Eine Socket-Datei ist ein besonderer Eintrag im Dateisystem, über den der Webserver und PHP auf demselben Rechner miteinander reden.

Dazu die Datei `/etc/php/8.5/fpm/pool.d/drupal.conf` anlegen:

```ini
[drupal]
user = www-data
group = www-data
listen = /run/php/php8.5-fpm-drupal.sock
listen.owner = www-data
listen.group = www-data

pm = dynamic
pm.max_children = 10
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3

php_admin_value[memory_limit] = 256M
```

- `[drupal]` ist der Name des Pools.
- `listen` ist die Socket-Datei, über die NGINX diesen Pool erreicht – eine andere als die von MediaWiki.
- Die `pm.*`-Zeilen steuern, wie viele PHP-Arbeiter gleichzeitig bereitstehen.

Danach PHP-FPM neu starten:

```bash
sudo systemctl restart php8.5-fpm

# Zur Kontrolle: die neue Socket-Datei sollte jetzt da sein
ls /run/php/
```

Wer strikt auf PHP 8.4 bleiben möchte (siehe Abschnitt „Welche Version"), legt diesen Pool stattdessen unter `/etc/php/8.4/fpm/pool.d/` an und passt den Pfad der Socket-Datei entsprechend an.

## Schritt 7: NGINX als Webserver einrichten

Damit die Seite öffentlich und verschlüsselt erreichbar ist, kommt NGINX davor. Es liefert die festen Dateien (Bilder, Stylesheets) selbst aus und reicht alle PHP-Aufrufe an den Drupal-Pool aus Schritt 6 weiter (siehe [Webserver](../server-einrichten/webserver.md)).

Zuerst das SSL-Zertifikat besorgen. Wie das mit **Certbot** und **Let's Encrypt** geht, steht ausführlich im Kapitel [Webserver](../server-einrichten/webserver.md); für eine feste Domain genügt:

```bash
sudo certbot certonly --nginx -d portal.meine-domain.de
```

Dann die Konfigurationsdatei `/etc/nginx/sites-available/drupal` anlegen:

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;
    server_name portal.meine-domain.de;

    ssl_certificate     /etc/letsencrypt/live/portal.meine-domain.de/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/portal.meine-domain.de/privkey.pem;

    root /var/www/drupal/web;
    index index.php;

    # Große Datei-Uploads zulassen (muss zu php.ini aus Schritt 1 passen)
    client_max_body_size 100m;

    # Kurze Adressen: alles, was keine echte Datei ist, geht an Drupal
    location / {
        try_files $uri /index.php?$query_string;
    }

    # PHP nur für index.php und update.php ausführen
    location ~ ^/(index|update)\.php(/|$) {
        include snippets/fastcgi-php.conf;
        fastcgi_param HTTPS on;
        fastcgi_pass unix:/run/php/php8.5-fpm-drupal.sock;
    }

    # Einstellungsdateien, den privaten Bereich und versteckte Dateien sperren
    location ~ ^/sites/[^/]+/(settings|services).*\.(php|yml)$ { deny all; }
    location ~ ^/sites/[^/]+/files/.*\.php$               { deny all; }
    location ~ ^/sites/.+/private/                        { deny all; }
    location ~ /\.(ht|git)                                { deny all; }

    # Feste Dateien lange im Browser zwischenspeichern lassen;
    # noch nicht erzeugte Vorschaubilder gehen an Drupal
    location ~ \.(css|js|gif|jpe?g|png|svg|webp|woff2?)$ {
        try_files $uri @rewrite;
        expires max;
        log_not_found off;
    }
    location @rewrite {
        rewrite ^ /index.php;
    }
}

server {
    listen 80;
    listen [::]:80;
    server_name portal.meine-domain.de;
    # Alle unverschlüsselten Aufrufe auf HTTPS umleiten
    return 301 https://$host$request_uri;
}
```

Der Kern ist der Block `location /`: Fragt der Browser eine echte Datei an (ein Bild, ein Stylesheet), liefert NGINX sie direkt. Fragt er eine Seitenadresse an, gibt es diese Datei nicht – dann übergibt `try_files` die Anfrage an `/index.php`, und Drupal erzeugt die Seite. Der PHP-Block führt bewusst nur `index.php` und `update.php` aus; jede andere PHP-Datei im Verzeichnis bleibt unausgeführt und kann nicht als Einfallstor dienen.

Die Datei aktiv schalten, prüfen und übernehmen:

```bash
sudo ln -s /etc/nginx/sites-available/drupal /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Jetzt ist die Seite unter `https://portal.meine-domain.de/` erreichbar. Die Anmeldung am Administratorbereich erfolgt unter `/user` mit den Zugangsdaten aus Schritt 5.

## Schritt 8: Wartungsaufgaben über einen Zeitplan

Drupal hat Aufgaben, die regelmäßig im Hintergrund laufen müssen: Zwischenspeicher aufräumen, Suchindex aktualisieren, E-Mails verschicken, nach Sicherheitsaktualisierungen sehen. Angestoßen werden sie über einen Zeitplan-Eintrag des Benutzers `www-data`:

```bash
sudo -u www-data crontab -e
```

Dort diese Zeile einfügen – sie arbeitet alle 15 Minuten die anstehenden Aufgaben ab:

```
*/15 * * * * cd /var/www/drupal && /usr/bin/php vendor/bin/drush core:cron >/dev/null 2>&1
```

Betreibt man auf demselben Server auch MediaWiki, steht dessen Zeile für `runJobs` bereits in dieser Tabelle; beide Einträge stören sich nicht.

## Aktualisieren

Drupal wird komplett über Composer aktuell gehalten. Vor einer Aktualisierung immer erst eine Sicherung anlegen (siehe unten). Die Schritte laufen im Projektverzeichnis als der Benutzer, dem die Dateien gehören:

```bash
cd /var/www/drupal

# Neue Fassungen von Drupal Core und Abhängigkeiten holen
sudo -u www-data composer update "drupal/core-*" --with-all-dependencies

# Datenbank an die neue Fassung anpassen und Zwischenspeicher leeren
sudo -u www-data php vendor/bin/drush updatedb --yes
sudo -u www-data php vendor/bin/drush cache:rebuild
```

Der Wechsel auf **Drupal 12** läuft genauso, nur wird dabei die geforderte Hauptversion angehoben:

```bash
sudo -u www-data composer require "drupal/core-recommended:^12" "drupal/core-composer-scaffold:^12" --update-with-all-dependencies
sudo -u www-data php vendor/bin/drush updatedb --yes
sudo -u www-data php vendor/bin/drush cache:rebuild
```

## Sicherung und Wiederherstellung

Zu einem Drupal-Betrieb gehört eine regelmäßige, automatische Sicherung an einen zweiten Ort. Drei Dinge müssen gesichert werden:

```bash
# 1. Die Datenbank – sie enthält Inhalte, Konten und Konfiguration
sudo -u postgres pg_dump drupal > /home/thorsten/backup/drupal-$(date +%F).sql

# 2. Die hochgeladenen Dateien
sudo tar -czf /home/thorsten/backup/drupal-files-$(date +%F).tar.gz \
  -C /var/www/drupal/web/sites/default files

# 3. Die Konfiguration als lesbare Textdateien (zusätzlich zur Datenbank)
cd /var/www/drupal
sudo -u www-data php vendor/bin/drush config:export --yes
```

Der Konfigurations-Export nach `config/sync/` ist besonders nützlich: Diese Textdateien lassen sich in einem Git-Repository versionieren, sodass jede Änderung an der Seitenstruktur nachvollziehbar bleibt. Wie sich der Datenbank-Export über einen Zeitplan automatisieren lässt, steht im Kapitel [Datenbank](../server-einrichten/datenbank.md).

Für die **Wiederherstellung** wird die Datenbank neu angelegt und der Export eingelesen, danach der Datei-Ordner zurückgelegt:

```bash
sudo -u postgres dropdb drupal
sudo -u postgres createdb -E UTF8 -O drupal drupal
sudo -u postgres psql -d drupal -c "CREATE EXTENSION IF NOT EXISTS pg_trgm;"
sudo -u postgres psql -d drupal -f /home/thorsten/backup/drupal-2026-09-06.sql

sudo tar -xzf /home/thorsten/backup/drupal-files-2026-09-06.tar.gz \
  -C /var/www/drupal/web/sites/default
sudo chown -R www-data:www-data /var/www/drupal/web/sites/default/files
```

## Schnell ausprobieren ohne Webserver

Für einen kurzen Test – etwa um zu prüfen, ob die Installation überhaupt läuft – bringt Drush einen eigenen kleinen Webserver mit:

```bash
cd /var/www/drupal
sudo -u www-data php vendor/bin/drush runserver 127.0.0.1:8888
```

Die Adresse `127.0.0.1` sorgt dafür, dass dieser Testserver nur vom Server selbst erreichbar ist. Um ihn trotzdem im Browser des eigenen Rechners zu öffnen, wird wie im Kapitel [XWiki einrichten](./xwiki.md) ein SSH-Tunnel aufgebaut:

```bash
# Auf dem eigenen Rechner ausführen
ssh -L 8888:127.0.0.1:8888 admin@SERVER-IP
```

Danach `http://localhost:8888/` im Browser aufrufen. Für den Dauerbetrieb ist dieser Weg nicht gedacht – dafür ist NGINX aus Schritt 7 zuständig.

## Für dieses Buch

Der Web Stack dieses Buchs läuft im Kern auf **MediaWiki**. **Drupal** kommt als zweiter, redaktioneller Auftritt hinzu, wenn neben der gemeinsam gepflegten Wissenssammlung eine Seite mit fester Redaktion und strukturierten Inhaltstypen entstehen soll (siehe [Content-Management-System](./cms.md)).

Empfohlen wird der Betrieb **direkt auf dem Server** (ohne Container, siehe [Containerisierung von Software](../server-einrichten/containerisierung.md)): PHP-FPM, PostgreSQL und NGINX laufen nebeneinander, Drupal liegt als Composer-Projekt unter `/var/www/drupal`. Damit Drupal und MediaWiki sich nicht behindern, bekommt Drupal einen eigenen PHP-FPM-Pool (Schritt 6).

Als Ausgangsfassung dient **Drupal 11**; der Wechsel auf **Drupal 12** ist vorgesehen, sobald diese erschienen ist – Server, PHP 8.5 und PostgreSQL 18 erfüllen deren Anforderungen bereits. Zwei Dinge gehören von Anfang an zum Betrieb: der Zeitplan-Eintrag für die Wartungsaufgaben (Schritt 8) und eine regelmäßige, automatische Sicherung von Datenbank, Datei-Ordner und Konfiguration an einen zweiten Ort.

## Fazit

Drupal besteht aus fünf zusammenspielenden Teilen: der Laufzeitumgebung PHP-FPM, dem Paketwerkzeug Composer, der Datenbank PostgreSQL, einem Webserver davor und der Kommandozeilen-Shell Drush. Die Einrichtung läuft in klaren Schritten: PHP und seine Erweiterungen installieren, in PostgreSQL eine leere Datenbank mit der Erweiterung `pg_trgm` anlegen, Drupal mit Composer aus dem Projektgerüst `drupal/recommended-project` zusammenstellen, die Dateirechte setzen, mit `drush site:install` ohne Browser installieren, einen eigenen PHP-FPM-Pool einrichten und zuletzt NGINX als verschlüsselten Zugang davorstellen. Weil Drupal 11.4 mit dem PHP 8.5 von Ubuntu 26.04 zurechtkommt und PostgreSQL 18 die Anforderungen übertrifft, ist der spätere Wechsel auf Drupal 12 unkritisch.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
