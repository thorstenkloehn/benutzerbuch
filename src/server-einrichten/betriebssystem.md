# Betriebssystem

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Ist der Server gemietet (siehe [Server mieten](./server-mieten.md)), fehlt ihm noch das Wichtigste: ein Betriebssystem. Das Betriebssystem ist das Grundprogramm, das den Rechner überhaupt bedienbar macht. Es verwaltet Festplatte, Arbeitsspeicher und Netzwerk und sorgt dafür, dass andere Programme starten können. Ohne Betriebssystem läuft kein einziges der Programme aus dem Kapitel [Inhalts-Software selbst betreiben](../grundlagen/inhalt-software-verwalten.md).

Die meisten Anbieter lassen die Wahl bei der Bestellung oder später per Mausklick im Kundenbereich. Dieses Kapitel erklärt, welche Betriebssysteme zur Auswahl stehen, wie sie zusammenhängen, und welches sich für eine selbst betriebene Wissenssammlung eignet.

## Warum fast immer Linux

Auf Servern läuft überwiegend **Linux** – eine Familie kostenloser, quelloffener Betriebssysteme. Dafür gibt es handfeste Gründe:

- Es kostet nichts und darf ohne Lizenzgebühren betrieben werden.
- Es läuft sparsam: Ein Linux-Server braucht keine grafische Oberfläche und kommt mit wenig Arbeitsspeicher aus.
- Fast alle Server-Programme werden zuerst für Linux entwickelt. MediaWiki, PostgreSQL, Meilisearch und Ollama sind dafür gemacht; ihre Anleitungen gehen von Linux aus.
- Es läuft über Jahre ohne Neustart und lässt sich vollständig über die Textkonsole fernsteuern.

Eine Linux-Ausgabe nennt man **Distribution** (kurz „Distro"). Alle Distributionen teilen denselben Kern, unterscheiden sich aber darin, wie neu ihre Programme sind, wie lange es Sicherheitsupdates gibt und wie sie bedient werden. Die folgende Übersicht ordnet die üblichen Angebote.

## Die zwei großen Linux-Familien

Fast jede Server-Distribution stammt aus einer von zwei Linien. Der wichtigste sichtbare Unterschied ist der Befehl, mit dem man Programme installiert.

- **Debian-Familie** (Debian, Ubuntu). Programme werden mit dem Befehl `apt` installiert. Pakete tragen die Endung `.deb`.
- **Red-Hat-Familie** (CentOS, AlmaLinux, Rocky Linux, Fedora). Programme werden mit dem Befehl `dnf` installiert. Pakete tragen die Endung `.rpm`.

Beide Familien sind solide. Anleitungen im Internet nennen oft nur Befehle für eine der beiden – wer die Familie seines Servers kennt, weiß dann, welche Zeile für ihn gilt.

## Debian-Familie

### Debian

Debian ist eine der ältesten Linux-Distributionen und wird von einer weltweiten Gemeinschaft Freiwilliger gepflegt, nicht von einer Firma. Die stabile Ausgabe erscheint etwa alle zwei Jahre. Während ihrer Laufzeit ändert sich an den Programmen nur wenig: Es kommen Sicherheitskorrekturen, aber keine großen neuen Versionen. Mit der zusätzlichen Langzeitpflege erhält jede Ausgabe rund fünf Jahre lang Updates.

Genau diese Ruhe ist auf einem Server ein Vorteil. Man richtet den Rechner einmal ein und muss sich jahrelang nur um Sicherheitsupdates kümmern. Debian gilt als sparsam, zuverlässig und gut dokumentiert.

### Ubuntu Server

Ubuntu baut auf Debian auf und wird von der Firma Canonical herausgegeben. Interessant für Server sind die **LTS-Ausgaben** („Long Term Support", zu Deutsch Langzeitunterstützung). Sie erscheinen alle zwei Jahre im April und werden fünf Jahre lang mit Updates versorgt; kostenlos lässt sich dieser Zeitraum für wenige private Rechner auf zehn Jahre verlängern.

Ubuntu bringt in der Regel etwas neuere Programmversionen mit als Debian und hat die größte Nutzergemeinde überhaupt. Zu fast jedem Problem findet sich eine Anleitung, die ausdrücklich Ubuntu nennt. Für Einsteiger ist das ein spürbarer Vorteil. Wichtig: Nur die LTS-Ausgaben nehmen; die Zwischenversionen (etwa 25.10) werden schon nach neun Monaten nicht mehr gepflegt.

## Red-Hat-Familie

Hintergrund: Die Firma Red Hat verkauft ein kostenpflichtiges Server-Linux namens „Red Hat Enterprise Linux" (RHEL) mit langer Update-Garantie. Rund um dieses Produkt sind mehrere kostenlose Distributionen entstanden.

### CentOS

CentOS war jahrelang die kostenlose Version von RHEL – gleiche Software, keine Support-Verträge, keine Gebühr. Diese klassische Form („CentOS Linux") wurde jedoch eingestellt. Die letzte Ausgabe, CentOS Linux 7, erhielt Mitte 2024 ihr letztes Update.

Heute gibt es nur noch **CentOS Stream**. Das ist eine Vorschau auf die jeweils nächste RHEL-Version: Änderungen landen hier zuerst und wandern später nach RHEL. Für einen Server, der einfach nur ruhig laufen soll, ist das die falsche Richtung – man bekommt Neuerungen früher, aber weniger erprobt. Wer den bewährten „kostenloses RHEL"-Gedanken sucht, greift zu einer der beiden folgenden Distributionen.

### AlmaLinux

AlmaLinux ist als Nachfolger des eingestellten CentOS entstanden und wird von einer gemeinnützigen Stiftung getragen, die von der Firma CloudLinux gegründet wurde. Es ist so gebaut, dass Programme, die für RHEL gedacht sind, unverändert laufen. Jede Ausgabe wird rund zehn Jahre lang mit Sicherheitsupdates versorgt – deutlich länger als bei Debian oder Ubuntu.

### Rocky Linux

Rocky Linux verfolgt dasselbe Ziel wie AlmaLinux und ist ebenfalls als CentOS-Nachfolger gestartet. Gegründet hat es einer der ursprünglichen CentOS-Mitgründer; hinter dem Projekt steht eine eigene Stiftung. Auch hier gilt: Programme für RHEL laufen unverändert, und jede Ausgabe wird etwa zehn Jahre lang gepflegt.

Zwischen AlmaLinux und Rocky Linux muss man sich für diesen Aufbau nicht lange entscheiden – beide sind gleichwertige, seriöse Wahl. Sie sind vor allem dann interessant, wenn man die besonders lange Update-Dauer schätzt.

### Fedora Server

Fedora ist das Feld, auf dem Red Hat neue Technik ausprobiert, bevor sie Jahre später in RHEL landet. Entsprechend neu ist die mitgelieferte Software – und entsprechend kurz die Pflege: Jede Ausgabe erhält nur etwa 13 Monate lang Updates, danach muss man auf die nächste wechseln.

Für einen Server, der lange unverändert laufen soll, bedeutet das häufige, größere Umstellungen. Fedora Server eignet sich für alle, die bewusst mit aktueller Technik arbeiten wollen und die regelmäßige Pflege einplanen. Für eine Wissenssammlung, die einfach nur verfügbar sein soll, ist es mehr Aufwand als nötig.

## Weitere Angebote

### Arch Linux

Arch Linux kennt keine festen Ausgaben. Es wird laufend aktualisiert („Rolling Release"): Jedes Programm ist immer auf dem neuesten Stand. Das Grundsystem wird bewusst schlank ausgeliefert, alles Weitere baut man selbst zusammen. Das Arch-Wiki ist eine der besten Linux-Anleitungssammlungen im Netz, wird aber auch von Nutzern anderer Distributionen gelesen.

Für einen Server bringt der ständige Fluss neuer Versionen ein Risiko mit: Nach einem Update kann sich das Verhalten eines Programms ändern, und man muss zeitnah nachbessern. Wer den Server nicht täglich im Blick hat, fährt mit einer Distribution mit festen Ausgaben ruhiger. Arch richtet sich an erfahrene Nutzer, die ihr System genau kennen wollen.

### CloudLinux

CloudLinux ist ein kostenpflichtiges Server-Linux auf RHEL-Grundlage. Es ist für Firmen gedacht, die Webspace an viele Kunden vermieten: Es kann jedem Kunden feste Grenzen für Rechenleistung setzen und die Kunden streng voneinander abschotten, damit einer allein nicht den ganzen Server auslastet.

Für eine einzelne, selbst betriebene Wissenssammlung gibt es keine solchen Mitnutzer. Die Zusatzfunktionen und die laufenden Kosten von CloudLinux bringen hier keinen Nutzen. Diese Distribution taucht in Anbieterlisten nur auf, weil viele Anbieter selbst im Webhosting-Geschäft sind.

### Proxmox VE

Proxmox VE ist ein Sonderfall. Es ist kein Betriebssystem, auf dem man MediaWiki und die anderen Programme direkt installiert, sondern ein **Wirtssystem für virtuelle Maschinen**. Auf einem physischen Rechner mit Proxmox legt man über eine Weboberfläche mehrere abgetrennte virtuelle Server an, jeder mit eigenem Betriebssystem.

Das ist nützlich, wenn man eigene Hardware besitzt – etwa einen Rechner zu Hause oder einen gemieteten dedizierten Server – und daraus mehrere getrennte Server machen will. Für einen bereits fertig gemieteten VPS ergibt Proxmox dagegen keinen Sinn: Der Anbieter hat die Virtualisierung schon übernommen. Man würde eine virtuelle Maschine in eine virtuelle Maschine setzen, was nur Leistung kostet. Innerhalb einer der virtuellen Maschinen von Proxmox nimmt man dann wieder eine der oben genannten Distributionen, meist Debian.

### Windows Server

Windows Server ist die Server-Ausgabe von Microsoft Windows. Sie ist kostenpflichtig, und die Lizenz richtet sich nach der Zahl der Rechenkerne; für einen kleinen Server kommen so schnell mehrere Hundert Euro zusammen.

Der hier beschriebene Aufbau spricht klar gegen Windows: MediaWiki, PostgreSQL, Meilisearch und Ollama stammen alle aus der Linux-Welt. Zwar gibt es für manche davon auch Windows-Fassungen, doch die Anleitungen, Erfahrungsberichte und Hilfsprogramme gehen fast immer von Linux aus. Windows Server lohnt sich nur, wenn ohnehin andere Windows-Programme auf dem Rechner laufen müssen.

## Kurzvergleich

| Distribution | Familie | Neue Software | Update-Dauer je Ausgabe | Für diesen Aufbau |
| --- | --- | --- | --- | --- |
| Debian | Debian | zurückhaltend | ca. 5 Jahre | empfohlen |
| Ubuntu Server (LTS) | Debian | moderat | 5 Jahre (privat bis 10) | empfohlen |
| CentOS Stream | Red Hat | Vorschau auf RHEL | laufend | nicht empfohlen |
| AlmaLinux | Red Hat | zurückhaltend | ca. 10 Jahre | gut geeignet |
| Rocky Linux | Red Hat | zurückhaltend | ca. 10 Jahre | gut geeignet |
| Fedora Server | Red Hat | sehr neu | ca. 13 Monate | zu viel Pflege |
| Arch Linux | eigenständig | immer neu | laufend | nur für Fortgeschrittene |
| CloudLinux | Red Hat | zurückhaltend | an RHEL angelehnt | kostenpflichtig, unnötig |
| Proxmox VE | Debian | — | — | nur für eigene Hardware |
| Windows Server | Windows | — | lange, kostenpflichtig | nur bei Windows-Zwang |

## Was bei dieser Wissenssammlung zählt

Drei Fragen entscheiden, welche Wahl praktisch ist:

- **Wie lange läuft der Server ohne größere Umstellung?** Je länger eine Ausgabe Updates bekommt, desto seltener muss man das ganze System anheben. Debian und Ubuntu LTS liegen bei rund fünf Jahren, AlmaLinux und Rocky Linux bei rund zehn. Fedora und Arch verlangen laufende Aufmerksamkeit.
- **Wie leicht findet man Hilfe?** Für Debian und besonders Ubuntu gibt es die meisten Anleitungen, auch für genau die Programme aus diesem Buch. Das erspart Einsteigern viel Sucherei.
- **Bekommt man die nötigen Programme fertig verpackt?** PostgreSQL mit der Erweiterung pgvector, Meilisearch und Ollama lassen sich auf Debian und Ubuntu mit wenigen Befehlen einrichten. Auf den Red-Hat-Distributionen geht das ebenfalls, verlangt aber gelegentlich einen zusätzlichen Handgriff.

## Fazit

Für eine selbst betriebene Wissenssammlung ist **Debian** in der stabilen Ausgabe oder **Ubuntu Server** in einer LTS-Ausgabe die beste Wahl: kostenlos, sparsam, jahrelang ruhig im Betrieb und mit der größten Sammlung passender Anleitungen. Wer eine besonders lange Update-Dauer bevorzugt, nimmt **AlmaLinux** oder **Rocky Linux** – beide gleichwertig. **CentOS Stream, Fedora und Arch Linux** bringen neuere Technik, aber mehr Pflegeaufwand, als dieser Aufbau braucht. **CloudLinux** und **Windows Server** kosten Geld ohne Gegenwert für diesen Zweck. **Proxmox VE** ist nur dann sinnvoll, wenn man eigene Hardware in mehrere virtuelle Server aufteilen möchte; auf einem gemieteten VPS gehört es nicht hin.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
