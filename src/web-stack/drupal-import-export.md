# Drupal-Inhalte importieren und exportieren

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Das Kapitel [Drupal einrichten](./drupal.md) beschreibt im Abschnitt „Sicherung und Wiederherstellung" die **vollständige Sicherung**: ein Abzug der ganzen PostgreSQL-Datenbank mit `pg_dump`, dazu der Ordner mit den hochgeladenen Dateien. Dieser Abzug enthält alles – auch die Benutzerkonten mit ihren verschlüsselten Passwörtern – und lässt sich nur in genau dieselbe Umgebung zurückspielen: dieselbe Datenbank, dieselbe Drupal-Fassung.

Dieses Kapitel behandelt einen anderen Fall. Die Inhalte einer Drupal-Seite – die Texte, die strukturierten Datensätze, die Kategorien – sollen so aus Drupal herausgeholt werden, dass ein **anderes System** sie weiterverarbeiten kann. Das andere System kann eine zweite Drupal-Seite sein, ein Testaufbau, ein Übersetzungsdienst oder ein Programm, das gar nichts mit Drupal zu tun hat. Zwei Bedingungen gelten dabei:

- **Datenbankneutral.** Die Ausgabe ist eine Datei in einem lesbaren Textformat (YAML, JSON, CSV oder XML), nicht ein Datenbankabzug. Sie hängt nicht davon ab, ob im Hintergrund PostgreSQL, MySQL oder etwas ganz anderes läuft.
- **Ohne Nutzerdaten.** In der Ausgabe stehen keine Benutzerkonten, keine E-Mail-Adressen, keine Passwörter, keine persönlichen Angaben von angemeldeten Personen. Nur die redaktionellen Inhalte und die Systemkonfiguration.

Alle Befehle werden in der Textkonsole des Servers eingegeben, im Projektverzeichnis `/var/www/drupal`. Das vorangestellte `sudo -u www-data` bedeutet: als der Benutzer ausführen, dem die Drupal-Dateien gehören.

## Was „datenbankneutral" und „ohne Nutzerdaten" bedeutet

Drupal legt alles in der Datenbank ab: Seiten, frühere Fassungen, Kategorien, Benutzerkonten und die gesamte Konfiguration. Ein Datenbankabzug ist deshalb der schnellste Weg zu einer Sicherung – aber der unbeweglichste. Er lässt sich nicht in ein anderes System einlesen und nicht gefahrlos weitergeben, weil die Konten mit darin stehen.

Der Ausweg besteht aus zwei getrennten Exporten:

- **Die Konfiguration** – also welche Inhaltstypen es gibt, welche Felder sie haben, wie die Seite aufgebaut ist. Sie wird als Sammlung von YAML-Dateien exportiert. YAML ist ein Textformat, in dem jede Zeile ein `Schlüssel: Wert`-Paar ist; es ist für Menschen lesbar und für jedes Programm einlesbar.
- **Die Inhalte** – die eigentlichen Datensätze, die in diese Struktur eingetragen wurden. Sie werden je nach Ziel als YAML, JSON, CSV oder XML exportiert.

Beide Exporte enthalten von sich aus **keine** Benutzerkonten. Nutzerdaten geraten nur auf einem Weg mit hinein: Jeder Inhalt hat einen Autor, und der Autor ist eine Verknüpfung zu einem Benutzerkonto. Zieht ein Export „alle abhängigen Daten" mit, kann er dabei auch das Autorkonto mitnehmen. Der Abschnitt „Nutzerdaten sicher heraushalten" weiter unten zeigt, wie man das verhindert.

## Die Konfiguration exportieren

Der Konfigurationsexport ist in Drupal eingebaut und braucht kein Zusatzmodul. Er schreibt die komplette Systemkonfiguration in lesbare YAML-Dateien:

```bash
cd /var/www/drupal
sudo -u www-data php vendor/bin/drush config:export --yes
```

Ohne weitere Angabe landet die Ausgabe im Ordner `config/sync`, der in Schritt 5 des Kapitels [Drupal einrichten](./drupal.md) festgelegt wurde. Für einen Export an eine andere Stelle dient `--destination`:

```bash
sudo -u www-data php vendor/bin/drush config:export --destination=/home/thorsten/export/config --yes
```

In diesen Dateien stehen die Inhaltstypen, die Felder, die Ansichten, die Sprachen und die Moduleinstellungen – aber keine Inhalte und keine Konten. Der Ordner lässt sich unverändert an ein anderes Drupal übergeben; dort liest ihn `drush config:import` wieder ein und stellt damit dieselbe Struktur her. Weil es reine Textdateien sind, lassen sie sich zusätzlich in einem Git-Repository versionieren (siehe [Git Hosting](../server-einrichten/git-hosting.md)), sodass jede Änderung an der Seitenstruktur nachvollziehbar bleibt.

## Die Inhalte exportieren – drei Wege

Für den Inhaltsexport gibt es drei gebräuchliche Wege. Welcher passt, hängt vom Zielsystem ab.

| Weg | Format | Zielsystem | Braucht |
| --- | --- | --- | --- |
| Views Data Export | CSV, JSON, XML | beliebig, auch Nicht-Drupal | Zusatzmodul |
| Default Content Deploy | YAML, JSON | ein anderes Drupal | Zusatzmodul |
| Drupal-Core ab 11.3 | YAML | ein anderes Drupal | nichts (eingebaut) |

### Weg 1: Views Data Export (CSV, JSON, XML)

Das Modul **Views Data Export** erweitert das eingebaute Werkzeug „Views" um Ausgabeformate für Dateien. „Views" ist der Listenbauer von Drupal: Damit stellt man im Browser zusammen, welche Inhalte in einer Liste erscheinen und welche Felder sie zeigt. Views Data Export macht aus so einer Liste eine herunterladbare CSV-, JSON- oder XML-Datei.

Das ist der richtige Weg, wenn das Zielsystem **kein Drupal** ist – eine Tabellenkalkulation, ein Übersetzungsdienst, ein selbst geschriebenes Programm. CSV (Werte durch Kommas getrennt) öffnet jede Tabellensoftware; JSON und XML liest jede Programmiersprache.

Zuerst das Modul mit Composer holen und einschalten:

```bash
cd /var/www/drupal
sudo -u www-data composer require drupal/views_data_export
sudo -u www-data php vendor/bin/drush pm:enable views_data_export --yes
```

Dann im Browser unter „Struktur → Ansichten" eine Ansicht anlegen, die genau die gewünschten Inhalte auflistet – zum Beispiel alle Datensätze vom Typ „Gerät" mit den Feldern Hersteller, Baujahr und Standort. In dieser Ansicht wird eine zusätzliche Anzeige vom Typ „Data export" hinzugefügt und dort das Format (CSV, JSON oder XML) gewählt. Wichtig: Als Felder nur die fachlichen Angaben aufnehmen, **nicht** das Feld „Autor" – dann bleiben die Nutzerdaten von vornherein außen vor.

Die fertige Ansicht lässt sich über die Kommandozeile ausführen und in eine Datei schreiben. Die drei Angaben sind der Maschinenname der Ansicht, der Name der Export-Anzeige und der Zielpfad:

```bash
sudo -u www-data php vendor/bin/drush views:data-export geraete_export data_export_1 \
  /home/thorsten/export/geraete.csv
```

Bei großen Mengen arbeitet das Modul die Ausgabe in Blöcken ab, damit der Arbeitsspeicher nicht überläuft; dazu muss die Ansicht nach einem eindeutigen Feld sortiert sein, etwa der internen Nummer des Datensatzes.

### Weg 2: Default Content Deploy (YAML für ein anderes Drupal)

Soll das Zielsystem **selbst ein Drupal** sein – eine zweite Instanz, ein Testaufbau, eine Bühne zum Bearbeiten –, ist ein Format sinnvoller, das die volle Struktur der Inhalte behält: alle Felder, alle Verknüpfungen, alle Übersetzungen. Dafür gibt es das Modul **Default Content Deploy**. Es exportiert Inhalte als einzelne YAML-Dateien, geordnet nach Typ, und kann sie auf der Zielseite wieder einlesen.

```bash
cd /var/www/drupal
sudo -u www-data composer require drupal/default_content_deploy
sudo -u www-data php vendor/bin/drush pm:enable default_content_deploy --yes
```

Das Modul kennt mehrere Exportbefehle:

```bash
# Alle Datensätze eines Typs exportieren
sudo -u www-data php vendor/bin/drush dcde node --bundle=geraet

# Einen einzelnen Datensatz mit allem, worauf er verweist (Bilder, Kategorien)
sudo -u www-data php vendor/bin/drush dcder node 42

# Den ganzen Inhalt der Seite exportieren
sudo -u www-data php vendor/bin/drush dcdes
```

Die Dateien landen in einem Inhaltsverzeichnis, das über die Einstellung `default_content_deploy_content_directory` in der `settings.php` festgelegt wird, zum Beispiel `../content`. Von dort lassen sie sich als Ordner an die Zielseite übergeben.

Für die Autoren geht Default Content Deploy einen sicheren Weg: Beim Import legt es Benutzerkonten nur mit Kennung und Anzeigename an, **ohne Passwort und ohne E-Mail-Adresse**. Persönliche Daten werden also nicht übertragen. Wer auch das vermeiden will, entfernt vor der Weitergabe den Unterordner `user/` aus dem Export.

### Weg 3: Drupal-Core ab 11.3 (ohne Zusatzmodul)

Seit **Drupal 11.3** (Juni 2025) bringt Drupal einen eigenen Befehl zum Inhaltsexport mit. Er schreibt Inhalte im selben YAML-Format, das früher nur das Zusatzmodul „Default Content" erzeugen konnte – jetzt aber ohne dass irgendein Modul installiert sein muss. Der Befehl gehört nicht zu Drush, sondern zum mitgelieferten Skript `core/scripts/drupal`:

```bash
cd /var/www/drupal

# Einen einzelnen Datensatz auf dem Bildschirm ausgeben
sudo -u www-data php core/scripts/drupal content:export node 3

# Denselben Datensatz in eine Datei schreiben
sudo -u www-data php core/scripts/drupal content:export node 3 > /home/thorsten/export/seite-3.yml

# Alle Datensätze eines Typs mit ihren abhängigen Daten in einen Ordner
sudo -u www-data php core/scripts/drupal content:export node \
  --bundle=blog --dir=/home/thorsten/export/inhalt --with-dependencies
```

Die Optionen im Einzelnen:

- `--dir=<ordner>` schreibt jeden Datensatz als eigene YAML-Datei in einen nach Typ geordneten Ordner, statt alles auf den Bildschirm auszugeben.
- `--bundle=<typ>` beschränkt den Export auf einen Inhaltstyp. Die Option darf mehrfach stehen: `--bundle=blog --bundle=nachricht`.
- `--with-dependencies` (kurz `-W`) nimmt alles mit, worauf ein Datensatz verweist – Bilder, Kategorien, verknüpfte Einträge. **Achtung:** Dazu gehört auch das Autorkonto. Wird diese Option benutzt, entsteht im Zielordner ein Unterordner `user/` mit den Autoren. Er muss vor der Weitergabe gelöscht werden (siehe nächster Abschnitt).

Ein `content:import`-Gegenstück ist im Drupal-Core noch nicht enthalten; für das Einlesen wird weiterhin ein Modul gebraucht (siehe „Die Inhalte wieder importieren").

## Nutzerdaten sicher heraushalten

Vor jeder Weitergabe eines Inhaltsexports sollten diese Punkte geprüft sein:

- **Keine Autorenfelder in Views Data Export.** In der Export-Ansicht nur fachliche Felder aufnehmen, nicht „Autor" oder „Verfasser".
- **Den Ordner `user/` löschen.** Entsteht er durch `--with-dependencies` oder durch einen Vollexport, wird er vor der Weitergabe entfernt:

  ```bash
  rm -rf /home/thorsten/export/inhalt/user
  ```

- **Keine Kommentare mitexportieren.** Kommentare enthalten Namen und oft E-Mail-Adressen der Verfasser. Den Inhaltstyp „comment" nicht in den Export aufnehmen.
- **Freitextfelder durchsehen.** In redaktionellen Texten können Namen, Telefonnummern oder Adressen stehen. Bei einer Weitergabe an Dritte lohnt ein Blick in die exportierten Dateien.
- **Die Konfiguration prüfen.** Der Konfigurationsexport kann Absenderadressen für E-Mails oder Schlüssel für Fremddienste enthalten. Die Dateien im Ordner `config/` vor der Weitergabe durchsehen und solche Werte entfernen.

## Die Inhalte wieder importieren

Auf der Zielseite werden die exportierten Dateien wieder eingelesen. Der Weg hängt davon ab, ob dort ein Kommandozeilenzugang besteht.

### Ohne Drush

Manche gehostete Drupal-Umgebungen bieten keinen Zugang zur Kommandozeile. Dann helfen diese Module, die vollständig über die Weboberfläche bedient werden:

- **Feeds.** Der Klassiker für den Import ohne Kommandozeile. Feeds liest CSV-, JSON-, XML- und RSS-Dateien und ordnet ihre Spalten den Feldern eines Inhaltstyps zu. Der Import läuft auf Knopfdruck als Stapelverarbeitung oder regelmäßig über den Zeitplan. Passt zu den Dateien aus Weg 1.
- **Single Content Sync.** Exportiert und importiert einzelne Datensätze oder ganze Auswahllisten als YAML, wahlweise als ZIP-Archiv mit den zugehörigen Dateien. Auf der Zielseite wird das Archiv über die Oberfläche hochgeladen; Verknüpfungen und Dateien kommen mit. Gut für kleinere Mengen und für den gelegentlichen Austausch.
- **Entity Share.** Kein Dateiaustausch, sondern eine direkte Verbindung: Die Zielseite holt sich die Inhalte über die eingebaute JSON:API direkt von der Quellseite. Voraussetzung ist, dass beide Seiten dieselben Inhaltstypen und Feldnamen haben. Sinnvoll, wenn zwei Drupal-Seiten dauerhaft Inhalte teilen.
- **Migrate über den Zeitplan.** Das eingebaute Migrate-System von Drupal braucht selbst kein Drush, nur einen Auslöser. Mit dem Modul **Migrate Plus** wird eine Migration als YAML-Konfiguration angelegt; ein Modul wie **Migrate Cron Scheduler** startet sie dann automatisch über den Zeitplan. Das Modul **Migrate Tools** bringt zusätzlich eine Übersichtsseite mit „Ausführen"-Schaltfläche mit.

### Mit Drush

Mit Kommandozeilenzugang ist der Import direkter:

- **CSV oder JSON (aus Weg 1).** Diese Dateien liest das Migrate-System ein. Dazu die Module `migrate_plus`, `migrate_tools` und – für CSV – `migrate_source_csv` installieren, eine kurze YAML-Datei anlegen, die Spalten auf Felder abbildet, und den Import starten:

  ```bash
  sudo -u www-data php vendor/bin/drush migrate:import geraete_csv
  ```

- **YAML aus Default Content Deploy (Weg 2).** Den Export-Ordner auf die Zielseite legen und einlesen:

  ```bash
  cd /var/www/drupal
  sudo -u www-data php vendor/bin/drush dcdi
  ```

- **YAML aus dem Drupal-Core (Weg 3).** Das Format entspricht dem des Moduls **Default Content**. Auf der Zielseite dieses Modul installieren und den Ordner einlesen – entweder indem die Dateien in ein eigenes Modul oder eine „Recipe" gelegt werden, oder mit dem Importbefehl aus dem Recipe-Werkzeug:

  ```bash
  sudo -u www-data php vendor/bin/drush content:import /home/thorsten/export/inhalt
  ```

  Beim Import bekommen die Datensätze auf der Zielseite neue interne Nummern; ihre dauerhafte Kennung (UUID) bleibt gleich, sodass ein zweiter Import dieselben Einträge aktualisiert statt sie zu verdoppeln.

## Für dieses Buch

Der Web Stack dieses Buchs nutzt Drupal als zweiten, redaktionellen Auftritt neben [MediaWiki](./mediawiki.md) (siehe [Content-Management-System](./cms.md)). Für den portablen Export gilt:

- **Die Struktur** wird immer mit `drush config:export` gesichert. Der Ordner `config/sync` gehört zusätzlich in ein Git-Repository.
- **Für ein zweites Drupal** (Test, Bühne, Übernahme) ist **Default Content Deploy** der erste Weg, weil es Felder, Verknüpfungen und Übersetzungen behält und die Autoren ohne persönliche Daten überträgt. Sollen zwei Drupal-Seiten dauerhaft Inhalte teilen, ist **Entity Share** die Alternative.
- **Für ein Nicht-Drupal-System** ist **Views Data Export** nach CSV oder JSON der Weg, mit einer Ansicht, die bewusst keine Autorenfelder enthält.
- **Vor jeder Weitergabe** wird der Ordner `user/` gelöscht, die Kommentare bleiben außen vor, und die Konfigurationsdateien werden auf Zugangsschlüssel und Absenderadressen durchgesehen.

Diese Exporte ersetzen **nicht** die vollständige Sicherung mit `pg_dump` aus dem Kapitel [Drupal einrichten](./drupal.md). Sie sind ein zusätzliches Werkzeug für den Fall, dass Inhalte ein anderes System erreichen sollen.

## Fazit

Ein Datenbankabzug ist die schnellste Sicherung, aber die unbeweglichste: Er enthält die Benutzerkonten und passt nur in dieselbe Umgebung zurück. Für die Weitergabe an ein anderes System werden Konfiguration und Inhalte getrennt und als Textdateien exportiert. Die Konfiguration erledigt der eingebaute Befehl `drush config:export`. Für die Inhalte gibt es drei Wege: **Views Data Export** nach CSV, JSON oder XML für beliebige Zielsysteme; **Default Content Deploy** nach YAML für ein anderes Drupal; und seit Drupal 11.3 der eingebaute Befehl `php core/scripts/drupal content:export`, ebenfalls nach YAML. Nutzerdaten bleiben dabei von selbst außen vor, solange keine Autorenfelder mitexportiert werden und der Ordner `user/` vor der Weitergabe gelöscht wird. Zurückgelesen werden die Dateien ohne Kommandozeile über **Feeds**, **Single Content Sync** oder **Entity Share**, mit Kommandozeile über das **Migrate**-System oder die Importbefehle der genannten Module.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
