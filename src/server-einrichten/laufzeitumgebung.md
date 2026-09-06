# Laufzeitumgebung

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Jedes Programm ist in einer Programmiersprache geschrieben. Manche Programme werden vor der Auslieferung in eine fertige, eigenständige Datei übersetzt und laufen dann allein – Meilisearch und Ollama aus dem Kapitel [Inhalts-Software selbst betreiben](../grundlagen/inhalt-software-verwalten.md) sind solche Fälle. Andere Programme brauchen dagegen ein Hilfsprogramm, das ihren Code erst auf dem Server ausführt. Dieses Hilfsprogramm heißt **Laufzeitumgebung**.

MediaWiki zum Beispiel besteht aus vielen Dateien mit PHP-Code. Ohne ein PHP-Programm auf dem Server passiert damit nichts. Genauso braucht das Automatisierungswerkzeug n8n aus dem Kapitel [KI-Agenten auf dem Server](./ki-agent.md) die Laufzeitumgebung Node.js, und viele kleine Zusatzwerkzeuge im KI-Umfeld sind in Python geschrieben.

Dieses Kapitel erklärt, was eine Laufzeitumgebung ist, stellt die sechs verbreitetsten vor und zeigt für jede, wie man sie auf einem Server mit **Ubuntu 26.04 LTS** (siehe [Betriebssystem](./betriebssystem.md)) einrichtet. Alle Befehle werden in der Textkonsole des Servers eingegeben. Das vorangestellte `sudo` bedeutet: mit Verwaltungsrechten ausführen.

## Was eine Laufzeitumgebung ist

Ein Kochrezept ist nur ein Blatt Papier, solange niemand in der Küche steht und es umsetzt. Die Laufzeitumgebung ist diese Küche samt Koch: Sie liest die Anweisungen des Programms Zeile für Zeile und führt sie aus.

Zur Laufzeitumgebung gehören meist drei Dinge:

- **Der Ausführer** selbst – das Programm, das den Code liest und abarbeitet.
- **Eine Sammlung fertiger Bausteine** („Standardbibliothek"), damit nicht jedes Programm das Rechnen mit Datum, Text oder Netzwerk neu erfinden muss.
- **Ein Paketwerkzeug**, mit dem ein Programm weitere Bausteine aus dem Internet nachlädt, die es benötigt.

Wichtig ist der Unterschied zwischen der vollständigen Entwicklungsausstattung und der reinen Laufzeit. Wer selbst Software *schreibt*, braucht das komplette Paket (bei Java „JDK" genannt, bei .NET „SDK"). Wer eine fertige Anwendung nur *betreiben* will, kommt mit der abgespeckten Laufzeit aus (bei Java „JRE", bei .NET „Runtime"). Die Laufzeit ist kleiner, schneller eingerichtet und bietet weniger Angriffsfläche. Für einen Server, auf dem fremde Programme nur laufen sollen, ist sie die richtige Wahl.

## Java

Java ist eine der ältesten noch weit verbreiteten Programmiersprachen für Server. Ihr Kennzeichen: Java-Code wird nicht für einen bestimmten Rechnertyp übersetzt, sondern in eine Zwischenform. Diese Zwischenform führt auf jedem Betriebssystem dieselbe Laufzeitumgebung aus, die **Java Virtual Machine**. Dadurch läuft dasselbe Java-Programm unverändert auf Windows, Linux und macOS.

Die freie Ausgabe von Java heißt **OpenJDK**. Im Umfeld einer Wissenssammlung begegnet Java vor allem bei Suchservern wie Apache Solr oder OpenSearch – Alternativen zu Meilisearch, die manche Anleitungen verwenden. Auch etliche Werkzeuge zur Datenverarbeitung setzen Java voraus.

Ubuntu 26.04 liefert OpenJDK 25 als Standardversion mit. Für den reinen Betrieb genügt die kopflose Laufzeit ohne grafische Bestandteile:

```bash
sudo apt update

# Installiert die Java-Laufzeitumgebung ohne grafische Bestandteile
sudo apt install openjdk-25-jre-headless

# Zeigt die installierte Version zur Kontrolle an
java -version
```

Wer Java-Software auch selbst übersetzen muss, ersetzt `openjdk-25-jre-headless` durch `openjdk-25-jdk-headless`.

## Python

Python ist eine Sprache, die auf gut lesbaren Code setzt und darum oft für kurze Hilfsprogramme, Auswertungen und Automatisierung genommen wird. Im KI-Umfeld ist Python die vorherrschende Sprache: Werkzeuge, die Texte für die Bedeutungssuche aufbereiten, Modelle testen oder Daten zwischen Diensten umschaufeln, sind fast immer in Python geschrieben.

Ubuntu bringt Python 3 bereits mit, allerdings ohne die üblichen Zusatzteile. Diese werden nachinstalliert:

```bash
# Python 3 mit Paketwerkzeug (pip) und Umgebungsverwaltung (venv);
# python-is-python3 sorgt dafür, dass der Befehl "python" Python 3 startet
sudo apt install python3 python3-pip python3-venv python-is-python3
```

Eine Besonderheit von Python sollte man kennen: Zusätzliche Bausteine werden nicht global installiert, sondern in einer abgetrennten Umgebung je Programm, einer sogenannten **virtuellen Umgebung**. Das verhindert, dass zwei Programme sich mit unterschiedlichen Versionen desselben Bausteins ins Gehege kommen. Angelegt wird eine solche Umgebung mit:

```bash
python -m venv .venv
source .venv/bin/activate
```

Neuere Ubuntu-Ausgaben bestehen sogar darauf: Ein direktes `pip install` außerhalb einer virtuellen Umgebung wird abgewiesen.

## ASP.NET Core Runtime

.NET (gesprochen „dot net") ist die Programmierplattform von Microsoft, seit einigen Jahren quelloffen und auch für Linux verfügbar. **ASP.NET Core** ist der Teil davon, mit dem Webanwendungen gebaut werden. Die **ASP.NET Core Runtime** ist die passende Laufzeitumgebung, um solche Webanwendungen zu betreiben; sie enthält die allgemeine .NET-Laufzeit bereits.

Im Umfeld dieses Buchs spielt .NET eine Nebenrolle, taucht aber bei einzelnen selbst betriebenen Zusatzdiensten auf – etwa bei manchen Verwaltungsoberflächen oder Schnittstellenprogrammen.

Auf Ubuntu 26.04 liegen die .NET-Pakete direkt in den Paketquellen des Systems; eine zusätzliche Paketquelle von Microsoft ist nicht mehr nötig:

```bash
sudo apt-get update
sudo apt-get install -y aspnetcore-runtime-10.0

# Zeigt die installierten Laufzeiten an
dotnet --list-runtimes
```

Die Zahl `10.0` benennt die Hauptversion. .NET 10 ist eine Ausgabe mit langer Pflegezusage („LTS") und darum die richtige Wahl für einen Server, der ruhig laufen soll.

## Node.js

JavaScript war ursprünglich nur die Sprache im Webbrowser. **Node.js** löst diese Sprache vom Browser und macht sie zu einer vollwertigen Server-Laufzeitumgebung. Sehr viele moderne Werkzeuge sind darauf gebaut – im Umfeld dieses Buchs vor allem das Automatisierungswerkzeug n8n, aber auch zahlreiche Kommandozeilenhelfer.

Das zugehörige Paketwerkzeug heißt **npm**. Damit lädt ein Node.js-Programm die Bausteine nach, die es braucht.

### Der einfache Weg: das Systempaket

Für einen einzelnen Dienst genügt die Fassung aus den Ubuntu-Paketquellen:

```bash
sudo apt install nodejs npm
node -v
```

### Der flexible Weg: nvm

Manche Programme verlangen eine ganz bestimmte Node.js-Version, und diese Versionen wechseln schnell. Dafür gibt es den **Node Version Manager** (`nvm`). Er wird für einen einzelnen Benutzer installiert, nicht systemweit, und kann mehrere Node.js-Versionen nebeneinander vorhalten:

```bash
# Installationsskript von nvm herunterladen und ausführen
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.7/install.sh | bash

# nvm in der laufenden Sitzung verfügbar machen (sonst hilft ein neues Anmelden)
\. "$HOME/.nvm/nvm.sh"

# Eine Node.js-Version installieren und benutzen
nvm install 24
nvm use 24
```

> **Hinweis:** Ein Skript aus dem Internet direkt auszuführen, sollte man nur bei Quellen tun, denen man vertraut. Wer sichergehen will, lädt das Skript zuerst herunter, liest es und führt es dann aus.

Für den Server empfiehlt sich eine Version mit gerader Nummer (22, 24, …); nur diese erhalten über Jahre Sicherheitsupdates. Ungerade Nummern sind kurzlebige Zwischenausgaben.

## PHP

PHP ist die Sprache, in der ein großer Teil des Webs geschrieben ist – darunter MediaWiki, die Grundlage der Wissenssammlung dieses Buchs. Ohne eine PHP-Laufzeitumgebung auf dem Server zeigt MediaWiki keine einzige Seite an.

Für den Betrieb hinter einem Webserver nimmt man die Ausführungsart **PHP-FPM** („FastCGI Process Manager"). Sie hält PHP als Hintergrunddienst bereit, an den der Webserver die Anfragen weiterreicht. Ubuntu 26.04 liefert PHP 8.5 mit. Die allgemeinen Paketnamen ohne Versionsnummer ziehen automatisch diese Fassung:

```bash
sudo apt install php-fpm php-cli \
  php-pgsql php-xml php-mbstring php-curl php-gd php-zip php-intl php-opcache

# Zeigt die installierte Version zur Kontrolle an
php -v
```

Die einzelnen `php-*`-Pakete sind Erweiterungen: `php-pgsql` verbindet PHP mit der Datenbank PostgreSQL, `php-mbstring` behandelt Texte mit Umlauten und anderen Sonderzeichen korrekt, `php-gd` verkleinert hochgeladene Bilder, und so fort. MediaWiki nennt in seiner Installationsprüfung genau, welche davon es erwartet.

### Composer

Zusätzliche PHP-Bausteine verwaltet das Werkzeug **Composer**. Es liegt zwar auch in den Paketquellen, dort aber oft veraltet. Die offizielle Anleitung lädt es darum direkt herunter und prüft dabei über eine Prüfsumme, dass die Datei unterwegs nicht verändert wurde:

```bash
# Installationsprogramm von Composer herunterladen
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

# Erwartete Prüfsumme laden und die heruntergeladene Datei damit vergleichen
EXPECTED_HASH="$(curl -sS https://composer.github.io/installer.sig)"
php -r "if (hash_file('sha384', 'composer-setup.php') === '$EXPECTED_HASH') { echo 'Installationsprogramm bestätigt' . PHP_EOL; } else { echo 'Installationsprogramm beschädigt' . PHP_EOL; unlink('composer-setup.php'); exit(1); }"

# Composer systemweit als Befehl "composer" einrichten und Installationsprogramm entfernen
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
php -r "unlink('composer-setup.php');"
```

## Lua

Lua ist eine sehr kleine, schnelle Sprache. Sie wird selten allein benutzt, sondern meist in ein größeres Programm eingebettet, um dieses erweiterbar zu machen. Der Zwischenspeicher Redis führt kleine Lua-Programme aus, um mehrere Schritte in einem Rutsch zu erledigen; der Webserver nginx lässt sich mit Lua um eigene Logik ergänzen; auch manche Konfigurationsdateien sind in Wahrheit Lua.

Für die Wissenssammlung dieses Buchs ist Lua kein Pflichtbestandteil. Wird es doch einmal gebraucht, genügt:

```bash
# Lua-Ausführer und das Paketwerkzeug luarocks
sudo apt install lua5.4 luarocks
lua5.4 -v
```

## Kurzvergleich

| Laufzeitumgebung | Paket für den Betrieb (Ubuntu 26.04) | Paketwerkzeug | In diesem Buch |
| --- | --- | --- | --- |
| Java (OpenJDK 25) | `openjdk-25-jre-headless` | Maven, Gradle | nur bei Suchservern wie Solr/OpenSearch |
| Python 3 | `python3` `python3-pip` `python3-venv` | pip (in virtueller Umgebung) | viele KI-Hilfswerkzeuge |
| ASP.NET Core Runtime | `aspnetcore-runtime-10.0` | NuGet | einzelne Zusatzdienste |
| Node.js | `nodejs` `npm` oder `nvm` | npm | n8n und Kommandozeilenhelfer |
| PHP 8.5 | `php-fpm` `php-cli` + Erweiterungen | Composer | Pflicht für MediaWiki |
| Lua 5.4 | `lua5.4` | luarocks | eingebettet in Redis, nginx |

## Für dieses Buch

Von den sechs Laufzeitumgebungen ist für den beschriebenen Aufbau nur eine wirklich unverzichtbar: **PHP** für MediaWiki. Wer zusätzlich einen KI-Agenten auf dem Server betreibt, kommt um **Node.js** (für n8n) und meist um **Python** (für kleinere Helfer) nicht herum. **Java**, **.NET** und **Lua** braucht man nur, wenn ein bestimmtes Zusatzprogramm sie voraussetzt – dann sagt dessen Anleitung das ausdrücklich.

Ein Grundsatz gilt für alle: auf dem Server nur die Laufzeit installieren, nicht die volle Entwicklungsausstattung, und keine Version aufspielen, für die es keine Sicherheitsupdates mehr gibt. Wer Programme in Containern betreibt (siehe [Containerisierung von Software](./containerisierung.md)), muss sich um die Laufzeitumgebungen gar nicht kümmern: Sie stecken dann bereits im jeweiligen Container.

## Fazit

Eine Laufzeitumgebung ist das Hilfsprogramm, das den Code einer Anwendung auf dem Server ausführt. **PHP** ist für MediaWiki zwingend, **Node.js** und **Python** kommen mit KI-Agenten dazu, **Java**, **.NET** und **Lua** nur bei Bedarf. Auf Ubuntu 26.04 lässt sich jede davon mit einem einzigen `apt`-Befehl einrichten; nur bei Node.js lohnt mit `nvm` ein zweiter Weg, wenn ein Programm eine feste Version verlangt. Für einen Server gilt: die schlanke Laufzeit statt der vollen Entwicklungsausstattung wählen und nur gepflegte Versionen betreiben.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
