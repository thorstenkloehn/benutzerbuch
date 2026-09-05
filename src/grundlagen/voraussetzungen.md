# Voraussetzungen

Bekannte Coding-Tools dieser Art kommen aktuell vor allem von drei großen KI-Firmen: Anthropic mit Claude Code, OpenAI mit Codex CLI und Google mit Antigravity CLI. Alle drei funktionieren nach einem ähnlichen Grundprinzip, unterscheiden sich aber im verwendeten KI-Modell. Dieses Kapitel konzentriert sich auf Claude Code.

## Installation

Bevor man eines der drei Tools benutzen kann, muss man es zuerst auf dem eigenen Computer installieren. Dafür öffnet man ein Terminal (ein Fenster, in das man Befehle als Text eingibt) und gibt einen einzigen Befehl ein, der das Programm herunterlädt und einrichtet.

### Claude Code (Anthropic)

Unter macOS, Linux oder in WSL (einer Linux-Umgebung innerhalb von Windows) genügt dieser Befehl:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Unter Windows in der PowerShell lautet er:

```powershell
irm https://claude.ai/install.ps1 | iex
```

Danach startet man das Programm mit dem Befehl `claude` und meldet sich beim ersten Start im Browser mit einem Claude-Account an (Pro, Max, Team oder Enterprise; der kostenlose Claude.ai-Zugang reicht dafür nicht aus).

### Codex CLI (OpenAI)

Codex CLI installiert man ebenfalls mit einem Befehl im Terminal:

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

Alternativ geht es auch über npm (ein Paketverwalter für JavaScript-Programme, der zusammen mit Node.js installiert wird):

```bash
npm install -g @openai/codex
```

Gestartet wird Codex CLI mit dem Befehl `codex`. Beim ersten Start meldet man sich entweder mit einem ChatGPT-Account (Plus, Pro, Business, Edu oder Enterprise) oder mit einem OpenAI-API-Schlüssel an.

### Antigravity CLI (Google)

Unter macOS und Linux installiert man Antigravity CLI mit:

```bash
curl -fsSL https://antigravity.google/cli/install.sh | bash
```

Unter Windows in der PowerShell:

```powershell
irm https://antigravity.google/cli/install.ps1 | iex
```

Gestartet wird das Programm mit dem Befehl `agy`. Beim ersten Start meldet man sich mit einem Google-Konto an oder benutzt stattdessen einen Gemini-API-Schlüssel.

### Hinweis zur Installation

Alle drei Befehle laden ein Skript herunter und führen es direkt aus. Das ist bei allen drei großen Anbietern so üblich, setzt aber Vertrauen in die Quelle voraus. Wer das Skript vorher ansehen möchte, kann es zuerst herunterladen und öffnen, statt es sofort auszuführen.

Claude Code lässt sich außerdem auf unterschiedliche Arten benutzen. Welche davon die passende ist, hängt davon ab, wie man am liebsten arbeitet.

## Direkt im Terminal

Die ursprüngliche und flexibelste Art ist, Claude Code direkt im Terminal zu benutzen (einem Fenster, in dem man Befehle als Text eingibt statt mit der Maus zu klicken). Dabei hat man die volle Kontrolle über den eigenen Code: Jeder Befehl und jede Änderung lässt sich genau nachvollziehen. Der Einstieg ist dafür etwas anspruchsvoller als bei den anderen Möglichkeiten, weil man sich zunächst an die Arbeit im Terminal gewöhnen muss.

## In Visual Studio Code

Wer lieber in einer gewohnten Programmierumgebung bleibt, kann Claude Code auch über die Erweiterung für Visual Studio Code benutzen (ein weit verbreitetes, kostenloses Programm zum Schreiben von Code). Claude Code fügt sich dort gut ein und lässt sich bedienen, ohne dass man das Terminal einzeln öffnen muss. Das macht den Einstieg leichter, vor allem für alle, die Visual Studio Code bereits kennen.

## Über die Claude-Desktop-App

Eine dritte Möglichkeit ist die Claude-Desktop-App, das allgemeine Chatprogramm von Anthropic. Sie eignet sich gut für allgemeine Aufgaben und Unterhaltungen mit Claude, ist für reine Programmieraufgaben aber weniger gut geeignet als Claude Code direkt im Terminal oder die Erweiterung für Visual Studio Code.

## Fazit

Für den Einstieg eignet sich die Erweiterung für Visual Studio Code am besten, wenn man bereits mit diesem Programm arbeitet. Wer die volle Kontrolle über jeden einzelnen Schritt haben möchte, benutzt Claude Code direkt im Terminal. Die Claude-Desktop-App ist dagegen eher für allgemeine Aufgaben gedacht als fürs Programmieren.
