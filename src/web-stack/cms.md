# Content-Management-System

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Ein Content-Management-System (kurz CMS, wörtlich „System zur Verwaltung von Inhalten") ist ein fertiges Programm, mit dem man die Texte, Bilder und Seiten einer Webseite pflegt, ohne sie programmieren zu müssen. Es ist – wie das [Wissenssystem](./wissensystem.md) – die oberste Schicht des Web Stacks: Der Besucher sieht nur die fertige Webseite, darunter arbeiten die [Datenbank](../server-einrichten/datenbank.md), in der alle Inhalte dauerhaft liegen, und der [Webserver](../server-einrichten/webserver.md), der die Seiten ausliefert.

Der Unterschied zum Wissenssystem liegt im Zweck. Ein Wiki wie MediaWiki ist auf viele gleichberechtigte Bearbeiter und verlinkten Fließtext ausgelegt. Ein CMS ist auf eine Redaktion ausgelegt: Wenige Personen mit festen Rollen pflegen Seiten, die einem Freigabeweg folgen und in einem festen Seitenaufbau erscheinen – eine Firmenseite, ein Nachrichtenportal, ein Behördenauftritt. Dieses Kapitel trifft die Wahl für den Fall, dass die Seite im Kern ein solcher redaktioneller Auftritt ist. Der Begriff CMS ist im Kapitel [Webframework](../server-einrichten/webframework.md) bereits kurz gegen die Begriffe „Anwendung" und „Framework" abgegrenzt.

Die Ausgangsfrage des Autors ist dieselbe wie beim Wissenssystem: Welches CMS ist industrietauglich, sehr robust, verwaltet eine Million Texteinträge mit verschiedenen Inhaltstypen und Datensätzen, geht dabei nicht kaputt, ist schnell und lässt sich vergleichend messen, hat den höchsten Reifegrad und läuft direkt auf dem Server – wobei als Datenbank ausschließlich **PostgreSQL** vorgesehen ist und die Software quelloffen sein soll, auch im Sinne quelloffener Software mit einem Unternehmen dahinter.

## Was schon feststeht

Vier Dinge sind durch die vorigen Kapitel bereits entschieden und schränken die Auswahl ein:

- **Die Datenbank ist PostgreSQL.** Das Kapitel [Datenbank](../server-einrichten/datenbank.md) richtet PostgreSQL samt der Erweiterungen für die Suche ein. Ein CMS, das PostgreSQL nicht unterstützt, scheidet aus – auch dann, wenn es sonst gut passen würde.
- **Es läuft „auf dem Metall".** Das Programm wird direkt auf dem Server installiert, ohne Zwischenschicht wie einen Container (siehe [Containerisierung von Software](../server-einrichten/containerisierung.md)). Es muss sich also als normales Paket oder als entpacktes Archiv auf **Ubuntu** einrichten lassen.
- **Es wird selbst betrieben.** Die Inhalte bleiben auf dem eigenen Server. Bezahldienste in fremder Hand kommen nicht in Frage.
- **Es ist quelloffen.** Der Quelltext muss offenliegen. Ein Unternehmen im Hintergrund, das mit einer kostenpflichtigen Zusatzausgabe oder mit Betreuung Geld verdient, ist ausdrücklich in Ordnung – solange die selbst betriebene Fassung frei nutzbar bleibt. Bei einigen jüngeren Programmen ist genau dieser Punkt inzwischen die entscheidende Frage; dazu unten mehr.

## Die Kriterien der Frage

Die Begriffe der Ausgangsfrage sind im Kapitel [Wissenssystem](./wissensystem.md) bereits im Einzelnen eingeordnet; sie gelten für ein CMS unverändert. Eine Million Einträge sind für ein ausgereiftes System eine kleine bis mittlere Größe. „Industrietauglich" und „höchster Reifegrad" meinen: seit vielen Jahren ohne Unterbrechung gepflegt, Sicherheitslücken zügig geschlossen, geregelter Versionsumstieg, genug andere Betreiber. „Robust" heißt: Ein Absturz zerreißt keine Daten, ein Programmfehler legt nicht alles lahm.

Ein Punkt verdient für ein CMS besondere Beachtung: **„verschiedene Inhaltstypen und Datensätze".** Bei einem CMS ist das keine Randfunktion, sondern die eigentliche Kerndisziplin. Ein Inhaltstyp ist eine Vorlage mit festen Feldern – etwa „Gerät" mit den Feldern Hersteller, Baujahr und Standort –, aus der sich sortierbare und filterbare Listen bauen lassen. An diesem Punkt unterscheiden sich die Kandidaten am stärksten.

## Die Kandidaten im Überblick

Für eine große, selbst betriebene Webseite kommen die folgenden Systeme in Frage. Entscheidend ist die Spalte „Datenbank".

| System | Technik | Datenbank | Seit | Stärke |
| --- | --- | --- | --- | --- |
| WordPress | PHP | nur MySQL/MariaDB | 2003 | mit Abstand verbreitetstes CMS, riesiges Erweiterungsangebot |
| Drupal | PHP | PostgreSQL, MySQL/MariaDB, SQLite | 2001 | strukturierte Inhalte von Haus aus, auf große Auftritte ausgelegt |
| TYPO3 | PHP | PostgreSQL, MySQL/MariaDB u. a. | 1998 | im deutschsprachigen Raum verbreitetes Unternehmens-CMS, feste LTS-Pflege |
| Joomla | PHP | MySQL/MariaDB; PostgreSQL nur eingeschränkt | 2005 | Mittelweg zwischen WordPress und Drupal |
| Plone | Python | eigene Objektdatenbank (ZODB) | 2001 | strenge Rechteverwaltung, im Behördenumfeld verbreitet |
| Strapi | JavaScript (Node.js) | PostgreSQL (empfohlen), MySQL, SQLite | 2015 | größtes Ökosystem unter den „Headless"-Systemen |
| Directus | JavaScript (Node.js) | PostgreSQL, MySQL u. a. | 2015 | legt sich über eine bestehende SQL-Datenbank |
| Payload | JavaScript (Node.js) | PostgreSQL, SQLite, MongoDB | 2022 | eng mit dem Frontend-Werkzeug Next.js verzahnt |

Die letzten drei sind **Headless-Systeme** (englisch „kopflos"). Ein solches CMS liefert nur die Inhalte über eine Schnittstelle und keine fertige Webseite; die Anzeigeseite ist ein eigenes Programm, das man mit einem [Webframework](../server-einrichten/webframework.md) selbst baut oder als statische Seite erzeugt (siehe [Docs-as-Code](../entwicklungs-rechner/docs-as-code.md)).

### Kandidaten, die ausscheiden

- **WordPress** unterstützt offiziell nur MySQL oder MariaDB. Für PostgreSQL gibt es nur eine Zusatzerweiterung, die nicht mit den aktuellen WordPress-Versionen Schritt hält, und sehr viele Erweiterungen setzen MySQL-eigene Befehle voraus. Mit der Festlegung auf PostgreSQL ist WordPress damit außen vor – so verbreitet es auch ist.
- **Plone** benutzt keine übliche relationale Datenbank, sondern legt seine Inhalte als Objekte in einer eigenen Ablage (ZODB) ab. Man kann diese Ablage zwar in PostgreSQL speichern lassen, doch dann ist PostgreSQL nur ein Behälter für Plones eigenes Format und keine Datenbank, die man normal abfragen kann. Das passt nicht zu einem Aufbau, der bewusst auf PostgreSQL setzt.
- **Joomla** kann PostgreSQL seit einigen Jahren, doch die Unterstützung gilt als lückenhaft und wird wenig getestet; empfohlen sind MySQL oder MariaDB. Das ist dasselbe Problem wie bei MediaWiki im Kapitel [Wissenssystem](./wissensystem.md): ein wenig begangener Weg, bei dem man vor jeder Erweiterung prüfen muss, ob sie mitspielt.

## Der Filter „nur PostgreSQL"

Übrig bleiben **Drupal**, **TYPO3**, **Strapi**, **Directus** und **Payload**. Hier ist ein genauer Blick nötig.

- **Drupal** unterstützt PostgreSQL gleichwertig zu MySQL; die aktuelle Hauptversion setzt PostgreSQL 16 oder neuer voraus. Für die eingebaute Suche erwartet Drupal die Erweiterung `pg_trgm` – dieselbe, die das Kapitel [Datenbank](../server-einrichten/datenbank.md) ohnehin einrichtet.
- **TYPO3** spricht die Datenbank über eine Zwischenschicht an (Doctrine DBAL) und unterstützt PostgreSQL 13 und neuer. Das funktioniert zuverlässig, ist aber etwas seltener anzutreffen als der Weg über MySQL.
- **Strapi**, **Directus** und **Payload** sind von Grund auf für PostgreSQL gebaut. Strapi und Payload laufen zusätzlich auf anderen Datenbanken; Directus ist ausdrücklich dafür gemacht, sich direkt auf eine bestehende SQL-Datenbank zu setzen und deren Tabellen zu lesen.

## Der Filter „industrietauglich, höchster Reifegrad"

- **Drupal** wird seit 2001 entwickelt, betreibt Auftritte von Regierungen, Universitäten und großen Nachrichtenseiten, hat einen festen Veröffentlichungsplan, ein eigenes Sicherheitsteam und einen dokumentierten Weg für den Versionsumstieg.
- **TYPO3** wird seit 1998 entwickelt und ist das verbreitete Unternehmens-CMS im deutschsprachigen Raum. Es hat einen besonders strengen LTS-Plan – die Version 14 LTS erschien im April 2026 und wird mehrere Jahre mit kostenlosen Aktualisierungen und danach optional gegen Bezahlung weiter gepflegt – sowie ein dichtes Netz an Agenturen und die TYPO3 GmbH für kommerzielle Betreuung.

**Strapi** ist das ausgereifteste der Headless-Systeme: ein Unternehmen im Hintergrund, ein quelloffener Kern und eine große Gemeinschaft. Es ist aber jünger und hat für sehr große Sammlungen einen kürzeren Erfolgsnachweis. **Directus** ist solide, **Payload** ist mit Baujahr 2022 das jüngste und am wenigsten erprobte.

## Der eigentliche Unterschied: Wie entstehen die Inhaltstypen?

An diesem Punkt entscheidet sich die Wahl, denn hier greift das Kriterium „verschiedene Inhaltstypen und Datensätze".

- **Drupal** nennt sie „Inhaltstypen" und „Felder". Man legt im Browser einen Typ „Gerät" mit den Feldern Hersteller, Baujahr und Standort an und baut daraus mit dem eingebauten Werkzeug „Views" sortierbare und filterbare Listen – ohne eine Zeile Programmcode. Das ist Drupals Kernstärke und deckt sich fast wörtlich mit der Anforderung.
- **TYPO3** kann dasselbe, braucht dafür aber etwas mehr Einrichtung (über die eingebauten „Content Blocks" oder eine Erweiterung).
- Die Headless-Systeme **Strapi**, **Directus** und **Payload** sind vollständig um Inhaltstypen herum gebaut: Man beschreibt sie in der Verwaltungsoberfläche oder im Code und bekommt sofort eine Schnittstelle darauf. Nur bekommt man daraus keine Webseite – die Anzeigeseite muss man selbst bauen.

Daraus folgt eine klare Trennlinie: Soll die Seite ein sofort nutzbarer redaktioneller Auftritt mit strukturierten Inhalten sein, ist **Drupal** die natürliche Wahl. Ist die Anzeigeseite ohnehin ein eigenes Programm – ein Shop, eine App, eine statisch gebaute Seite –, ist ein Headless-System mit **Strapi** der sauberere Schnitt.

## Zur Geschwindigkeit: Was Vergleichsmessungen taugen

Für CMS gilt dasselbe wie für Wissenssysteme und Webframeworks: Vergleichsmessungen sagen wenig über den Alltag aus. Eine Million Seiten sind für PostgreSQL eine normale Größe. Über die Antwortzeit entscheiden drei Dinge, die alle unterhalb des CMS liegen:

- **Die richtigen Indizes in der Datenbank** – der Unterschied ist oft der Faktor Tausend.
- **Ein Zwischenspeicher für fertige Seiten**, entweder im CMS selbst, in einem vorgeschalteten Zwischenspeicher (etwa Varnish) oder direkt im [Webserver](../server-einrichten/webserver.md). Ist eine Seite einmal erzeugt und wird kurz vorgehalten, spielt die Geschwindigkeit von CMS und Datenbank für den nächsten Abruf keine Rolle mehr.
- **Sparsames Laden** – Listen seitenweise holen statt alles auf einmal.

Drupal wie TYPO3 bringen einen mehrstufigen Zwischenspeicher von Haus aus mit. Was ein CMS-Vergleich am Ende misst, ist meist nur, wie beherzt jedes System im Auslieferungszustand zwischenspeichert – nicht die eigentliche Geschwindigkeit.

## Für dieses Buch

Der Web Stack dieses Buchs läuft auf **MediaWiki** – einem Wiki, keinem CMS –, weil die Sammlung im Kern ein gemeinsam bearbeitetes Nachschlagewerk ist. Ein CMS wird erst zum Thema, wenn daneben ein zweiter, redaktioneller Auftritt entsteht: eine Firmenseite, ein Blog, ein Portal.

Für diesen Fall gilt, mit der Festlegung auf PostgreSQL:

- **Drupal** ist die Empfehlung: PostgreSQL wird offiziell und gleichwertig unterstützt (mit der Erweiterung `pg_trgm`, die im Kapitel [Datenbank](../server-einrichten/datenbank.md) ohnehin eingerichtet wird), strukturierte Inhaltstypen sind eingebaut, und das System ist seit 2001 ausgereift. Drupal läuft auf derselben PHP-Laufzeitumgebung, die MediaWiki ohnehin braucht.
- **TYPO3** ist die Alternative, besonders wenn deutschsprachige Agenturbetreuung wichtig ist.
- **Strapi** (Headless) ist die Wahl, wenn die Anzeigeseite ein eigenes Programm ist.
- **WordPress**, **Joomla** und **Plone** scheiden an der Festlegung auf PostgreSQL aus.

Ein Blick auf die Lizenzen lohnt sich: **Drupal**, **TYPO3** und **WordPress** stehen unter der GPL und sind vollständig frei. **Strapi** hat einen quelloffenen Kern (MIT-Lizenz) und eine kostenpflichtige Zusatzausgabe. **Directus** hat 2023 und erneut 2025 die Lizenz gewechselt: Die selbst betriebene Fassung ist für kleinere Betreiber (unter einer festgelegten Umsatz- und Mitarbeitergrenze) kostenlos, jede Version wird nach vier Jahren vollständig frei – aber oberhalb der Grenze ist eine kostenpflichtige Lizenz nötig. Wer Directus einsetzen will, sollte die aktuellen Bedingungen vorher prüfen.

## Fazit

Ein CMS ist die oberste Schicht des Web Stacks für einen redaktionellen Auftritt – im Unterschied zum Wiki, das für gemeinsam bearbeiteten Fließtext gedacht ist. Die Festlegung auf PostgreSQL, auf den Betrieb direkt auf dem Server und auf höchsten Reifegrad grenzt die Auswahl auf zwei Systeme ein: **Drupal** und **TYPO3**. Drupal ist die Wahl, wenn die Seite ein sofort nutzbarer Auftritt mit strukturierten Inhaltstypen sein soll; TYPO3 ist die Alternative im deutschsprachigen Unternehmensumfeld. Ist die Anzeigeseite ein eigenes Programm, ist ein Headless-System mit **Strapi** der klarere Schnitt. **WordPress**, **Joomla** und **Plone** scheiden an PostgreSQL aus. Eine Million Einträge sind für alle verbliebenen Systeme unproblematisch; ob die Seite schnell bleibt, entscheidet nicht das CMS, sondern die Indizes in der Datenbank und ein Zwischenspeicher für fertige Seiten.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
