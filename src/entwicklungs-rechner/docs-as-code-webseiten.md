# Docs-as-Code für Webseiten und Blogs

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Das Kapitel [Docs-as-Code](./docs-as-code.md) beschreibt das Verfahren am Beispiel eines Buchs: ein durchgehender Text mit fester Reihenfolge. Für eine allgemeine Webseite oder einen Blog gilt dasselbe Verfahren – reiner Text in Git, Prüfung vor der Übernahme, automatischer Bau –, aber die Anforderungen sind andere. Es gibt keine feste Lesereihenfolge, dafür viele einzelne Beiträge, Übersichtsseiten, Schlagwort-Seiten, Archive nach Jahr und Monat, Autorenseiten und oft auch Seiten, die aus reinen Datensätzen entstehen.

Dieses Kapitel beantwortet die Frage: Welche Werkzeuge für Webseiten und Blogs sind so ausgereift und so robust, dass sie auch sehr große Inhaltsmengen – bis in den Bereich einer Million einzelner Beiträge – zuverlässig verarbeiten, verschiedene Inhaltstypen und Datensätze sauber verwalten, dabei schnell bleiben und sich verlässlich messen lassen?

## Worauf es bei großen Mengen ankommt

Eine fertige Webseite besteht am Ende nur aus HTML-, CSS- und Bilddateien. Diese Dateien auszuliefern ist für jeden Webserver mühelos – ob es hundert oder eine Million sind, spielt kaum eine Rolle. Das ist der schnelle Teil: einfache Dateien direkt „vom Blech" (bare metal), ohne Datenbank, ohne Programm dazwischen. Ein vorgelagertes Auslieferungsnetz (**CDN**, ein Netz aus Zwischenspeichern nah bei den Besuchern) liefert sie mit nahezu Leitungsgeschwindigkeit aus.

Der Engpass liegt nicht beim Ausliefern, sondern beim **Bau**:

- **Die Bauzeit von Grund auf.** Aus einer Million Textdateien alle HTML-Seiten neu zu erzeugen, dauert je nach Werkzeug wenige Minuten oder viele Stunden. Werkzeuge, die für große Mengen taugen, brauchen für die volle Million höchstens einige Minuten.
- **Der inkrementelle Bau.** Nach einer kleinen Änderung darf nicht alles neu gebaut werden, sondern nur die betroffenen Seiten. Ausgereifte Werkzeuge erkennen, welche Seiten von einer Änderung abhängen, und bauen nur diese neu.
- **Der Speicherbedarf.** Manche Werkzeuge halten während des Baus die gesamte Webseite im Arbeitsspeicher. Bei sehr vielen Seiten reicht der Speicher dann nicht mehr, und der Bau bricht ab.
- **Die Suche.** Eine im Browser laufende Volltextsuche lädt eine Index-Datei herunter. Bei sehr vielen Seiten wird diese Datei zu groß. Dann braucht es eine serverseitige Suche (siehe [Inhalts-Software selbst betreiben](../grundlagen/inhalt-software-verwalten.md)).
- **Verschiedene Inhaltstypen und Datensätze.** Neben Beiträgen fallen Bilder, Tabellen, Kennzahlen und ganze Datensätze an. Gute Werkzeuge trennen Text und Daten sauber (siehe unten).

## Was „ausgereift" bedeutet

Ausgereift heißt bei diesen Werkzeugen dasselbe wie im Kapitel [Docs-as-Code](./docs-as-code.md): Das Werkzeug gibt es seit vielen Jahren, es hat einen festen Veröffentlichungsplan, eine große Gemeinschaft, eine klare Vorgehensweise beim Umstieg auf neue Versionen und bekannte große Webseiten, die darauf laufen. Für einen Blog, der zehn Jahre bestehen soll, ist das der entscheidende Punkt – wichtiger als jede einzelne Funktion.

## Die Werkzeuge im Überblick

| Werkzeug | Sprache des Werkzeugs | Im Umlauf seit | Besonders geeignet für |
| --- | --- | --- | --- |
| **Hugo** | Go | 2013 | Sehr große Blogs und Webseiten; schnellster Bau; eine einzige Programmdatei |
| **Zola** | Rust | 2017 | Wie Hugo, sehr schneller Bau, eine Programmdatei; kleinere Gemeinschaft |
| **Jekyll** | Ruby | 2008 | Am längsten erprobt; Standard bei GitHub Pages; bei großen Mengen langsam |
| **Eleventy (11ty)** | JavaScript | 2018 | Sehr anpassbar; schlanke Ausgabe ohne unnötiges JavaScript |
| **Astro** | JavaScript | 2021 | Moderne inhaltslastige Webseiten; Inhalts-Sammlungen mit Prüfung |
| **Next.js** | JavaScript (React) | 2016 | Sehr großes Ökosystem; auch gemischter Betrieb (siehe [Webframework](../server-einrichten/webframework.md)) |
| **Gatsby** | JavaScript (React) | 2015 | War lange verbreitet; Bau bei vielen Seiten sehr langsam; rückläufig |
| **Pelican** | Python | 2010 | Blogs in der Python-Welt |
| **Hexo** | JavaScript | 2012 | Blogs; im asiatischen Raum weit verbreitet |

### Hugo – die ausgereifte Wahl für sehr große Webseiten

**Hugo** wird als eine einzige eigenständige Programmdatei ausgeliefert; es gibt keine weiteren Teile zu installieren. Nach eigenen Angaben baut Hugo eine durchschnittliche Seite in unter einer Millisekunde, sodass selbst Webseiten mit vielen Tausend Seiten in unter einer Sekunde fertig sind. Für eine Million Beiträge bewegt sich der vollständige Bau je nach Rechner im Bereich weniger Minuten. Hugo baut auch inkrementell und meldet auf Wunsch, wie viel Zeit und Speicher der Bau gebraucht hat. Große Nachrichten- und Unternehmensseiten laufen damit. Wenn allein die schiere Menge zählt, ist Hugo die naheliegende Wahl.

Für Fälle, in denen selbst Hugo an Grenzen kommt, kann Hugo den Bau in **Abschnitte** teilen und nur einzelne Abschnitte neu erzeugen. So lässt sich eine sehr große Webseite in mehreren Läufen bauen, statt in einem einzigen.

### Zola – gleiche Idee, andere Grundlage

**Zola** verfolgt denselben Ansatz wie Hugo: eine einzige Programmdatei, sehr schneller Bau, keine Zusatzteile. Es ist in der Sprache Rust geschrieben und in manchen Vergleichen noch etwas schneller als Hugo. Die Gemeinschaft ist kleiner, es gibt weniger fertige Vorlagen und Erweiterungen. Für eine überschaubare, aber sehr große Webseite mit klaren Anforderungen ist Zola eine gute Wahl; wo viele fertige Bausteine gebraucht werden, ist Hugo im Vorteil.

### Jekyll – am längsten erprobt, aber langsam

**Jekyll** ist der Urvater dieser Werkzeuge und die Standard-Grundlage von GitHub Pages. Es ist außerordentlich stabil und gut dokumentiert. Sein Nachteil zeigt sich genau bei der Frage dieses Kapitels: Bei mehreren Tausend Seiten wird der Bau spürbar langsam, bei Hunderttausenden ist er kaum noch praktikabel. Für einen kleinen bis mittleren Blog ist Jekyll solide; für sehr große Mengen ist es die falsche Wahl.

### Eleventy und Astro – flexibel und modern

**Eleventy** ist bewusst schlank und lässt sich weitgehend anpassen. Es erzeugt standardmäßig reines HTML ohne mitgeliefertes JavaScript, was die Seiten leicht macht. Bei sehr großen Mengen liegt es in der Geschwindigkeit im Mittelfeld – schneller als Jekyll, langsamer als Hugo.

**Astro** ist auf inhaltslastige Webseiten ausgelegt. Es liefert ebenfalls überwiegend reines HTML aus und lädt JavaScript nur dort nach, wo es wirklich gebraucht wird (dieses Vorgehen heißt „Inseln", englisch *islands*). Astro bringt **Inhalts-Sammlungen** mit: Man legt für jeden Inhaltstyp fest, welche Angaben im Dateikopf stehen müssen, und der Bau bricht ab, wenn ein Beitrag diese Vorgabe verletzt. Das hält große Sammlungen sauber. Astro ist noch vergleichsweise jung, wächst aber schnell und wird bereits von großen Webseiten eingesetzt.

### Next.js und Gatsby – die React-Werkzeuge

**Next.js** kann Seiten beim Bau als statische Dateien erzeugen, aber auch beim Aufruf durch den Server (siehe [Webframework](../server-einrichten/webframework.md)). Für sehr große Webseiten bietet es einen Mittelweg: Nur die wichtigsten Seiten werden beim Bau erzeugt, die übrigen entstehen beim ersten Aufruf und werden dann zwischengespeichert. So lässt sich eine Million Seiten betreiben, ohne sie alle im Voraus zu bauen. Der Preis dafür ist ein laufender Server statt reiner Dateien.

**Gatsby** war einige Jahre sehr beliebt, gilt inzwischen aber als rückläufig. Der Bau ist bei vielen Seiten langsam und speicherhungrig. Für neue Projekte in großem Umfang ist es nicht die erste Wahl.

## Verschiedene Inhaltstypen und Datensätze verwalten

Ein Blog besteht nicht nur aus Fließtext. Ausgereifte Werkzeuge trennen drei Dinge:

- **Der Text** steht in Markdown, eine Datei je Beitrag.
- **Feste Angaben zum Beitrag** – Titel, Datum, Autor, Schlagworte, Kurzbeschreibung – stehen im Kopf der Datei, dem **Frontmatter**. Daraus baut das Werkzeug automatisch Übersichts-, Schlagwort- und Archivseiten.
- **Reine Datensätze** – etwa eine Liste von Produkten, Orten oder Kennzahlen – stehen in getrennten Datendateien (Formate YAML, JSON, TOML oder CSV). Das Werkzeug erzeugt daraus Tabellen oder ganze Seiten, ohne dass man jede von Hand anlegt.

Größere Redaktionen legen die Inhalte oft nicht als Dateien ab, sondern in einem **Headless-CMS** – einer Inhaltsverwaltung, die eine bequeme Schreiboberfläche bietet, die Inhalte aber nicht selbst anzeigt, sondern nur über eine Schnittstelle herausgibt. Der Generator holt die Inhalte beim Bau von dort ab und macht daraus statische Seiten. So schreiben Redakteure in einer vertrauten Oberfläche, und die ausgelieferte Webseite bleibt trotzdem eine Sammlung einfacher Dateien.

Für Bilder haben Hugo, Astro und Eleventy eine eingebaute **Bildverarbeitung**: Sie erzeugen beim Bau verkleinerte Fassungen in mehreren Größen und modernen Formaten, sodass jeder Besucher nur die passende Größe lädt.

## Messbarkeit: den eigenen Fall nachbauen

Einen einheitlichen, neutralen Geschwindigkeitsvergleich – wie ihn Webframeworks mit dem TechEmpower-Vergleich haben (siehe [Webframework](../server-einrichten/webframework.md)) – gibt es für diese Werkzeuge nicht. Veröffentlichte Vergleiche messen unterschiedliche Seiten, unterschiedliche Vorlagen und unterschiedliche Rechner; ihre Zahlen sind nur grobe Anhaltspunkte.

Der einzige verlässliche Test ist, die eigene Inhaltsmenge nachzubilden und selbst zu messen:

1. **Testinhalte erzeugen.** Ein kleines Skript legt so viele Beispielbeiträge an, wie später erwartet werden – etwa hunderttausend oder eine Million, mit realistischem Frontmatter und realistischer Textlänge.
2. **Den Bau von Grund auf messen.** Wie lange dauert ein vollständiger Bau ohne Zwischenspeicher? Wie viel Arbeitsspeicher wird dabei belegt?
3. **Den inkrementellen Bau messen.** Wie lange dauert der Bau, nachdem sich ein einziger Beitrag geändert hat?
4. **Auf der Zielumgebung messen.** Der Bau-Server im automatischen Ablauf (**CI/CD**) hat oft weniger Leistung als der eigene Rechner. Die Messung zählt nur dort, wo später wirklich gebaut wird.

Hugo bringt für Schritt 2 und 3 eigene Anzeigen mit (Bauzeit je Vorlage, belegter Speicher). Bei den anderen Werkzeugen misst man die Gesamtzeit von außen.

Als grobe Ordnung – wie im Kapitel [Docs-as-Code](./docs-as-code.md) – gilt: Werkzeuge, die als eine einzige Programmdatei in Go oder Rust ausgeliefert werden (**Hugo**, **Zola**), bauen am schnellsten und verkraften die größten Mengen. Werkzeuge auf Basis von Python oder Ruby (**Pelican**, **Jekyll**) liegen im Mittelfeld bis hinten. Werkzeuge, die im Bau ein ganzes Webprogramm mitführen (**Gatsby**, teils **Next.js**), sind bei sehr vielen Seiten am langsamsten und brauchen am meisten Speicher.

## Reifegrad im Überblick

| Anforderung | Ausgereifte Wahl | Warum |
| --- | --- | --- |
| Sehr großer Blog, reine Dateien | Hugo | Bau in Minuten auch bei einer Million Beiträgen; eine Programmdatei; große Gemeinschaft |
| Sehr große Webseite, klare Anforderungen | Hugo, Zola | Schnellster Bau, geringster Speicherbedarf |
| Eine Million Seiten ohne vollständigen Vorab-Bau | Next.js | Wichtige Seiten beim Bau, der Rest beim ersten Aufruf mit Zwischenspeicher |
| Kleiner bis mittlerer Blog, höchste Stabilität | Jekyll | Seit 2008, Standard bei GitHub Pages |
| Inhaltslastige Webseite mit vielen Inhaltstypen | Astro | Inhalts-Sammlungen mit Prüfung; schlanke Ausgabe |
| Schlanke, stark angepasste Ausgabe | Eleventy | Reines HTML, weitgehend anpassbar |

## Für dieses Buch

Dieses Handbuch ist ein Buch mit fester Reihenfolge und nutzt daher [mdBook](./docs-as-code.md). Entstünde stattdessen ein Blog oder eine allgemeine Webseite in großem Umfang, wäre **Hugo** die ausgereifte Wahl: reine Dateien in Git, Bau in Minuten auch bei sehr großen Mengen, saubere Trennung von Text und Daten und eine einzige Programmdatei ohne weitere Abhängigkeiten. Erst wenn die Zahl der Seiten so groß wird, dass sich ein vollständiger Vorab-Bau nicht mehr lohnt, käme der gemischte Betrieb mit **Next.js** in Frage – um den Preis eines laufenden Servers statt reiner Dateien.

## Fazit

Für Webseiten und Blogs gilt dasselbe Docs-as-Code-Verfahren wie für ein Buch, nur mit vielen einzelnen Beiträgen statt einem durchgehenden Text. Ob ein Werkzeug „eine Million Beiträge aushält", entscheidet sich nicht am Ausliefern der Seiten – das ist für jeden Webserver und jedes Auslieferungsnetz mühelos –, sondern an der Bauzeit von Grund auf, am inkrementellen Bau, am Speicherbedarf und an der sauberen Trennung von Text und Daten. Am besten schneiden die Werkzeuge ab, die als eine einzige Programmdatei ausgeliefert werden: **Hugo** und, mit kleinerer Gemeinschaft, **Zola**. **Jekyll** ist am längsten erprobt, aber bei großen Mengen zu langsam. **Astro** und **Eleventy** sind moderne, flexible Alternativen im Mittelfeld. Wo eine Million Seiten nicht mehr sinnvoll im Voraus gebaut werden können, bietet **Next.js** einen Mittelweg mit laufendem Server. Verlässliche Zahlen liefert nur ein eigener Test mit der eigenen Inhaltsmenge auf der eigenen Bau-Umgebung.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
