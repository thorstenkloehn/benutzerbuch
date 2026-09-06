# Programmiersprachen

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Auf dem Entwicklungs-Rechner – dem eigenen Computer, an dem gearbeitet wird – entsteht der Code. Damit man Code schreiben, ausprobieren und in ein fertiges Programm übersetzen kann, muss für jede Programmiersprache ihr **Werkzeugkasten** installiert sein. Dieser Werkzeugkasten besteht meist aus drei Teilen: dem Programm, das den Code ausführt oder übersetzt, einer Sammlung fertiger Bausteine und einem **Paketwerkzeug**, das weitere Bausteine aus dem Internet nachlädt.

Das Kapitel [Laufzeitumgebung](../server-einrichten/laufzeitumgebung.md) beschreibt das Gegenstück auf dem Server: Dort wird nur die schlanke Laufzeit eingerichtet, mit der fertige Programme betrieben werden. Auf dem Entwicklungs-Rechner ist es umgekehrt – hier braucht man die **vollständige Entwicklungsausstattung**, weil man Software nicht nur laufen lässt, sondern selbst baut.

Dieses Kapitel zeigt für acht verbreitete Sprachen, wie man ihren Werkzeugkasten auf einem Rechner mit **Ubuntu** einrichtet: **Python**, **Java**, **C und C++**, **Go**, **Node.js**, **Rust**, **.NET** und **PHP**. Ubuntu ist hier gewählt, weil der Rest des Buchs ebenfalls damit arbeitet (siehe [IDE](./ide.md) und [Betriebssystem](../server-einrichten/betriebssystem.md)). Alle Befehle werden in einem **Terminal** eingegeben – einem Fenster, in das man Anweisungen als Text tippt. Ein vorangestelltes `sudo` bedeutet: mit Verwaltungsrechten ausführen; das System fragt dann nach dem Passwort.

## Ein gemeinsames Hilfsmittel vorab: curl

Mehrere der folgenden Installationen laden Dateien aus dem Netz. Dafür wird das kleine Werkzeug **curl** benötigt. Es ist mit einem Befehl eingerichtet und danach für alle weiteren Schritte vorhanden:

```bash
sudo apt update
sudo apt install curl
```

## Laufzeit oder volle Ausstattung

Bei einigen Sprachen gibt es zwei Pakete: die reine Laufzeit zum Betreiben fertiger Programme und die volle Ausstattung zum Entwickeln. Bei Java heißt die volle Ausstattung **JDK** (Java Development Kit), bei .NET **SDK** (Software Development Kit). Auf dem Entwicklungs-Rechner wählt man immer die volle Ausstattung – sie enthält den Übersetzer, die Fehlersuche und die Werkzeuge, die zum Bauen nötig sind. Die schlanke Laufzeit ist nur für den Server gedacht.

## Python

Python ist eine Sprache, die auf gut lesbaren Code setzt. Sie wird oft für kurze Hilfsprogramme, Auswertungen und Automatisierung genommen und ist im KI-Umfeld die vorherrschende Sprache. Python-Code wird nicht vorab übersetzt, sondern von einem Ausführer Zeile für Zeile abgearbeitet.

Ubuntu bringt Python 3 bereits mit, aber ohne die üblichen Zusatzteile. Diese werden nachinstalliert:

```bash
# python3-pip ist das Paketwerkzeug, python3-venv legt abgetrennte Umgebungen an;
# python-is-python3 sorgt dafür, dass der Befehl "python" Python 3 startet
sudo apt install python3 python3-pip python3-venv python-is-python3

# Zeigt die installierte Version zur Kontrolle an
python --version
```

Eine Besonderheit von Python sollte man kennen: Zusätzliche Bausteine werden nicht für den ganzen Rechner installiert, sondern in einer abgetrennten Umgebung je Projekt, einer sogenannten **virtuellen Umgebung**. Das verhindert, dass zwei Projekte sich mit unterschiedlichen Versionen desselben Bausteins ins Gehege kommen. Angelegt und eingeschaltet wird eine solche Umgebung im Projektordner mit:

```bash
python -m venv .venv
source .venv/bin/activate
```

Solange die Umgebung eingeschaltet ist, landet jedes `pip install` nur in diesem einen Ordner. Neuere Ubuntu-Ausgaben bestehen sogar darauf: Ein `pip install` außerhalb einer virtuellen Umgebung wird abgewiesen.

## Java

Java ist eine der ältesten noch weit verbreiteten Sprachen für Server. Java-Code wird nicht für einen bestimmten Rechnertyp übersetzt, sondern in eine Zwischenform, die auf jedem Betriebssystem dieselbe Laufzeitumgebung ausführt – die **Java Virtual Machine**. Dadurch läuft dasselbe Programm unverändert auf Windows, Linux und macOS.

Die freie Ausgabe von Java heißt **OpenJDK**. Zum Entwickeln wird das vollständige **JDK** installiert, nicht die abgespeckte Laufzeit:

```bash
# Vollständiges Entwicklungspaket ("jdk"), Fassung ohne grafische Bestandteile ("headless")
sudo apt install openjdk-26-jdk-headless

# Zeigt die installierte Version zur Kontrolle an
javac -version
```

Wer eine bestimmte ältere Java-Version für ein Projekt braucht, kann mehrere Fassungen nebeneinander installieren und mit dem Befehl `sudo update-alternatives --config java` zwischen ihnen umschalten.

Für größere Java-Projekte kommt zusätzlich ein **Bauwerkzeug** dazu, das den Ablauf vom Quellcode zum fertigen Programm steuert und Bausteine nachlädt. Verbreitet sind **Maven** und **Gradle**:

```bash
sudo apt install maven
```

## C und C++

C und C++ sind hardwarenahe Sprachen. Ihr Code wird vollständig in Maschinenbefehle für genau einen Rechnertyp übersetzt und läuft danach ohne Laufzeitumgebung allein. Diese Übersetzung erledigt ein **Compiler**. Viele grundlegende Systemprogramme und Bibliotheken sind in C oder C++ geschrieben; auch andere Sprachen greifen beim Bauen oft auf einen C-Compiler zurück.

Ubuntu fasst die nötigen Werkzeuge im Sammelpaket **build-essential** zusammen. Dazu kommen üblicherweise noch **CMake** als Bauwerkzeug und **gdb** zur Fehlersuche:

```bash
# Compiler (gcc/g++), Binder und Standard-Werkzeuge zum Bauen
sudo apt install build-essential

# Bauwerkzeug, das große C/C++-Projekte plattformübergreifend steuert
sudo apt install cmake

# Debugger: hält ein laufendes Programm an und zeigt seinen Zustand
sudo apt install gdb
```

Nach der Installation steht der Compiler als Befehl `gcc` (für C) und `g++` (für C++) bereit. Ein `gcc --version` zeigt die Fassung an.

## Go

Go – auch **Golang** genannt – ist eine Sprache von Google, die auf einfache Sprachregeln und schnelles Übersetzen setzt. Wie bei C entsteht am Ende eine eigenständige Datei, die ohne Laufzeitumgebung läuft. Viele moderne Server- und Kommandozeilenwerkzeuge sind in Go geschrieben.

Die Fassung in den Ubuntu-Paketquellen ist oft mehrere Versionen alt. Für die Entwicklung lädt man Go darum direkt von der offiziellen Seite. Die aktuelle Versionsnummer steht auf `go.dev/dl`; im folgenden Beispiel ist es `1.26.5`:

```bash
cd $HOME

# Eine eventuell vorhandene ältere Installation entfernen
sudo rm -rf /usr/local/go

# Das offizielle Archiv herunterladen
wget https://go.dev/dl/go1.26.5.linux-amd64.tar.gz

# Nach /usr/local entpacken; dort erwartet Go seine Dateien
sudo tar -C /usr/local -xzf go1.26.5.linux-amd64.tar.gz
```

Damit das Terminal den Befehl `go` findet, muss der Ordner mit den Go-Programmen in die Liste der Suchpfade (**PATH**) aufgenommen werden. Diese drei Zeilen ergänzen die persönliche Startdatei `~/.bashrc`, die bei jeder neuen Terminal-Sitzung gelesen wird:

```bash
# Ort des go-Befehls bekannt machen
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc

# GOPATH ist der Ordner, in dem Go heruntergeladene Bausteine und selbst
# gebaute Programme ablegt
echo 'export GOPATH=$HOME/go' >> ~/.bashrc
echo 'export PATH=$PATH:$GOPATH/bin' >> ~/.bashrc

# Die geänderte Startdatei sofort für die laufende Sitzung übernehmen
source ~/.bashrc

# Zeigt die installierte Version zur Kontrolle an
go version
```

## Node.js

JavaScript war ursprünglich nur die Sprache im Webbrowser. **Node.js** löst diese Sprache vom Browser und macht sie zu einer vollwertigen Laufzeitumgebung für Werkzeuge und Server. Das zugehörige Paketwerkzeug heißt **npm**. Sehr viele Entwicklerwerkzeuge – auch solche, die man nur nebenbei benutzt – sind auf Node.js gebaut.

Auf dem Entwicklungs-Rechner verlangen verschiedene Projekte oft verschiedene Node.js-Versionen. Darum installiert man hier nicht das Systempaket, sondern den **Node Version Manager** (`nvm`). Er wird für den eigenen Benutzer eingerichtet, nicht für den ganzen Rechner, und hält mehrere Node.js-Versionen nebeneinander vor:

```bash
# Installationsskript von nvm herunterladen und ausführen
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# nvm in der laufenden Sitzung verfügbar machen (sonst hilft ein neues Anmelden)
\. "$HOME/.nvm/nvm.sh"

# Eine Node.js-Version installieren und benutzen
nvm install 25
nvm use 25

# Zeigt die Versionen von Node.js und npm zur Kontrolle an
node -v
npm -v
```

> **Hinweis:** Ein Skript aus dem Internet direkt auszuführen, sollte man nur bei Quellen tun, denen man vertraut. Wer sichergehen will, ruft die Adresse zuerst im Browser auf, liest den Inhalt und führt ihn erst danach aus.

Für ein Projekt, das lange gepflegt werden soll, wählt man eine Version mit gerader Nummer (24, 26, …); nur diese erhalten über Jahre Sicherheitsupdates. Ungerade Nummern sind kurzlebige Zwischenausgaben.

## Rust

Rust ist eine jüngere hardwarenahe Sprache. Sie erzeugt wie C eigenständige Programme, verhindert aber schon beim Übersetzen eine ganze Klasse von Speicherfehlern. Der Editor Zed aus dem Kapitel [IDE](./ide.md) ist ein Beispiel für ein größeres Programm in Rust.

Rust wird nicht über die Ubuntu-Paketquellen eingerichtet, sondern über das offizielle Verwaltungsprogramm **rustup**. Es installiert den Compiler und das Paketwerkzeug **Cargo** in den persönlichen Ordner – Verwaltungsrechte sind nicht nötig, `sudo` entfällt hier:

```bash
# Installationsskript von rustup herunterladen und ausführen
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Das Skript fragt nach der gewünschten Einrichtung; für den Standard genügt die Eingabetaste. Danach muss das Terminal einmal neu geöffnet oder die Startdatei neu gelesen werden:

```bash
source "$HOME/.cargo/env"

# Zeigt die installierten Versionen zur Kontrolle an
rustc --version
cargo --version
```

Aktualisiert wird Rust später mit `rustup update`.

## .NET

.NET (gesprochen „dot net") ist die Programmierplattform von Microsoft, seit einigen Jahren quelloffen und auch für Linux verfügbar. Damit werden vor allem Webanwendungen und Kommandozeilenwerkzeuge in der Sprache **C#** gebaut.

Zum Entwickeln wird das vollständige **SDK** installiert. Auf Ubuntu 26.04 liegen die .NET-Pakete direkt in den Paketquellen des Systems; eine zusätzliche Paketquelle von Microsoft ist nicht mehr nötig:

```bash
sudo apt update
sudo apt install -y dotnet-sdk-10.0

# Zeigt die installierten Fassungen zur Kontrolle an
dotnet --info
```

Die Zahl `10.0` benennt die Hauptversion. .NET 10 hat eine lange Pflegezusage („LTS") und ist darum die richtige Wahl für ein Projekt, das ruhig laufen soll.

Viele Aufgaben erledigen kleine Zusatzwerkzeuge, die als **globale Werkzeuge** nachinstalliert werden – etwa `dotnet-ef` für Datenbank-Änderungen. Sie landen im Ordner `~/.dotnet/tools`, der ebenfalls in die Liste der Suchpfade aufgenommen werden muss:

```bash
dotnet tool install --global dotnet-ef
dotnet tool install --global dotnet-aspnet-codegenerator
dotnet tool install --global Microsoft.Web.LibraryManager.Cli

# Ordner mit den globalen Werkzeugen bekannt machen und sofort übernehmen
echo 'export PATH=$HOME/.dotnet/tools:$PATH' >> ~/.bashrc
source ~/.bashrc
```

## PHP

PHP ist die Sprache, in der ein großer Teil des Webs geschrieben ist – darunter MediaWiki, die Grundlage der Wissenssammlung dieses Buchs (siehe [Wissenssystem](../web-stack/wissensystem.md)). Wer an solchen Anwendungen arbeitet oder eigene Erweiterungen dafür schreibt, braucht PHP auch auf dem Entwicklungs-Rechner.

Damit sich die Umgebung wie auf dem Server verhält, installiert man dieselben Bestandteile: die Ausführungsart **PHP-FPM** für den Betrieb hinter einem Webserver, die Kommandozeilen-Fassung **php-cli** zum Testen und die üblichen Erweiterungen. Die Paketnamen ohne Versionsnummer ziehen automatisch die Fassung, die Ubuntu mitbringt:

```bash
sudo apt install php-fpm php-cli \
  php-pgsql php-xml php-mbstring php-curl php-gd php-zip php-intl php-xmlrpc php-opcache

# Zeigt die installierte Version zur Kontrolle an
php -v
```

Die einzelnen `php-*`-Pakete sind Erweiterungen: `php-pgsql` verbindet PHP mit der Datenbank PostgreSQL, `php-mbstring` behandelt Texte mit Umlauten korrekt, `php-gd` verkleinert Bilder, und so fort. Welche eine Anwendung erwartet, nennt sie in ihrer Installationsprüfung.

Zum Entwickeln lohnt sich eine Datenbank auf demselben Rechner. PostgreSQL wird mit einem Befehl eingerichtet:

```bash
sudo apt install postgresql
```

Zusätzliche PHP-Bausteine verwaltet das Werkzeug **Composer**. Seine Einrichtung ist im Kapitel [Laufzeitumgebung](../server-einrichten/laufzeitumgebung.md) beschrieben und funktioniert auf dem Entwicklungs-Rechner genauso.

## Kurzvergleich

| Sprache | Paket zum Entwickeln (Ubuntu) | Übersetzt zu | Paketwerkzeug |
| --- | --- | --- | --- |
| Python 3 | `python3` `python3-pip` `python3-venv` | wird direkt ausgeführt | pip (in virtueller Umgebung) |
| Java | `openjdk-26-jdk-headless` | Zwischenform für die Java Virtual Machine | Maven, Gradle |
| C / C++ | `build-essential` `cmake` `gdb` | eigenständige Datei je Rechnertyp | – (oft über CMake) |
| Go | Archiv von `go.dev` | eigenständige Datei je Rechnertyp | go (eingebaut) |
| Node.js | über `nvm` | wird direkt ausgeführt | npm |
| Rust | über `rustup` | eigenständige Datei je Rechnertyp | Cargo |
| .NET (C#) | `dotnet-sdk-10.0` | Zwischenform für die .NET-Laufzeit | NuGet |
| PHP | `php-fpm` `php-cli` + Erweiterungen | wird direkt ausgeführt | Composer |

## Für dieses Buch

Für die Arbeit am Handbuch selbst wird keine dieser Sprachen zwingend gebraucht – die Kapitel sind einfache Markdown-Dateien (siehe [Docs-as-Code](./docs-as-code.md)). Sobald aber ein eigenes Programm oder eine Erweiterung für MediaWiki entsteht, richtet man den passenden Werkzeugkasten ein.

Zwei Grundsätze helfen, den Entwicklungs-Rechner übersichtlich zu halten. Erstens: Versionen, die schnell wechseln – vor allem Node.js, aber auch Go und Rust –, verwaltet man über das jeweilige eigene Werkzeug (`nvm`, die Neuinstallation von Go, `rustup`), nicht über die Systempaketquellen. So lassen sich mehrere Fassungen nebeneinander halten und einzeln aktualisieren. Zweitens: Wer mit **Containern** arbeitet (siehe [Containerisierung von Software](../server-einrichten/containerisierung.md)), kann den Werkzeugkasten einer Sprache auch ganz in einen Container legen und den eigenen Rechner sauber halten.

## Fazit

Auf dem Entwicklungs-Rechner braucht jede Programmiersprache ihren vollständigen Werkzeugkasten: den Übersetzer oder Ausführer, fertige Bausteine und ein Paketwerkzeug. **Python** und **PHP** kommen weitgehend aus den Ubuntu-Paketquellen, **Java**, **C/C++** und **.NET** ebenfalls, wobei man dort das volle Entwicklungspaket (JDK, `build-essential`, SDK) statt der schlanken Laufzeit wählt. **Go** lädt man direkt von der offiziellen Seite, **Node.js** über `nvm` und **Rust** über `rustup` – diese drei, weil sich ihre Versionen häufig ändern und man mehrere Fassungen nebeneinander benötigt. Für den Server gilt das Gegenteil: Dort wird nur die schlanke Laufzeit eingerichtet, wie im Kapitel [Laufzeitumgebung](../server-einrichten/laufzeitumgebung.md) beschrieben.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
