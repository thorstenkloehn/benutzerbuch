# Containerisierung von Software

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Die Programme aus dem Kapitel [Inhalts-Software selbst betreiben](../grundlagen/inhalt-software-verwalten.md) lassen sich auf zwei Arten auf einem Server einrichten. Der erste Weg ist die direkte Installation: Man lädt jedes Programm mit dem Paketbefehl des Betriebssystems (siehe [Betriebssystem](./betriebssystem.md)) herunter und startet es als Hintergrunddienst. Der zweite Weg heißt **Containerisierung**: Jedes Programm läuft dann in einem eigenen, abgeschlossenen Paket, einem sogenannten Container.

Dieses Buch beschreibt in den anderen Kapiteln den direkten Weg. Er ist für eine einzelne, kleine Wissenssammlung übersichtlich und braucht kein zusätzliches Werkzeug. Trotzdem gehört die Containerisierung zum Grundwissen für den Betrieb eines Servers: Ein großer Teil der Anleitungen im Internet setzt sie voraus, und viele Programme werden heute zuerst als Container ausgeliefert. Dieses Kapitel erklärt darum, was ein Container ist, welche Eigenschaften ihn ausmachen, welche Programme es dafür gibt und wie man Docker auf einem Ubuntu-Server einrichtet.

## Was ein Container ist

Der Name ist wörtlich gemeint. Ein Schiffscontainer ist eine genormte Kiste: Was drin ist, spielt für den Kran und das Schiff keine Rolle, denn außen sind alle Container gleich. Ein Software-Container macht dasselbe mit einem Programm. Er packt das Programm zusammen mit allem, was es zum Laufen braucht – Hilfsbibliotheken, Einstellungen, kleine Zusatzwerkzeuge – in ein einziges Bündel. Dieses Bündel läuft dann auf jedem Rechner gleich, egal welche Programme sonst darauf installiert sind.

Die Vorlage für einen Container heißt **Image** (englisch für „Abbild"). Ein Image ist eine unveränderliche Datei, die ein fertig eingerichtetes Programm enthält. Aus einem Image startet man einen oder mehrere Container. Der Container ist die laufende Ausführung, das Image die Blaupause dazu.

Ein Container ist nicht dasselbe wie eine virtuelle Maschine (siehe den Abschnitt zu Proxmox VE im Kapitel [Betriebssystem](./betriebssystem.md)). Eine virtuelle Maschine bringt ein komplettes eigenes Betriebssystem mit und braucht entsprechend viel Arbeitsspeicher und Festplattenplatz. Ein Container teilt sich den Kern des Betriebssystems mit dem Wirt und enthält nur das Programm selbst. Dadurch startet er in Sekunden und belegt wenig Platz.

## Wesentliche Eigenschaften

Sechs Merkmale beschreiben, was einen Container ausmacht:

- **Abgeschlossen.** Jeder Container ist vom Rest des Systems getrennt. Ein Programm im Container sieht nur seine eigenen Dateien und Prozesse, nicht die der anderen Container oder des Wirts. Stürzt ein Container ab, bleiben die übrigen unberührt.
- **Alles dabei.** Das Image enthält das Programm samt aller benötigten Bibliotheken in genau der passenden Version. Es gibt keinen Streit mehr darüber, ob auf dem Server die richtige Fassung einer Hilfsbibliothek liegt.
- **Sparsam.** Weil sich alle Container den Betriebssystemkern des Wirts teilen, kostet ein Container kaum mehr Leistung als das Programm selbst. Auf einem kleinen Server können problemlos ein Dutzend Container nebeneinander laufen.
- **Immer gleich.** Dasselbe Image liefert auf dem Entwicklungsrechner, auf dem Testserver und im echten Betrieb dasselbe Ergebnis. Der Satz „Bei mir läuft es doch" verliert seinen Sinn.
- **Wegwerfbar.** Ein Container ist dazu gedacht, jederzeit gelöscht und neu gestartet zu werden. Alles, was dauerhaft bleiben soll – zum Beispiel die Datenbankinhalte von PostgreSQL –, wird deshalb in einem getrennten Speicherbereich abgelegt, einem sogenannten **Volume**. Das Volume überlebt den Container.
- **Aus einer Textdatei beschrieben.** Wie ein Image aufgebaut ist, steht in einer kurzen Textdatei (bei Docker heißt sie `Dockerfile`). Welche Container zusammen eine Anwendung ergeben, steht in einer weiteren Textdatei (`compose.yaml`). Diese Dateien lassen sich versionieren und weitergeben; aus ihnen entsteht die gesamte Umgebung neu.

## Die wichtigsten Programme

### Docker

Docker ist das mit Abstand verbreitetste Container-Werkzeug und für viele gleichbedeutend mit dem Thema. Es besteht aus mehreren Teilen: der **Docker Engine**, einem Hintergrunddienst, der die Container verwaltet; dem Befehl `docker` für die Bedienung über die Textkonsole; und **Docker Compose**, mit dem sich mehrere Container gemeinsam aus einer `compose.yaml`-Datei starten lassen. Zu Docker gibt es die meisten Anleitungen und die größte Sammlung fertiger Images (im „Docker Hub"). Für den Einstieg ist Docker die naheliegende Wahl.

Ein Punkt verdient Aufmerksamkeit: Die Docker Engine läuft standardmäßig mit vollen Systemrechten. Wer einen Benutzer berechtigt, Docker zu bedienen, gibt ihm damit faktisch die volle Kontrolle über den Server. Dazu weiter unten mehr.

### Podman

Podman verfolgt dasselbe Ziel wie Docker und versteht fast dieselben Befehle – oft genügt es, `docker` durch `podman` zu ersetzen. Zwei Unterschiede sind wichtig. Erstens kommt Podman ohne ständig laufenden Hintergrunddienst aus. Zweitens laufen Container bei Podman von Haus aus ohne Systemrechte („rootless"), was die Sicherheit erhöht. Podman stammt aus dem Umfeld von Red Hat und ist auf den Distributionen der Red-Hat-Familie (siehe [Betriebssystem](./betriebssystem.md)) vorinstalliert oder leicht nachzurüsten. Auf Debian und Ubuntu ist es ebenfalls verfügbar.

### Kubernetes (K8s)

Kubernetes ist kein Ersatz für Docker oder Podman, sondern eine Ebene darüber. Es verteilt Container über viele Server hinweg, startet abgestürzte Container neu, verschiebt Last von einem Rechner auf den anderen und rollt Aktualisierungen ohne Unterbrechung aus. Die Abkürzung „K8s" steht für das Wort Kubernetes mit acht ausgelassenen Buchstaben zwischen „K" und „s".

Für eine einzelne selbst betriebene Wissenssammlung ist Kubernetes deutlich überdimensioniert. Der Aufwand für Einrichtung und Pflege übersteigt den Nutzen bei Weitem. Es wird hier nur genannt, weil der Begriff in vielen Anleitungen auftaucht. Wer trotzdem in die Technik hineinschnuppern will, greift zu einer abgespeckten Ausgabe wie **k3s** oder **MicroK8s**, die auf einem einzelnen Server läuft.

### Weitere Werkzeuge

- **containerd** und **runc.** Das sind die unteren Bausteine, die unter Docker die eigentliche Arbeit erledigen. Man bedient sie normalerweise nicht direkt, hört die Namen aber in Fehlermeldungen und Anleitungen.
- **LXC** und **Incus** (früher LXD). Diese Werkzeuge starten sogenannte Systemcontainer. Ein Systemcontainer verhält sich eher wie ein sehr sparsamer kompletter Rechner mit eigenem Betriebssystem-Nutzerbereich, während ein Docker-Container üblicherweise nur ein einzelnes Programm enthält.
- **Portainer.** Eine Weboberfläche, mit der sich Docker-Container per Mausklick im Browser verwalten lassen, statt über die Textkonsole. Praktisch für alle, die sich die Befehle nicht merken wollen.
- **Docker Compose.** Streng genommen kein eigenständiges Programm mehr, sondern ein fester Bestandteil von Docker. Es ist das Werkzeug der Wahl, sobald mehrere Container zusammenspielen – also genau im Fall dieses Buchs mit MediaWiki, PostgreSQL, Meilisearch und Ollama.

### Kurzvergleich

| Werkzeug | Aufgabe | Für diesen Aufbau |
| --- | --- | --- |
| Docker | Container einzeln oder als Gruppe betreiben | gut geeignet, größte Anleitungssammlung |
| Podman | wie Docker, ohne Dauerdienst, ohne Systemrechte | gut geeignet, sicherer voreingestellt |
| Docker Compose | mehrere Container aus einer Textdatei starten | empfohlen, sobald mehr als ein Programm läuft |
| Kubernetes (K8s) | Container über viele Server verteilen | überdimensioniert |
| k3s / MicroK8s | Kubernetes für einen einzelnen Server | nur zum Ausprobieren |
| LXC / Incus | Systemcontainer statt Einzelprogramm-Container | Sonderfall |
| Portainer | Container im Browser verwalten | optionale Erleichterung |

## Docker auf Ubuntu einrichten

Die folgenden Schritte gelten für **Ubuntu 26.04 LTS** und ebenso für die älteren Ausgaben 24.04 und 22.04. Alle Befehle werden in der Textkonsole des Servers eingegeben. Das Zeichen `sudo` am Anfang bedeutet: mit Verwaltungsrechten ausführen.

### Schritt 1: Alte Pakete entfernen

Ubuntu bringt teils veraltete oder unvollständige Docker-Pakete mit. Diese werden zuerst entfernt, damit sie sich nicht mit der offiziellen Fassung ins Gehege kommen:

```bash
for paket in docker.io docker-doc docker-compose podman-docker containerd runc; do
  sudo apt-get remove -y $paket
done
```

Meldet der Befehl, dass ein Paket gar nicht installiert war, ist das in Ordnung.

### Schritt 2: Die Paketquelle von Docker eintragen

Damit Ubuntu die aktuelle Docker-Fassung findet, wird die offizielle Paketquelle des Herstellers hinzugefügt. Dazu gehört ein digitaler Schlüssel, mit dem Ubuntu prüft, dass die Pakete echt sind:

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Schritt 3: Docker installieren

```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

Damit sind die Docker Engine, der Bedienbefehl `docker` und Docker Compose eingerichtet.

### Schritt 4: Prüfen, ob es läuft

```bash
sudo docker run --rm hello-world
```

Docker lädt ein winziges Test-Image herunter, startet es und gibt eine Erfolgsmeldung aus. Der Zusatz `--rm` löscht den Test-Container gleich wieder.

## Wichtige Einstellungen

Nach der Installation sind einige Anpassungen sinnvoll.

### Docker ohne „sudo" bedienen – mit Bedacht

Damit ein normaler Benutzer `docker` ohne vorangestelltes `sudo` verwenden kann, wird er in die Gruppe `docker` aufgenommen:

```bash
sudo usermod -aG docker $USER
```

Danach einmal ab- und wieder anmelden. Wichtig zu wissen: Wer in dieser Gruppe ist, kann über Docker die volle Kontrolle über den Server erlangen. Auf einem Server sollte deshalb nur der Betreiber selbst in dieser Gruppe sein. Wer das vermeiden will, betreibt Docker im **rootless-Modus** (das Docker-Projekt liefert dafür das Skript `dockerd-rootless-setuptool.sh` mit) oder nutzt gleich Podman, das ohne Systemrechte arbeitet.

### Automatischer Start nach einem Neustart

Damit die Docker Engine nach jedem Server-Neustart von selbst hochfährt:

```bash
sudo systemctl enable --now docker.service containerd.service
```

Einzelne Container starten nach einem Neustart nur dann wieder, wenn man ihnen das mitgibt – bei `docker run` mit dem Zusatz `--restart unless-stopped`, in einer `compose.yaml` mit der Zeile `restart: unless-stopped`.

### Protokolldateien begrenzen

Ohne Vorgabe schreiben Container ihre Protokolle unbegrenzt auf die Festplatte, bis diese voll ist. Eine Obergrenze wird in der Datei `/etc/docker/daemon.json` gesetzt:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

Danach `sudo systemctl restart docker` ausführen. Jeder Container behält so höchstens drei Protokolldateien zu je 10 Megabyte.

### Speicherort der Daten kennen

Images, Container und Volumes liegen unter `/var/lib/docker`. Dieses Verzeichnis kann groß werden. Es sollte auf einer Festplatte mit genügend Platz liegen und in die Datensicherung einbezogen werden – allerdings nur die Volumes, denn nur dort stehen die eigenen Inhalte. Der Befehl `docker system df` zeigt den Verbrauch, `docker system prune` entfernt nicht mehr benötigte Reste.

### Docker und die Firewall

Docker trägt seine Regeln direkt in die Netzwerkfilter des Betriebssystems ein und umgeht dabei die verbreitete Firewall `ufw`. Ein mit `-p 8080:80` nach außen geöffneter Container ist deshalb aus dem Internet erreichbar, auch wenn `ufw` den Port eigentlich sperrt. Wer eine Firewall betreibt, veröffentlicht Container-Ports darum nur an die lokale Adresse (`-p 127.0.0.1:8080:80`) und stellt einen Webserver davor, oder richtet die Filterregeln von Hand ein.

## Für dieses Buch

Die anderen Kapitel richten MediaWiki, PostgreSQL, Meilisearch und Ollama ohne Container ein. Dieser Weg bleibt die Empfehlung für eine erste, kleine Wissenssammlung: weniger Werkzeuge, weniger Schichten, weniger, das schiefgehen kann. Container werden interessant, sobald man die Sammlung auf einen neuen Server umziehen will, mehrere getrennte Umgebungen (Test und Betrieb) braucht oder ein Programm ausprobieren möchte, ohne den Server dauerhaft zu verändern. Dann lohnt sich der Blick auf Docker Compose, mit dem sich der gesamte Aufbau in einer einzigen Textdatei beschreiben lässt.

## Fazit

Ein Container packt ein Programm mit allem Nötigen in ein abgeschlossenes, sparsames und überall gleich laufendes Bündel. **Docker** ist das gängige Werkzeug dafür, **Podman** die von Haus aus sicherere Alternative, **Kubernetes** eine Nummer zu groß für einen einzelnen Server. Auf Ubuntu wird Docker über die offizielle Paketquelle des Herstellers eingerichtet; danach sollte man den Zugang zur `docker`-Gruppe eng halten, den automatischen Start aktivieren, die Protokolle begrenzen und die Wechselwirkung mit der Firewall beachten. Für die Wissenssammlung dieses Buchs ist Containerisierung kein Muss – aber ein Werkzeug, das man kennen sollte, sobald der Aufbau wächst.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
