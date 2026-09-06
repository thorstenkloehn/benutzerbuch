# KI-Agenten auf dem Server

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Ein KI-Agent ist ein Programm, das eine Aufgabe nicht nur beantwortet, sondern selbstständig ausführt: mehrere Schritte hintereinander, mit Werkzeugen, die es selbst bedient. Das Kapitel [Desktop-Agenten](../grundlagen/desktop-agenten.md) beschreibt solche Programme für den eigenen Computer. Dieses Kapitel geht einen Schritt weiter: Der Agent läuft dann nicht mehr auf dem Arbeitsplatzrechner, sondern rund um die Uhr auf dem gemieteten Server (siehe [Server mieten](./server-mieten.md)).

Das ist nützlich, wenn ein Agent Dinge tun soll, während niemand am Rechner sitzt: neue Seiten der Wissenssammlung nachts prüfen, auf eingehende Nachrichten reagieren, regelmäßig Daten zusammentragen. Damit die Inhalte dabei im eigenen Haus bleiben, arbeiten alle hier vorgestellten Programme mit **Ollama** zusammen – dem Dienst, der Sprachmodelle auf dem eigenen Rechner ausführt (siehe [Inhalts-Software selbst betreiben](../grundlagen/inhalt-software-verwalten.md)).

Dieses Kapitel stellt drei Programme vor – **n8n**, **OpenClaw** und **Hermes Agent** –, richtet einen Ubuntu-Server dafür ein und zeigt, wie jedes von ihnen mit einem lokalen Sprachmodell verbunden wird.

## Die drei Programme im Überblick

| Programm | Grundidee | Bedienung |
| --- | --- | --- |
| n8n | Abläufe als Baukasten aus verbundenen Kästchen | Weboberfläche im Browser |
| OpenClaw | ein Assistent, den man über Messenger anspricht | Chat-App (Signal, Telegram …) plus Textkonsole |
| Hermes Agent | ein Agent für die Textkonsole, der dazulernt | Textkonsole, zusätzlich über Messenger erreichbar |

Alle drei sind quelloffen und dürfen kostenlos selbst betrieben werden. Sie schließen sich nicht aus – man kann mit einem beginnen und die anderen später daneben stellen.

### n8n – Abläufe zusammenstecken

n8n ist ein Werkzeug für **Automatisierung**. Man baut einen Ablauf (dort „Workflow" genannt) aus einzelnen Bausteinen zusammen, die wie Kästchen auf einer Fläche liegen und mit Linien verbunden werden. Ein Baustein holt zum Beispiel eine neue Wiki-Seite ab, der nächste schickt ihren Text an ein Sprachmodell, der übernächste trägt das Ergebnis irgendwo ein. Für Hunderte Dienste gibt es fertige Bausteine; ein eigener Baustein „AI Agent" lässt das Modell selbst entscheiden, welche Werkzeuge es in welcher Reihenfolge benutzt.

n8n eignet sich, wenn der Ablauf klar umrissen ist und immer gleich abläuft. Die Weboberfläche macht sichtbar, was in welchem Schritt passiert, und das erleichtert die Fehlersuche. n8n steht unter einer quelloffenen Lizenz (der „Sustainable Use License"): Der Quelltext liegt offen, und der Betrieb für eigene Zwecke ist kostenlos; nur das Weiterverkaufen als eigener Dienst ist eingeschränkt.

### OpenClaw – der Assistent im Messenger

OpenClaw ist ein selbst betriebener Assistent, den man über gewohnte Chat-Programme anspricht – etwa Signal, Telegram, Slack oder WhatsApp. Auf dem Server läuft ein Vermittler (im Projekt „Gateway" genannt), der die Nachrichten entgegennimmt, an ein Sprachmodell weiterreicht und die Antworten zurückschickt. Der Agent kann dabei auf dem Server Befehle ausführen, im Browser klicken, Dateien bearbeiten oder E-Mails schreiben.

OpenClaw eignet sich, wenn man den Agenten wie eine Person im Chat behandeln möchte: kurze Anweisung schicken, Ergebnis abwarten. Das Projekt ist noch jung, entwickelt sich schnell und wird von einer gemeinnützigen Stiftung offen gepflegt. Es steht unter der MIT-Lizenz, einer der freizügigsten Lizenzen überhaupt.

Ein Hinweis zur Vorsicht: Ein Agent, der Nachrichten aus einem Messenger annimmt und daraufhin Befehle auf dem Server ausführt, ist eine mächtige und zugleich heikle Kombination. Er sollte nur mit den nötigsten Rechten laufen und nicht auf demselben Server wie die öffentliche Wissenssammlung.

### Hermes Agent – der lernende Agent in der Konsole

Hermes Agent stammt von der Forschungsgruppe Nous Research und läuft in der **Textkonsole**. Er führt Aufgaben mit über 40 Werkzeugen aus, kann Teilaufgaben an Unter-Agenten abgeben und Abläufe zu festen Zeiten starten. Die Besonderheit ist eine eingebaute Lernschleife: Aus erledigten Aufgaben legt der Agent kleine wiederverwendbare „Skills" an und erinnert sich über Sitzungen hinweg an frühere Gespräche.

Hermes Agent eignet sich für alle, die ohnehin auf dem Server über die Textkonsole arbeiten und einen Helfer für wiederkehrende Handgriffe suchen. Wie OpenClaw kann er über einen eingebauten Zugang zusätzlich aus Messengern angesprochen werden. Er steht unter der MIT-Lizenz und arbeitet mit lokalen Modellen über Ollama, LM Studio oder vergleichbare Dienste.

## Den Server vorbereiten

Grundlage ist ein Server mit **Ubuntu 26.04 LTS** (siehe [Betriebssystem](./betriebssystem.md)). Alle Befehle werden in der Textkonsole des Servers eingegeben. `sudo` am Anfang bedeutet: mit Verwaltungsrechten ausführen.

### Schritt 1: Das System aktualisieren

```bash
sudo apt update && sudo apt upgrade -y
```

### Schritt 2: Docker einrichten

n8n wird am einfachsten als Container betrieben, OpenClaw und Hermes Agent lassen sich ebenfalls in einen Container packen. Die Einrichtung von Docker beschreibt das Kapitel [Containerisierung von Software](./containerisierung.md) Schritt für Schritt. Kurz zusammengefasst:

```bash
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

### Schritt 3: Ollama einrichten

Ollama stellt die Sprachmodelle bereit, die alle drei Agenten nutzen. Es wird mit einem Befehl installiert:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Danach lädt man ein Modell herunter, zum Beispiel ein kleineres, das auch ohne starke Grafikkarte läuft:

```bash
ollama pull llama3.1:8b
```

Ollama lauscht danach auf dem eigenen Rechner unter der Adresse `http://localhost:11434`. Läuft ein Agent in einem Docker-Container, erreicht er Ollama nicht über `localhost`, sondern über die besondere Adresse `http://host.docker.internal:11434`. Diese muss dem Container beim Start mit dem Zusatz `--add-host=host.docker.internal:host-gateway` bekannt gemacht werden.

Für den echten Betrieb („Produktionsserver") gelten zwei Regeln:

- **Ollama nicht ins Internet öffnen.** Der Dienst hat keine Anmeldung. Er darf nur vom Server selbst und von den Containern darauf erreichbar sein, niemals von außen. Die Firewall des Anbieters sollte den Port `11434` von außen sperren.
- **Modellgröße zur Hardware wählen.** Ein Modell mit 8 Milliarden Parametern (Kurzform „8b") läuft auf bescheidener Hardware, antwortet aber langsamer. Für flüssige Antworten braucht der Server eine Grafikkarte mit ausreichend Speicher. Ohne Grafikkarte bleibt man besser bei kleinen Modellen und plant längere Wartezeiten ein.

## n8n einrichten

n8n wird als einzelner Container gestartet. Zuerst wird ein dauerhafter Speicherbereich angelegt, damit die Abläufe einen Neustart überstehen:

```bash
docker volume create n8n_data

docker run -d --restart unless-stopped --name n8n \
  -p 127.0.0.1:5678:5678 \
  --add-host=host.docker.internal:host-gateway \
  -v n8n_data:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
```

Der Zusatz `-p 127.0.0.1:5678:5678` sorgt dafür, dass die Weboberfläche nur vom Server selbst erreichbar ist. Für den Zugriff über das Internet stellt man einen Webserver mit Verschlüsselung und Passwortschutz davor; wie das geht, gehört in ein eigenes Kapitel.

Die Oberfläche ist danach unter `http://localhost:5678` erreichbar (bei Fernzugriff über einen verschlüsselten Tunnel). Beim ersten Aufruf legt man ein Benutzerkonto an.

### n8n mit Ollama verbinden

In einem Ablauf fügt man einen Baustein vom Typ **AI Agent** ein und weist ihm ein Sprachmodell zu. Als Modell wählt man **Ollama Chat Model** und trägt bei den Zugangsdaten die Adresse `http://host.docker.internal:11434` ein. Anschließend lässt sich der gewünschte Modellname (etwa `llama3.1:8b`) auswählen. Ab jetzt bearbeitet der Baustein jede Anfrage mit dem lokalen Modell.

## OpenClaw einrichten

OpenClaw wird mit einem Skript installiert. Das Projekt entwickelt sich schnell – die aktuelle Anleitung steht unter `docs.openclaw.ai`. Zum Zeitpunkt der Erstellung dieses Kapitels lautet der Weg:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw onboard --install-daemon
```

Der Befehl `onboard` führt durch die Einrichtung: Man wählt einen Messenger aus, hinterlegt die Zugangsdaten dafür und legt fest, welches Sprachmodell benutzt wird. Mit `--install-daemon` richtet OpenClaw sich als Hintergrunddienst ein, der nach jedem Server-Neustart von selbst wieder startet.

> **Wichtig:** Skripte aus dem Internet direkt auszuführen, sollte man nur bei Quellen tun, denen man vertraut. Wer sichergehen will, lädt das Skript zuerst herunter, liest es und führt es dann aus. Alternativ bietet das Projekt eine `docker-compose.yml` an, mit der OpenClaw in einem Container läuft.

### OpenClaw mit Ollama verbinden

Bei der Einrichtung – oder später in der Einstellungsdatei von OpenClaw – wählt man als Modellanbieter einen lokalen Dienst und trägt die Ollama-Adresse ein: `http://localhost:11434` bei direkter Installation, `http://host.docker.internal:11434` im Container. Als Modell wird der Name des mit `ollama pull` geladenen Modells angegeben. Danach laufen alle Anfragen über den eigenen Server, ohne dass Nachrichten an einen fremden Anbieter gehen.

## Hermes Agent einrichten

Auch Hermes Agent wird per Skript installiert:

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

Das Installationsprogramm bringt die nötigen Hilfsprogramme (unter anderem Python und Node.js) selbst mit. Gestartet wird der Agent danach mit dem Befehl `hermes` in der Textkonsole. Beim ersten Start fragt er nach dem Modellanbieter.

Hinweis: Läuft die Verbindung zum Server über SSH, endet ein per Konsole gestarteter Agent, sobald die Verbindung getrennt wird. Damit er weiterläuft, startet man ihn in einer Sitzungsverwaltung wie `tmux` oder `screen` oder richtet ihn als Dienst ein. Der gleiche Sicherheitshinweis wie bei OpenClaw gilt auch hier.

### Hermes Agent mit Ollama verbinden

In der Einrichtung wählt man einen lokalen, OpenAI-kompatiblen Anbieter und trägt als Adresse die Ollama-Schnittstelle ein: `http://localhost:11434/v1`. Als Modell wird der geladene Ollama-Modellname eingetragen. Ollama bietet neben seiner eigenen auch eine Schnittstelle im OpenAI-Format an; darüber sprechen die meisten Agenten mit lokalen Modellen.

## Welches Programm wofür

| Aufgabe | Passendes Programm |
| --- | --- |
| Fester, wiederkehrender Ablauf mit vielen Diensten | n8n |
| Agent wie ein Chat-Kontakt per Messenger bedienen | OpenClaw |
| Helfer in der Textkonsole für Server-Handgriffe | Hermes Agent |
| Nur ausprobieren, ohne den Server zu verändern | jedes davon als Docker-Container |

## Für dieses Buch

Für die Wissenssammlung dieses Buchs ist ein KI-Agent auf dem Server kein Pflichtbestandteil. Er wird interessant, sobald wiederkehrende Aufgaben anfallen: neue Seiten automatisch einordnen, Zusammenfassungen erzeugen, die Bedeutungssuche mit frischen Zahlenreihen versorgen. Für solche festen Abläufe ist **n8n** der übersichtlichste Einstieg, weil jeder Schritt sichtbar bleibt. **OpenClaw** und **Hermes Agent** sind die richtige Wahl, wenn man den Agenten frei anweisen möchte statt einen festen Ablauf zu bauen – mit dem Bewusstsein, dass ein frei handelnder Agent auf einem Server sorgfältig abgesichert werden muss.

## Fazit

Ein KI-Agent auf dem Server erledigt Aufgaben, während niemand am Rechner sitzt. **n8n** baut feste Abläufe aus verbundenen Bausteinen und zeigt sie in einer Weboberfläche. **OpenClaw** macht aus einem Messenger die Fernbedienung für einen Assistenten. **Hermes Agent** ist ein lernender Helfer für die Textkonsole. Alle drei sind quelloffen, laufen auf einem Ubuntu-Server mit Docker und arbeiten mit lokalen Sprachmodellen über **Ollama** zusammen, sodass keine Inhalte an fremde Anbieter abfließen. Im echten Betrieb gilt: Ollama nie ins Internet öffnen, die Modellgröße zur Hardware wählen und einem frei handelnden Agenten nur die nötigsten Rechte geben.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
