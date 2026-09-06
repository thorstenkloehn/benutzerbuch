# IDE

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Auf dem Entwicklungs-Rechner – dem eigenen Computer, an dem gearbeitet wird – braucht es ein Programm zum Schreiben und Ändern von Text und Code. Für dieses Buch ist das die Stelle, an der die Kapitel entstehen und an der ein KI-Agent wie Claude Code (siehe [Voraussetzungen](../grundlagen/voraussetzungen.md)) mitarbeitet.

Ein solches Programm heißt **Editor**, wenn es vor allem Text bearbeitet, und **IDE**, wenn viele Hilfen fürs Programmieren fest eingebaut sind. „IDE" steht für „Integrated Development Environment", auf Deutsch „integrierte Entwicklungsumgebung": Editor, Fehlersuche, Versionsverwaltung und Werkzeuge zum Ausführen von Code stecken in einem einzigen Fenster. Der Übergang ist fließend – moderne Editoren lassen sich mit Erweiterungen so weit ausbauen, dass sie einer IDE nahekommen.

Dieses Kapitel zeigt, wie sich vier verbreitete Vertreter auf einem Rechner mit **Ubuntu** einrichten lassen: **Visual Studio Code**, die **JetBrains**-Programme, **Zed** und **Vim**. Ubuntu ist hier gewählt, weil es die größte Sammlung passender Anleitungen hat und weil der Rest des Buchs ebenfalls mit Ubuntu arbeitet (siehe [Betriebssystem](../server-einrichten/betriebssystem.md)). Alle Befehle werden in einem **Terminal** eingegeben – einem Fenster, in das man Anweisungen als Text tippt. Ein vorangestelltes `sudo` bedeutet: mit Verwaltungsrechten ausführen; das System fragt dann nach dem Passwort.

## Die vier Programme im Überblick

| Programm | Art | Grundlage | Besonders geeignet für |
| --- | --- | --- | --- |
| **Visual Studio Code** | Editor mit vielen Erweiterungen | frei nutzbar, von Microsoft | Der breite Standard; gute Anbindung an KI-Agenten |
| **JetBrains** (IntelliJ IDEA, PyCharm u. a.) | vollständige IDE | teils kostenlos, teils kostenpflichtig | Große Projekte in einer festen Programmiersprache |
| **Zed** | schneller Editor | quelloffen, in Rust geschrieben | Sehr schnelles Arbeiten; gemeinsames Bearbeiten |
| **Vim** | Editor für das Terminal | quelloffen, sehr alt und stabil | Arbeiten ganz ohne Maus, auch über eine Fernverbindung |

Man muss sich nicht auf ein Programm festlegen. Viele arbeiten mit einem Hauptprogramm und haben Vim zusätzlich für schnelle Änderungen auf dem Server.

## Visual Studio Code

Visual Studio Code – kurz **VS Code** – ist der am weitesten verbreitete Editor dieser Art. Er ist kostenlos, läuft auf allen gängigen Betriebssystemen und lässt sich über einen eingebauten Laden mit Erweiterungen an fast jede Sprache und Aufgabe anpassen. Für dieses Buch ist wichtig: Für Claude Code gibt es eine fertige Erweiterung, sodass der KI-Agent ohne getrenntes Terminalfenster direkt im Editor arbeitet (siehe [Voraussetzungen](../grundlagen/voraussetzungen.md)).

### Installation über das .deb-Paket

Ein **.deb-Paket** ist die Installationsdatei für die Ubuntu-Familie – vergleichbar mit einer Setup-Datei unter Windows. Die aktuelle Datei wird von der offiziellen Seite `code.visualstudio.com` heruntergeladen (Schaltfläche „.deb", 64-Bit). Danach im Ordner mit der heruntergeladenen Datei:

```bash
sudo apt install ./code_*.deb
```

Der Punkt und der Schrägstrich vor dem Dateinamen sind wichtig: Sie sagen `apt`, dass eine Datei aus dem aktuellen Ordner gemeint ist und kein Paket aus dem Netz. Das Sternchen steht für den wechselnden Versionsteil im Dateinamen.

Bei der Installation trägt das Paket zusätzlich die Paketquelle von Microsoft in das System ein. Das hat einen praktischen Vorteil: Von da an kommen neue Fassungen von VS Code automatisch mit den übrigen Systemaktualisierungen, ohne dass man erneut eine Datei herunterladen muss.

```bash
sudo apt update
sudo apt upgrade
```

Gestartet wird der Editor über das Anwendungsmenü oder im Terminal mit `code`. Ein bestimmter Ordner öffnet sich mit `code .` – der Punkt steht für „der Ordner, in dem ich gerade bin".

### Hinweis zur Snap-Fassung

Ubuntu bietet VS Code auch als **Snap** an – ein Paketformat von Canonical, bei dem das Programm stärker vom übrigen System abgeschottet läuft. Für VS Code führt diese Abschottung gelegentlich zu Reibung, etwa beim Zugriff auf andere Werkzeuge im Terminal. Das .deb-Paket ist deshalb die ruhigere Wahl.

## Die JetBrains-Programme

JetBrains ist eine Firma, die für viele Programmiersprachen jeweils eine eigene, vollständige IDE anbietet: **IntelliJ IDEA** für Java und verwandte Sprachen, **PyCharm** für Python, **WebStorm** für Webentwicklung, **CLion** für C und C++, **RustRover** für Rust und weitere. Diese Programme nehmen einem viel Handarbeit ab – sie verstehen den Code tief, finden Fehler früh und benennen Dinge projektweit sicher um. Der Preis dafür: Sie brauchen mehr Arbeitsspeicher und starten langsamer als ein schlanker Editor. Einige Ausgaben (die „Community"-Fassungen von IntelliJ IDEA und PyCharm) sind kostenlos, die übrigen kosten eine jährliche Gebühr; für Lernende und quelloffene Projekte gibt es sie kostenfrei.

### Installation über die Toolbox App

Der von JetBrains empfohlene Weg ist nicht, jede IDE einzeln zu installieren, sondern zuerst die **Toolbox App**. Das ist ein kleines Verwaltungsprogramm, über das sich alle JetBrains-IDEs mit einem Klick installieren, aktuell halten und wieder entfernen lassen. Es verwaltet auch mehrere Versionen nebeneinander und die Lizenzen.

Die Toolbox App wird von `jetbrains.com/toolbox-app` als **.tar.gz**-Archiv für Linux heruntergeladen – eine gepackte Datei, vergleichbar mit einem ZIP-Archiv. Danach im Ordner mit dem Download:

```bash
# Archiv in den Ordner /opt entpacken (dort liegen zusätzlich installierte Programme)
sudo tar -xzf jetbrains-toolbox-*.tar.gz -C /opt

# In den entpackten Ordner wechseln und das Programm einmalig starten
cd /opt/jetbrains-toolbox-*
./jetbrains-toolbox
```

Beim ersten Start trägt sich die Toolbox App selbst in das Anwendungsmenü ein; danach genügt der normale Programmstart. Über ihr Fenster wird dann die gewünschte IDE ausgewählt und installiert.

Startet die Toolbox App nicht und meldet einen Fehler zu „FUSE", fehlt eine Hilfsbibliothek zum Einbinden solcher Programmdateien. Sie wird mit einem Befehl nachgerüstet:

```bash
sudo apt install libfuse2t64
```

### Einfachere Alternative über Snap

Wer nur eine einzige kostenlose IDE braucht und auf die Verwaltung über die Toolbox App verzichten kann, installiert sie direkt als Snap. Beispiel für die kostenlose Fassung von IntelliJ IDEA:

```bash
sudo snap install intellij-idea-community --classic
```

`--classic` erlaubt dem Programm den vollen Zugriff auf das System, den eine IDE braucht.

## Zed

**Zed** ist ein neuerer Editor, der auf Geschwindigkeit ausgelegt ist. Er ist in der Programmiersprache Rust geschrieben, quelloffen und reagiert auch bei großen Dateien ohne spürbare Verzögerung. Eingebaut sind unter anderem das gemeinsame Bearbeiten einer Datei durch mehrere Personen in Echtzeit und eine Anbindung an KI-Modelle. Die Linux-Fassung kam 2024 dazu und wird seither stetig weiterentwickelt.

### Installation über das offizielle Skript

JetBrains und VS Code werden als Paket installiert; Zed bringt ein eigenes Installationsskript mit. Der folgende Befehl lädt es herunter und führt es aus:

```bash
curl -f https://zed.dev/install.sh | sh
```

Das Skript legt Zed im persönlichen Ordner unter `~/.local` ab – es sind also keine Verwaltungsrechte nötig, `sudo` entfällt hier. Ein solcher Befehl, der ein Skript aus dem Netz sofort ausführt, setzt Vertrauen in die Quelle voraus; das ist bei vielen Entwicklerwerkzeugen üblich (auch die KI-Agenten in [Voraussetzungen](../grundlagen/voraussetzungen.md) werden so eingerichtet). Wer das Skript zuerst ansehen möchte, ruft `https://zed.dev/install.sh` im Browser auf und führt den Inhalt erst danach aus.

Nach der Installation wird der Editor mit `zed` gestartet, ein bestimmter Ordner mit `zed .`. Aktualisiert wird Zed von selbst; das Skript lässt sich zum Nachrüsten aber auch erneut ausführen.

Zed zeichnet seine Oberfläche über die Grafikschnittstelle **Vulkan**. Auf den meisten Ubuntu-Systemen ist die nötige Unterstützung vorhanden; fehlt sie, hilft:

```bash
sudo apt install mesa-vulkan-drivers
```

## Vim

**Vim** ist ein Editor, der vollständig im Terminal läuft – ohne Fenster, ohne Maus, allein über die Tastatur. Das wirkt zunächst sperrig, hat aber einen handfesten Nutzen: Vim ist auf praktisch jedem Linux-Server bereits vorhanden, startet sofort und funktioniert auch über eine reine Textverbindung zu einem entfernten Rechner (**SSH**). Für schnelle Änderungen an einer Einstellungsdatei auf dem Server ist es deshalb das naheliegende Werkzeug, auch für alle, die sonst mit einem grafischen Editor arbeiten.

### Installation

Ubuntu bringt nur eine abgespeckte Fassung mit. Die vollständige Fassung wird mit einem Befehl nachgeholt:

```bash
sudo apt update
sudo apt install vim
```

Gestartet wird der Editor mit `vim` gefolgt vom Dateinamen, zum Beispiel `vim notiz.txt`. Wichtig für den Anfang: Vim kennt verschiedene Zustände. Nach dem Start ist man im **Normalmodus** – Tastendrücke sind hier Befehle, kein Text. Mit der Taste `i` wechselt man in den **Einfügemodus** und schreibt wie gewohnt. Die Taste `Esc` führt in den Normalmodus zurück. Von dort speichert und beendet die Eingabe `:wq` gefolgt von `Enter`; ohne Speichern beendet `:q!`.

Eine eigene Einrichtung – Farben, Zeilennummern, Einrückung – kommt in die Datei `~/.vimrc` im persönlichen Ordner. Sie ist beim ersten Mal leer und wird nach und nach ergänzt.

### Neovim als moderne Fortführung

Aus Vim ist das Projekt **Neovim** hervorgegangen. Es ist mit Vim weitgehend gleich zu bedienen, aber innen aufgeräumt und leichter mit Erweiterungen auszubauen; viele fertige Einrichtungspakete bauen darauf. Ubuntu liefert eine aktuelle Fassung mit:

```bash
sudo apt install neovim
```

Gestartet wird es mit `nvim`. Wer die jeweils neueste Fassung möchte, fügt vorher die offizielle Paketquelle des Projekts hinzu:

```bash
sudo add-apt-repository ppa:neovim-ppa/stable
sudo apt update
sudo apt install neovim
```

## Für dieses Buch

Für die Arbeit an diesem Handbuch wird **Visual Studio Code** empfohlen, installiert über das **.deb-Paket**. Der Grund ist die Zusammenarbeit mit dem KI-Agenten: Für Claude Code gibt es eine Erweiterung, die sich sauber in den Editor einfügt, und die meisten Anleitungen im Umfeld der KI-Agenten gehen von VS Code aus (siehe [Voraussetzungen](../grundlagen/voraussetzungen.md)). Die Kapitel selbst sind einfache Markdown-Dateien (siehe [Docs-as-Code](./docs-as-code.md)), für die kein schweres Werkzeug nötig ist.

Zusätzlich lohnt es sich, **Vim** oder **Neovim** einzurichten – nicht als Hauptprogramm, sondern für schnelle Änderungen direkt auf dem Server, wo kein grafischer Editor zur Verfügung steht.

Eine vollständige **JetBrains**-IDE ist sinnvoll, sobald neben dem Buch ein größeres Programm in einer festen Sprache entsteht. **Zed** ist eine gute Wahl für alle, denen VS Code zu träge ist und die auf dessen großen Erweiterungsladen verzichten können.

## Fazit

Auf dem Entwicklungs-Rechner braucht es ein Programm zum Bearbeiten von Text und Code. **Visual Studio Code** ist der breite Standard, wird über ein **.deb-Paket** installiert und hält sich danach über die Systemaktualisierungen selbst aktuell; für dieses Buch ist es wegen der Claude-Code-Erweiterung die erste Wahl. Die **JetBrains**-Programme sind vollständige Entwicklungsumgebungen für große Projekte und werden am besten über die **Toolbox App** verwaltet. **Zed** ist ein besonders schneller, quelloffener Editor und wird über ein offizielles Skript eingerichtet. **Vim** – oder seine Fortführung **Neovim** – läuft im Terminal, ist auf jedem Server vorhanden und deshalb für schnelle Änderungen über eine Fernverbindung unverzichtbar.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
