# Claude Desktop

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Claude Desktop ist das Chatprogramm von Anthropic zum Installieren auf dem eigenen Computer. Es bietet dieselben Bereiche wie auf macOS und Windows: den normalen Chat, den Cowork-Modus für Aufgaben, die Claude weitgehend selbstständig abarbeitet (siehe [Desktop-Agenten](./desktop-agenten.md)), sowie einen Code-Bereich mit eingebautem Terminal und Editor. Für Linux gibt es das Programm seit Kurzem als Beta-Version. Beta heißt: Es funktioniert im Alltag schon, einzelne Funktionen fehlen aber noch oder ändern sich.

## Unter Linux installieren (Beta)

### Voraussetzungen

- Ubuntu ab Version 22.04 oder Debian ab Version 12
- Ein Prozessor vom Typ x86_64 (die üblichen Intel- und AMD-Chips) oder arm64

Andere Linux-Varianten, die auf Debian aufbauen und diese Bedingungen erfüllen, laufen möglicherweise ebenfalls, sind von Anthropic aber nicht geprüft. Auf Systemen ohne Debian-Grundlage, etwa Fedora oder Arch Linux, lässt sich Claude Desktop nicht installieren. Dort nutzt man stattdessen die Kommandozeilen-Version Claude Code (siehe [Voraussetzungen](./voraussetzungen.md)).

### Warum über ein Paket-Repository?

Ein Repository ist eine Bezugsquelle für Software, aus der sich das Betriebssystem selbst bedient. Trägt man die Bezugsquelle von Anthropic einmal ein, holt sich Ubuntu neue Versionen von Claude Desktop danach automatisch mit den übrigen Systemaktualisierungen. Man muss also nicht regelmäßig von Hand eine neue Datei herunterladen und darüberinstallieren.

### Schritt 1: Hilfsprogramme bereitstellen

Alle Befehle werden in einem Terminal eingegeben (ein Fenster, in das man Anweisungen als Text tippt). Zwei kleine Programme werden benötigt: `curl` lädt Dateien aus dem Internet, `gnupg` prüft digitale Signaturen. Auf frisch eingerichteten Systemen fehlen sie manchmal. Dieser Befehl installiert beide:

```bash
sudo apt install curl gnupg
```

Das vorangestellte `sudo` führt den Befehl mit Administratorrechten aus; das System fragt dabei nach dem eigenen Passwort.

### Schritt 2: Signaturschlüssel von Anthropic herunterladen

Der Schlüssel dient dem Betriebssystem als Echtheitsnachweis: Damit erkennt es, dass die installierten Pakete tatsächlich von Anthropic stammen und unterwegs nicht verändert wurden.

```bash
sudo curl -fsSLo /usr/share/keyrings/claude-desktop-archive-keyring.asc https://downloads.claude.ai/claude-desktop/key.asc
```

Läuft der Befehl durch, gibt er nichts aus. Zur Kontrolle lässt sich der Fingerabdruck des Schlüssels anzeigen:

```bash
gpg --show-keys /usr/share/keyrings/claude-desktop-archive-keyring.asc
```

Angezeigt werden sollte die Zeichenfolge `31DDDE24DDFAB679F42D7BD2BAA929FF1A7ECACE`. Weicht sie ab oder meldet das Programm, die Datei enthalte keine gültigen Daten, ist der Download fehlgeschlagen. Dann prüft man die Internetverbindung und wiederholt Schritt 2.

### Schritt 3: Bezugsquelle eintragen

Der folgende Befehl schreibt die Adresse des Repositorys in eine Konfigurationsdatei des Systems:

```bash
echo "deb [arch=amd64,arm64 signed-by=/usr/share/keyrings/claude-desktop-archive-keyring.asc] https://downloads.claude.ai/claude-desktop/apt/stable stable main" | sudo tee /etc/apt/sources.list.d/claude-desktop.list
```

### Schritt 4: Claude Desktop installieren

```bash
sudo apt update && sudo apt install claude-desktop
```

`apt update` liest zuerst die neue Bezugsquelle ein, danach lädt `apt install` das Programm und richtet es ein.

### Schritt 5: Starten und anmelden

Claude Desktop erscheint anschließend im Anwendungsmenü. Alternativ startet man es im Terminal mit:

```bash
claude-desktop
```

Beim ersten Start meldet man sich mit einem Anthropic-Konto an, entweder über ein claude.ai-Abo oder über die Anmeldung der eigenen Organisation. Ein API-Schlüssel aus der Claude Console wird in Claude Desktop nicht angenommen; wer sich auf diese Weise anmelden möchte, nutzt Claude Code im Terminal. Das Programm sollte außerdem nicht als Benutzer `root` gestartet werden, sondern als das normale eigene Benutzerkonto.

## Aktualisieren

Unter Linux aktualisiert sich Claude Desktop nicht von selbst. Neue Versionen kommen mit den übrigen Systemaktualisierungen:

```bash
sudo apt update && sudo apt upgrade
```

Die grafische Aktualisierungsverwaltung der jeweiligen Linux-Variante zeigt neue Versionen ebenfalls an.

## Entfernen

```bash
sudo apt remove claude-desktop
```

Damit werden auch der Repository-Eintrag und der Signaturschlüssel wieder gelöscht.

## Was in der Linux-Beta noch fehlt

Einige Funktionen der Windows- und macOS-Version sind unter Linux noch nicht enthalten, darunter die Steuerung von Maus und Bildschirm durch Claude sowie die Spracheingabe. Für den Cowork-Modus muss zusätzlich die Hardware-Virtualisierung des Rechners eingeschaltet sein; fehlt sie, weist die App im Cowork-Bereich darauf hin.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
