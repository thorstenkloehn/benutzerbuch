# Webframework

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Die Wissenssammlung dieses Buchs läuft auf MediaWiki. MediaWiki ist eine fertige Anwendung: Man muss sie nur einrichten, nicht programmieren. Sobald aber ein eigenes Zusatzprogramm dazukommen soll – eine Schnittstelle, über die andere Programme die Inhalte abrufen, ein Werkzeug, das Texte aus einer alten Datenbank einliest, oder eine kleine Weboberfläche für eine Aufgabe, die MediaWiki nicht abdeckt –, schreibt man dieses Programm selbst. Dann stellt sich die Frage nach dem **Webframework**.

Dieses Kapitel erklärt, was ein Webframework ist, worin es sich von einer fertigen Anwendung wie MediaWiki unterscheidet, und stellt die Frameworks vor, die als besonders ausgereift und belastbar gelten – also solche, die auch mit sehr vielen Inhalten und über viele Jahre zuverlässig laufen. Es geht dabei nicht um die Einrichtung Schritt für Schritt, denn welches Framework überhaupt in Frage kommt, hängt vom geplanten Zusatzprogramm ab. Das Kapitel liefert die Grundlage für diese Wahl.

## Was ein Webframework ist

Wer ein Haus baut, gießt nicht jeden Ziegel selbst und erfindet auch nicht die Statik neu. Es gibt Normmaße, fertige Träger, geprüfte Bauteile. Ein Webframework ist dieser Satz geprüfter Bauteile für Webprogramme. Es nimmt einem die immer gleichen Grundaufgaben ab, die jedes Webprogramm hat:

- **Anfragen verstehen.** Aus der rohen Anfrage des Browsers die wichtigen Angaben herauslesen – welche Adresse, welche Formulardaten, welcher angemeldete Benutzer.
- **Die richtige Funktion aufrufen.** Anhand der Adresse entscheiden, welcher Teil des eigenen Codes zuständig ist. Das nennt man **Routing**.
- **Mit der Datenbank reden.** Inhalte aus der Datenbank holen und speichern, ohne dass man die Datenbanksprache SQL von Hand zusammensetzen muss. Der Vermittler dafür heißt **ORM** (englisch für „objektrelationale Abbildung").
- **Antworten erzeugen.** Aus den Daten eine fertige HTML-Seite oder eine Antwort im JSON-Format bauen.
- **Vor Angriffen schützen.** Eingaben prüfen, Passwörter sicher speichern, gängige Angriffsmuster von vornherein abwehren.

Ein Framework ist also kein fertiges Programm, sondern ein Gerüst, in das man den eigenen Code einhängt. Es gibt die Struktur vor; die Fachlogik – was das Programm konkret tun soll – schreibt man selbst.

### Framework, Anwendung, Content-Management-System

Drei Begriffe werden leicht verwechselt:

- Eine **Anwendung** wie MediaWiki ist fertig. Man installiert sie und füllt sie mit Inhalten.
- Ein **Content-Management-System** (CMS) wie WordPress, Drupal oder TYPO3 ist ebenfalls eine fertige Anwendung, aber eine, die stark auf das Verwalten von Seiteninhalten zugeschnitten und über Erweiterungen anpassbar ist. Für ein Wissensportal ist ein CMS oft schon die ganze Lösung.
- Ein **Framework** ist der Rohbau darunter. MediaWiki, WordPress und Drupal bringen ihren eigenen Unterbau mit; andere Anwendungen sind sichtbar auf einem bekannten Framework gebaut.

Wer nur eine Wissenssammlung betreiben will, bleibt bei der fertigen Anwendung. Das Framework wird erst zum Thema, wenn ein Zusatzprogramm entsteht, für das es nichts Fertiges gibt.

## Was „robust" bei einem Framework bedeutet

Die Frage „Welches Framework hält auch eine Million Einträge aus, ohne kaputtzugehen?" lässt sich nicht am Framework allein beantworten. Drei Dinge spielen zusammen:

- **Die Datenbank trägt die Last, nicht das Framework.** Eine Million Texteinträge sind für eine ausgewachsene Datenbank wie PostgreSQL (siehe [Inhalts-Software selbst betreiben](../grundlagen/inhalt-software-verwalten.md)) eine normale Größe. Ob das schnell bleibt, entscheidet sich an den Datenbank-Indizes und an der Frage, ob das Programm immer nur die gerade benötigten Einträge lädt – niemals alle auf einmal.
- **Das Framework muss sparsam mit Daten umgehen.** Ein gutes Framework holt Einträge seitenweise (**Pagination**), liefert große Mengen häppchenweise aus (**Streaming**) und lädt verknüpfte Daten gebündelt statt in tausend Einzelabfragen. Ob das passiert, liegt aber am Code, den man selbst schreibt – das Framework macht es nur möglich und bequem.
- **Reifegrad heißt: Es bricht nicht bei der nächsten Aktualisierung.** Ein ausgereiftes Framework hat einen festen Veröffentlichungsplan, eine Ausgabe mit langer Pflegezusage (**LTS**), eine klare Vorgehensweise beim Umstieg auf neue Versionen und eine große Gemeinschaft, die Fehler früh findet. Das ist der Unterschied zwischen einem Framework, auf das man einen Server für zehn Jahre stellt, und einem, das nach zwei Jahren nicht mehr gepflegt wird.

Verschiedene Inhaltstypen und Datensätze – Artikel, Bilder, Kategorien, Verweise, Nutzerkonten – verwaltet man über das Datenmodell. Jedes hier genannte Framework kann beliebig viele solcher Typen abbilden; die Arbeit steckt im sauberen Entwurf des Modells, nicht in der Wahl des Frameworks.

## Ausgereifte Frameworks im Überblick

Die folgenden Frameworks gelten als besonders belastbar: Sie sind seit vielen Jahren im Einsatz, werden von großen Auftritten verwendet, haben einen verlässlichen Pflegeplan und eine breite Gemeinschaft. Sie sind nach der Programmiersprache geordnet; die passende Laufzeitumgebung dazu beschreibt das Kapitel [Laufzeitumgebung](./laufzeitumgebung.md).

### Django und FastAPI (Python)

**Django** ist das Rundum-sorglos-Framework für Python. Es bringt ORM, Nutzerverwaltung, Sicherheitsfunktionen und eine fertige Verwaltungsoberfläche für die eigenen Daten von Haus aus mit. Genau diese eingebaute Verwaltungsoberfläche macht Django stark, wenn viele verschiedene Inhaltstypen zu pflegen sind: Man beschreibt das Datenmodell einmal und bekommt die Eingabemasken dazu geschenkt. Django gibt es seit 2005, es hat einen festen Veröffentlichungsplan mit LTS-Ausgaben und wird von sehr großen Auftritten eingesetzt.

**FastAPI** ist der modernere, schlankere Gegenentwurf – kein Rundum-Paket, sondern spezialisiert auf Schnittstellen im JSON-Format. Es ist schnell, erzeugt automatisch eine Beschreibung der eigenen Schnittstelle und eignet sich gut, wenn das Zusatzprogramm nur Daten bereitstellen und keine Webseiten anzeigen soll.

### Laravel und Symfony (PHP)

Da MediaWiki in PHP geschrieben ist, liegt ein PHP-Framework für Zusatzprogramme nahe: Die Laufzeitumgebung ist ohnehin schon auf dem Server.

**Laravel** ist das meistgenutzte moderne PHP-Framework. Es gilt als angenehm zu bedienen, bringt viele fertige Bausteine mit (Warteschlangen für Hintergrundaufgaben, Zwischenspeicher, Nutzerverwaltung) und hat eine sehr große Gemeinschaft. **Symfony** ist der Baukasten darunter: eine Sammlung einzeln nutzbarer, sehr sorgfältig gepflegter Bauteile, aus denen unter anderem auch Teile von Laravel und dem CMS Drupal bestehen. Symfony hat einen besonders strengen Veröffentlichungsplan mit LTS-Ausgaben, die mehrere Jahre Unterstützung erhalten.

### Ruby on Rails (Ruby)

**Rails** hat vieles von dem geprägt, was heute an allen Frameworks selbstverständlich ist. Es ist seit 2004 im Einsatz, sehr ausgereift und darauf ausgelegt, dass ein kleines Team schnell viel erreicht. Große, seit Jahren wachsende Auftritte laufen auf Rails. Wer keine Ruby-Kenntnisse im Umfeld hat, wählt es aber selten neu.

### Spring Boot (Java)

**Spring Boot** ist der Standard für Java im Unternehmensumfeld. Es ist auf Langlebigkeit, Stabilität und die Anbindung an andere Unternehmenssysteme ausgelegt. Der Preis dafür ist ein höherer Einstiegsaufwand und mehr Verbrauch an Arbeitsspeicher als bei den schlankeren Frameworks. Für ein kleines Zusatzprogramm neben einer Wissenssammlung ist Spring Boot meist überdimensioniert; seine Stärke spielt es aus, wenn viele Dienste über Jahre zusammenwachsen.

### ASP.NET Core (C#)

**ASP.NET Core** ist das Web-Framework von Microsoft, seit einigen Jahren quelloffen und auch auf Linux zu Hause. Es ist ausgereift, gut dokumentiert und in Vergleichsmessungen regelmäßig unter den schnellsten der weit verbreiteten Frameworks. Die passende Laufzeit lässt sich auf Ubuntu direkt aus den Paketquellen einrichten.

### Express und NestJS (JavaScript, Node.js)

**Express** ist ein sehr kleines, seit vielen Jahren bewährtes Framework für Node.js – es kümmert sich fast nur um das Routing und überlässt den Rest Zusatzbausteinen. **NestJS** legt eine feste Struktur darüber und ähnelt damit eher den Rundum-Frameworks. Node.js ist ohnehin auf dem Server, wenn dort ein KI-Agent mit n8n läuft (siehe [KI-Agenten auf dem Server](./ki-agent.md)).

### Gin und Echo (Go)

Frameworks in der Sprache Go werden vor der Auslieferung zu einer einzelnen, eigenständigen Datei übersetzt, die ohne separate Laufzeitumgebung startet. **Gin** und **Echo** sind schlanke, schnelle Frameworks für Schnittstellen. Sie sind ausgereift, brauchen wenig Arbeitsspeicher und eignen sich gut für einen kleinen, dauerhaft laufenden Zusatzdienst.

### Actix Web und Axum (Rust)

In der Sprache Rust geschriebene Frameworks wie **Actix Web** und **Axum** stehen in Vergleichsmessungen regelmäßig an der Spitze. Rust ist dafür bekannt, ganze Klassen von Programmierfehlern schon beim Übersetzen zu verhindern. Der Einstieg ist allerdings anspruchsvoll, und für die meisten Zusatzprogramme neben einer Wissenssammlung ist die Höchstgeschwindigkeit gar nicht der Engpass.

## Zur Geschwindigkeit: Was Benchmarks sagen – und was nicht

Es gibt einen bekannten, offen einsehbaren Vergleich, die **TechEmpower-Benchmarks**. Dabei werden Hunderte Frameworks unter gleichen Bedingungen gemessen: einfache Antworten, Datenbankabfragen, das Zusammensetzen einer HTML-Seite. Die jüngste Runde (Runde 23, Februar 2025) zeigt das übliche Bild:

| Framework (Sprache) | Einordnung im Vergleich |
| --- | --- |
| Actix Web, Axum (Rust) | ganz oben |
| ASP.NET Core (C#) | sehr weit oben unter den verbreiteten Frameworks |
| Gin, Echo (Go) | weit oben |
| Spring Boot (Java) | oberes Mittelfeld |
| Express, NestJS (Node.js) | Mittelfeld |
| Rails (Ruby), Django (Python) | unteres Mittelfeld |
| Laravel (PHP) | unteres Mittelfeld |

Diese Reihenfolge sagt weniger aus, als es zunächst scheint. Zwei Punkte sind wichtig:

- **Die Messung testet den Leerlauf des Frameworks.** In einem echten Programm liegt die Wartezeit fast immer woanders: bei der Datenbank, beim Suchdienst, bei einer Anfrage an ein anderes Programm. Ein Framework, das im Test doppelt so schnell ist, macht das Gesamtprogramm selten spürbar schneller.
- **Ein Zwischenspeicher hebt jedes Framework auf ein anderes Niveau.** Wird eine fertige Antwort für kurze Zeit vorgehalten (etwa mit Redis oder direkt im Webserver), fällt das Framework als Zeitfaktor praktisch heraus.

Als grobe Regel gilt: Solange ein Zusatzprogramm nicht Tausende Anfragen pro Sekunde beantworten muss, ist die Framework-Geschwindigkeit nachrangig. Wichtiger sind eine gute Datenbankgestaltung, sinnvolle Indizes und ein Zwischenspeicher.

## Direkt auf dem Server oder im Container

Ein selbst geschriebenes Programm mit Webframework lässt sich auf zwei Wegen betreiben:

- **Direkt auf dem Server** („bare metal", also ohne Zwischenschicht): Man richtet die Laufzeitumgebung ein, legt das Programm ab und startet es als Hintergrunddienst über `systemd`. Davor sitzt der Webserver als Reverse Proxy (siehe [Webserver](./webserver.md)). Dieser Weg hat die wenigsten beweglichen Teile.
- **Im Container** (siehe [Containerisierung von Software](./containerisierung.md)): Das Programm wird mit seiner Laufzeitumgebung in ein Abbild gepackt. Das erleichtert den Umzug auf einen anderen Server und das Nebeneinander von Test- und Betriebsfassung.

Für ein einzelnes kleines Zusatzprogramm ist der direkte Weg meist der einfachere. Sobald mehrere solche Programme zusammenkommen, wird der Container attraktiver.

## Für dieses Buch

Für die beschriebene Wissenssammlung wird **kein** Webframework benötigt: MediaWiki, PostgreSQL, der Suchdienst und der KI-Agent decken alles ab. Ein Framework kommt erst ins Spiel, wenn ein eigenes Zusatzprogramm entsteht.

Für diesen Fall bieten sich zwei Wege an:

- **Bleibt man bei PHP**, weil die Laufzeitumgebung wegen MediaWiki ohnehin da ist, sind **Laravel** (bequem, große Gemeinschaft) oder **Symfony** (sehr sorgfältig gepflegt) die naheliegenden Frameworks.
- **Geht es vor allem um eine Datenschnittstelle**, ist **FastAPI** (Python) oder **Gin** (Go) schlank und schnell eingerichtet. Läuft ohnehin schon Node.js für den KI-Agenten, ist **Express** eine gute Wahl.

**Django** lohnt sich, wenn das Zusatzprogramm selbst viele eigene Inhaltstypen verwalten soll – die eingebaute Verwaltungsoberfläche spart dann viel Arbeit. **Spring Boot**, **ASP.NET Core** und die Rust-Frameworks sind für ein kleines Zusatzprogramm überdimensioniert; sie sind die richtige Wahl in größeren, über Jahre wachsenden Systemen.

## Fazit

Ein Webframework ist der geprüfte Rohbau für ein selbst geschriebenes Webprogramm; es nimmt Routing, Datenbankzugriff, Antworterzeugung und Grundschutz ab. Für eine reine Wissenssammlung mit MediaWiki braucht man keines. Wird ein Zusatzprogramm nötig, gelten **Django** und **FastAPI** (Python), **Laravel** und **Symfony** (PHP), **Ruby on Rails**, **Spring Boot** (Java), **ASP.NET Core** (C#), **Express** und **NestJS** (Node.js), **Gin** und **Echo** (Go) sowie **Actix Web** und **Axum** (Rust) als besonders ausgereift. Ob ein Framework „eine Million Einträge aushält", entscheidet sich nicht am Framework, sondern an der Datenbank, an sauberem Datenzugriff und an einem Zwischenspeicher. Geschwindigkeits-Benchmarks messen den Leerlauf des Frameworks und sagen für den Alltag wenig aus. Für dieses Buch gilt: bei PHP bleiben, wenn nur ein kleines Zusatzprogramm gebraucht wird, und das Programm direkt auf dem Server hinter dem Webserver betreiben.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
