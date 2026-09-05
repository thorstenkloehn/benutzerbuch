# Inhalts-Software selbst betreiben

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Wer eine eigene Wissenssammlung aufbaut, kann die dafür nötigen Programme mieten oder selbst betreiben. „Selbst betreiben" (oft „Selfhosting" genannt) heißt: Die Software läuft auf einem Rechner, über den man selbst bestimmt, statt auf den Servern eines fremden Anbieters. Der Vorteil ist, dass die Inhalte im eigenen Haus bleiben und niemand sie abschalten oder verteuern kann. Der Preis dafür ist etwas mehr Arbeit bei Einrichtung und Pflege.

Dieses Kapitel beschreibt einen erprobten Satz an Programmen für eine solche Sammlung. Jedes Programm hat eine klare Aufgabe. Zusammen ergeben sie eine Wissensdatenbank, in der man Texte schreibt, schnell darin sucht und Fragen in normaler Sprache stellen kann.

## Die Bausteine im Überblick

| Programm | Aufgabe |
| --- | --- |
| MediaWiki | Ort, an dem die Texte geschrieben und gelesen werden |
| PostgreSQL | Datenbank, die alle Inhalte dauerhaft speichert |
| Meilisearch | Volltextsuche über die Wörter in den Texten |
| pgvector | Erweiterung von PostgreSQL für die Suche nach Bedeutung |
| Ollama | führt Sprachmodelle auf dem eigenen Rechner aus |

Alle fünf sind Open-Source-Software: Ihr Quelltext liegt offen, und man darf sie kostenlos benutzen und selbst betreiben.

## MediaWiki – der Ort für die Texte

MediaWiki ist die Software, mit der auch die Wikipedia läuft. Sie stellt eine Webseite bereit, deren Seiten man direkt im Browser bearbeitet. Zu jeder Seite wird die komplette Änderungsgeschichte gespeichert, sodass sich jeder frühere Stand wiederherstellen lässt. Seiten lassen sich untereinander verlinken und in Kategorien einordnen.

MediaWiki ist auf viele Seiten und viele Bearbeiter ausgelegt und seit Jahren bewährt. Für eine Wissenssammlung, die wachsen soll, ist das eine solide Grundlage. Die Software selbst kümmert sich nur um das Anzeigen und Bearbeiten der Seiten. Wo die Texte tatsächlich liegen, ist ihre Sache nicht – dafür ist die Datenbank zuständig.

## PostgreSQL – die Datenbank darunter

Eine Datenbank ist ein Programm, das Daten geordnet ablegt und auf Anfrage schnell wieder herausgibt. MediaWiki speichert dort jede Seite, jede Version und jede Einstellung. Ohne Datenbank gäbe es nichts anzuzeigen.

MediaWiki kann mehrere Datenbanken verwenden. Hier fällt die Wahl auf PostgreSQL, weil es besonders zuverlässig ist und sich um zusätzliche Funktionen erweitern lässt. Eine davon – die Suche nach Bedeutung – wird weiter unten wichtig. Für den Anfang genügt der Gedanke: PostgreSQL ist der Tresor, in dem alle Inhalte liegen, und MediaWiki ist die Theke, an der man sie bekommt.

## Meilisearch – die schnelle Volltextsuche

Sobald eine Sammlung ein paar Hundert Seiten hat, braucht man eine gute Suche. Die eingebaute Suche von MediaWiki ist einfach und wird bei vielen Seiten langsam. Deshalb übernimmt ein eigenes Programm diese Aufgabe: Meilisearch.

Meilisearch ist eine schlanke Suchmaschine, die man selbst betreibt. Sie führt ein eigenes Verzeichnis aller Wörter in allen Texten (einen sogenannten Index) und liefert Treffer schon während des Tippens. Auch Tippfehler verzeiht sie: Wer „Meiliserch" eingibt, findet trotzdem die richtigen Seiten. Meilisearch durchsucht dabei die Wörter selbst – es findet also genau das, wonach man buchstäblich gefragt hat.

## pgvector – die Suche nach Bedeutung

Manchmal weiß man nicht das genaue Wort, das im Text steht. Man sucht „Auto reparieren", aber die passende Seite heißt „Wagen instand setzen". Eine reine Wortsuche findet sie nicht. Hier hilft die Suche nach Bedeutung, auch semantische Suche genannt.

Dafür wird jeder Text in eine lange Zahlenreihe umgerechnet, die seinen Inhalt beschreibt (fachlich: ein „Vektor" oder „Embedding"). Texte mit ähnlicher Bedeutung ergeben ähnliche Zahlenreihen. Sucht man nun etwas, wird auch die Suchanfrage in eine solche Zahlenreihe umgerechnet, und das System sucht die Texte mit den ähnlichsten Werten.

Diese Zahlenreihen müssen irgendwo gespeichert und schnell verglichen werden. Genau das leistet **pgvector**, eine Erweiterung für PostgreSQL. Damit wird aus der ohnehin vorhandenen Datenbank zugleich eine Vektordatenbank – ein zweites, getrenntes Programm ist dafür nicht nötig. Wer die Zahlenreihen lieber außerhalb von PostgreSQL verwalten möchte, kann stattdessen eine eigenständige Vektordatenbank einsetzen; für den hier beschriebenen Aufbau reicht pgvector.

In der Praxis kombiniert man beide Suchen: Meilisearch für die genaue Wortsuche, pgvector für die Suche nach Bedeutung. Zusammen finden sie deutlich mehr als jede für sich.

## Ollama – Sprachmodelle auf dem eigenen Rechner

Zwei Dinge aus den vorigen Abschnitten brauchen ein Sprachmodell: das Umrechnen der Texte in Zahlenreihen und das Beantworten von Fragen in normaler Sprache. Man könnte dafür einen Online-Dienst benutzen, müsste dann aber jeden Text dorthin schicken. Wer die Inhalte im eigenen Haus behalten will, lässt das Modell ebenfalls selbst laufen.

Ollama ist ein Programm, das genau das einfach macht. Es lädt ein Sprachmodell herunter und stellt es auf dem eigenen Rechner bereit, ansprechbar über eine feste Schnittstelle. Andere Programme – etwa das Modul, das die Zahlenreihen für pgvector erzeugt – fragen dann bei Ollama an, statt bei einem fremden Anbieter.

Für gute Ergebnisse braucht Ollama einen leistungsfähigen Rechner, am besten mit einer starken Grafikkarte. Kleinere Modelle laufen auch auf normaler Hardware, nur langsamer.

## Wie die Teile zusammenspielen

Ein typischer Ablauf sieht so aus:

1. Jemand schreibt oder ändert eine Seite in **MediaWiki**.
2. Der Text wird in **PostgreSQL** gespeichert.
3. Im Hintergrund meldet MediaWiki die Änderung an **Meilisearch**, das seinen Wort-Index aktualisiert.
4. Ebenfalls im Hintergrund lässt **Ollama** den Text in eine Zahlenreihe umrechnen, die in **pgvector** landet.
5. Sucht später jemand etwas, fragen Wortsuche (Meilisearch) und Bedeutungssuche (pgvector) parallel, und die Treffer werden zusammengeführt.
6. Für eine Frage in normaler Sprache holt sich das System die passenden Seiten aus Schritt 5 und lässt **Ollama** daraus eine Antwort formulieren.

Jeder Baustein lässt sich einzeln austauschen, ohne die anderen anzufassen: ein anderes Wiki, eine andere Suchmaschine, ein anderes Modell. Das ist der Vorteil klar getrennter Aufgaben.

## Fazit

Für eine selbst betriebene Wissenssammlung genügt ein kleiner, überschaubarer Satz an Programmen: MediaWiki als Oberfläche, PostgreSQL als Speicher, Meilisearch für die Wortsuche, pgvector für die Bedeutungssuche und Ollama für die Sprachmodelle. Alle sind quelloffen und kostenlos. Wichtig ist weniger die konkrete Auswahl als das Prinzip dahinter: Jedes Programm macht eine Sache, die Teile sind sauber getrennt, und die Inhalte bleiben auf einem Rechner, über den man selbst bestimmt.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
