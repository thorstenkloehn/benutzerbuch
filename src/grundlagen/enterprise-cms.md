# Enterprise-CMS

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Ein **Content-Management-System** (kurz CMS, wörtlich „System zur Verwaltung von Inhalten") ist ein fertiges Programm, mit dem eine Redaktion Texte, Bilder und Seiten pflegt, ohne sie zu programmieren. Welches CMS für den Aufbau dieses Buchs in Frage käme, klärt das Kapitel [Content-Management-System](../web-stack/cms.md) im Einzelnen. Dieses Kapitel setzt darauf auf und stellt die engere Frage aus [Enterprise Web-Stack](./enterprise-web-stack.md): Welches CMS ist auf sehr große Auftritte ausgelegt, läuft gleichwertig auf **PostgreSQL**, ist **quelloffen** (gern mit einem Unternehmen dahinter) – und hat das beste **KI-Ökosystem**?

## Die Kandidaten im Überblick

Die Reihenfolge folgt grob dem KI-Ökosystem. Entscheidend bleibt die Spalte „Datenbank".

| System | Technik | Datenbank | Seit | KI-Ökosystem |
| --- | --- | --- | --- | --- |
| Drupal | PHP | PostgreSQL, MySQL/MariaDB, SQLite | 2001 | sehr groß: eigene KI-Initiative seit 2025, KI-Agenten im Kern |
| TYPO3 | PHP | PostgreSQL, MySQL/MariaDB u. a. | 1998 | groß: viel Beispielcode, KI-Erweiterungen vorhanden |
| Strapi | JavaScript (Node.js) | PostgreSQL (empfohlen), MySQL, SQLite | 2015 | groß: „Strapi AI", großes Headless-Umfeld |
| Payload | JavaScript (Node.js) | PostgreSQL, SQLite, MongoDB | 2022 | mittel: eng mit Next.js verzahnt, seit 2025 bei Figma |
| Directus | JavaScript (Node.js) | PostgreSQL, MySQL u. a. | 2015 | mittel: gute Schnittstelle für Agenten, aber Lizenzfrage |
| Wagtail | Python (auf Django) | PostgreSQL, MySQL, SQLite | 2014 | mittel: erbt das Python- und Django-Ökosystem |
| WordPress | PHP | nur MySQL/MariaDB | 2003 | sehr groß, aber **kein PostgreSQL** |
| Plone | Python | eigene Objektdatenbank (ZODB) | 2001 | klein, und **kein PostgreSQL** im üblichen Sinn |

Die letzten drei der oberen Gruppe – **Strapi**, **Payload**, **Directus** – sind **Headless-Systeme**: Sie liefern nur die Inhalte über eine Schnittstelle, die Anzeigeseite baut man mit einem [Webframework](../server-einrichten/webframework.md) selbst. Das ist der Gegenentwurf zum monolithischen Aufbau, den [Enterprise Web-Stack](./enterprise-web-stack.md) bevorzugt – dafür ist die Schnittstelle für einen KI-Agenten besonders leicht zu bedienen.

## Kandidaten, die ausscheiden

- **WordPress** unterstützt offiziell nur MySQL oder MariaDB. Für PostgreSQL gibt es nur eine Zusatzerweiterung, die den aktuellen Versionen hinterherhinkt, und sehr viele Erweiterungen setzen MySQL-eigene Befehle voraus. Trotz des mit Abstand größten Ökosystems ist WordPress mit der Festlegung auf PostgreSQL außen vor.
- **Plone** legt seine Inhalte als Objekte in einer eigenen Ablage (ZODB) ab, nicht in einer üblichen relationalen Datenbank. Man kann diese Ablage in PostgreSQL speichern lassen, doch dann ist PostgreSQL nur ein Behälter für Plones eigenes Format und keine Datenbank, die man normal abfragen kann.
- **Reine Bezahldienste** wie Contentful, Sanity oder Storyblok sind zwar bequem und haben oft gute KI-Funktionen, laufen aber nicht auf dem eigenen Server. Sie widersprechen der Vorgabe „selbst betrieben".
- **Ghost** (Node.js) ist ein starkes Redaktions- und Newsletter-System, unterstützt aber nur SQLite und MySQL – kein PostgreSQL.

## Nach KI-Ökosystem sortiert

- **Drupal (PHP).** Drupal ist beim Thema KI derzeit am weitesten. Seit 2025 gibt es eine offizielle **Drupal-KI-Initiative**, hinter der mehrere Firmen mit gemeinsamer Finanzierung stehen; ihr Ziel ist, dass KI-Agenten in Drupal Seiten, Bausteine und ganze Kampagnen aus einer Zielvorgabe erzeugen. Dazu kommt Drupals alte Stärke: **Inhaltstypen und Felder** legt man im Browser an – ein Typ „Gerät" mit Hersteller, Baujahr, Standort – und baut daraus mit dem eingebauten Werkzeug „Views" sortierbare Listen. Genau diese klare, benannte Struktur ist auch für einen Agenten gut lesbar. PostgreSQL wird gleichwertig unterstützt (mit der Erweiterung `pg_trgm`, die das Kapitel [Datenbank](../server-einrichten/datenbank.md) ohnehin einrichtet).
- **TYPO3 (PHP).** Seit 1998 entwickelt, das verbreitete Unternehmens-CMS im deutschsprachigen Raum, mit besonders strengem Pflegeplan (LTS) und einem dichten Netz aus Agenturen. KI-Erweiterungen gibt es, sie sind aber weniger gebündelt als bei Drupal. Für sehr viel Beispielcode und deutschsprachige Betreuung ist TYPO3 die solide Wahl. PostgreSQL wird über eine Zwischenschicht unterstützt, etwas seltener anzutreffen als der Weg über MySQL.
- **Strapi (Headless, Node.js).** Das ausgereifteste der Headless-Systeme: ein Unternehmen im Hintergrund, ein quelloffener Kern und eine große Gemeinschaft. Strapi ist von Grund auf für PostgreSQL gebaut und hat mit „Strapi AI" eine eigene KI-Funktion. Der Nachteil bleibt: Man bekommt keine fertige Webseite, die Anzeigeseite ist ein zweites Programm.
- **Payload (Headless, Node.js).** Baujahr 2022, eng mit dem Frontend-Werkzeug Next.js verzahnt, quelloffen unter der sehr freien MIT-Lizenz. Mitte 2025 wurde das Unternehmen von Figma übernommen; der Programmtext bleibt laut Ankündigung offen und frei. Payload ist modern und für PostgreSQL gebaut, aber das jüngste und am wenigsten erprobte System in dieser Liste.
- **Directus (Headless, Node.js).** Directus legt sich über eine bestehende SQL-Datenbank und macht deren Tabellen sofort über eine Schnittstelle nutzbar – für einen KI-Agenten ein sehr direkter Zugang zu den Daten. Der Haken ist die Lizenz (siehe nächster Abschnitt).
- **Wagtail (auf Django, Python).** Wagtail ist ein CMS, das auf dem Webframework Django aufsetzt. Es erbt damit das gesamte Python- und Django-Ökosystem, einschließlich der guten KI-Bausteine der Sprache. Wagtail ist auf redaktionelle Auftritte mit sauberem Seitenaufbau ausgelegt, kleiner als Drupal, aber solide und vollständig frei (BSD-Lizenz).

## Kommerziell quelloffen: die Lizenzfrage

- **Drupal**, **TYPO3** und **Wagtail** sind vollständig frei (GPL bzw. BSD). Firmen wie Acquia (Drupal) oder die TYPO3 GmbH verdienen mit Betreuung und Betrieb, nicht mit einer verschlossenen Ausgabe.
- **Payload** ist unter der MIT-Lizenz frei; die Übernahme durch Figma ändert daran zunächst nichts.
- **Strapi** hat einen quelloffenen Kern (MIT) und eine kostenpflichtige Zusatzausgabe – der klassische „offene Kern".
- **Directus** hat 2023 und erneut 2025 die Lizenz gewechselt: Die selbst betriebene Fassung ist nur für kleinere Betreiber (unter einer Umsatz- und Mitarbeitergrenze) kostenlos; jede Version wird nach vier Jahren vollständig frei. Wer Directus einsetzen will, sollte die aktuellen Bedingungen vorher prüfen.

## Für dieses Buch

Der Web-Stack dieses Buchs läuft auf **MediaWiki**, keinem CMS. Ein CMS wird erst zum Thema, wenn daneben ein redaktioneller Auftritt entsteht. Für diesen Fall, mit der Festlegung auf PostgreSQL:

- **Drupal** ist die Empfehlung: gleichwertige PostgreSQL-Unterstützung, strukturierte Inhaltstypen eingebaut, seit 2001 ausgereift – und mit der KI-Initiative das aktivste KI-Ökosystem aller quelloffenen CMS. Drupal läuft auf derselben PHP-Laufzeitumgebung, die MediaWiki ohnehin braucht.
- **TYPO3** ist die Alternative, besonders wenn deutschsprachige Agenturbetreuung wichtig ist.
- **Strapi** (Headless) ist die Wahl, wenn die Anzeigeseite ohnehin ein eigenes Programm ist und eine für Agenten bequeme Schnittstelle im Vordergrund steht.
- **WordPress**, **Plone** und **Ghost** scheiden an der Festlegung auf PostgreSQL aus.

## Fazit

Nach dem KI-Ökosystem geordnet steht **Drupal** an der Spitze der quelloffenen Enterprise-CMS: gleichwertige PostgreSQL-Unterstützung, eingebaute Inhaltstypen und seit 2025 eine eigene KI-Initiative mit Agenten im Kern. **TYPO3** ist die verbreitete Alternative im deutschsprachigen Unternehmensumfeld. Unter den Headless-Systemen ist **Strapi** das reifste, **Payload** das modernste (seit 2025 bei Figma) und **Directus** das mit der offensten Datenschnittstelle – aber der unklarsten Lizenz. **Wagtail** erbt als Django-CMS das starke Python-Ökosystem. **WordPress**, **Plone** und **Ghost** scheiden an PostgreSQL aus, reine Bezahldienste an der Vorgabe „selbst betrieben".

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
