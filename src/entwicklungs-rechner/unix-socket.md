# Unix-Socket

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Viele Dienste auf einem Server nehmen Verbindungen nicht über einen Netzwerk-Port entgegen, sondern über eine **Socket-Datei** – einen sogenannten Unix-Socket. Das Kapitel [XWiki einrichten](../web-stack/xwiki.md) zeigt das an einem Beispiel: Statt auf Port 9000 kann der Anwendungsserver Tomcat auch über die Datei `/var/lib/tomcat10/xwiki.sock` ansprechbar sein. Der Webserver NGINX, der auf demselben Rechner läuft, kommt damit gut zurecht.

Beim Testen vom Entwicklungs-Rechner aus entsteht dann aber ein Problem: Ein Browser, das Werkzeug `curl` in einer älteren Fassung oder ein grafisches Datenbank-Programm sprechen nur über Ports, nicht über Socket-Dateien. Dieses Kapitel erklärt, was ein Unix-Socket ist, und zeigt, wie man mit einem einzigen Befehl eine Brücke von einem Port zu einer Socket-Datei baut – lokal auf dem Server und über eine verschlüsselte Verbindung bis auf den eigenen Rechner.

Alle Befehle werden in einem **Terminal** eingegeben – einem Fenster, in das man Anweisungen als Text tippt. Ein vorangestelltes `sudo` bedeutet: mit Verwaltungsrechten ausführen.

## Was ein Unix-Socket ist

Wenn zwei Programme miteinander reden, brauchen sie einen gemeinsamen Kanal. Der bekannte Weg ist ein **Netzwerk-Port**: Ein Programm lauscht auf einer Nummer wie 9000, ein anderes verbindet sich mit `127.0.0.1:9000`. Das funktioniert auch dann, wenn die beiden Programme auf verschiedenen Rechnern laufen.

Ein **Unix-Socket** ist der zweite Weg. Statt einer Portnummer gibt es eine Datei im Dateisystem, zum Beispiel `/var/lib/tomcat10/xwiki.sock`. Ein Programm lauscht an dieser Datei, ein anderes verbindet sich mit ihr. Beide müssen dafür auf demselben Rechner laufen – die Datei liegt ja auf genau einer Festplatte.

Der Name kommt daher, dass diese Technik aus der Unix-Welt stammt; genauer heißt sie **Unix Domain Socket**. Die Socket-Datei ist keine gewöhnliche Datei mit Inhalt. Sie ist nur ein Treffpunkt: Wer sie öffnet, landet bei dem Programm, das dort lauscht. Öffnet man sie versehentlich mit einem Texteditor, sieht man nichts Sinnvolles.

Zwei Eigenschaften machen den Unix-Socket für Server attraktiv:

- **Kein offener Port.** Ein Port kann aus Versehen für das ganze Netz geöffnet werden. Eine Socket-Datei kann das nicht – sie ist grundsätzlich nur auf demselben Rechner erreichbar.
- **Zugriff über Dateirechte.** Wer die Socket-Datei benutzen darf, regeln dieselben Lese- und Schreibrechte wie bei jeder anderen Datei. Im XWiki-Beispiel sorgt `rw-rw----` dafür, dass nur der Tomcat-Benutzer und die Gruppe des Webservers an den Socket kommen.

Verbreitet sind Unix-Sockets unter anderem bei **PostgreSQL** (`/var/run/postgresql/.s.PGSQL.5432`), bei **PHP-FPM** (`/run/php/php8.5-fpm.sock`, siehe [MediaWiki einrichten](../web-stack/mediawiki.md)), bei **Docker** (`/var/run/docker.sock`) und eben bei Tomcat.

## Das Problem beim Testen

Solange alle beteiligten Programme auf dem Server liegen, ist der Unix-Socket kein Hindernis. NGINX erreicht PHP-FPM und Tomcat problemlos über deren Socket-Dateien.

Beim Entwickeln will man den Dienst aber oft direkt ansehen, bevor der Webserver davorsteht – zum Beispiel den Einrichtungsassistenten von XWiki einmal im Browser durchklicken. Dann sollen Werkzeuge auf den Dienst zugreifen, die nur Ports kennen:

- ein **Browser** öffnet `http://localhost:9000/`, aber niemals eine Socket-Datei
- ältere Fassungen von **`curl`** können keine Unix-Sockets ansprechen
- grafische **Datenbank-Programme** wie DBeaver oder pgAdmin erwarten Wirtsname und Portnummer

In all diesen Fällen fehlt ein Zwischenstück, das auf einer Seite einen Port anbietet und auf der anderen Seite die Socket-Datei benutzt. Genau das leistet `socat`.

## socat: die Brücke in einem Befehl

**`socat`** (kurz für „socket cat") ist ein kleines Kommandozeilen-Werkzeug, das zwei Datenkanäle miteinander verbindet und alles, was auf der einen Seite hereinkommt, unverändert an die andere Seite weitergibt. Die beiden Seiten dürfen von ganz verschiedener Art sein – ein Port hier, eine Datei dort.

Installiert wird es aus den Paketquellen von Ubuntu:

```bash
sudo apt update
sudo apt install socat
```

Um einen lokalen Port 9000 mit der XWiki-Socket-Datei zu verbinden:

```bash
socat TCP-LISTEN:9000,bind=127.0.0.1,fork,reuseaddr \
  UNIX-CONNECT:/var/lib/tomcat10/xwiki.sock
```

Solange dieser Befehl läuft, nimmt `socat` auf Port 9000 Verbindungen an und leitet sie an die Socket-Datei weiter. Ein Aufruf von `http://127.0.0.1:9000/` landet dann bei Tomcat. Mit `Strg`+`C` wird `socat` wieder beendet; danach ist der Port geschlossen.

### Die Bestandteile des Befehls

`socat` bekommt genau zwei Adressen. Die erste beschreibt, wo Verbindungen hereinkommen, die zweite, wohin sie gehen sollen. An jede Adresse werden mit Komma weitere Angaben angehängt.

| Baustein | Bedeutung |
| --- | --- |
| `TCP-LISTEN:9000` | auf dem Netzwerk-Port 9000 auf Verbindungen warten |
| `bind=127.0.0.1` | dabei nur auf dem Rechner selbst erreichbar sein, nicht im Netz |
| `fork` | für jede neue Verbindung einen eigenen Ableger starten, damit mehrere gleichzeitig möglich sind |
| `reuseaddr` | den Port sofort wieder benutzen dürfen, ohne kurze Wartezeit nach dem Beenden |
| `UNIX-CONNECT:/var/lib/tomcat10/xwiki.sock` | sich mit dieser Socket-Datei verbinden |

`bind=127.0.0.1` ist wichtig. Ohne diese Angabe würde `socat` den Port auf allen Netzwerk-Adressen öffnen und damit den Unix-Socket, der eigentlich vor dem Netz geschützt ist, für Fremde erreichbar machen. Der Zusatz sorgt dafür, dass die Brücke nur auf dem Server selbst existiert.

`fork` fehlt in vielen kurzen Beispielen, wird aber fast immer gebraucht: Ohne `fork` bedient `socat` genau eine Verbindung und beendet sich danach. Ein Browser öffnet für eine einzige Seite oft mehrere Verbindungen parallel – ohne `fork` bricht das sofort ab.

### Die Richtung umdrehen

Der umgekehrte Fall kommt ebenfalls vor: Ein Werkzeug spricht nur über eine Socket-Datei, der Dienst hört aber auf einem Port. Dann werden die beiden Adressen einfach getauscht:

```bash
socat UNIX-LISTEN:/tmp/meine.sock,fork,reuseaddr \
  TCP-CONNECT:127.0.0.1:9000
```

`socat` legt dabei die Datei `/tmp/meine.sock` an und reicht alles, was dort ankommt, an Port 9000 weiter. Nach dem Beenden bleibt die Datei manchmal liegen und muss mit `rm /tmp/meine.sock` entfernt werden.

## Bis auf den eigenen Rechner: SSH-Tunnel

Die `socat`-Brücke hilft auf dem Server. Auf dem Entwicklungs-Rechner ist der Server-Socket damit aber noch nicht erreichbar. Dafür sorgt ein **SSH-Tunnel** – dieselbe Technik, die das Kapitel [XWiki einrichten](../web-stack/xwiki.md) für den Port 9000 verwendet.

Moderne Fassungen von OpenSSH (ab Version 6.7, auf aktuellem Ubuntu immer gegeben) können am entfernten Ende eines Tunnels statt eines Ports auch eine Socket-Datei ansprechen:

```bash
ssh -L 9000:/var/lib/tomcat10/xwiki.sock admin@SERVER-IP
```

Der Teil `-L 9000:/var/lib/tomcat10/xwiki.sock` öffnet auf dem eigenen Rechner den Port 9000 und verbindet ihn durch die verschlüsselte SSH-Verbindung direkt mit der Socket-Datei auf dem Server. Ein `socat` auf dem Server ist in diesem Fall gar nicht nötig. Solange das SSH-Fenster offen ist, landet jeder Aufruf von `http://localhost:9000/` im Browser des eigenen Rechners bei Tomcat.

`socat` auf dem Server und der SSH-Tunnel lösen also dieselbe Aufgabe an zwei verschiedenen Stellen. Wenn SSH die Socket-Datei direkt ansprechen kann, ist der Tunnel allein die einfachere Wahl. `socat` bleibt nützlich, wenn auf dem Server selbst ein Werkzeug an den Socket soll oder wenn eine ältere SSH-Fassung im Spiel ist.

## Andere Wege zum selben Ziel

`socat` ist die kürzeste Lösung für den schnellen Test. Für den Dauerbetrieb gibt es solidere Möglichkeiten.

| Werkzeug | Einsatz | Anmerkung |
| --- | --- | --- |
| **socat** | schneller Test im Terminal | ein Befehl, läuft nur, solange das Fenster offen ist |
| **SSH-Tunnel** (`ssh -L`) | Zugriff vom eigenen Rechner | verschlüsselt, kein zusätzliches Programm auf dem Server |
| **`systemd-socket-proxyd`** | dauerhafte Brücke auf dem Server | wird als Dienst eingerichtet, startet nach einem Neustart von selbst wieder |
| **NGINX** (`stream`-Baustein) | dauerhafte Brücke, wenn NGINX ohnehin läuft | ein kleiner `stream { … }`-Abschnitt in der Konfiguration |
| **`ncat`** (aus dem Nmap-Paket) | Ersatz für `socat`, falls dieses fehlt | Aufruf etwas umständlicher |

Für eine Brücke, die dauerhaft bestehen soll, ist ein per `systemd` eingerichteter Dienst der richtige Weg. Ein `socat`-Befehl im Terminal ist dagegen als Wegwerf-Lösung für einen einzelnen Testlauf gedacht.

## Fehlersuche

- **`Connection refused`** beim Verbinden mit dem Port: `socat` läuft nicht oder hört auf einem anderen Port. Prüfen mit `ss -tlnp | grep 9000`.
- **`socat: … Permission denied`** bei `UNIX-CONNECT`: Der Benutzer, der `socat` startet, darf die Socket-Datei nicht benutzen. Die Rechte zeigt `ls -l /var/lib/tomcat10/xwiki.sock`. Meist hilft es, `socat` mit `sudo` oder als der Benutzer des Dienstes zu starten.
- **`socat: … No such file or directory`**: Der Pfad zur Socket-Datei stimmt nicht, oder der Dienst läuft nicht und hat den Socket deshalb nicht angelegt.
- **Die erste Seite lädt, weitere Anfragen hängen**: Die Angabe `fork` fehlt. Den Befehl mit `,fork` an der ersten Adresse neu starten.

## Für dieses Buch

Für die Einrichtung von XWiki (siehe [XWiki einrichten](../web-stack/xwiki.md)) reicht in fast allen Fällen der **SSH-Tunnel auf die Socket-Datei** aus – er braucht kein zusätzliches Programm auf dem Server und ist verschlüsselt.

**`socat`** ist das Mittel der Wahl, wenn direkt auf dem Server ein Werkzeug an einen Unix-Socket muss – etwa `curl` für einen kurzen Test – oder wenn die Socket-Datei aus einem anderen Grund kurz als Port erreichbar sein soll. Der Befehl läuft nur, solange das Terminal-Fenster offen ist; danach ist der Port wieder zu. Für eine dauerhafte Brücke wird stattdessen ein `systemd`-Dienst oder der `stream`-Baustein von NGINX eingerichtet.

## Fazit

Ein **Unix-Socket** ist ein Treffpunkt für zwei Programme auf demselben Rechner in Form einer Datei im Dateisystem – ohne Port, mit Zugriffsschutz über die Dateirechte. Viele Serverdienste bieten sich so an. Werkzeuge wie Browser oder grafische Datenbank-Programme sprechen dagegen nur über Ports. **`socat`** baut die Brücke in einem einzigen Befehl: `TCP-LISTEN` auf der einen Seite, `UNIX-CONNECT` auf der anderen, dazu `bind=127.0.0.1` für die Beschränkung auf den Rechner selbst und `fork` für mehrere gleichzeitige Verbindungen. Vom eigenen Rechner aus erreicht ein **SSH-Tunnel** die Socket-Datei auf dem Server sogar ohne Zusatzprogramm. Für den Dauerbetrieb übernehmen `systemd-socket-proxyd` oder NGINX dieselbe Aufgabe zuverlässiger.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
