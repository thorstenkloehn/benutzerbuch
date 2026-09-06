# Allgemeine Einstellungen

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Bevor auf dem Entwicklungs-Rechner – dem eigenen Computer, an dem gearbeitet wird – ein Editor eingerichtet oder die erste Programmiersprache installiert wird, sollte das System einmal grundlegend vorbereitet sein. Dieses Kapitel führt durch die Schritte, die für fast jedes spätere Kapitel die Grundlage bilden: das System aktualisieren, einige Hilfsbibliotheken einspielen, den Browser installieren, die Versionsverwaltung **Git** einrichten und den Zugang zum Passwortspeicher der Datenbank vorbereiten.

Als Grundlage dient wie im übrigen Buch **Ubuntu**. Alle Befehle werden in einem **Terminal** eingegeben – einem Fenster, in das man Anweisungen als Text tippt. Ein vorangestelltes `sudo` bedeutet: mit Verwaltungsrechten ausführen; das System fragt dann nach dem Passwort. Wer das Terminal noch nie geöffnet hat, findet es im Anwendungsmenü unter „Terminal" oder über die Tastenkombination `Strg`+`Alt`+`T`.

Die Reihenfolge der Abschnitte ist bewusst gewählt. Am besten arbeitet man sie einmal von oben nach unten ab.

## System aktualisieren

Ein frisch installiertes Ubuntu ist selten auf dem neuesten Stand. Der erste Schritt bringt die Liste der verfügbaren Pakete und danach die installierte Software auf den aktuellen Stand:

```bash
# Die Liste der verfügbaren Pakete neu einlesen
sudo apt-get update

# Alle installierten Pakete auf die neueste Fassung bringen
sudo apt-get upgrade
```

`apt-get` ist das Programm, mit dem Ubuntu Software verwaltet – vergleichbar mit einem App-Laden, nur über die Tastatur bedient. Der Befehl `update` lädt dabei nur das Verzeichnis neu, `upgrade` installiert die Aktualisierungen tatsächlich. Bei `upgrade` fragt das System einmal nach, ob die aufgelisteten Änderungen ausgeführt werden sollen; mit `J` und `Enter` wird bestätigt.

### Hilfsbibliotheken für den Passwortspeicher

Viele Entwicklerwerkzeuge – darunter Git und die KI-Agenten aus dem Kapitel [Voraussetzungen](../grundlagen/voraussetzungen.md) – müssen Zugangsdaten speichern, damit man sie nicht bei jedem Schritt neu eintippt. Ubuntu hat dafür einen verschlüsselten Speicher, den **Schlüsselbund** (englisch *keyring*). Damit die Werkzeuge diesen Speicher nutzen können, wird die Bibliothek **libsecret** benötigt:

```bash
sudo apt install libsecret-1-0 libsecret-tools libsecret-1-dev libglib2.0-dev
```

Die vier Pakete haben unterschiedliche Aufgaben: `libsecret-1-0` ist die eigentliche Bibliothek, `libsecret-tools` liefert das Kommando `secret-tool` zum Nachsehen, was gespeichert ist. Die Pakete mit `-dev` im Namen und `libglib2.0-dev` enthalten die Bauteile, mit denen sich das kleine Hilfsprogramm übersetzen lässt, das Git mit dem Schlüsselbund verbindet. Ohne diese Pakete lässt sich dieser Verbinder auf manchen Ubuntu-Fassungen nicht einrichten.

### Zusätzliche Treiber

Ubuntu bringt für die meiste Hardware freie Treiber mit. Für einige Bauteile – vor allem Grafikkarten von Nvidia – gibt es zusätzlich Treiber des Herstellers, die mehr Leistung bringen. Dieser Befehl sucht die passenden heraus und installiert sie:

```bash
sudo ubuntu-drivers install
```

Nach der Installation ist ein Neustart des Rechners nötig, damit die neuen Treiber verwendet werden. Meldet der Befehl, dass keine zusätzlichen Treiber verfügbar sind, ist das kein Fehler – dann genügen die freien Treiber, und der Schritt kann übersprungen werden.

## Google Chrome installieren

Ubuntu bringt den Browser **Firefox** bereits mit. Zusätzlich ist **Google Chrome** nützlich: Viele Weboberflächen werden vor allem in Chrome getestet, und für automatische Tests einer Webseite (siehe [Webseiten und Blogs](./docs-as-code-webseiten.md)) wird oft Chrome im Hintergrund gesteuert.

Chrome liegt nicht in den Paketquellen von Ubuntu, sondern wird als einzelne Installationsdatei von Google geladen. Eine solche `.deb`-Datei ist die Installationsdatei für die Ubuntu-Familie, vergleichbar mit einer Setup-Datei unter Windows:

```bash
# In den Ordner für flüchtige Dateien wechseln
cd /tmp

# Die aktuelle Installationsdatei von Google herunterladen
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb

# Die heruntergeladene Datei installieren
sudo dpkg -i google-chrome-stable_current_amd64.deb
```

`wget` lädt eine Datei aus dem Netz, `dpkg -i` spielt eine `.deb`-Datei ein. Beschwert sich `dpkg` über fehlende Bestandteile, holt der folgende Befehl sie nach und schließt die Installation ab:

```bash
sudo apt install -f
```

Bei der Installation trägt Chrome zusätzlich die Paketquelle von Google in das System ein. Neue Fassungen kommen dadurch später automatisch mit den übrigen Systemaktualisierungen.

## Git einrichten

**Git** ist die Versionsverwaltung, mit der Änderungen an Dateien festgehalten und rückgängig gemacht werden können. Für dieses Buch ist es die Grundlage: Jedes Kapitel wird als Datei in einem Git-Verlauf gespeichert. Zusammen mit Git wird die **GitHub CLI** (`gh`) installiert – ein Zusatzwerkzeug, das die Arbeit mit dem Online-Dienst GitHub über das Terminal erledigt.

```bash
sudo apt-get install git gh
```

Nach der Installation trägt man einmalig seinen Namen und seine E-Mail-Adresse ein. Diese beiden Angaben schreibt Git in jede festgehaltene Änderung, damit später erkennbar ist, von wem sie stammt:

```bash
git config --global user.email "name@example.com"
git config --global user.name "Vorname Nachname"
```

`--global` bedeutet: Die Einstellung gilt für alle Projekte dieses Benutzers, nicht nur für ein einzelnes.

### Anmeldung bei GitHub

Damit sich Git und `gh` mit dem eigenen GitHub-Konto verbinden können, meldet man sich einmal an:

```bash
gh auth login
```

Das Programm stellt dann einige Fragen: ob es um GitHub.com oder eine eigene Firmen-Installation geht, ob die Verbindung über HTTPS oder SSH laufen soll und ob die Anmeldung im Browser erfolgen darf. Für den Standardfall wählt man **GitHub.com**, **HTTPS** und die Anmeldung über den Browser. `gh` zeigt dann einen kurzen Zahlencode an, den man auf der geöffneten Webseite einträgt. Danach ist die Anmeldung dauerhaft gespeichert, und auch Git nutzt sie für den Zugriff auf die eigenen Projekte.

## Passwort-Zugang zur Datenbank vorbereiten

Wer lokal mit der Datenbank **PostgreSQL** arbeitet (siehe [Datenbank](../server-einrichten/datenbank.md)), muss sonst bei jedem Zugriff das Datenbank-Passwort eingeben. Das lässt sich mit einer kleinen Datei im persönlichen Ordner abkürzen. Sie heißt `.pgpass` – der Punkt am Anfang macht sie zu einer versteckten Datei:

```bash
nano ~/.pgpass
```

`nano` ist ein einfacher Editor im Terminal; `~` steht für den persönlichen Ordner. In die geöffnete Datei kommt eine Zeile nach diesem Muster:

```text
localhost:5432:*:dein_benutzer:dein_passwort
```

Die fünf durch Doppelpunkte getrennten Felder bedeuten: Rechner, Anschlussnummer (bei PostgreSQL üblicherweise `5432`), Datenbank (`*` steht für „alle"), Benutzername und Passwort. Gespeichert wird in `nano` mit `Strg`+`O` und `Enter`, geschlossen mit `Strg`+`X`.

PostgreSQL benutzt diese Datei nur, wenn sonst niemand sie lesen kann. Deshalb werden die Zugriffsrechte eng gesetzt:

```bash
chmod 0600 ~/.pgpass
```

`0600` bedeutet: nur der Besitzer darf lesen und schreiben, sonst niemand. Fehlt dieser Schritt, weist PostgreSQL die Datei stillschweigend ab.

## Sudo ohne Passwort (optional)

Bei der Arbeit auf dem Entwicklungs-Rechner fällt viel `sudo` an, und jedes Mal fragt das System nach dem Passwort. Man kann das für den eigenen Benutzer abschalten. Das ist bequem, senkt aber die Sicherheit spürbar: Jedes Programm, das unter dem eigenen Benutzer läuft, kann dann ohne Rückfrage Verwaltungsrechte erlangen. Auf einem Rechner, der nur zum Entwickeln dient und an dem sonst niemand arbeitet, ist das vertretbar – auf einem Server oder einem gemeinsam genutzten Rechner sollte man darauf verzichten.

Geändert wird die Einstellung mit einem eigenen Befehl, der die Konfigurationsdatei sicher öffnet und vor dem Speichern auf Fehler prüft:

```bash
sudo visudo
```

Am Ende der Datei wird eine Zeile ergänzt – `dein_benutzername` durch den eigenen Anmeldenamen ersetzen (der Befehl `whoami` zeigt ihn an):

```text
dein_benutzername ALL=(ALL) NOPASSWD:ALL
```

Danach speichern und schließen. Öffnet `visudo` den Editor `vi`, der ungewohnt zu bedienen ist, hilft vorab die Umstellung auf `nano`:

```bash
sudo EDITOR=nano visudo
```

Wer die Bequemlichkeit möchte, ohne den vollen Verzicht auf die Passwortabfrage, kann `NOPASSWD` auch auf einzelne Befehle beschränken. Das sprengt aber den Rahmen dieses Kapitels und ist in der Regel nicht nötig.

## Nächste Schritte

Das System ist jetzt vorbereitet. Danach folgen üblicherweise diese Kapitel:

- [IDE](./ide.md) – ein Programm zum Schreiben von Text und Code einrichten.
- [Programmiersprachen](./programmiersprachen.md) – den Werkzeugkasten der benötigten Sprachen installieren.
- [Docs-as-Code](./docs-as-code.md) – wie die Kapitel dieses Buchs als Markdown-Dateien entstehen.

## Fazit

Die Grundeinrichtung eines Entwicklungs-Rechners besteht aus wenigen Schritten: das System mit `apt-get update` und `apt-get upgrade` aktualisieren, mit **libsecret** den Zugang zum verschlüsselten **Schlüsselbund** schaffen, mit `ubuntu-drivers install` passende Treiber nachrüsten, **Google Chrome** als zweiten Browser installieren und **Git** samt **GitHub CLI** mit Namen, E-Mail und einmaliger Anmeldung einrichten. Eine `.pgpass`-Datei mit den Rechten `0600` erspart später die ständige Passworteingabe bei **PostgreSQL**. Das Abschalten der `sudo`-Passwortabfrage ist bequem, aber nur auf einem allein genutzten Entwicklungs-Rechner vertretbar.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
