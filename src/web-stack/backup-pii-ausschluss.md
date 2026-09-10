# Backup mit PII-Ausschluss

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Zu jedem Programm des Web Stacks gehört eine Sicherung. Die Kapitel [MediaWiki einrichten](./mediawiki.md), [XWiki einrichten](./xwiki.md) und [Drupal einrichten](./drupal.md) beschreiben dafür jeweils die **vollständige Sicherung**: ein Abzug der ganzen PostgreSQL-Datenbank mit `pg_dump`, dazu der Ordner mit den hochgeladenen Dateien. Dieser Abzug enthält alles – auch die Benutzerkonten mit ihren E-Mail-Adressen und verschlüsselten Passwörtern – und lässt sich nur in genau dieselbe Umgebung zurückspielen.

Dieses Kapitel behandelt einen anderen Fall: einen Export **nur der Inhalte**, in einem lesbaren Textformat, **ohne persönliche Daten**. Ein solcher Export lässt sich gefahrlos weitergeben – an ein Testsystem, an einen Übersetzungsdienst, an eine öffentliche Spiegelseite, an ein Programm, das eine [RAG-Wissensdatenbank](../server-einrichten/ki-agent.md) daraus baut, oder an eine ganz andere Software. Für Drupal ist dieser Weg bereits im eigenen Kapitel [Inhalte importieren und exportieren](./drupal-import-export.md) ausführlich beschrieben; hier kommen MediaWiki und XWiki dazu, und davor eine Einordnung, welche Systeme das überhaupt sauber können.

## Was „PII" bedeutet

**PII** steht für das englische „personally identifiable information", auf Deutsch **personenbezogene Daten**: alle Angaben, über die sich eine Person erkennen lässt. Name, E-Mail-Adresse, Telefonnummer, Anschrift, aber auch ein Anmeldename oder die Angabe „diese Zeile hat Benutzerin X am 3. März geändert".

In einem Web Stack stecken solche Daten an mehr Stellen, als man zunächst denkt:

- in den **Benutzerkonten** – E-Mail-Adresse, Passwort-Hash, echter Name;
- in der **Autorenangabe** an jedem Inhalt – jeder Artikel, jeder Datensatz ist mit einem Konto verknüpft;
- in **Kommentaren** und auf **Diskussionsseiten** – dort stehen Namen und oft E-Mail-Adressen;
- in **Unterschriften** auf Wiki-Diskussionsseiten – `~~~~` wird zu Benutzername und Zeitstempel;
- in **Bearbeitungskommentaren** – die kurze Notiz „Tippfehler von Herrn Meier korrigiert";
- in den **Metadaten hochgeladener Bilder** – eine Kamera schreibt Aufnahmeort und manchmal den Namen des Fotografen in die Datei;
- im **Freitext der Inhalte selbst** – in einem redaktionellen Text kann eine Telefonnummer stehen.

Ein Export „ohne PII" muss all diese Stellen berücksichtigen, nicht nur die Benutzertabelle. Der Grundsatz dahinter heißt **Datenminimierung**: Wer Inhalte weitergibt, gibt nur die Inhalte weiter – nicht die Menschen, die daran gearbeitet haben.

## Was „datenbankneutral" bedeutet

Ein `pg_dump` ist an PostgreSQL gebunden. Ein datenbankneutraler Export ist dagegen eine Datei in einem allgemeinen Textformat – **YAML**, **JSON**, **XML** oder **CSV**. Sie hängt nicht davon ab, welche Datenbank im Hintergrund läuft, und lässt sich mit jeder Programmiersprache und jeder Tabellensoftware öffnen. Genau diese Eigenschaft macht den Export überhaupt erst weitergabefähig: Das Zielsystem muss kein PostgreSQL sprechen und kein Drupal, MediaWiki oder XWiki sein.

## Der Reifegrad des PII-Ausschlusses

Nicht jedes System kann einen sauberen, portablen Export ohne persönliche Daten gleich gut. Die folgende Einteilung ordnet die Systeme danach ein:

- **Stufe 4 – eingebaut und trennscharf.** Getrennte Befehle für Aufbau (Konfiguration) und Inhalte, ein Textformat, die Benutzerkonten bleiben von sich aus außen vor, alles über die Kommandozeile steuerbar.
- **Stufe 3 – eingebaut, aber Nacharbeit nötig.** Ein Inhaltsexport ist da, doch Autorennamen oder Bearbeitungsnotizen müssen von Hand entfernt werden.
- **Stufe 2 – nur über ein Zusatzmodul.** Der Kern kann es nicht; ein nachinstalliertes Modul liefert den portablen Export.
- **Stufe 1 – nur der Datenbankabzug.** Es gibt keinen inhaltlichen Export; man muss den SQL-Abzug selbst von persönlichen Daten befreien.
- **Stufe 0 – ungeeignet.** Kein Kommandozeilenzugang, kein Textexport, oder das Format ist proprietär.

## Content-Management-Systeme: die Topliste

Geordnet nach dem Reifegrad des PII-Ausschlusses, nicht nach Verbreitung. Alle genannten Systeme sind quelloffen; einige verlangen für den gewerblichen Betrieb eine kostenpflichtige Lizenz, bei offenem Programmtext. Die Spalte „PostgreSQL" zeigt, ob das System die im Kapitel [Datenbank](../server-einrichten/datenbank.md) eingerichtete Datenbank gleichwertig unterstützt.

| System | Technik | PostgreSQL | Reifegrad | Weg zum Export ohne persönliche Daten |
| --- | --- | --- | --- | --- |
| Drupal | PHP | gleichwertig | Stufe 4 | `drush config:export` für den Aufbau, `drush content:export` (Core 11.3) oder Default Content Deploy für die Inhalte; YAML, Autoren ohne persönliche Daten. Siehe [eigenes Kapitel](./drupal-import-export.md). |
| Statamic | PHP | ohne Datenbank möglich | Stufe 4 | Inhalte liegen als Markdown- und YAML-Dateien im Dateisystem; kopieren genügt, im Inhalt steht kein Konto. |
| Grav | PHP | ohne Datenbank | Stufe 4 | wie Statamic: die Inhalte sind Markdown-Dateien in einem Ordner. |
| Strapi | JavaScript | empfohlen | Stufe 3 | `strapi export` (Funktion „Data Transfer") über die Kommandozeile, verschlüsselbares Archiv; Verwaltungskonten und Zugriffstoken bleiben ausgenommen, die Konfiguration ist abwählbar. |
| Wagtail | Python (Django) | gleichwertig | Stufe 3 | `manage.py dumpdata` mit `--exclude auth.user` (JSON); für ganze Seitenbäume das Modul `wagtail-import-export`. |
| TYPO3 | PHP | über Zwischenschicht | Stufe 3 | `typo3 impexp:export` erzeugt eine XML-Datei; Verweise auf Redaktionskonten müssen herausgefiltert werden. |
| Directus | JavaScript | gleichwertig | Stufe 3 | `directus schema snapshot` (YAML) für den Aufbau, getrennt von den Daten; die Daten über die Schnittstelle als JSON oder CSV, die Nutzer-Sammlung dabei auslassen. Lizenzgrenze für größere Betreiber beachten. |
| Payload | JavaScript | gleichwertig | Stufe 3 | kein Kernbefehl; ein kurzes Skript über die eingebaute „Local API" oder das Gemeinschaftsmodul `payload-plugin-import-export`, Ausgabe als JSON. |
| Ghost | JavaScript | ohne (SQLite/MySQL) | Stufe 3 | JSON-Export über das `ghost`-Werkzeug oder die Verwaltungsschnittstelle; die Ausgabe enthält den Abschnitt `users` mit E-Mail-Adressen – vor der Weitergabe entfernen. |
| WordPress | PHP | ohne (MySQL/MariaDB) | Stufe 2 | `wp export` (WP-CLI) erzeugt eine WXR-XML-Datei; sie enthält im Autorenblock auch die E-Mail-Adresse und keine strukturierten Zusatzfelder ohne weiteres Modul. |
| Plone | Python | ohne (Objektablage) | Stufe 2 | `plone.exportimport` (ab Plone 6.1) oder das ältere `collective.exportimport`, JSON; den Mitglieder-Ordner auslassen. |
| Craft CMS | PHP | gleichwertig | Stufe 2 | Projektkonfiguration als `project.yaml`; die Inhalte selbst nur über ein Zusatz-Plugin. |
| October CMS | PHP | möglich | Stufe 2 | Themen- und Vorlagendateien im Dateisystem; Datensätze nur über `php artisan`-Zusatzbefehle einzelner Plugins. |
| Joomla | PHP | eingeschränkt | Stufe 1 | kein Inhaltsexport im Kern; nur der Datenbankabzug oder eine kostenpflichtige Zusatzkomponente. |
| Concrete CMS | PHP | ohne (MySQL/MariaDB) | Stufe 1 | Paket- und Themenexport vorhanden, aber kein trennscharfer Inhaltsexport. |
| Contao | PHP | ohne (MySQL/MariaDB) | Stufe 1 | kein portabler Inhaltsexport; nur der Datenbankabzug. |

An der Spitze steht **Drupal**: Aufbau und Inhalte werden mit getrennten Befehlen über die Kommandozeile ausgegeben, im Textformat YAML, und die Autoren werden von den mitgelieferten Werkzeugen ohne Passwort und E-Mail übertragen. **Statamic** und **Grav** stehen technisch gleichauf, kommen aber ganz ohne Datenbank aus – die Inhalte sind schlicht Dateien im Dateisystem, in denen nie ein Konto steht. Sie passen damit nicht zu einem Aufbau, der bewusst auf PostgreSQL setzt, sind für den reinen PII-Ausschluss aber vorbildlich. Die **Headless-Systeme** Strapi, Directus und Payload liefern einen guten, über die Kommandozeile steuerbaren Export, verlangen aber bei den Nutzerdaten einen bewussten Handgriff. **WordPress** und **Joomla** fallen doppelt zurück: unsauberer Inhaltsexport und keine gleichwertige PostgreSQL-Unterstützung.

## Wissenssysteme: die Topliste

Ebenfalls nach dem Reifegrad des PII-Ausschlusses geordnet. Die Auswahl baut auf den Kapiteln [Wissenssystem](./wissensystem.md) und [Enterprise-Wissenssystem](../grundlagen/enterprise-wissenssystem.md) auf.

| System | Technik | PostgreSQL | Reifegrad | Weg zum Export ohne persönliche Daten |
| --- | --- | --- | --- | --- |
| DokuWiki | PHP | ohne (Dateien) | Stufe 4 | die Seiten liegen als reine Textdateien unter `data/pages/`; kopieren genügt. Änderungs- und Nutzerdaten stehen getrennt in `data/meta/` und `data/changes` und werden einfach weggelassen. |
| Git-gestützte Wikis (Gollum, Wiki.js im Git-Modus) | verschieden | – | Stufe 4 | jede Seite ist eine Markdown-Datei in einem Git-Verlauf; ein Klon ohne Verlauf (`git clone --depth 1`) enthält keine Bearbeiterangaben. |
| XWiki | Java | erste Wahl | Stufe 4 | XAR-Export über die Export-Aktion (`curl`), mit `history=false` und `backup=false`; die verbliebenen Autorfelder im XAR nachträglich leeren. Siehe unten. |
| MediaWiki | PHP | zweitrangig | Stufe 3 | `dumpBackup` mit den Filtern `notalk` und Namensraum-Beschränkung; für eine öffentliche Fassung `<contributor>` und `<comment>` in der XML-Datei leeren. Siehe unten. |
| Foswiki | Perl | ohne (Dateien) | Stufe 3 | die „Topics" sind Textdateien; eine Kopie des `data/`-Baums ohne das Benutzer-Web (`Main/`). |
| BookStack | PHP | ohne (MySQL/MariaDB) | Stufe 3 | ZIP-Export je Buch als HTML, Markdown oder PDF über Oberfläche und Schnittstelle; Autoren erscheinen nur als Klartextname. |
| Outline | JavaScript | gleichwertig (+ Redis) | Stufe 3 | Export als Markdown-ZIP über Oberfläche oder Schnittstelle; die Dokument-Metadaten nennen den Ersteller. Lizenz nicht mehr quelloffen im engen Sinn. |
| Docmost | JavaScript | gleichwertig (+ Redis) | Stufe 3 | Seiten- und Bereichsexport als Markdown- oder HTML-ZIP; sehr junges Projekt. |
| PmWiki | PHP | ohne (Dateien) | Stufe 3 | Seiten als Textdateien im Ordner `wiki.d/`; die Autor-Zeile (`author=`) je Datei entfernen. |
| Tiki Wiki | PHP | ohne (MySQL/MariaDB) | Stufe 2 | Struktur- und Seitenexport vorhanden, aber kein trennscharfer Filter für persönliche Daten. |
| TWiki | Perl | ohne (Dateien) | Stufe 2 | wie Foswiki, aber älter und weniger gepflegt; Foswiki ist die aktive Abspaltung. |
| Trilium Notes | JavaScript | ohne (SQLite) | Stufe 2 | `.zip`-Export je Notizzweig als HTML oder Markdown; Einzelnutzer-System mit wenig Metadaten. |
| MoinMoin | Python | ohne (Dateien) | Stufe 2 | Seiten als Dateien; die neue Version 2 ist noch nicht stabil. |
| Confluence | Java | – | Stufe 0 | die selbst betreibbare Ausgabe wurde eingestellt; übrig bleiben Bezahldienst und eine teure „Data Center"-Lizenz. |

Auffällig ist, dass die **dateibasierten Wikis** (DokuWiki, Foswiki, PmWiki) beim PII-Ausschluss vorn liegen: Wo jede Seite eine eigene Textdatei ist und die Nutzerdaten in getrennten Dateien stehen, ist der saubere Export eine Frage des richtigen `cp`-Befehls. Der Preis dafür ist, dass diese Systeme keine Datenbank nutzen und bei sehr großen Sammlungen mit anspruchsvoller Suche zäh werden (siehe [Wissenssystem](./wissensystem.md)). Von den datenbankgestützten Systemen ist **XWiki** am weitesten, weil die Export-Aktion mit dem Schalter `backup=false` genau die Metadaten weglässt, um die es geht.

## MediaWiki: Inhalte ohne Nutzerdaten exportieren und importieren

### Wo bei MediaWiki persönliche Daten stecken

MediaWiki legt alles in der Datenbank ab. Der eingebaute XML-Export (im Kapitel [MediaWiki einrichten](./mediawiki.md) als „inhaltliche Sicherung" beschrieben) enthält **keine** E-Mail-Adressen und **keine** Passwörter. Er enthält aber:

- die **Namensräume `Benutzer:` und `Benutzer_Diskussion:`** – dort legen angemeldete Personen persönliche Seiten an;
- **Diskussionsseiten** zu Artikeln – mit Unterschriften aus Benutzername und Datum;
- in jeder `<revision>` einen Block `<contributor>` mit `<username>` und interner Nummer;
- oft ein `<comment>` – die Bearbeitungsnotiz, die einen Namen enthalten kann.

Für die Weitergabe an ein **eigenes zweites Wiki** ist das meist unkritisch. Für eine **öffentliche** Fassung müssen diese Stellen raus.

### Der Export

Alle Befehle im Projektverzeichnis `/var/www/mediawiki`. Das vorangestellte `sudo -u www-data` führt sie als der Benutzer aus, dem die MediaWiki-Dateien gehören.

```bash
cd /var/www/mediawiki

# Nur Inhaltsnamensräume, ohne Diskussionsseiten, ohne Versionsgeschichte
sudo -u www-data php maintenance/run.php dumpBackup \
  --current \
  --filter=notalk \
  --filter=namespace:0,6,14 \
  --output=gzip:/home/thorsten/export/inhalt.xml.gz
```

Die drei Angaben im Einzelnen:

- `--current` nimmt nur die jeweils neueste Fassung jeder Seite auf, nicht jede frühere Bearbeitung. Jede frühere Bearbeitung würde einen weiteren Bearbeiter nennen.
- `--filter=notalk` lässt alle Diskussionsnamensräume weg – dort stehen die Unterschriften.
- `--filter=namespace:0,6,14` beschränkt den Export auf Artikel (Namensraum 0), Dateibeschreibungen (6) und Kategorien (14). Damit fallen `Benutzer:` (2) und `Benutzer_Diskussion:` (3) von vornherein weg.

### Die restlichen Autorennamen entfernen

In den verbliebenen Seiten steht noch in jeder Fassung der letzte Bearbeiter. Für eine öffentliche Fassung werden `<username>` und `<comment>` geleert:

```bash
zcat /home/thorsten/export/inhalt.xml.gz \
  | sed -E \
      -e 's#<username>[^<]*</username>#<username>Autor</username>#g' \
      -e 's#<comment>[^<]*</comment>##g' \
  | gzip > /home/thorsten/export/inhalt-anonym.xml.gz
```

MediaWiki schreibt jeden dieser Einträge auf eine eigene Zeile, deshalb reicht hier die einfache Textersetzung. Die interne Bearbeiternummer im `<contributor>`-Block bleibt stehen – eine Zahl ohne Namen; wer auch sie entfernen will, nimmt ein richtiges XML-Werkzeug. Die fertige Datei sollte vor der Weitergabe stichprobenartig durchgesehen werden.

Die **hochgeladenen Bilder** liegen nicht in der XML-Datei. Sie werden wie in [MediaWiki einrichten](./mediawiki.md) beschrieben getrennt gesichert – und vor einer Weitergabe von ihren EXIF-Metadaten befreit (siehe „Was in jedem Fall zu prüfen ist").

### Der Import

Der Export wird in ein **frisch eingerichtetes** Wiki eingelesen. Danach werden die abgeleiteten Verzeichnisse neu aufgebaut:

```bash
cd /var/www/mediawiki
zcat /home/thorsten/export/inhalt-anonym.xml.gz \
  | sudo -u www-data php maintenance/run.php importDump
sudo -u www-data php maintenance/run.php rebuildrecentchanges
sudo -u www-data php maintenance/run.php initSiteStats
sudo -u www-data php maintenance/run.php refreshLinks
```

Für die im Export genannten Benutzernamen legt `importDump` **keine** vollwertigen Konten mit Passwort und E-Mail an; die Namen bleiben reiner Text an den jeweiligen Bearbeitungen. Die Datei `LocalSettings.php` mit den Einstellungen wird durch den XML-Export **nicht** übertragen – der Aufbau des Zielwikis wird getrennt eingerichtet.

## Drupal: der Kurzweg

Für Drupal gilt das eigene Kapitel [Inhalte importieren und exportieren](./drupal-import-export.md). In Kürze und über die Kommandozeile:

```bash
cd /var/www/drupal

# Der Aufbau: Inhaltstypen, Felder, Ansichten – als YAML, ohne Inhalte, ohne Konten
sudo -u www-data php vendor/bin/drush config:export --yes

# Die Inhalte: alle Datensätze eines Typs mit ihren abhängigen Daten
sudo -u www-data php core/scripts/drupal content:export node \
  --bundle=blog --dir=/home/thorsten/export/inhalt --with-dependencies

# Das durch --with-dependencies mitgezogene Autorkonto wieder entfernen
rm -rf /home/thorsten/export/inhalt/user
```

Für ein Nicht-Drupal-Zielsystem tritt an die Stelle von `content:export` das Modul **Views Data Export** mit einer Ansicht, die bewusst kein Autorenfeld enthält, und einer Ausgabe nach CSV, JSON oder XML.

## XWiki: XAR-Export über die Kommandozeile

Ein **XAR** ist das portable Paketformat von XWiki – eine ZIP-Datei mit je einer XML-Datei pro Seite. Die Export-Aktion lässt sich mit `curl` von der Kommandozeile aufrufen:

```bash
curl -u admin:'ADMINPASSWORT' -G \
  "https://wiki.meine-domain.de/xwiki/bin/export/XWiki/XWikiPreferences" \
  --data-urlencode "format=xar" \
  --data-urlencode "name=inhalt" \
  --data-urlencode "history=false" \
  --data-urlencode "backup=false" \
  --data-urlencode "pages=Doku.%" \
  --data-urlencode "pages=Handbuch.%" \
  -o /home/thorsten/export/inhalt.xar
```

Die Angaben im Einzelnen:

- `format=xar` wählt das Paketformat (statt HTML oder PDF).
- `history=false` lässt die Versionsgeschichte weg – jede frühere Fassung nennt ihren Bearbeiter.
- `backup=false` erzeugt **kein** „Backup-Paket". Ein Backup-Paket würde Ersteller, Autor und Zeitstempel jeder Seite so einbetten, dass sie beim Import unverändert wiederhergestellt werden. Ohne diesen Schalter wird die importierende Person zum Autor.
- `pages=Doku.%` nimmt alle Seiten im Bereich „Doku" auf; das `%` ist ein Platzhalter. Die Angabe darf mehrfach stehen.

Auch dann steht in jeder Seiten-XML im XAR noch der zuletzt speichernde Nutzer. Für eine öffentliche Fassung wird das Paket ausgepackt, gesäubert und neu gepackt:

```bash
cd /tmp && rm -rf xar && mkdir xar && cd xar
unzip -q /home/thorsten/export/inhalt.xar

find . -name '*.xml' -exec sed -i -E \
  -e 's#<author>[^<]*</author>#<author>xwiki:XWiki.Autor</author>#g' \
  -e 's#<creator>[^<]*</creator>#<creator>xwiki:XWiki.Autor</creator>#g' \
  -e 's#<contentAuthor>[^<]*</contentAuthor>#<contentAuthor>xwiki:XWiki.Autor</contentAuthor>#g' \
  -e 's#<comment>[^<]*</comment>##g' {} +

zip -qr /home/thorsten/export/inhalt-anonym.xar .
```

### Der Import

Auf der Zielseite wird das XAR wieder eingelesen. Zwei Wege stehen offen:

- **Über die Oberfläche:** Administration → Inhalt → Importieren, das XAR hochladen. Beim Import lässt sich „Verlauf zurücksetzen" wählen, damit keine fremden Bearbeiterangaben übernommen werden.
- **Über die Kommandozeile** mit der REST-Schnittstelle:

  ```bash
  curl -u admin:'ADMINPASSWORT' -X POST \
    --data-binary @/home/thorsten/export/inhalt-anonym.xar \
    "https://ziel.meine-domain.de/xwiki/rest/wikis/xwiki?backup=false&history=RESET"
  ```

  `backup=false` übernimmt keine eingebetteten Autor- und Wiederherstellungsdaten, `history=RESET` legt jede Seite ohne Versionsgeschichte neu an.

Der Import-Endpunkt der REST-Schnittstelle war in älteren XWiki-Fassungen ohne Anmeldung erreichbar (CVE-2026-33137). Vor dem Einsatz sollte eine aktuelle, gepflegte XWiki-Version installiert sein (siehe die Versionshinweise im Kapitel [XWiki einrichten](./xwiki.md)).

Die Konfigurationsdateien unter `/etc/xwiki/` – mit dem Datenbankpasswort – gehören nicht in einen weitergegebenen Export.

## Was in jedem Fall zu prüfen ist

Vor jeder Weitergabe eines Inhaltsexports, unabhängig vom System:

- **Benutzer- und Mitgliederbereiche ausschließen.** Eigene Namensräume, Ordner oder Sammlungen für Konten gehören nicht in den Export.
- **Kommentare und Diskussionsseiten weglassen.** Sie enthalten Namen und oft E-Mail-Adressen.
- **Autor- und Bearbeiterfelder leeren**, wenn der Export öffentlich wird.
- **Bearbeitungskommentare und Änderungsnotizen leeren** – die kurze Zeile pro Speichervorgang.
- **Den Freitext durchsehen.** In redaktionellen Inhalten können Namen, Telefonnummern und Anschriften stehen. Bei einer Weitergabe an Dritte lohnt der Blick in die exportierten Dateien.
- **Konfigurationsdateien prüfen.** Ein Konfigurationsexport kann Absenderadressen für E-Mails, Zugangsschlüssel für Fremddienste oder ein Datenbankpasswort enthalten.
- **Bild-Metadaten entfernen.** Fotos tragen oft Aufnahmeort und Kameradaten im EXIF-Block. Ein Sammelbefehl entfernt sie:

  ```bash
  exiftool -all= -overwrite_original /home/thorsten/export/bilder/
  ```

- **Log- und Statistiktabellen nie mitexportieren.** Zugriffsprotokolle enthalten IP-Adressen.

## Für dieses Buch

Der Web Stack dieses Buchs läuft auf **MediaWiki**, gegebenenfalls mit **Drupal** als zweitem, redaktionellem Auftritt (siehe [Content-Management-System](./cms.md)). Für den Export ohne persönliche Daten gilt:

- **MediaWiki:** `dumpBackup --current --filter=notalk --filter=namespace:0,6,14`. Für eine öffentliche Fassung zusätzlich `<username>` und `<comment>` leeren. Die vollständige Sicherung bleibt davon unberührt – `pg_dump` plus der Ordner `images/`, an einen zweiten Ort.
- **Drupal:** der Weg aus dem Kapitel [Inhalte importieren und exportieren](./drupal-import-export.md) – `drush config:export` für den Aufbau, `content:export` oder Default Content Deploy für die Inhalte, danach den Ordner `user/` löschen.
- **XWiki** (falls es statt MediaWiki eingesetzt wird): XAR-Export über `curl` mit `history=false` und `backup=false`, danach die Autorfelder im Paket leeren.
- Die reinen Textformate lassen sich zusätzlich in einem Git-Repository versionieren (siehe [Git Hosting](../server-einrichten/git-hosting.md)), sodass jede Änderung am weitergegebenen Stand nachvollziehbar bleibt.

Diese Exporte **ersetzen nicht** die vollständige Sicherung mit `pg_dump`. Sie sind ein zusätzliches Werkzeug für den Fall, dass Inhalte ein anderes System erreichen sollen, ohne die Menschen mitzunehmen, die daran gearbeitet haben. Wer Inhalte aus fremden Quellen übernimmt, prüft sie davor gegen das Urheberrecht (siehe [Urheberrecht und Duplicate Content](../grundlagen/urheberrecht-duplicate-content.md)).

## Fazit

Ein Datenbankabzug ist die schnellste Sicherung, aber die unbeweglichste: Er enthält die Benutzerkonten und passt nur in dieselbe Umgebung zurück. Für die Weitergabe an ein anderes System braucht es einen Export **nur der Inhalte**, in einem allgemeinen Textformat, **ohne persönliche Daten**. Persönliche Daten stecken dabei nicht nur in der Benutzertabelle, sondern auch in Autorenangaben, Kommentaren, Unterschriften, Bearbeitungsnotizen und Bild-Metadaten. Nach dem Reifegrad des PII-Ausschlusses stehen bei den CMS **Drupal** und die dateibasierten Systeme vorn, bei den Wissenssystemen die dateibasierten Wikis und **XWiki**. Bei MediaWiki erzeugt `dumpBackup` mit Namensraum- und `notalk`-Filter einen brauchbaren Export, der für eine öffentliche Fassung noch um die Bearbeiternamen bereinigt wird. Bei XWiki liefert die Export-Aktion mit `backup=false` ein XAR ohne einbettete Wiederherstellungs-Metadaten. In jedem Fall gilt: vor der Weitergabe die exportierten Dateien durchsehen.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
