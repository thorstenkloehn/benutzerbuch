# Enterprise Web-Stack

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Ein **Web-Stack** ist der Stapel aus Programmen, der zusammen eine Webseite betreibt: unten die [Datenbank](../server-einrichten/datenbank.md), in der Mitte der [Webserver](../server-einrichten/webserver.md), oben das Programm, mit dem Inhalte geschrieben und angezeigt werden. „Stack" ist das englische Wort für „Stapel". **Enterprise** (englisch für „Unternehmen") meint hier keine Firmengröße, sondern einen Anspruch: Der Auftritt soll sehr groß werden können, über viele Jahre laufen, bei einem Fehler nicht auseinanderfallen und sich von einem Team pflegen lassen, das wechselt.

Dieses Kapitel klärt die Begriffe, die bei dieser Wahl immer wieder vorkommen, und teilt das oberste Programm des Stacks in drei Fälle auf. Für jeden Fall gibt es ein eigenes Unterkapitel mit einer Bestenliste:

- [Enterprise-Webframework](./enterprise-webframework.md) – wenn die Anwendung selbst geschrieben wird.
- [Enterprise-CMS](./enterprise-cms.md) – wenn eine Redaktion Seiten pflegt.
- [Enterprise-Wissenssystem](./enterprise-wissenssystem.md) – wenn viele Personen ein Nachschlagewerk gemeinsam schreiben.

Verwandte Kapitel gehen den umgekehrten Weg: [Webframework](../server-einrichten/webframework.md), [Content-Management-System](../web-stack/cms.md) und [Wissenssystem](../web-stack/wissensystem.md) treffen die Wahl für den konkreten Aufbau dieses Buchs. Die Unterkapitel hier ordnen dieselben Programme noch einmal, aber nach einer zusätzlichen Frage: Wie gut lässt sich damit **mit Hilfe von Künstlicher Intelligenz** arbeiten?

## Die drei Fälle im Überblick

| Fall | Wer pflegt die Inhalte? | Was ist der Kern? | Kapitel |
| --- | --- | --- | --- |
| Webframework | Programmiererinnen und Programmierer | eine eigene Anwendung, für die es nichts Fertiges gibt | [Enterprise-Webframework](./enterprise-webframework.md) |
| CMS | eine Redaktion mit festen Rollen | Seiten mit festem Aufbau und Freigabeweg | [Enterprise-CMS](./enterprise-cms.md) |
| Wissenssystem | viele gleichberechtigte Autoren | verlinkter Fließtext, ein Nachschlagewerk | [Enterprise-Wissenssystem](./enterprise-wissenssystem.md) |

Die Grenzen sind fließend. Ein CMS wie Drupal kann auch als Rohbau für eine eigene Anwendung dienen, ein Wiki lässt sich mit Zusatzprogrammen wie ein CMS benutzen. Die Tabelle nennt den jeweils typischen Fall.

## „Batterien inklusive" und „monolithisch"

Zwei Wörter beschreiben, wie ein Programm zugeschnitten ist.

**Batterien inklusive** (englisch „batteries included") heißt: Alles, was ein Auftritt fast immer braucht, ist schon eingebaut – die Anmeldung von Benutzern, die Verbindung zur Datenbank, ein Schutz gegen die üblichen Angriffe, eine Verwaltungsoberfläche, das Verschicken von E-Mails, Hintergrundaufgaben. Das Gegenteil ist ein Baukasten aus vielen kleinen Einzelteilen, die man selbst aussuchen und zusammenfügen muss.

**Monolithisch** (von griechisch „monólithos", „aus einem Stein") heißt: Der ganze Auftritt ist ein einziges Programm in einer einzigen Ablage. Das Gegenteil sind viele kleine Programme, die über das Netz miteinander reden – das nennt man **Microservices** (englisch „kleine Dienste"). Ein weiterer Gegenentwurf im CMS-Bereich heißt **Headless** (englisch „kopflos"): Das CMS liefert nur die Inhalte über eine Schnittstelle, die Anzeigeseite ist ein zweites, getrenntes Programm.

Für einen großen, langlebigen Auftritt mit einem kleinen Team hat der monolithische, batterienreiche Aufbau handfeste Vorteile: weniger bewegliche Teile, ein Ort für den ganzen Code, ein einziger Vorgang beim Aktualisieren. Microservices lohnen sich erst, wenn viele Teams unabhängig voneinander an getrennten Bereichen arbeiten. Die Bestenlisten in den Unterkapiteln bevorzugen darum bewusst den Monolithen.

## „Kommerziell quelloffen"

**Quelloffen** (englisch „open source") heißt: Der Programmtext liegt offen, jeder darf ihn lesen, ändern und weitergeben. Bei manchen quelloffenen Programmen steht eine Gemeinnützige oder eine lose Gemeinschaft dahinter (etwa bei Django oder MediaWiki). Bei anderen steht ein **Unternehmen** dahinter, das mit dem Programm Geld verdient – meist so:

- **Bezahlte Betreuung:** Das Programm ist frei, die Firma verkauft Hilfe, Schulung und Wartungsverträge.
- **Bezahlter Betrieb:** Die Firma nimmt einem das Aufsetzen und Betreiben gegen Gebühr ab (englisch „managed hosting").
- **Offener Kern** (englisch „open core"): Ein freier Grundteil, dazu kostenpflichtige Zusatzausgaben mit weiteren Funktionen.

Der letzte Weg hat einen Haken: Einige Anbieter haben in den letzten Jahren die **Lizenz** – die Nutzungsbedingungen des Programmtexts – nachträglich verschärft, sodass die selbst betriebene Fassung ab einer bestimmten Firmengröße Geld kostet. Für einen Auftritt, der zehn Jahre halten soll, ist die Lizenz darum kein Randthema. Die Unterkapitel weisen bei jedem Kandidaten darauf hin. „Kommerziell quelloffen" ist also ausdrücklich in Ordnung – solange die frei betreibbare Fassung frei bleibt.

## Was ein gutes KI-Ökosystem ausmacht

Ein **Ökosystem** ist alles, was rund um ein Programm gewachsen ist: Zusatzbausteine, Anleitungen, erfahrene Leute, Werkzeuge. Ein gutes **KI-Ökosystem** heißt: Die Arbeit an diesem Web-Stack lässt sich gut von einem Sprachmodell und einem [KI-Agenten](../server-einrichten/ki-agent.md) unterstützen. Vier Dinge entscheiden darüber:

- **Wie gut kennt das Sprachmodell das Programm?** Ein Sprachmodell hat aus öffentlichem Programmtext gelernt (siehe [Text erstellen](./text-erstellen.md)). Für ein altes, weit verbreitetes Framework wie Django oder Laravel gibt es Millionen Zeilen Beispielcode; das Modell schlägt dafür zuverlässigen Code vor. Für ein junges Nischenprojekt rät es öfter daneben.
- **Gibt es offizielle KI-Bausteine?** Manche Projekte liefern selbst fertige Teile, um ein Sprachmodell einzubinden – etwa „Spring AI" für Java, die KI-Erweiterungen für Drupal oder die KI-Pakete für Laravel. Dann muss man die Anbindung nicht von Hand bauen.
- **Gibt es einen fertigen MCP-Server?** **MCP** (englisch „model context protocol", „Protokoll für den Modellkontext") ist eine gemeinsame Sprache, über die ein KI-Agent ein Programm untersuchen und bedienen darf. Ein mitgelieferter MCP-Server lässt den Agenten die eigene Anwendung „von innen" sehen – welche Datenfelder es gibt, welche Adressen, welche Einstellungen. Laravel und Drupal bringen so etwas mit.
- **Passt der Aufbau zur Arbeitsweise eines Agenten?** Ein monolithisches Programm mit festen Namensregeln ist für einen Agenten leichter zu überblicken als ein Feld verstreuter kleiner Dienste. „Batterien inklusive" und „ein Ort für den Code" helfen also nicht nur Menschen, sondern auch der Maschine.

Die passende Programmiersprache dazu und ihre Rolle im KI-Umfeld beschreibt das Kapitel [Programmiersprachen für KI](./ki-programmiersprachen.md).

## Was schon feststeht

Für alle drei Unterkapitel gelten dieselben vier Vorgaben aus den bisherigen Kapiteln:

- **Die Datenbank ist PostgreSQL.** Das Kapitel [Datenbank](../server-einrichten/datenbank.md) richtet PostgreSQL samt der Erweiterungen für die Suche ein. Ein Programm, das PostgreSQL nicht gleichwertig unterstützt, ist im Nachteil.
- **Es läuft direkt auf dem Server**, ohne Zwischenschicht wie einen Container (siehe [Containerisierung von Software](../server-einrichten/containerisierung.md)). Es muss sich also als Paket oder entpacktes Archiv auf **Ubuntu** einrichten lassen.
- **Es wird selbst betrieben.** Die Inhalte bleiben auf dem eigenen Server; reine Bezahldienste in fremder Hand scheiden aus.
- **Es ist quelloffen** – mit oder ohne Unternehmen dahinter, solange die selbst betriebene Fassung frei nutzbar bleibt.

## Für dieses Buch

Der Web-Stack dieses Buchs steht bereits: **MediaWiki** auf **PostgreSQL**, direkt auf dem Server. Die Enterprise-Frage stellt sich erst, wenn daneben etwas Neues entsteht – eine Firmenseite, ein Portal, eine eigene Anwendung. Für diesen Fall führen die drei Unterkapitel die Kandidaten auf und ordnen sie nach ihrem KI-Ökosystem.

Als grobe Richtung schon hier: Am besten von einem Sprachmodell unterstützen lassen sich die alten, weit verbreiteten Stacks in **Python** (Django), **PHP** (Laravel, Drupal, TYPO3) und **Java** (Spring Boot, XWiki) – schlicht, weil davon am meisten öffentlicher Beispielcode existiert und die Projekte selbst inzwischen KI-Bausteine mitliefern.

## Fazit

Ein **Enterprise Web-Stack** ist der Programmstapel für einen Auftritt, der groß werden, lange laufen und robust bleiben soll. Das oberste Programm dieses Stapels fällt in drei Fälle: ein selbst geschriebenes Programm auf einem **Webframework**, ein **CMS** für eine Redaktion oder ein **Wissenssystem** für ein gemeinsames Nachschlagewerk. Für alle drei gilt dieselbe Vorgabe: PostgreSQL, direkt auf dem Server, selbst betrieben, quelloffen – auch mit einem Unternehmen dahinter. Neu ist die Frage nach dem **KI-Ökosystem**: Wie viel öffentlicher Beispielcode existiert, ob es offizielle KI-Bausteine und einen fertigen MCP-Server gibt und ob der Aufbau zur Arbeitsweise eines Agenten passt. Die drei Unterkapitel führen das für Webframeworks, CMS und Wissenssysteme im Einzelnen aus.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
