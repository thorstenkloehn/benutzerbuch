# Wissenssystem

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Ein Wissenssystem ist das Programm, in dem die Texte geschrieben, gelesen und geordnet werden. Es ist die oberste Schicht des Web Stacks: Der Besucher sieht nur dieses Programm. Darunter liegen die [Datenbank](../server-einrichten/datenbank.md), in der alle Inhalte dauerhaft gespeichert sind, und der [Webserver](../server-einrichten/webserver.md), der die Seiten ausliefert. Im Kapitel [Inhalts-Software selbst betreiben](../grundlagen/inhalt-software-verwalten.md) ist dieses oberste Programm bereits als ein Baustein aufgetaucht; im Kapitel [Freies Wissen](../grundlagen/freies-wissen.md) stehen mehrere davon in einer Liste. Dieses Kapitel trifft die Wahl.

Die Ausgangsfrage des Autors lautete: Welches Wissenssystem ist industrietauglich, sehr robust, verwaltet eine Million Texteinträge mit verschiedenen Inhaltstypen und Datensätzen, geht dabei nicht kaputt, ist schnell und lässt sich vergleichend messen, hat den höchsten Reifegrad und läuft direkt auf dem Server – wobei als Datenbank ausschließlich **PostgreSQL** vorgesehen ist. Diese Anforderungen werden im Folgenden einzeln geprüft und daraus eine Empfehlung abgeleitet.

## Was schon feststeht

Drei Dinge sind durch die vorigen Kapitel bereits entschieden und schränken die Auswahl ein:

- **Die Datenbank ist PostgreSQL.** Das Kapitel [Datenbank](../server-einrichten/datenbank.md) richtet PostgreSQL 18 ein, samt der Erweiterungen für die Suche. Ein Wissenssystem, das PostgreSQL nicht unterstützt, scheidet damit aus – auch dann, wenn es sonst gut passen würde.
- **Es läuft „auf dem Metall".** Das Programm wird direkt auf dem Server installiert, ohne Zwischenschicht wie einen Container (siehe [Containerisierung von Software](../server-einrichten/containerisierung.md)). Das bedeutet: Es muss sich als normales Paket oder als entpacktes Archiv auf **Ubuntu** einrichten lassen und darf nicht zwingend eine Container-Umgebung voraussetzen.
- **Es wird selbst betrieben.** Die Inhalte bleiben auf dem eigenen Server. Bezahldienste in fremder Hand und Programme ohne offenen Quelltext kommen nicht in Frage.

## Die Kriterien der Frage, einzeln betrachtet

Die Begriffe aus der Ausgangsfrage klingen anspruchsvoller, als sie in der Praxis sind. Es lohnt sich, jeden einzeln einzuordnen.

- **„Eine Million Texteinträge"** ist für ein ausgereiftes Wissenssystem eine kleine bis mittlere Größe. Die deutschsprachige Wikipedia hat ein Vielfaches davon. Über die Geschwindigkeit entscheidet nicht die Menge der Seiten, sondern ob die Datenbank die passenden **Indizes** hat – ein Index ist ein zusätzliches Verzeichnis, das das Suchen abkürzt, so wie das Stichwortverzeichnis am Ende eines Buchs. Dieser Punkt ist im Kapitel [Datenbank](../server-einrichten/datenbank.md) ausführlich behandelt.
- **„Verschiedene Inhaltstypen und Datensätze"** meint: nicht nur Fließtext, sondern auch feste Felder – etwa eine Liste von Geräten mit Hersteller, Baujahr und Standort, die sich sortieren und filtern lässt. Reine Wikis können das nur mit Zusatzprogrammen; einige Systeme bringen es von Haus aus mit. Das ist der Punkt, an dem sich die Kandidaten am stärksten unterscheiden.
- **„Industrietauglich" und „höchster Reifegrad"** bedeuten dasselbe aus zwei Richtungen: Das Programm wird seit vielen Jahren ohne Unterbrechung gepflegt, Sicherheitslücken werden zügig geschlossen, der Umstieg auf neue Versionen ist geregelt, und es gibt genug andere Betreiber, dass man bei Problemen nicht allein ist.
- **„Robust" / „geht nicht kaputt"** heißt: Ein Absturz des Servers hinterlässt keine zerrissenen Daten (dafür sorgt die Datenbank, siehe [Datenbank](../server-einrichten/datenbank.md)), und ein Programmfehler legt nicht die ganze Sammlung lahm.
- **„Schnell und benchmarkbar"** – vergleichende Messungen unter gleichen Bedingungen. Für Wissenssysteme sind solche Messungen selten aussagekräftig, weil das Ergebnis fast immer an der Datenbank, den Indizes und einem Zwischenspeicher hängt, nicht am Wissenssystem selbst. Dazu weiter unten mehr.

## Die Kandidaten im Überblick

Aus der Liste in [Freies Wissen](../grundlagen/freies-wissen.md) kommen die folgenden Programme für eine große, selbst betriebene Sammlung in Frage. Entscheidend ist die Spalte „Datenbank".

| System | Technik | Datenbank | Seit | Stärke |
| --- | --- | --- | --- | --- |
| MediaWiki | PHP | MariaDB/MySQL; PostgreSQL zweitrangig | 2002 | die Software der Wikipedia; größtes Ökosystem an Erweiterungen |
| XWiki | Java | PostgreSQL (empfohlen), MariaDB | 2004 | strukturierte Daten und kleine Anwendungen von Haus aus |
| Wiki.js | JavaScript | PostgreSQL (ab der nächsten Hauptversion die einzige) | 2017 | modernes Bedienbild, Seiten zusätzlich als Git-Ablage |
| Outline | JavaScript | PostgreSQL + Redis | 2016 | schneller Editor, gut für ein Team, weniger für ein offenes Nachschlagewerk |
| Docmost | JavaScript | PostgreSQL + Redis | 2024 | beliebig verschachtelte Seiten, gemeinsames Bearbeiten in Echtzeit |
| BookStack | PHP | nur MariaDB/MySQL | 2015 | sehr einfacher Einstieg durch feste Gliederung |
| DokuWiki | PHP | keine (einfache Dateien) | 2004 | sehr wartungsarm, kein Datenbankbetrieb nötig |

Zwei Kandidaten fallen sofort heraus:

- **BookStack** unterstützt ausschließlich MariaDB oder MySQL. Mit der Festlegung auf PostgreSQL ist es damit außen vor, so angenehm es sonst zu bedienen wäre.
- **DokuWiki** benutzt gar keine Datenbank, sondern legt jede Seite als einzelne Datei ab. Das macht es extrem wartungsarm, passt aber nicht zu einem Aufbau, der bewusst auf PostgreSQL setzt, und wird bei einer Million Seiten und anspruchsvoller Suche zäh.

## Der Filter „nur PostgreSQL"

Übrig bleiben **MediaWiki**, **XWiki**, **Wiki.js**, **Outline** und **Docmost**. Hier ist ein genauer Blick nötig, denn „unterstützt PostgreSQL" bedeutet nicht bei allen dasselbe.

- **XWiki** empfiehlt PostgreSQL ausdrücklich als Datenbank für den ernsthaften Betrieb. Die Unterstützung ist gleichwertig zu allen anderen; nichts fühlt sich wie ein Nebengleis an.
- **Wiki.js**, **Outline** und **Docmost** sind von Grund auf für PostgreSQL gebaut. Bei Wiki.js wird PostgreSQL ab der nächsten Hauptversion sogar die einzige unterstützte Datenbank sein. Outline und Docmost brauchen zusätzlich **Redis**, einen schnellen Zwischenspeicher – ein weiteres Programm, das eingerichtet und überwacht werden will.
- **MediaWiki** unterstützt PostgreSQL zwar seit vielen Jahren, aber erklärtermaßen nur **zweitrangig**. Empfohlen für den Produktivbetrieb sind MariaDB oder MySQL. In der Praxis heißt das: Der Kern läuft auf PostgreSQL, aber einzelne wichtige Erweiterungen (etwa die verbreitete Übersetzungs-Erweiterung „Translate") setzen MySQL voraus. Wer MediaWiki auf PostgreSQL betreibt, geht einen weniger begangenen Weg und muss bei jeder Erweiterung vorher prüfen, ob sie mitspielt.

## Der Filter „industrietauglich, höchster Reifegrad"

Von den fünf verbliebenen Systemen haben zwei einen deutlich längeren und breiteren Erfolgsnachweis als die anderen drei:

- **MediaWiki** wird seit 2002 entwickelt, betreibt die Wikipedia mit Millionen Seiten und Milliarden Zugriffen und hat das mit Abstand größte Ökosystem an Erweiterungen, Anleitungen und erfahrenen Betreibern.
- **XWiki** wird seit 2004 entwickelt, wird von einem Unternehmen (XWiki SAS) kommerziell gepflegt und ist von Anfang an als „Enterprise-Wiki" für Firmen ausgelegt. Große Installationen mit Hunderten gleichzeitigen Benutzern sind dokumentiert.

**Wiki.js**, **Outline** und **Docmost** sind moderne, gut gemachte Programme, aber jünger und mit kleinerem Umfeld. Wiki.js hat einen langen, holprigen Übergang zwischen zwei Hauptversionen hinter sich; Docmost ist erst seit 2024 öffentlich. Für ein Team-Notizbuch mit einigen Tausend Seiten sind alle drei eine gute Wahl. Für den Anspruch „eine Million Einträge, darf auf keinen Fall kaputtgehen, muss in zehn Jahren noch gepflegt sein" sind sie die riskantere Wette.

## MediaWiki oder XWiki – der eigentliche Unterschied

Damit stehen sich zwei ausgereifte Systeme gegenüber. Die Wahl zwischen ihnen hängt an zwei Fragen.

**Erste Frage: Fließtext oder auch strukturierte Daten?**

- **MediaWiki** ist auf verlinkten Fließtext ausgelegt – Artikel, die aufeinander verweisen und in Kategorien stehen. Feste Datenfelder gibt es nur über Erweiterungen wie „Semantic MediaWiki" oder eine angeschlossene Wikibase. Das funktioniert, ist aber zusätzlicher Aufbau.
- **XWiki** bringt strukturierte Einträge von Haus aus mit. Man legt eine Vorlage mit festen Feldern an – etwa „Gerät" mit Hersteller, Baujahr, Standort – und bekommt daraus Formulare, sortierbare Listen und Auswertungen, ohne eine Zeile Programmcode. Wenn „verschiedene Inhaltstypen und Datensätze" wörtlich gemeint ist, ist das der Punkt, der für XWiki spricht.

**Zweite Frage: Wie wörtlich ist „nur PostgreSQL"?**

- Ist PostgreSQL fest gesetzt und soll das Wissenssystem wirklich ohne Umwege darauf laufen, ist **XWiki** die natürliche Wahl: PostgreSQL ist dort erste Wahl, nicht geduldet.
- Ist MediaWiki aus anderen Gründen gewünscht (das Wikipedia-Bedienbild, eine bestimmte Erweiterung, vorhandene Erfahrung), sollte man ehrlich abwägen, ob man den Preis der zweitrangigen PostgreSQL-Unterstützung zahlen will – oder für MediaWiki doch **MariaDB** einsetzt. MariaDB und PostgreSQL können auf demselben Server nebeneinander laufen; die übrigen Kapitel dieses Buchs ändern sich dadurch kaum, da die Suchdienste und die Bedeutungssuche auch mit einer MariaDB-gestützten MediaWiki-Installation zusammenarbeiten.

Ein Mischweg ist ebenfalls möglich und in der Praxis verbreitet: MediaWiki mit MariaDB für die Texte, PostgreSQL mit `pgvector` daneben allein für die Bedeutungssuche. Dann hat jede Datenbank die Aufgabe, für die sie am besten unterstützt ist.

## Zur Geschwindigkeit: Was Vergleichsmessungen taugen

Für Wissenssysteme kursieren kaum belastbare Vergleichsmessungen, und das aus gutem Grund: Die Antwortzeit einer Wiki-Seite hängt fast vollständig an drei Dingen, die alle unterhalb des Wissenssystems liegen:

- **Die richtigen Indizes in der Datenbank.** Eine Abfrage ohne passenden Index durchsucht die ganze Tabelle; mit Index springt sie sofort zum Ziel. Der Unterschied ist oft der Faktor Tausend.
- **Ein Zwischenspeicher für fertige Seiten.** Wird eine einmal erzeugte Seite für kurze Zeit vorgehalten (im Wissenssystem selbst, im [Webserver](../server-einrichten/webserver.md) oder in einem eigenen Zwischenspeicher wie Redis), spielt die Geschwindigkeit von Datenbank und Wissenssystem für den nächsten Abruf keine Rolle mehr.
- **Sparsames Laden.** Gute Systeme holen Listen seitenweise statt alles auf einmal.

MediaWiki wie XWiki bringen diese Mechanik mit. Der Unterschied in der reinen Geschwindigkeit ist im Alltag einer Wissenssammlung nicht spürbar. Wer trotzdem messen will, misst am Ende die Datenbank und den Zwischenspeicher – nicht das Wissenssystem.

## Für dieses Buch

Die Kapitel [Inhalts-Software selbst betreiben](../grundlagen/inhalt-software-verwalten.md) und [Datenbank](../server-einrichten/datenbank.md) beschreiben den Aufbau mit **MediaWiki**. Das ist die Wahl für den Fall, dass die Sammlung im Kern ein verlinktes Nachschlagewerk nach dem Vorbild der Wikipedia ist: größtes Ökosystem, längster Erfolgsnachweis, die meisten Anleitungen. Dabei gilt die oben genannte Einschränkung – MediaWikis PostgreSQL-Unterstützung ist zweitrangig, weshalb für MediaWiki auch **MariaDB** eine vertretbare Wahl ist.

Wenn die strikte Festlegung auf PostgreSQL im Vordergrund steht **oder** die Sammlung von Anfang an auch strukturierte Datensätze führen soll, ist **XWiki** das passendere System: PostgreSQL ist dort erste Wahl, und feste Datenfelder sind eingebaut. XWiki braucht mehr Arbeitsspeicher (es läuft in einer Java-Laufzeitumgebung) und hat ein kleineres Umfeld als MediaWiki, ist aber genauso ausgereift und industrietauglich.

Die jüngeren Systeme **Wiki.js**, **Outline** und **Docmost** sind für ein Team-Wissensnetz eine gute Wahl, aber nicht für den Anspruch „eine Million Einträge, über viele Jahre, darf nicht kaputtgehen". **BookStack** und **DokuWiki** scheiden an der Festlegung auf PostgreSQL aus.

## Fazit

Ein Wissenssystem ist die oberste Schicht des Web Stacks – das Programm, in dem geschrieben und gelesen wird. Die Festlegung auf PostgreSQL, auf den Betrieb direkt auf dem Server und auf höchsten Reifegrad grenzt die Auswahl auf zwei Systeme ein: **MediaWiki** und **XWiki**. MediaWiki ist die Wahl für ein Nachschlagewerk nach Wikipedia-Vorbild, mit der Einschränkung, dass PostgreSQL dort nur zweitrangig unterstützt wird. XWiki ist die Wahl, wenn PostgreSQL fest gesetzt ist oder neben Fließtext auch strukturierte Datensätze verwaltet werden sollen. Eine Million Einträge sind für beide unproblematisch; ob die Sammlung schnell bleibt, entscheidet nicht das Wissenssystem, sondern die Indizes in der Datenbank und ein Zwischenspeicher.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
