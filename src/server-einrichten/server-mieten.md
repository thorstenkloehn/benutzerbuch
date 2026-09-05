# Server mieten

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Die Programme aus dem Kapitel [Inhalts-Software selbst betreiben](../grundlagen/inhalt-software-verwalten.md) müssen irgendwo laufen – auf einem Rechner, der ständig eingeschaltet und aus dem Internet erreichbar ist. Ein solcher Rechner heißt Server. Man kann ihn selbst hinstellen oder ihn bei einem Anbieter mieten. Dieses Kapitel vergleicht die üblichen Wege, nennt für jeden die Vor- und Nachteile und gibt am Ende eine Empfehlung samt Anbieternamen. Ein Sonderfall am Schluss betrifft reine Textsammlungen, die sich ohne echten Server veröffentlichen lassen.

## Was der Server hier leisten muss

Bevor man Angebote vergleicht, sollte klar sein, was der Rechner können muss:

- **Dauerbetrieb.** Die Wissenssammlung soll jederzeit erreichbar sein, also läuft der Server rund um die Uhr.
- **Feste Adresse.** Er braucht eine gleichbleibende Adresse im Internet (eine feste IP-Adresse), damit ein Name wie `wiki.example.de` immer auf denselben Rechner zeigt.
- **Eigene Programme installieren.** MediaWiki, PostgreSQL mit der Erweiterung pgvector, Meilisearch und Ollama sind eigenständige Programme. Der Server muss es erlauben, sie selbst einzurichten und als Hintergrunddienste laufen zu lassen.
- **Genug Leistung.** Rechenkerne (CPU), Arbeitsspeicher (RAM) und Festplattenplatz müssen zur Größe der Sammlung passen. Besonders Ollama braucht viel Leistung; mit einer starken Grafikkarte (GPU) antwortet es deutlich schneller.

An diesen vier Punkten entscheidet sich, welche Angebote überhaupt infrage kommen.

## Die Wege im Überblick

### Webhosting (Webspace)

Beim Webhosting mietet man fertigen Speicherplatz auf einem Server, den der Anbieter betreibt und wartet. Man lädt seine Dateien hoch, mehr Zugriff gibt es nicht. Üblich ist damit der Betrieb von PHP-Anwendungen wie MediaWiki – aber nur, solange der Anbieter die passende Umgebung vorgibt.

Für den hier beschriebenen Aufbau reicht das nicht: Eigene Hintergrunddienste wie Meilisearch oder Ollama lassen sich nicht installieren, und an der Datenbank kann man keine Erweiterung wie pgvector nachrüsten. Man hat keine Kontrolle über die Umgebung. Webhosting ist damit nur eine Option, wenn man sich dauerhaft auf ein reines MediaWiki ohne die Zusatzprogramme beschränkt.

### Eigener Rechner zu Hause mit dynamischem DNS

Statt zu mieten, kann man einen Rechner zu Hause an den eigenen Internetanschluss hängen. Das Problem: Die meisten Anschlüsse bekommen von Zeit zu Zeit eine neue IP-Adresse zugeteilt. Ein fester Name würde dann ins Leere zeigen.

Hier hilft dynamisches DNS (auch DynDNS oder DDNS). Ein kleines Programm auf dem Rechner oder im Router merkt, wenn sich die IP-Adresse ändert, und meldet die neue Adresse sofort an einen Dienst, der den Namen verwaltet. So bleibt der Rechner unter demselben Namen erreichbar, obwohl sich seine Adresse ändert.

Trotzdem ist dieser Weg für eine Wissenssammlung im Dauerbetrieb wenig geeignet:

- Der Rechner muss ununterbrochen laufen – auch nachts, bei Gewitter und im Urlaub.
- Private Anschlüsse haben eine langsame Upload-Richtung; wenn viele Seiten oder große Antworten abgerufen werden, wird es zäh.
- Manche Anschlüsse vergeben gar keine von außen erreichbare Adresse mehr, sondern teilen eine Adresse unter vielen Kunden auf. Dann funktioniert dynamisches DNS nicht ohne Weiteres.
- Ein aus dem Internet erreichbarer Rechner im eigenen Netz ist ein Sicherheitsrisiko, wenn man ihn nicht sorgfältig abschottet.

### Container-VPS (LXC)

VPS steht für „virtueller privater Server“ – ein abgetrennter Bereich auf einem großen Rechner, den man wie einen eigenen kleinen Server benutzt. Bei der Container-Variante (oft „LXC“ genannt) teilen sich alle Kunden auf demselben großen Rechner denselben Betriebssystem-Kern. Das ist günstig, bringt aber Einschränkungen: Manche systemnahen Funktionen sind gesperrt, und man ist stärker davon abhängig, wie der Anbieter den großen Rechner einstellt.

Für einfache Zwecke reicht das oft. Für einen Aufbau aus mehreren Diensten, bei dem man volle Kontrolle möchte, ist die nächste Variante die bessere Wahl.

### KVM-VPS – die Empfehlung für den Anfang

Bei einem KVM-VPS bekommt man eine vollständige virtuelle Maschine mit eigenem Betriebssystem-Kern. Sie verhält sich wie ein eigener Rechner: Man wählt das Betriebssystem, installiert beliebige Programme und richtet Hintergrunddienste ein. Andere Kunden auf demselben großen Rechner sind sauber abgetrennt.

Für eine kleine bis mittlere Wissenssammlung ist ein KVM-VPS meist die richtige Größe. Er kostet monatlich einen festen, überschaubaren Betrag. Wächst die Sammlung, kann man bei den meisten Anbietern später auf ein größeres Paket wechseln oder einen zweiten Server dazunehmen.

### Cloud-Server

Ein Cloud-Server ist technisch dasselbe wie ein KVM-VPS, wird aber anders abgerechnet: nach Stunden oder Minuten statt pauschal im Monat. Man kann ihn in wenigen Minuten erstellen und wieder löschen und zahlt nur für die Zeit, in der er läuft.

Das lohnt sich vor allem zum Ausprobieren: einen Server aufsetzen, den Aufbau testen, ihn wieder abschalten. Für den Dauerbetrieb ist der Preisunterschied zum festen Monatspaket meist gering.

### Dedizierter Server (Root-Server)

Bei einem dedizierten Server mietet man einen ganzen physischen Rechner für sich allein. Niemand sonst teilt sich die Hardware. Man hat die volle Kontrolle und die volle Leistung – und kann alles installieren, was auch auf einem KVM-VPS läuft, nur eben mit mehr Reserven.

Das ist die richtige Wahl, wenn die Sammlung groß ist oder wenn Ollama zügig antworten soll. Server mit eingebauter Grafikkarte gibt es ebenfalls zu mieten, sie sind aber deutlich teurer. Kleinere Sprachmodelle laufen auch ohne Grafikkarte, nur langsamer. Der feste Monatspreis liegt höher als bei einem VPS.

### Kurzvergleich

| Weg | Kontrolle | Eigene Dienste | Kosten | Für diesen Aufbau |
| --- | --- | --- | --- | --- |
| Webhosting | gering | nein | niedrig | nur reines MediaWiki |
| Rechner zu Hause + DynDNS | hoch | ja | Stromkosten | nicht empfohlen |
| Container-VPS (LXC) | mittel | teilweise | sehr niedrig | eingeschränkt |
| KVM-VPS | hoch | ja | niedrig, fest | empfohlen für den Anfang |
| Cloud-Server | hoch | ja | nach Nutzung | gut zum Testen |
| Dedizierter Server | sehr hoch | ja | höher, fest | für große Sammlungen und schnelles Ollama |

## Wo der Server stehen sollte

Eine Wissenssammlung enthält oft auch personenbezogene Daten – schon Benutzerkonten mit Namen und E-Mail-Adressen zählen dazu. Für deren Verarbeitung gilt in der Europäischen Union die Datenschutz-Grundverordnung (DSGVO). Steht der Server in der EU, ist die rechtliche Lage am einfachsten. Bei einem Standort außerhalb der EU muss man zusätzlich prüfen und vertraglich absichern, dass das Schutzniveau der DSGVO eingehalten wird.

Deshalb die Empfehlung: einen Anbieter mit Rechenzentrum in Deutschland oder in einem anderen EU-Land wählen. Zwei Dinge sollte man dabei zusätzlich beachten:

- **Wo sitzt das Unternehmen?** Auch ein Anbieter mit Rechenzentrum in der EU kann seinen Firmensitz außerhalb haben. Für die einfachste Rechtslage sollten beide in der EU liegen.
- **Auftragsverarbeitungsvertrag.** Seriöse Anbieter stellen einen solchen Vertrag bereit, der regelt, wie sie mit den Daten umgehen. Fehlt er, ist das ein schlechtes Zeichen.

## Bandbreite und Traffic

Zwei Begriffe tauchen in Serverangeboten ständig auf und werden oft verwechselt:

- **Bandbreite** ist die Geschwindigkeit der Anbindung – wie viele Daten pro Sekunde durch die Leitung passen, angegeben in Megabit oder Gigabit pro Sekunde (Mbit/s, Gbit/s).
- **Traffic** ist die Datenmenge, die pro Monat insgesamt übertragen wird, angegeben in Gigabyte oder Terabyte (GB, TB).

Für eine Wissenssammlung mit wenigen Nutzern sind beide Werte selten ein Problem. Übliche VPS-Angebote nennen eine Anbindung von 1 Gbit/s (die man sich mit anderen Kunden teilt) und entweder eine großzügige Traffic-Grenze von mehreren Terabyte oder gar keine feste Grenze bei „fairer Nutzung“. Das reicht für Texte, Bilder und die Antworten des Sprachmodells bequem aus.

Hohe Bandbreiten von mehreren Hundert Mbit/s fest zugesichert oder gar mehrere Gbit/s braucht man erst, wenn viele Menschen gleichzeitig große Dateien oder Videos abrufen. Das ist bei einem privaten oder betrieblichen Wiki normalerweise nicht der Fall.

Wichtiger als große Zahlen ist ein Blick ins Kleingedruckte: Gibt es eine Traffic-Grenze, und was passiert, wenn sie überschritten wird? Manche Anbieter drosseln dann nur die Geschwindigkeit bis zum Monatsende, andere berechnen jedes zusätzliche Gigabyte. Ersteres ist harmlos, Letzteres kann teuer werden.

## Anbieter

Die folgenden Anbieter haben Rechenzentren in Deutschland oder der EU und sind seit Jahren am Markt. Angebote und Preise ändern sich laufend, deshalb sollte man vor dem Abschluss die aktuelle Leistungsbeschreibung, den genauen Standort und die Vertragsbedingungen prüfen und aktuelle Erfahrungsberichte lesen.

**KVM-VPS und Cloud-Server:**

- **Hetzner** (Deutschland und Finnland) – „Hetzner Cloud“ für nach Nutzung abgerechnete Server, dazu feste VPS-Pakete.
- **netcup** (Deutschland und Österreich) – feste Monatspakete, oft mit viel Arbeitsspeicher zum kleinen Preis.
- **IONOS** (Deutschland) – VPS- und Cloud-Angebote des Anbieters, der früher 1&1 hieß.
- **Contabo** (Firmensitz in Deutschland, Rechenzentren auch außerhalb der EU – auf den Standort achten) – viel Leistung für wenig Geld.
- **OVHcloud** und **Scaleway** (Frankreich) – große europäische Anbieter mit breiter Auswahl.

**Dedizierte Server:**

- **Hetzner** – neben aktueller Hardware auch eine „Serverbörse“ mit gebrauchten Geräten zu deutlich niedrigeren Preisen.
- **netcup** und **IONOS** – ebenfalls dedizierte Server aus deutschen Rechenzentren.
- **OVHcloud** mit den günstigeren Marken **Kimsufi** und **So you Start** – preiswerte dedizierte Server, teils an Standorten in Frankreich oder Kanada (Standort prüfen).

Daneben gibt es viele kleinere Anbieter und Wiederverkäufer mit teils sehr günstigen Preisen. Bei ihnen lohnt ein besonders genauer Blick auf Firmensitz, Standort der Technik, Support und Erfahrungsberichte, bevor man dort wichtige Daten ablegt.

## Sonderfall: statische Seite mit Git veröffentlichen

Nicht jede Sammlung braucht einen echten Server. Wenn die Inhalte nur aus Texten bestehen, die sich selten ändern, und niemand direkt auf der Webseite schreibt, reicht eine **statische Seite**: fertig gebaute HTML-Dateien ohne Datenbank und ohne laufendes Programm im Hintergrund. Dieses Handbuch selbst ist ein Beispiel – es wird aus Textdateien zu HTML gebaut und als solche Seite veröffentlicht.

Der übliche Ablauf: Man schreibt die Texte, verwaltet sie mit dem Versionsverwaltungs-Programm Git (siehe das Kapitel [Inhalts-Software selbst betreiben](../grundlagen/inhalt-software-verwalten.md) für den Umgang mit Git) und schickt sie zu einem Dienst, der daraus die Webseite baut und ausliefert.

**Fertige Dienste:**

- **GitHub Pages** – kostenloser Webseiten-Dienst der Plattform GitHub. Weit verbreitet und zuverlässig.
- **Codeberg Pages** – dasselbe bei Codeberg, einem gemeinnützigen Verein mit Sitz in Deutschland. Wer Wert auf einen europäischen Anbieter legt, ist hier gut aufgehoben.

Beide liefern nur fertige Dateien aus und verarbeiten kaum personenbezogene Daten der Besucher, was die rechtliche Lage einfach hält.

**Selbst betreiben:**

- **All-in-One: eine Git-Forge mit Pages-Funktion.** Eine „Forge“ ist eine Software, die Git-Projekte im Browser verwaltet. **Forgejo** ist eine solche quelloffene Forge – dieselbe Software, auf der auch Codeberg läuft – und bringt mit „Forgejo Pages“ eine eingebaute Funktion zum Veröffentlichen statischer Seiten mit. Damit hat man Projektverwaltung und Webseiten-Auslieferung aus einer Hand auf dem eigenen Server.
- **Minimalistisch: Webserver plus Git-Hook.** Man betreibt einen schlanken Webserver wie **Nginx** und richtet einen „Git-Hook“ ein – ein kleines Skript, das bei jedem Hochladen neuer Texte automatisch die Seite neu baut und in das Verzeichnis legt, aus dem der Webserver ausliefert. Wenig Software, dafür etwas mehr Handarbeit bei der Einrichtung.

Beide Wege laufen problemlos auf einem der oben beschriebenen KVM-VPS.

## Fazit

Für eine selbst betriebene Wissenssammlung ist ein **KVM-VPS** bei einem Anbieter mit Rechenzentrum in Deutschland oder der EU der beste Einstieg: volle Kontrolle über die eigenen Programme, fester und niedriger Monatspreis, später erweiterbar. Ein **Cloud-Server** hilft beim Ausprobieren, ein **dedizierter Server** lohnt sich für große Sammlungen und ein schnell antwortendes Ollama. Webhosting und ein Rechner zu Hause reichen für diesen Aufbau nicht aus. Wer nur selten geänderte Texte veröffentlicht, kommt ganz ohne Server aus und nutzt einen Pages-Dienst wie GitHub Pages oder Codeberg Pages oder betreibt eine Git-Forge wie Forgejo selbst.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
