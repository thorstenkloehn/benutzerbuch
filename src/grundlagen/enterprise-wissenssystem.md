# Enterprise-Wissenssystem

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Ein **Wissenssystem** ist das Programm, in dem viele Personen gemeinsam ein Nachschlagewerk schreiben – verlinkter Fließtext, Artikel, die aufeinander verweisen. Welches Wissenssystem für den Aufbau dieses Buchs gewählt wurde und warum, steht im Kapitel [Wissenssystem](../web-stack/wissensystem.md). Dieses Kapitel setzt darauf auf und stellt die engere Frage aus [Enterprise Web-Stack](./enterprise-web-stack.md): Welches Wissenssystem trägt sehr große Sammlungen, läuft gleichwertig auf **PostgreSQL**, ist **quelloffen** – und lässt sich am besten mit einem Sprachmodell zusammen betreiben?

## Was „KI-Ökosystem" bei einem Wissenssystem heißt

Bei einem Wissenssystem geht es weniger um Beispielcode als bei einem Framework. Wichtiger sind drei Fragen:

- **Kennt das Sprachmodell das Format?** Die Auszeichnungssprache **Wikitext** von MediaWiki hat jedes große Sprachmodell millionenfach gesehen, weil die Wikipedia damit geschrieben ist. Ein Modell kann Wikitext darum fast so sicher lesen und schreiben wie normale Sprache.
- **Kommt ein KI-Agent an die Inhalte?** Für die **RAG**-Methode (englisch „retrieval-augmented generation", „Erzeugung mit vorgeschaltetem Nachschlagen") holt ein Programm passende Textstellen aus der Sammlung und legt sie dem Sprachmodell zur Antwort bei. Dafür braucht es einen sauberen Zugang zu allen Texten – über eine Schnittstelle oder einen [KI-Agenten](../server-einrichten/ki-agent.md).
- **Bringt das System KI-Funktionen schon mit?** Einige neuere Wissenssysteme haben eine Suche in eigener Sprache, Zusammenfassungen oder einen Frage-Antwort-Bereich bereits eingebaut.

## Die Kandidaten im Überblick

Aus der Liste im Kapitel [Freies Wissen](./freies-wissen.md) kommen die folgenden Programme für eine große, selbst betriebene Sammlung in Frage. Entscheidend ist die Spalte „Datenbank".

| System | Technik | Datenbank | Seit | KI-Ökosystem |
| --- | --- | --- | --- | --- |
| MediaWiki | PHP | MariaDB/MySQL; PostgreSQL zweitrangig | 2002 | sehr groß: Wikitext ist jedem Modell vertraut, RAG-freundlich |
| XWiki | Java | PostgreSQL (empfohlen), MariaDB | 2004 | groß: eigene KI-Erweiterung, strukturierte Daten eingebaut |
| Outline | JavaScript (Node.js) | PostgreSQL + Redis | 2016 | mittel: KI-Funktionen eingebaut, aber Lizenzfrage |
| Docmost | JavaScript (Node.js) | PostgreSQL + Redis | 2024 | mittel: KI eingebaut, sehr jung |
| Wiki.js | JavaScript (Node.js) | PostgreSQL (bald die einzige) | 2017 | mittel: modern, aber holpriger Versionswechsel |
| BookStack | PHP | nur MariaDB/MySQL | 2015 | klein, und **kein PostgreSQL** |
| DokuWiki | PHP | keine (einfache Dateien) | 2004 | klein, keine Datenbank |

**Confluence** von Atlassian wäre der bekannteste Vertreter, ist aber kein Kandidat: Die selbst betreibbare „Server"-Ausgabe wurde eingestellt, übrig bleiben ein Bezahldienst und eine teure „Data Center"-Lizenz. Damit widerspricht Confluence gleich zwei Vorgaben.

## Kandidaten, die ausscheiden

- **BookStack** unterstützt ausschließlich MariaDB oder MySQL. Mit der Festlegung auf PostgreSQL ist es außen vor, so angenehm es sonst zu bedienen ist.
- **DokuWiki** benutzt gar keine Datenbank, sondern legt jede Seite als einzelne Datei ab. Das ist sehr wartungsarm, passt aber nicht zu einem Aufbau, der bewusst auf PostgreSQL setzt, und wird bei sehr vielen Seiten mit anspruchsvoller Suche zäh.

## Nach KI-Ökosystem sortiert

- **MediaWiki (PHP).** MediaWiki wird seit 2002 entwickelt, betreibt die Wikipedia mit Millionen Seiten und hat das mit Abstand größte Ökosystem an Erweiterungen und erfahrenen Betreibern. Der KI-Vorteil liegt im **Format**: Wikitext und der Aufbau von Wikipedia-Artikeln sind jedem Sprachmodell vertraut, weil ein großer Teil des Trainingstextes daher stammt. Ein Modell schreibt und ändert MediaWiki-Seiten darum sehr sicher. Für RAG gibt es eine gut dokumentierte Schnittstelle, über die ein Agent alle Texte abrufen kann. Der bekannte Nachteil bleibt: PostgreSQL wird nur **zweitrangig** unterstützt, einzelne wichtige Erweiterungen setzen MySQL voraus. Für MediaWiki ist darum auch **MariaDB** eine vertretbare Wahl.
- **XWiki (Java).** XWiki wird seit 2004 von einem Unternehmen (XWiki SAS) kommerziell gepflegt und ist von Anfang an als „Enterprise-Wiki" ausgelegt. PostgreSQL ist dort **erste Wahl**, nicht geduldet. XWiki bringt zwei Dinge mit, die beim KI-Einsatz zählen: **strukturierte Einträge** von Haus aus (eine Vorlage „Gerät" mit festen Feldern, daraus Formulare und sortierbare Listen) und eine eigene KI-Erweiterung, über die sich ein Sprachmodell und eine Bedeutungssuche direkt anbinden lassen. XWiki braucht mehr Arbeitsspeicher, weil es in einer Java-Laufzeitumgebung läuft.
- **Outline (Node.js).** Ein schneller, moderner Editor, gut für ein Team. Outline hat KI-Funktionen – Suche in eigener Sprache, Zusammenfassungen – bereits eingebaut und ist von Grund auf für PostgreSQL gebaut, braucht zusätzlich aber **Redis** als schnellen Zwischenspeicher. Die Lizenz ist nicht mehr quelloffen im engen Sinn (siehe unten).
- **Docmost (Node.js).** Ein „Notion-artiges" Wiki mit beliebig verschachtelten Seiten, gemeinsamem Bearbeiten in Echtzeit und eingebauten KI-Funktionen. PostgreSQL und Redis sind Pflicht. Docmost ist erst seit 2024 öffentlich – für ein Team-Wiki gut, für den Anspruch „eine Million Einträge, zehn Jahre, darf nicht kaputtgehen" noch zu jung.
- **Wiki.js (Node.js).** Modernes Bedienbild, Seiten zusätzlich als Git-Ablage. Ab der nächsten Hauptversion ist PostgreSQL die einzige unterstützte Datenbank. Der lange, holprige Übergang zwischen den Hauptversionen ist der Grund, warum Wiki.js hier nicht weiter oben steht.

## Strukturierte Daten

„Verschiedene Inhaltstypen und Datensätze" – nicht nur Fließtext, sondern auch feste Felder – sind der Punkt, an dem sich die Kandidaten am stärksten unterscheiden:

- **XWiki** kann das von Haus aus.
- **MediaWiki** braucht dafür Erweiterungen: **Semantic MediaWiki** legt Bedeutungen in den Text („dieses Feld ist der Hersteller"), oder eine angeschlossene **Wikibase** – dieselbe Software, die hinter der Datensammlung Wikidata steht – hält die Datensätze getrennt vom Fließtext.
- **Outline**, **Docmost** und **Wiki.js** sind auf Fließtext ausgelegt; feste Datenfelder sind nicht ihre Stärke.

## Für dieses Buch

Die Sammlung dieses Buchs läuft auf **MediaWiki**, weil sie im Kern ein verlinktes Nachschlagewerk nach Wikipedia-Vorbild ist: größtes Ökosystem, längster Erfolgsnachweis, und ein Format, das jedes Sprachmodell sicher beherrscht. Dabei gilt die Einschränkung bei PostgreSQL – weshalb **MariaDB** für MediaWiki eine vertretbare Wahl ist, gegebenenfalls mit PostgreSQL daneben allein für die Bedeutungssuche.

Steht die strikte Festlegung auf PostgreSQL im Vordergrund **oder** sollen von Anfang an strukturierte Datensätze geführt werden, ist **XWiki** das passendere System: PostgreSQL ist dort erste Wahl, feste Datenfelder und eine KI-Erweiterung sind eingebaut. **Outline** und **Docmost** sind gute Team-Wikis mit eingebauter KI, aber jünger und mit kleinerem Umfeld. **BookStack** und **DokuWiki** scheiden an PostgreSQL aus.

## Fazit

Nach dem KI-Ökosystem geordnet stehen zwei ausgereifte Systeme oben: **MediaWiki**, dessen Format jedes Sprachmodell sicher beherrscht und das sich gut für die RAG-Methode anzapfen lässt – mit dem Nachteil der nur zweitrangigen PostgreSQL-Unterstützung; und **XWiki**, das PostgreSQL bevorzugt, strukturierte Datensätze und eine KI-Erweiterung eingebaut hat. **Outline**, **Docmost** und **Wiki.js** sind moderne Wikis mit teils eingebauten KI-Funktionen, aber jünger, mit kleinerem Umfeld und bei Outline mit einer nicht mehr im engen Sinn quelloffenen Lizenz. **BookStack** und **DokuWiki** scheiden an der Festlegung auf PostgreSQL aus, **Confluence** an der eingestellten Selbstbetriebs-Ausgabe.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
