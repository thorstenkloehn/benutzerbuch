# Sprachmodell selbst betreiben

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Ein Sprachmodell ist ein Programm, das Text fortsetzt und umformt (siehe [Text erstellen](../grundlagen/text-erstellen.md)). Man kann es bei einem Anbieter im Internet nutzen oder auf einem eigenen Rechner laufen lassen. Der zweite Weg heißt „selbst betreiben" (siehe [Inhalts-Software selbst betreiben](../grundlagen/inhalt-software-verwalten.md)): Die Inhalte bleiben im eigenen Haus, und es fallen keine Kosten pro Anfrage an.

Die Kapitel [Text erstellen](../grundlagen/text-erstellen.md) und [KI-Agenten auf dem Server](./ki-agent.md) setzen dafür **Ollama** ein – das einfachste Programm für diesen Zweck. Dieses Kapitel schaut hinter Ollama: Aus welchen Teilen besteht ein selbst betriebenes Sprachmodell, welche Programme gibt es außer Ollama, und wie holt man aus einer bestimmten Hardware die beste Geschwindigkeit heraus. Am Ende werden zwei echte Rechner eingerichtet: ein Desktop-PC mit Grafikkarte und ein gemieteter Server ohne Grafikkarte.

## Die vier Bausteine

Ein selbst betriebenes Sprachmodell besteht aus vier Teilen, die aufeinander aufbauen:

| Baustein | Aufgabe |
| --- | --- |
| Hardware und Treiber | liefern die Rechenleistung und machen die Grafikkarte ansprechbar |
| Inferenz-Engine | lädt das Modell in den Speicher und rechnet die Antworten aus |
| Benutzeroberfläche | die Chat-Seite im Browser |
| Erweiterungen | Anbindung an eigene Texte und eine Schnittstelle für andere Programme |

„Inferenz" ist das Fachwort dafür, aus einer Anfrage eine Antwort auszurechnen. Die **Inferenz-Engine** ist also das Programm, das die eigentliche Arbeit macht. Nur die ersten beiden Bausteine sind Pflicht; Oberfläche und Erweiterungen kommen je nach Bedarf dazu.

## Baustein 1: Hardware und Treiber

### Der Speicher entscheidet über die Modellgröße

Ein Modell muss vollständig in den Speicher passen – am besten in den schnellen Speicher der Grafikkarte (das **VRAM**), sonst in den Arbeitsspeicher des Rechners (das **RAM**). Passt es nirgends ganz hinein, wird es sehr langsam.

Wie groß ein Modell ist, gibt man in der Zahl seiner **Parameter** an, meist in Milliarden (englisch „billion", abgekürzt „B"). Ein Modell mit 8 Milliarden Parametern heißt kurz „8B".

Damit ein Modell weniger Speicher braucht, wird es **quantisiert**: Jede seiner Zahlen wird von 16 Bit auf 4 Bit eingedampft. Das Modell braucht danach nur noch rund ein Viertel des Speichers, verliert aber kaum an Qualität. Die gängige Stufe dafür heißt `Q4_K_M`. Als grobe Faustregel gilt:

| Modellgröße | Speicherbedarf bei 4 Bit (grob) | passt auf |
| --- | --- | --- |
| 3B | 2–3 GB | fast jede Grafikkarte, auch reines RAM |
| 7–8B | 5–6 GB | Karten ab 6 GB VRAM |
| 13–14B | 9–10 GB | Karten ab 12 GB VRAM |
| 30–34B | rund 20 GB | Karten ab 24 GB VRAM |
| 70B | rund 40 GB | mehrere Karten oder viel RAM |

### Treiber und Beschleunigung

Damit ein Programm die Grafikkarte zum Rechnen nutzen kann, braucht es den passenden Treiber und eine Rechenbibliothek:

- **NVIDIA-Karten:** die Rechenbibliothek heißt **CUDA**. Nötig ist der NVIDIA-Treiber; CUDA selbst bringen Ollama und llama.cpp bereits mit.
- **AMD-Karten:** die Rechenbibliothek heißt **ROCm** und unterstützt vor allem neuere Karten.
- **Apple-Rechner:** die Rechenbibliothek **Metal** ist im Betriebssystem eingebaut.
- **Ohne Grafikkarte:** dann rechnet der Hauptprozessor (die **CPU**). Moderne Prozessoren haben eingebaute Rechenbefehle wie **AVX2** oder **AVX-512**, die das beschleunigen. Das funktioniert, ist aber deutlich langsamer als mit einer Grafikkarte.

## Baustein 2: Die Inferenz-Engine

Die Inferenz-Engine lädt die Modelldatei und rechnet die Antwort Wort für Wort aus. Nach außen stellt sie eine Schnittstelle bereit, über die andere Programme Anfragen schicken – fast immer im sogenannten **OpenAI-Format**, das sich als gemeinsame Sprache durchgesetzt hat.

### Ollama – der einfache Einstieg

**Ollama** baut auf llama.cpp auf und versteckt dessen Einstellungen. Ein Befehl installiert es, ein weiterer lädt und startet ein Modell. Ollama erkennt eine vorhandene Grafikkarte von selbst und verteilt das Modell passend auf VRAM und RAM. Für die meisten ist das die richtige Wahl.

### llama.cpp – nah an der Hardware

**llama.cpp** ist in den Programmiersprachen C und C++ geschrieben und steckt als Kern in Ollama und vielen anderen Programmen. Man kann es selbst übersetzen („kompilieren") und dabei genau auf den eigenen Prozessor abstimmen. Auf reiner CPU holt es dadurch mehr Tempo heraus als Ollama. Es lädt Modelle im Dateiformat **GGUF**, in dem quantisierte Modelle üblicherweise verteilt werden.

### vLLM und TGI – für viele Anfragen gleichzeitig

**vLLM** und **TGI** (von der Firma Hugging Face) sind auf **Durchsatz** ausgelegt: viele Nutzer parallel bei voller Auslastung der Grafikkarte. Ihr rechenintensiver Teil ist ebenfalls maschinennah geschrieben, die Steuerung darum herum in Python beziehungsweise Rust. Beide brauchen eine Grafikkarte mit viel Speicher. Für einen einzelnen Nutzer sind sie überdimensioniert.

### Weitere Engines

- **ExLlamaV2** und **ExLlamaV3** sind auf NVIDIA-Karten für einen einzelnen Nutzer besonders schnell.
- In der Programmiersprache **Rust** gibt es jüngere Projekte wie **mistral.rs**. Sie sind vielversprechend, aber noch weniger erprobt.
- **ML-Compiler** wie **MLC-LLM**, **TVM** und **TensorRT-LLM** übersetzen ein Modell einmalig in hochoptimierten Maschinencode für genau eine Hardware. Das bringt die beste Geschwindigkeit, verlangt aber eine aufwendige Einrichtung.

| Engine | Sprache | Stärke | Grafikkarte nötig |
| --- | --- | --- | --- |
| Ollama | Go, C/C++ | einfachster Einstieg | nein |
| llama.cpp | C/C++ | reine CPU und gemischt, feine Abstimmung | nein |
| vLLM | Python, CUDA | viele Anfragen parallel | ja, viel VRAM |
| TGI | Rust, Python | Serverbetrieb mit vielen Nutzern | ja |
| ExLlamaV2/V3 | Python, CUDA | schnell für einen Nutzer | ja (NVIDIA) |
| MLC-LLM | Compiler | maximale Geschwindigkeit je Gerät | wahlweise |

### Welche Engine holt das Meiste aus „Bare Metal"?

„Bare Metal" heißt: direkt auf der Hardware, ohne eine Virtualisierungsschicht dazwischen. Welche Engine dann am schnellsten ist, hängt von der Hardware und von der Zahl gleichzeitiger Nutzer ab. Es gibt keine, die immer vorn liegt.

- **Nur CPU:** **llama.cpp**, selbst kompiliert mit den Rechenbefehlen des eigenen Prozessors (AVX2 oder AVX-512). Ollama nutzt dieselbe Technik, ist aber nicht auf den einzelnen Prozessor zugeschnitten. Entscheidend für das Tempo ist weniger die Zahl der Kerne als die **Speicherbandbreite** – also wie schnell Daten zwischen RAM und Prozessor fließen. Schneller RAM mit mehreren Kanälen bringt hier am meisten.
- **Nur Grafikkarte, ein Nutzer:** **ExLlamaV2/V3** oder **llama.cpp** mit CUDA; für die absoluten Höchstwerte **TensorRT-LLM** oder **MLC-LLM**.
- **Nur Grafikkarte, viele Nutzer gleichzeitig:** **vLLM** oder **TGI**.

## Baustein 3: Die Benutzeroberfläche

Ohne Oberfläche bedient man das Modell nur über die Textkonsole. Für den Alltag ist eine Chat-Seite im Browser bequemer:

- **Open WebUI** ist die verbreitetste. Sie sieht aus wie ein gewohnter Chat-Assistent und spricht mit Ollama oder jeder anderen Engine im OpenAI-Format. Sie wird meist als Docker-Container betrieben (siehe [Containerisierung von Software](./containerisierung.md)).
- **Text Generation WebUI** (auch „oobabooga" genannt) hat mehr Schalter, lädt Modelle selbst und bringt mehrere Engines mit. Sie eignet sich für alle, die viel ausprobieren wollen.
- **Direkt in vorhandener Software:** Für **MediaWiki** und **Drupal** gibt es Erweiterungen, die ein Chatfenster oder eine „Frage ans Wiki"-Funktion einbauen (siehe [Wissenssystem](../web-stack/wissensystem.md) und [Content-Management-System](../web-stack/cms.md)). Dann braucht es keine zweite Oberfläche.

## Baustein 4: Erweiterungen

- **RAG** (für „retrieval-augmented generation", etwa „Erzeugung mit Nachschlagen") liefert dem Modell zu jeder Frage die passenden eigenen Texte mit. Das Modell antwortet dann aus diesen Texten statt aus dem, was es beim Training gelernt hat. Dafür braucht es eine Vektordatenbank (siehe **pgvector** in [Inhalts-Software selbst betreiben](../grundlagen/inhalt-software-verwalten.md)). So beantwortet auch ein kleines lokales Modell Fragen zu Inhalten, die es nie gesehen hat.
- **Die OpenAI-kompatible Schnittstelle** (oft „API-Bridge" genannt) sorgt dafür, dass Programme, die eigentlich für einen Online-Dienst gebaut wurden, mit dem lokalen Modell sprechen können. Ollama und die meisten Engines bringen sie bereits mit.

## Rechner 1: Desktop-PC mit Grafikkarte

Als Beispiel dient ein Desktop-Rechner mit einem Intel-Core-i5-Prozessor (6 Kerne), 16 GB RAM und einer NVIDIA GeForce GTX 1660 mit 6 GB VRAM, auf dem **Ubuntu 26.04 LTS** läuft (siehe [Betriebssystem](./betriebssystem.md)). Damit passt ein Modell mit 7 bis 8 Milliarden Parametern in 4-Bit-Quantisierung knapp in den Grafikspeicher.

### Schritt 1: NVIDIA-Treiber installieren

Ubuntu bringt ein Hilfsprogramm mit, das die passende Treiberversion selbst auswählt:

```bash
sudo ubuntu-drivers install
sudo reboot
```

Nach dem Neustart prüft man, ob der Treiber geladen ist:

```bash
nvidia-smi
```

Der Befehl zeigt die Karte, ihren Speicher und die Treiberversion an. Erscheint stattdessen eine Fehlermeldung, hat der Treiber nicht geladen. Häufigste Ursache ist „Secure Boot" im BIOS: Entweder man schaltet es dort ab, oder man bestätigt den Treiber einmalig über die Abfrage, die beim nächsten Start erscheint.

Eine eigene CUDA-Installation ist für Ollama und llama.cpp nicht nötig – die Rechenbibliothek ist mitgeliefert. Nur wer vLLM einsetzt oder eine Engine selbst kompiliert, installiert zusätzlich das „CUDA Toolkit".

### Schritt 2: Ollama installieren

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Ein Skript aus dem Internet direkt auszuführen, sollte man nur bei Quellen tun, denen man vertraut. Wer sichergehen will, lädt das Skript zuerst herunter, liest es und führt es dann aus.

Ollama erkennt die GTX 1660 von selbst und legt das Modell in den Grafikspeicher.

### Schritt 3: Ein Modell laden

```bash
ollama run llama3.1:8b
```

Weitere gute Modelle in dieser Größe sind **Gemma** (`gemma3`) und **Qwen** (`qwen3`). Für eine deutsche Sprachprüfung eignet sich ein mittelgroßes Modell mit 7 bis 14 Milliarden Parametern (siehe [Text erstellen](../grundlagen/text-erstellen.md)). Bei 6 GB VRAM bleibt man bei 7 bis 8B in `Q4`; ein 14B-Modell passt nur teilweise auf die Karte und wird spürbar langsamer.

### Schritt 4 (wahlweise): Eine Oberfläche einrichten

```bash
docker run -d --restart unless-stopped --name open-webui \
  -p 127.0.0.1:3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  ghcr.io/open-webui/open-webui:main
```

Die Oberfläche ist danach unter `http://localhost:3000` erreichbar. Der Zusatz `-p 127.0.0.1:3000:8080` sorgt dafür, dass sie nur vom Rechner selbst erreichbar ist. Beim ersten Aufruf legt man ein Konto an; Ollama findet Open WebUI unter der Adresse `http://host.docker.internal:11434` von allein.

### Mehr Speicher nachrüsten

Das Beispiel-Mainboard hat vier RAM-Steckplätze, von denen zwei frei sind; bis zu 64 GB RAM sind möglich. Mehr RAM macht die Grafikkarte nicht schneller, erlaubt aber, größere Modelle teilweise im RAM zu halten (langsamer). Wer mehr Tempo will, ist mit einer Grafikkarte mit mehr VRAM besser bedient als mit mehr RAM.

## Rechner 2: Gemieteter Server ohne Grafikkarte

Als Beispiel dient ein gemieteter KVM-VPS (siehe [Server mieten](./server-mieten.md)) mit einem AMD-EPYC-Prozessor (4 zugeteilte Kerne), 20 GB RAM und ohne echte Grafikkarte – nur mit einer virtuellen Anzeige für die Textkonsole. Auch hier läuft **Ubuntu 26.04 LTS**.

Ohne Grafikkarte rechnet der Prozessor. Das funktioniert, ist aber deutlich langsamer: Ein 8B-Modell liefert je nach Server nur wenige Wörter pro Sekunde.

### Schritt 1: Keine Treiber nötig

Es gibt keine Grafikkarte, also auch keinen Grafiktreiber einzurichten. Wichtig ist nur, dass der Prozessor die modernen Rechenbefehle beherrscht. Das prüft man mit:

```bash
lscpu | grep -o 'avx[0-9_]*' | sort -u
```

Erscheint `avx2` (oder zusätzlich `avx512f` und ähnliche), ist alles Nötige vorhanden. AMD-EPYC-Prozessoren beherrschen beides.

### Schritt 2: Ollama installieren

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Ollama merkt, dass keine Grafikkarte vorhanden ist, und rechnet auf dem Prozessor. Der gleiche Sicherheitshinweis wie oben gilt auch hier.

### Schritt 3: Ein kleineres Modell wählen

Auf dem Prozessor zählt jede Milliarde Parameter doppelt. Gut geeignet sind Modelle mit 3 bis 8B:

```bash
ollama pull llama3.2:3b
```

Ein 3B-Modell antwortet flott, ein 8B-Modell (`llama3.1:8b`) liefert bessere Sprache, braucht aber mehr Geduld. Die Geschwindigkeit hängt vor allem an der Speicherbandbreite; mehr als vier bis acht Rechen-Stränge („Threads") bringen meist wenig zusätzliches Tempo.

### Schritt 4: Das Letzte herausholen mit llama.cpp

Wer mehr Tempo braucht, als Ollama auf dem Prozessor liefert, kompiliert llama.cpp selbst:

```bash
sudo apt update
sudo apt install -y build-essential cmake git libcurl4-openssl-dev
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
cmake -B build
cmake --build build --config Release -j
```

Der Übersetzer erkennt die Rechenbefehle des Prozessors automatisch und nutzt sie. Danach lädt man ein Modell im GGUF-Format und startet den eingebauten Server:

```bash
./build/bin/llama-server -hf ggml-org/gpt-oss-20b-GGUF -c 4096
```

`llama-server` stellt dieselbe OpenAI-kompatible Schnittstelle bereit wie Ollama, hier unter Port `8080`.

### Schritt 5: Nicht ins Internet öffnen

Weder Ollama noch `llama-server` haben eine Anmeldung. Sie dürfen nur vom Server selbst erreichbar sein, niemals von außen. Die Firewall des Anbieters sollte die Ports `11434` (Ollama) und `8080` (llama-server) von außen sperren. Für den Zugriff über das Internet stellt man einen Webserver mit Verschlüsselung und Passwortschutz davor oder nutzt einen verschlüsselten Tunnel (siehe [KI-Agenten auf dem Server](./ki-agent.md)).

## Für dieses Buch

- Für die Sprachprüfung der Kapitel genügt **Ollama** auf einem der beiden Rechner. Andere Engines lohnen sich erst, wenn viele Menschen gleichzeitig zugreifen (dann **vLLM** oder **TGI**) oder wenn man das letzte bisschen CPU-Tempo braucht (dann **llama.cpp** selbst kompiliert).
- Auf dem **Desktop mit GTX 1660** läuft ein 7- bis 8B-Modell in `Q4` flüssig genug für schnelle Zwischendurchgänge.
- Der **Server ohne Grafikkarte** taugt nur für kleine Modelle und für Aufgaben, bei denen Wartezeit keine Rolle spielt – etwa nächtliche Läufe eines Agenten (siehe [KI-Agenten auf dem Server](./ki-agent.md)).
- Die Schlussredaktion bleibt beim großen Cloud-Modell (siehe [KI-Abo auswählen](../grundlagen/ki-abo.md)); die lokalen Modelle übernehmen die häufigen, schnellen Schritte.

## Fazit

Ein selbst betriebenes Sprachmodell besteht aus vier Bausteinen: **Hardware und Treiber**, der **Inferenz-Engine**, einer **Benutzeroberfläche** und **Erweiterungen** für eigene Texte. Die Modellgröße richtet sich nach dem verfügbaren Speicher, am besten dem der Grafikkarte; die 4-Bit-Quantisierung senkt den Bedarf auf rund ein Viertel. **Ollama** ist der einfachste Weg und reicht für den Einzelbetrieb. **llama.cpp** holt auf reiner CPU das Meiste heraus, **vLLM** und **TGI** bedienen viele Nutzer gleichzeitig. Ein Desktop mit einer 6-GB-Grafikkarte fährt ein 7- bis 8B-Modell flüssig; ein gemieteter Server ohne Grafikkarte schafft nur kleine Modelle mit Geduld. In beiden Fällen gilt: Die Schnittstelle nie ungeschützt ins Internet öffnen.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
