# Docs-as-Code

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Dieses Buch ist selbst nach dem Verfahren entstanden, um das es in diesem Kapitel geht. Jedes Kapitel ist eine einfache Textdatei im Format Markdown unter dem Ordner `src/`. Aus diesen Dateien baut ein kleines Programm die fertigen HTML-Seiten. Die Textdateien liegen in einer Versionsverwaltung, jede Änderung ist nachvollziehbar, und der Bau läuft automatisch. Dieses Vorgehen heißt **Docs-as-Code** – auf Deutsch etwa „Dokumentation wie Programmcode".

Das Kapitel erklärt, was Docs-as-Code ist, was „robust" in diesem Zusammenhang bedeutet, und stellt die wichtigsten Gruppen von Werkzeugen vor: Buch-Generatoren, Dokumentations-Frameworks, Notizbuch-Systeme mit ausführbarem Code, Generatoren für lokale Notizsammlungen sowie die neueren Ansätze mit Künstlicher Intelligenz. Bei jeder Gruppe geht es besonders um die Frage, welche Werkzeuge als ausgereift gelten und auch sehr große Textmengen zuverlässig verarbeiten.

## Was Docs-as-Code ist

Programmierer schreiben ihren Code als reinen Text, legen ihn in eine Versionsverwaltung, lassen Änderungen von Kollegen prüfen und bauen daraus mit einem festen Ablauf ein lauffähiges Programm. Docs-as-Code überträgt genau diesen Ablauf auf Dokumentation. Fünf Bausteine gehören dazu:

- **Reiner Text als Quelle.** Der Inhalt wird in einer einfachen Auszeichnungssprache geschrieben – meist **Markdown**, seltener **AsciiDoc** oder **reStructuredText**. Das sind Textdateien, die man mit jedem Editor öffnen kann. Formatierung wie Überschriften oder Listen steht als Zeichen im Text (`# Überschrift`, `- Listenpunkt`), nicht in einem unsichtbaren Format wie bei einem Textverarbeitungsprogramm.
- **Versionsverwaltung mit Git.** Jede Änderung wird gespeichert, mit Datum, Verfasser und einer kurzen Beschreibung. Man kann jederzeit sehen, was sich zwischen zwei Ständen geändert hat, und zu einem alten Stand zurückkehren.
- **Prüfung vor der Übernahme.** Änderungen laufen über einen sogenannten Merge Request oder Pull Request: Erst schlägt jemand eine Änderung vor, dann sieht ein anderer sie durch und übernimmt sie. So wie bei einem Vier-Augen-Prinzip.
- **Automatischer Bau.** Sobald eine Änderung übernommen ist, baut ein Server ohne weiteres Zutun die Webseite neu. Dieser Ablauf heißt **CI/CD** (englisch für „fortlaufende Integration und Auslieferung").
- **Ein Generator erzeugt die Ausgabe.** Ein Programm, der **Static-Site-Generator** (Generator für statische Webseiten), macht aus den Textdateien fertige HTML-Seiten, oft auch eine PDF- oder E-Book-Fassung. „Statisch" heißt: Es entstehen einfache Dateien, die jeder Webserver ohne Datenbank ausliefern kann.

Der Nutzen: Der Text gehört einem selbst und steckt nicht in einem fremden Dienst fest. Man kann ihn durchsuchen, mit anderen Ständen vergleichen, offline bearbeiten und mit denselben Werkzeugen verwalten wie Code. Der Nachteil: Man muss den Umgang mit Git lernen, und es gibt keine Oberfläche, in der man den Text wie in einem Textverarbeitungsprogramm formatiert sieht, während man ihn schreibt.

### Abgrenzung zum Wiki

Die Wissenssammlung dieses Buchs läuft nicht nach Docs-as-Code, sondern auf einem **Wiki** (MediaWiki). Das ist Absicht. Ein Wiki ist eine laufende Anwendung mit Datenbank, in der viele Menschen direkt im Browser schreiben, ohne Git zu kennen. Docs-as-Code passt besser, wenn wenige Personen an einem abgeschlossenen Text arbeiten, die Änderungen streng geprüft werden sollen und der Text auch als Buch erscheinen soll. Beide Ansätze schließen sich nicht aus – dieses Projekt nutzt beide nebeneinander.

## Was „robust" bei Docs-as-Code bedeutet

Die Frage „Welches Werkzeug verkraftet eine Million Textinhalte, ohne kaputtzugehen?" zielt auf einen anderen Punkt als bei einer laufenden Anwendung. Fertige HTML-Seiten auszuliefern ist für jeden Webserver mühelos, egal wie viele es sind – das ist der schnelle Teil, „auf dem Blech" (bare metal) ohne Zwischenschichten. Der Engpass liegt woanders:

- **Die Bauzeit.** Aus einer Million Textdateien alle HTML-Seiten zu erzeugen, kann je nach Generator Sekunden oder viele Stunden dauern. Ausgereifte Generatoren bauen **inkrementell**: Sie erzeugen nach einer Änderung nur die betroffenen Seiten neu, nicht alles.
- **Der Suchindex.** Viele Generatoren bauen eine Volltextsuche ein, die im Browser läuft. Bei sehr vielen Seiten wird diese Index-Datei zu groß zum Herunterladen. Dann braucht es eine serverseitige Suche (siehe [Inhalts-Software selbst betreiben](../grundlagen/inhalt-software-verwalten.md)).
- **Verschiedene Inhaltstypen und Datensätze.** Neben Fließtext fallen Bilder, Tabellen, Codebeispiele, Kennzahlen und ganze Datensätze an. Gute Werkzeuge trennen Inhalt und Daten: Der Text steht in Markdown, feste Angaben stehen im Kopf der Datei (**Frontmatter**) oder in getrennten Datendateien (YAML, JSON, CSV), aus denen der Generator Seiten oder Tabellen erzeugt.
- **Der Reifegrad.** Ausgereift heißt: Das Werkzeug gibt es seit vielen Jahren, es hat einen festen Veröffentlichungsplan, eine große Gemeinschaft, eine klare Vorgehensweise beim Umstieg auf neue Versionen und bekannte, große Projekte, die es einsetzen. Das ist der Unterschied zwischen einem Werkzeug, auf das man ein Buch für zehn Jahre stellt, und einem, das nach zwei Jahren nicht mehr gepflegt wird.

Einen einheitlichen, neutralen Geschwindigkeitsvergleich wie bei Webframeworks gibt es für Dokumentations-Generatoren nicht. Als grobe Ordnung gilt: Generatoren, die als eine einzige eigenständige Programmdatei ausgeliefert werden (geschrieben in Go oder Rust), bauen am schnellsten; Generatoren auf Basis von Python liegen im Mittelfeld; Generatoren, die im Bau ein ganzes Webprogramm mitpacken (JavaScript), sind bei sehr vielen Seiten am langsamsten. Der einzige verlässliche Test ist, die eigene Textmenge nachzubilden und die Bauzeit zu messen.

## Buch-Generatoren (Markdown/AsciiDoc)

Diese Werkzeuge erzeugen ein Buch: einen durchgehenden Text von vorne nach hinten, mit Kapiteln, Inhaltsverzeichnis und meist auch einer PDF- oder E-Book-Fassung. Sie eignen sich für Handbücher, Lehrbücher und Nachschlagewerke – überall dort, wo der Inhalt eine feste Reihenfolge hat.

| Werkzeug | Sprache des Werkzeugs | Auszeichnungssprache | Besonders geeignet für |
| --- | --- | --- | --- |
| **mdBook** | Rust | Markdown | Schlanke Handbücher, sehr schneller Bau; nutzt dieses Buch |
| **Sphinx** | Python | reStructuredText, Markdown | Sehr große Nachschlagewerke mit vielen Querverweisen |
| **Antora** | JavaScript | AsciiDoc | Sehr große Dokumentation aus vielen getrennten Quellen, mit Versionen |
| **Quarto** | – (eigenständig) | Markdown | Wissenschaftliche Bücher mit Rechenergebnissen im Text |
| **Bookdown** | R | Markdown | Bücher aus der Statistik-Welt |
| **Pandoc** | Haskell | Markdown u. v. m. | Umwandlung in nahezu jedes Ausgabeformat |

**Sphinx** gilt als das ausgereifteste Werkzeug für sehr umfangreiche Dokumentation. Es gibt es seit 2008, die Dokumentation der Programmiersprache Python und die des Linux-Kernels werden damit gebaut. Seine Stärke sind Querverweise und Stichwortverzeichnisse über Tausende von Seiten, die auch bei einem Umbau nicht ins Leere zeigen.

**Antora** ist eigens für sehr große, aufgeteilte Dokumentation gebaut: Der Inhalt liegt in vielen getrennten Ablagen, Antora fügt ihn zu einer Webseite zusammen und verwaltet dabei mehrere Versionen nebeneinander. Der Softwarehersteller Red Hat pflegt damit seine Produkthandbücher.

**mdBook** ist bewusst einfach gehalten. Es baut auch große Bücher in Sekunden und kann eingebettete Codebeispiele beim Testen ausführen (siehe unten). Für ein Handbuch wie dieses ist es die passende Größe.

## Dokumentations-Frameworks (moderne Web-Dokumentation, Komponenten, API-Docs)

Diese Werkzeuge stellen nicht das Buch in den Mittelpunkt, sondern die Dokumentations-Webseite eines Produkts: mit eingebauter Suche, mehreren Versionen, mehreren Sprachen und interaktiven Bausteinen auf den Seiten. Häufig wird damit auch die Beschreibung einer Programmierschnittstelle (**API**) dargestellt.

| Werkzeug | Grundlage | Besonders geeignet für |
| --- | --- | --- |
| **Material for MkDocs** | Python (MkDocs) | Der einfache Einstieg; sehr gute eingebaute Suche |
| **Docusaurus** | JavaScript (React), von Meta | Interaktive Bausteine in den Seiten, Versionen, viele Sprachen |
| **Starlight** | JavaScript (Astro) | Moderne, schnelle Dokumentations-Webseiten |
| **VitePress** | JavaScript (Vue) | Schneller Bau; baut die Dokumentation von Vue und Vite |
| **Antora** | JavaScript | Dokumentation aus vielen Quellen und mit Versionen (siehe oben) |
| **Redoc / Swagger UI** | JavaScript | Reine API-Beschreibungen aus einer OpenAPI-Datei |

**Material for MkDocs** ist unter diesen Werkzeugen am weitesten verbreitet. Es ist schnell eingerichtet, die eingebaute Suche ist gut, und es wird von sehr vielen Projekten eingesetzt. **Docusaurus** ist mächtiger, aber aufwendiger: Es erlaubt echte Programmbausteine mitten im Text (Format **MDX**, also Markdown mit eingebauten Bedienelementen) und bringt Versions- und Übersetzungsverwaltung mit. Beide sind bei großen Projekten bewährt; bei sehr vielen Seiten wächst die Bauzeit spürbar, weshalb der inkrementelle Bau wichtig wird.

Für reine API-Beschreibungen erzeugen **Redoc** und **Swagger UI** die Seiten unmittelbar aus einer formalen Beschreibungsdatei (OpenAPI). Ändert sich die Schnittstelle, ändert sich die Dokumentation automatisch mit.

## Notizbuch-Systeme mit ausführbarem Code

Ein „Notizbuch" (englisch *notebook*) mischt Text und Programmcode in einem Dokument. Beim Bau wird der Code ausgeführt, und sein Ergebnis – eine Zahl, eine Tabelle, ein Diagramm – landet direkt im fertigen Text. So bleiben Erklärung und Rechenergebnis immer zusammen und immer aktuell.

| Werkzeug | Sprachen für den Code | Speicherform | Besonderheit |
| --- | --- | --- | --- |
| **Jupyter** | Python, R, Julia u. a. | JSON (`.ipynb`) | Der weit verbreitete Standard |
| **Jupyter Book / MyST** | wie Jupyter | Markdown + Notizbücher | Ganze Bücher aus Notizbüchern; mit Zwischenspeicher |
| **Quarto** | Python, R, Julia, JavaScript | Markdown (`.qmd`) | Ein System für Buch, Webseite und Folien |
| **marimo** | Python | reine Python-Datei (`.py`) | Reagiert wie eine Tabellenkalkulation; gut für Git |
| **Observable Framework** | JavaScript | Markdown | Datenauswertungen und Übersichtsseiten |

Für Docs-as-Code ist die **Speicherform** entscheidend. Klassische Jupyter-Notizbücher liegen als JSON-Datei vor, in der auch die Ausgaben mitgespeichert werden. Solche Dateien lassen sich mit Git schlecht vergleichen – schon ein erneuter Lauf verändert die halbe Datei. **marimo** speichert das Notizbuch als gewöhnliche Python-Datei, **Quarto** und **MyST** als Markdown. Diese Formen zeigen im Vergleich sauber, was sich geändert hat, und passen deshalb besser in den Docs-as-Code-Ablauf.

Der Engpass bei großen Sammlungen ist die Ausführungszeit: Wird bei jedem Bau sämtlicher Code neu gerechnet, dauert es lange. Ausgereifte Systeme haben deshalb einen **Zwischenspeicher** (Jupyter Cache, bei Quarto „freeze" genannt): Nur geänderter Code wird neu ausgeführt, der Rest kommt aus dem Speicher.

Auch **mdBook** kennt eine einfache Form davon: `mdbook test` führt die in den Text eingebetteten Rust-Beispiele aus und meldet, wenn eines nicht mehr stimmt.

## Generatoren für lokale Notizsammlungen

Hier geht es um die persönliche Wissensablage: viele kleine, untereinander verlinkte Notizen, lokal auf dem eigenen Rechner geschrieben, oft mit **Rückverweisen** (Backlinks – jede Notiz zeigt, welche anderen Notizen auf sie verweisen). Im Netz wird eine solche veröffentlichte Sammlung „digitaler Garten" genannt.

| Werkzeug | Art | Besonderheit |
| --- | --- | --- |
| **Obsidian** | Schreibprogramm | Arbeitet auf einem Ordner lokaler Markdown-Dateien; sehr große Sammlungen |
| **Quartz** | Generator | Macht aus einem Obsidian-Ordner eine Webseite mit Rückverweisen |
| **Logseq** | Schreibprogramm | Gliederung in Stichpunkten, lokale Markdown-Dateien |
| **Hugo** | Generator (Go) | Extrem schnell, verkraftet Zehntausende Seiten in Sekunden |
| **Zola** | Generator (Rust) | Eine einzige Programmdatei, sehr schneller Bau |
| **Eleventy** | Generator (JavaScript) | Sehr anpassbar |
| **Jekyll** | Generator (Ruby) | Standard bei GitHub Pages, bei vielen Seiten aber langsam |

Für die Sammlung selbst ist **Obsidian** verbreitet; es kommt auch mit vielen Tausend Notizen gut zurecht, weil es einfach lokale Textdateien bearbeitet. Für die Veröffentlichung ist dann der Generator der Engpass. **Quartz** ist eigens für digitale Gärten gemacht und stellt Rückverweise und eine Übersichtskarte dar. Wer allein auf Geschwindigkeit bei sehr vielen Seiten achtet, wählt **Hugo** oder **Zola**: Beide bauen große Sammlungen in unter einer Sekunde bis wenigen Sekunden. **Jekyll** ist am längsten im Umlauf, wird bei großen Sammlungen aber deutlich langsam.

## KI- und LLM-Wiki-Konzepte

Mit den Sprachmodellen sind zwei neuere Ansätze entstanden, bei denen eine Künstliche Intelligenz an der Wissenssammlung mitarbeitet. „LLM" steht für *Large Language Model*, also ein großes Sprachmodell.

**LLM-Wiki.** Ein Agent baut die Wissenssammlung selbst auf – aus vorhandenem Material wie Programmcode oder Dokumenten – und hält sie danach fortlaufend aktuell. Der Gedanke dahinter: Das Verständnis sammelt sich an, statt bei jeder Frage neu erarbeitet zu werden. Bekannte Vertreter sind **DeepWiki**, das aus einer Code-Ablage auf GitHub ein durchblätterbares Wiki mit Chat-Funktion erzeugt, sowie **OpenWiki** und ähnliche quelloffene Werkzeuge, die ein solches Wiki über die Änderungen im Git regelmäßig nachführen.

**Co-Wiki.** Mensch und Agent bearbeiten dieselbe Sammlung gemeinsam: Der Agent schreibt Entwürfe, ergänzt fehlende Abschnitte oder beantwortet Fragen auf Grundlage der vorhandenen Artikel; ein Mensch prüft und gibt frei. Dienste wie GitBook oder Mintlify bauen eine solche KI-Unterstützung fest in die Dokumentations-Webseite ein.

Wichtig ist dabei die redaktionelle Kontrolle. Von einer KI erzeugte Inhalte können falsch sein und müssen geprüft werden, bevor sie stehen bleiben – genau das schreibt auch die Arbeitsweise dieses Buchs vor (Prüfschritte und der Transparenzhinweis nach Art. 50 EU AI Act). Die verlässliche Fassung bleibt der von Menschen geprüfte Text unter Git; das KI-Wiki ist eine Zuarbeit, keine Quelle der Wahrheit.

## KI-Agenten-Tauglichkeit und RAG-Integration

Docs-as-Code ist die beste Grundlage, wenn ein KI-Agent (siehe [KI-Agenten auf dem Server](../server-einrichten/ki-agent.md)) auf die Inhalte zugreifen soll. Der übliche Weg dafür heißt **RAG** (englisch für „Erzeugung mit Anreicherung durch Abruf"): Der Agent sucht zuerst die passenden Textstellen heraus und schreibt seine Antwort dann nur auf deren Grundlage. Reiner Markdown-Text hilft dabei an mehreren Stellen:

- **Sauberes Zerteilen.** RAG zerlegt die Texte in Häppchen. Überschriften in Markdown sind natürliche Trennstellen – jeder Abschnitt wird ein sinnvolles Häppchen.
- **Feste Adressen.** Jede Seite und jeder Abschnitt hat eine feste Web-Adresse. Der Agent kann seine Antwort damit belegen und auf die Stelle verweisen.
- **Angaben im Dateikopf.** Das Frontmatter (Titel, Datum, Schlagworte) lässt sich zum Filtern nutzen.
- **Git-Verlauf.** Der Agent kann erkennen, was sich seit wann geändert hat.

Dazu gibt es einen jungen Standard namens **`llms.txt`**: eine Datei im Wurzelverzeichnis der Webseite, die alle wichtigen Seiten mit einer kurzen Beschreibung auflistet – ähnlich wie `robots.txt` für Suchmaschinen. Oft kommt eine zweite Datei `llms-full.txt` dazu, die den gesamten Inhalt als reinen Markdown-Text enthält. Für einen Agenten ist das viel sparsamer als die HTML-Seiten: Der Verbrauch an Recheneinheiten (Token) sinkt um etwa 90 Prozent. Nach Angaben des Anbieters GitBook stammen inzwischen rund 40 Prozent der Zugriffe auf Dokumentations-Seiten von KI-Agenten und nicht mehr von Menschen.

Manche Werkzeuge erzeugen `llms.txt` von selbst (etwa Mintlify), für andere gibt es Zusatzbausteine (für MkDocs und Docusaurus). Den Abruf durch den Agenten übernehmen entweder eine selbst gebaute RAG-Kette (mit Bausteinkästen wie LlamaIndex, LangChain oder Haystack) oder ein fertiger Dienst, der die Dokumentation als Chat zugänglich macht.

## Reifegrad im Überblick

| Aufgabe | Ausgereifte Wahl für große Mengen | Warum |
| --- | --- | --- |
| Handbuch, schlank | mdBook | Sehr schneller Bau, einfache Struktur |
| Nachschlagewerk, sehr groß | Sphinx | Seit 2008, Querverweise über Tausende Seiten, große Projekte |
| Dokumentation aus vielen Quellen, mit Versionen | Antora | Eigens dafür gebaut, im Einsatz bei großen Herstellern |
| Produkt-Dokumentation mit Suche | Material for MkDocs, Docusaurus | Weit verbreitet, bei großen Projekten bewährt |
| Text mit Rechenergebnissen | Quarto, Jupyter Book | Zwischenspeicher für den Code, mehrere Sprachen |
| Lokale Notizsammlung, sehr viele Seiten | Hugo, Zola | Bau in Sekunden auch bei Zehntausenden Seiten |
| Zugriff durch KI-Agenten | jedes Markdown-Werkzeug + `llms.txt` | Reiner Text zerteilt und belegt sich sauber |

## Für dieses Buch

Dieses Handbuch nutzt **mdBook**. Das passt: Der Inhalt hat eine feste Reihenfolge, die Menge bleibt überschaubar, der Bau ist in Sekunden fertig, und eingebettete Codebeispiele lassen sich testen.

Die eigentliche Wissenssammlung läuft bewusst nicht nach Docs-as-Code, sondern als **Wiki** – weil dort viele Personen ohne Git-Kenntnisse direkt im Browser schreiben sollen. Beide Wege bestehen nebeneinander.

Für die Zusammenarbeit mit einem KI-Agenten gilt: Die Kapitel bleiben reiner Markdown-Text, und sobald die Webseite online geht, kommt eine `llms.txt`-Datei dazu. Der Agent bekommt sowohl das Buch als auch das Wiki über eine RAG-Kette zugänglich gemacht.

Sollte das Buch sehr stark wachsen oder mehrere Versionen nebeneinander brauchen, sind **Antora** oder **Sphinx** die ausgereiften Alternativen.

## Fazit

Docs-as-Code heißt, Dokumentation wie Programmcode zu behandeln: als reinen Text in Git, mit Prüfung vor der Übernahme und automatischem Bau. Ob ein Werkzeug „eine Million Inhalte aushält", entscheidet sich nicht am Ausliefern der Seiten – das ist für jeden Webserver mühelos –, sondern an der Bauzeit, am Suchindex und an der sauberen Trennung von Text und Daten. Als besonders ausgereift gelten **Sphinx** und **Antora** für sehr große Nachschlagewerke, **Material for MkDocs** und **Docusaurus** für Produkt-Dokumentation, **mdBook** für schlanke Handbücher, **Quarto** und **Jupyter Book** für Text mit Rechenergebnissen sowie **Hugo** und **Zola** für sehr große lokale Notizsammlungen. Von KI erzeugte Wikis sind eine nützliche Zuarbeit, ersetzen aber nicht die redaktionelle Prüfung. Für den Zugriff durch KI-Agenten ist reiner Markdown-Text zusammen mit einer `llms.txt`-Datei die beste Grundlage. Für dieses Buch bleibt es bei mdBook, mit dem Wiki daneben und einer geplanten `llms.txt` für den Agenten.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
