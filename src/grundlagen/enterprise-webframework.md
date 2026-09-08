# Enterprise-Webframework

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Wenn für eine geplante Anwendung nichts Fertiges passt, schreibt man sie selbst – auf einem **Webframework**, dem geprüften Rohbau für Webprogramme. Was ein Framework ist und welche Frameworks als besonders belastbar gelten, steht ausführlich im Kapitel [Webframework](../server-einrichten/webframework.md). Dieses Kapitel setzt darauf auf und stellt eine engere Frage aus dem Kapitel [Enterprise Web-Stack](./enterprise-web-stack.md): Welche Frameworks sind **„Batterien inklusive"** und **monolithisch**, laufen auf **PostgreSQL**, taugen für sehr große Auftritte – und lassen sich am besten mit Hilfe eines Sprachmodells bearbeiten?

## Batterien inklusive: was dazugehört

Ein batterienreiches Framework bringt die immer gleichen Grundbausteine schon mit, statt sie einzeln zusammensuchen zu lassen:

- **ORM** – der Vermittler zur Datenbank, damit man die Datenbanksprache SQL nicht von Hand schreiben muss (englisch „object-relational mapping").
- **Migrationen** – Anweisungen, die den Aufbau der Datenbank Schritt für Schritt mitwachsen lassen.
- **Benutzerverwaltung** – Anmeldung, Passwörter, Rechte.
- **Verwaltungsoberfläche** – fertige Eingabemasken für die eigenen Daten.
- **Hintergrundaufgaben** – eine Warteschlange für Dinge, die später erledigt werden (E-Mail verschicken, Bilder umrechnen).
- **Grundschutz** – Abwehr der gängigen Angriffsmuster von Haus aus.

Je mehr davon eingebaut ist, desto weniger Fremdteile muss man aussuchen, aktuell halten und im Zusammenspiel prüfen – ein großer Vorteil über zehn Jahre Betrieb.

## Die Kandidaten im Überblick

Die Reihenfolge folgt grob dem KI-Ökosystem: oben die Frameworks, für die es am meisten öffentlichen Beispielcode und die reifsten KI-Bausteine gibt.

| Framework | Sprache | Seit | Batterien inklusive | PostgreSQL | KI-Ökosystem |
| --- | --- | --- | --- | --- | --- |
| Django | Python | 2005 | sehr viel (ORM, Admin, Auth) | erste Wahl | sehr groß: ganze Python-KI-Welt, viel Beispielcode |
| Laravel | PHP | 2011 | sehr viel (inkl. Warteschlangen, Cache) | ja | sehr groß: „Boost" (MCP-Server) und KI-Pakete offiziell |
| Ruby on Rails | Ruby | 2004 | sehr viel (ab Version 8 ohne Zusatzdienste) | ja | groß: viel Beispielcode, lange Tradition fester Regeln |
| Spring Boot | Java | 2014 | viel (über Bausteine „Starter") | ja | groß: „Spring AI" seit 2025 offiziell, mit MCP |
| ASP.NET Core | C# | 2016 | viel (EF Core, Identity) | ja (Npgsql) | groß: Microsoft-KI-Bausteine, gute Dokumentation |
| AdonisJS | TypeScript | 2016 | viel (ORM „Lucid", Auth) | ja | mittel: TypeScript-KI-Welt groß, Framework kleiner |
| Phoenix | Elixir | 2015 | viel (ORM „Ecto", Live-Oberflächen) | Standard | mittel: wenig Beispielcode, aber sehr feste Regeln |
| Frappe | Python | 2010 | sehr viel (ganze Plattform hinter ERPNext) | wächst (lange nur MariaDB) | klein: Nische, wenig Beispielcode |
| Encore | Go/TypeScript | 2021 | viel, aber auf einen Betreiber zugeschnitten | ja | klein: jung, kleines Umfeld |
| RedwoodSDK / Wasp | JavaScript/TypeScript | 2020 / 2021 | viel | ja | klein: jung, im Umbruch |

Die ersten fünf Zeilen sind die tragfähige Wahl für einen großen, langlebigen Auftritt. Die unteren fünf sind entweder jung, im Umbruch oder eine Nische – interessant, aber mit kürzerem Erfolgsnachweis.

## Nach KI-Ökosystem sortiert

- **Django (Python).** Django bringt ORM, Benutzerverwaltung und eine fertige Verwaltungsoberfläche mit; man beschreibt das Datenmodell einmal und bekommt die Eingabemasken geschenkt. Der KI-Vorteil ist zweifach: Erstens ist Django seit 2005 im Einsatz und millionenfach öffentlich dokumentiert, ein Sprachmodell schlägt dafür sehr verlässlichen Code vor. Zweitens ist Python ohnehin die Sprache mit den meisten KI-Bausteinen (siehe [Programmiersprachen für KI](./ki-programmiersprachen.md)) – die Anbindung an ein Sprachmodell entsteht im selben Projekt ohne Sprachwechsel.
- **Laravel (PHP).** Das meistgenutzte moderne PHP-Framework, sehr batterienreich und mit riesiger Gemeinschaft. Laravel ist beim Thema KI am weitesten vorne: Seit 2025 gibt es „Laravel Boost", einen offiziellen **MCP-Server**, der einem KI-Agenten die eigene Anwendung von innen zeigt und ihm auf die installierten Pakete zugeschnittene Hinweise gibt, dazu offizielle Pakete zum Einbinden von Sprachmodellen. Wer PHP schon wegen MediaWiki auf dem Server hat, bekommt hier das reifste KI-Ökosystem.
- **Ruby on Rails (Ruby).** Rails hat den Stil „eine richtige Art, es zu tun" geprägt – feste Namen, feste Ordnerstruktur. Genau das hilft einem Agenten: Er kann aus dem Namen einer Datei auf ihren Inhalt schließen. Ab Version 8 (Ende 2024) braucht Rails für Warteschlange, Cache und Live-Verbindungen keinen Zusatzdienst mehr, alles läuft über die PostgreSQL-Datenbank. Es gibt viel Beispielcode, aber weniger als für Django oder Laravel.
- **Spring Boot (Java).** Der Standard für Java im Unternehmensumfeld – auf Langlebigkeit und die Anbindung an andere Firmensysteme ausgelegt. Seit Mai 2025 gibt es „Spring AI" als offiziellen Baustein, inklusive MCP-Unterstützung. Der Preis ist ein höherer Einstiegsaufwand und mehr Verbrauch an Arbeitsspeicher. Die richtige Wahl, wenn viele Dienste über Jahre zusammenwachsen.
- **ASP.NET Core (C#).** Das Web-Framework von Microsoft, seit Jahren quelloffen und auf Linux zu Hause. Es ist ausgereift, sehr gut dokumentiert und in Vergleichsmessungen regelmäßig unter den schnellsten. Microsoft liefert eigene KI-Bausteine mit; die Dokumentation ist so gründlich, dass ein Sprachmodell selten daneben rät.
- **AdonisJS (TypeScript).** AdonisJS ist der Versuch, das Laravel-Gefühl in die Sprache TypeScript zu holen: ORM, Benutzerverwaltung, feste Struktur. Die TypeScript-KI-Welt ist groß, das Framework selbst aber deutlich kleiner als Django oder Laravel – entsprechend weniger zugeschnittener Beispielcode.
- **Phoenix (Elixir).** Technisch stark, besonders für Seiten mit vielen gleichzeitigen Live-Verbindungen, und PostgreSQL ist die Standarddatenbank. Elixir ist aber eine kleine Sprache; ein Sprachmodell hat weniger Beispiele gesehen. Die sehr festen Regeln des Frameworks gleichen das teilweise aus.
- **Frappe, Encore, RedwoodSDK, Wasp.** Alle vier sind batterienreich und modern, aber jung oder im Umbau. Frappe (die Plattform hinter der Unternehmenssoftware ERPNext) war lange auf MariaDB festgelegt und öffnet sich erst nach und nach für PostgreSQL. Für einen Auftritt, der zehn Jahre halten soll, sind sie die riskantere Wette – und für ein Sprachmodell die schwierigeren, weil es dafür wenig gelernt hat.

## PostgreSQL bei den Kandidaten

Bei den oberen fünf ist PostgreSQL gleichwertig oder sogar bevorzugt:

- **Django** nennt PostgreSQL in der eigenen Dokumentation die empfohlene Datenbank und bietet dafür Zusatzfunktionen, die andere Datenbanken nicht bekommen.
- **Rails** und **Phoenix** setzen PostgreSQL faktisch als Standard.
- **Laravel**, **Spring Boot** und **ASP.NET Core** sprechen die Datenbank über eine Zwischenschicht an und unterstützen PostgreSQL vollständig; verbreiteter ist bei Laravel MySQL, was aber nur bedeutet, dass es dafür mehr Anleitungen gibt.

Die im Kapitel [Datenbank](../server-einrichten/datenbank.md) eingerichtete Erweiterung `pg_trgm` für die unscharfe Suche steht allen zur Verfügung.

## Zur Vergleichbarkeit

Für die Frage „Welches Framework ist schnell genug?" gilt das im Kapitel [Webframework](../server-einrichten/webframework.md) Gesagte unverändert: Die offenen Vergleichsmessungen (TechEmpower) testen den Leerlauf des Frameworks, nicht den Alltag. In einer echten Anwendung liegt die Wartezeit fast immer bei der Datenbank, beim Suchdienst oder bei einer Anfrage an ein anderes Programm. Ein Zwischenspeicher für fertige Antworten hebt jedes dieser Frameworks auf ein Niveau, auf dem die reine Framework-Geschwindigkeit keine Rolle mehr spielt.

## Für dieses Buch

Für die Wissenssammlung dieses Buchs wird **kein** Webframework gebraucht. Entsteht ein eigenes Zusatzprogramm, gilt:

- **Bleibt man bei PHP** (die Laufzeitumgebung ist wegen MediaWiki ohnehin da), ist **Laravel** die Wahl – batterienreich, große Gemeinschaft und das derzeit beste KI-Ökosystem unter allen Frameworks.
- **Soll das Programm viele eigene Inhaltstypen verwalten**, ist **Django** stark: Die eingebaute Verwaltungsoberfläche spart viel Arbeit, und die KI-Anbindung entsteht ohne Sprachwechsel im selben Python-Projekt.
- **Spring Boot** und **ASP.NET Core** sind die Wahl in einem größeren Verbund vieler Dienste, nicht für ein einzelnes kleines Zusatzprogramm.

## Fazit

Für einen großen, langlebigen Auftritt ist ein **monolithisches, batterienreiches** Framework die richtige Grundlage. Nach dem KI-Ökosystem geordnet stehen oben **Django** (Python), **Laravel** (PHP), **Ruby on Rails** (Ruby), **Spring Boot** (Java) und **ASP.NET Core** (C#): alle seit vielen Jahren im Einsatz, alle mit viel öffentlichem Beispielcode und inzwischen mit offiziellen KI-Bausteinen – Laravel sogar mit einem eigenen MCP-Server. **AdonisJS** und **Phoenix** sind solide, aber mit kleinerem Umfeld; **Frappe**, **Encore**, **RedwoodSDK** und **Wasp** sind jung oder im Umbruch. PostgreSQL wird von allen ernsthaften Kandidaten voll unterstützt, bei Django, Rails und Phoenix ist es die Standardwahl.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
