# Desktop-Agenten

Ein Desktop-Agent ist ein Programm, das nicht nur im Chat antwortet, sondern selbstständig Aufgaben auf dem eigenen Computer erledigt: Dateien lesen, anlegen oder ändern, im Browser klicken oder mehrere Schritte hintereinander ausführen, ohne dass man jeden einzelnen davon selbst anstößt. Anders als bei Claude Code, Codex CLI oder Antigravity CLI (siehe [Voraussetzungen](./voraussetzungen.md)) läuft ein Desktop-Agent nicht im Terminal, sondern in einer eigenen grafischen Anwendung mit Fenstern und Schaltflächen.

Drei bekannte Beispiele dafür sind aktuell:

- **Claude Cowork** (Anthropic)
- **ChatGPT Agent** (OpenAI)
- **Antigravity 2.0** (Google)

## Claude Cowork (Anthropic)

Claude Cowork ist ein Modus innerhalb der Claude-Desktop-App, neben den Modi Chat und Code. Man wählt einen Ordner auf dem eigenen Computer aus, und Claude arbeitet darin: Es liest vorhandene Dateien, legt neue an und ändert sie, während es eine Aufgabe abarbeitet — mit möglichst wenig Rückfragen zwischendurch. Cowork richtet sich damit vor allem an Anwender ohne Programmiererfahrung, die von den Fähigkeiten profitieren möchten, die Claude Code bisher nur Entwicklern bot. Die Claude-Desktop-App mit Cowork gibt es für macOS und Windows, ab einem bezahlten Claude-Plan.

## ChatGPT Agent (OpenAI)

ChatGPT Agent ist Teil der ChatGPT-Desktop-App und bündelt dort mehrere Bereiche in einem Fenster: Chat für normale Unterhaltungen, Work für Aufgaben im Alltag und Codex für Programmieraufgaben mit lokalen Dateien. Im Work-Bereich kann der Agent über eine Funktion namens „Computer Use" den Computer im Hintergrund bedienen: klicken, tippen und Dateien verschieben, ähnlich wie ein Mensch es tun würde. Über Erweiterungen lässt sich ChatGPT Agent außerdem mit Diensten wie E-Mail, Kalender oder Cloud-Speicher verbinden, um Aufgaben über mehrere Programme hinweg zu erledigen.

## Antigravity 2.0 (Google)

Antigravity 2.0 ist eine eigenständige Desktop-Anwendung von Google für macOS, Linux und Windows. Sie dient als zentrale Steuerung für mehrere KI-Agenten gleichzeitig: Man kann darin verschiedene Agenten starten, ihren Fortschritt beobachten und auch Aufgaben einplanen, die zu einem späteren Zeitpunkt automatisch laufen.

### Antigravity 2.0 installieren

Auf Linux gibt es bei der Installation eine Besonderheit, die man kennen sollte: Die Anwendung bringt eine Sandbox-Komponente namens `chrome-sandbox` mit, die aus Sicherheitsgründen bestimmte Rechte braucht, um Browser-Funktionen nutzen zu können. Ohne diese Rechte startet Antigravity entweder gar nicht oder nur eingeschränkt.

Der Datei muss dafür der Benutzer `root` als Besitzer zugewiesen werden, und sie braucht die Rechte `4755` (das sogenannte Setuid-Bit, das der Datei erlaubt, kurzzeitig mit Root-Rechten zu laufen). Dazu wechselt man im Terminal in den entpackten Programmordner und führt folgende Befehle aus:

```bash
cd /home/thorsten/Downloads/Antigravity/Antigravity-x64
sudo chown root:root chrome-sandbox
sudo chmod 4755 chrome-sandbox
./antigravity
```

Diese Schritte sind nur unter Linux nötig. Unter macOS und Windows lässt sich Antigravity 2.0 wie gewohnt installieren und direkt starten.

## Fazit

Neben diesen drei Beispielen gibt es bereits weitere Desktop-Agenten, und es werden mit großer Sicherheit noch mehr dazukommen — dieses Handbuch kann und will keine vollständige, ständig aktuelle Liste davon sein. Wichtiger als die Anzahl der verfügbaren Werkzeuge ist ein Grundsatz, der für alle gilt: Ein Desktop-Agent kann viele Schritte selbstständig ausführen, doch die Kontrolle über wichtige Entscheidungen sollte am Ende immer beim Menschen bleiben, etwa beim Aufbau einer Webseite oder einer Anwendung. Wer Ergebnisse ungeprüft übernimmt, überlässt der KI mehr, als ihr gut tut. Ein Sprachmodell wird außerdem nicht von selbst besser, nur weil man es häufiger benutzt — es hilft, die eigenen Grundlagen zu verstehen, statt sich blind auf Vorschläge zu verlassen. Die folgenden Kapitel zeigen deshalb, wie man mit einem Desktop-Agenten gezielt Wissen aufbaut und sammelt, statt ihm die Arbeit einfach vollständig zu überlassen.
