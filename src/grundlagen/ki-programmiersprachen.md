# Programmiersprachen für KI

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Für die reine Schreibarbeit an diesem Buch muss niemand programmieren. Sobald aber ein eigener KI-Agent, ein Skript oder eine Anbindung an das Redaktionssystem entsteht (siehe [KI-Agenten auf dem Server](../server-einrichten/ki-agent.md) und [Content-Management-System](../web-stack/cms.md)), stellt sich die Frage: In welcher Sprache schreibt man so etwas?

Die Antwort ist nicht „in einer", sondern „in mehreren – je nach Aufgabe". Eine KI-Anwendung besteht aus drei übereinanderliegenden Schichten, und jede Schicht hat ihre eigene bevorzugte Programmiersprache. Eine **Programmiersprache** ist die Sprache, in der ein Mensch einem Computer Anweisungen aufschreibt. Ein Überblick über die einzelnen Sprachen und ihre Einrichtung auf dem eigenen Rechner steht im Kapitel [Programmiersprachen](../entwicklungs-rechner/programmiersprachen.md); dieses Kapitel ordnet sie nach ihrer Rolle im KI-Umfeld ein.

## Die drei Schichten im Überblick

| Schicht | Aufgabe | Übliche Sprachen |
| --- | --- | --- |
| Entwicklung und Training | ein Modell bauen, trainieren, ausprobieren | Python |
| Laufzeit und Beschleunigung | ein fertiges Modell schnell rechnen lassen | C++, Rust, CUDA |
| Integration und Orchestrierung | das Modell mit Datenbank, Oberfläche und Suche verbinden | TypeScript, Go, C#, Java, Python |

Ein **Modell** ist die trainierte Datei, die aus einer Eingabe eine Antwort errechnet. **Trainieren** heißt, dem Modell diese Fähigkeit über viele Beispiele beizubringen. Die oberste Schicht baut das Modell, die mittlere lässt es schnell laufen, die dritte bindet es in eine nutzbare Anwendung ein.

Neben diesem Aufbau gab es früher einen ganz anderen Ansatz, wie ein Computer „intelligent" wird – mit eigenen Programmiersprachen. Er wird weiter unten im Abschnitt „Eine ältere Linie" beschrieben.

## Schicht 1: Entwicklung und Training – Python

Wer ein Modell entwirft, trainiert oder mit neuen Ideen ausprobiert, arbeitet fast immer mit **Python**. Python ist eine Sprache, die auf gut lesbaren Code setzt und Anweisungen Zeile für Zeile abarbeitet, ohne sie vorher in Maschinencode zu übersetzen.

Für diese Schicht zählt Python aus drei Gründen:

- **Schnelles Ausprobieren.** Eine Idee lässt sich in wenigen Zeilen aufschreiben und sofort testen. Wer forscht, ändert seinen Code ständig – langes Übersetzen würde nur bremsen.
- **Rechnen mit Zahlengittern.** Ein Modell ist im Kern eine riesige Sammlung von Zahlen, angeordnet in Gittern (**Matrizen**). Python-Bausteine wie **NumPy** rechnen mit solchen Gittern in einem Schritt statt Zahl für Zahl.
- **Ein großes Ökosystem.** Rund um Python sind über Jahre die Werkzeuge für Künstliche Intelligenz gewachsen: **PyTorch** und **TensorFlow** zum Bauen und Trainieren von Modellen, dazu tausende fertige Bausteine für Datenaufbereitung und Auswertung.

Dass Python selbst langsam rechnet, fällt dabei kaum ins Gewicht: Die eigentliche Rechenarbeit geben diese Bausteine an hardwarenahen Code weiter (siehe Schicht 2). Python gibt nur die Anweisungen und wartet auf das Ergebnis.

## Schicht 2: Laufzeit und Beschleunigung – C++, Rust, CUDA

Ist ein Modell fertig trainiert, soll es im Betrieb möglichst schnell und sparsam antworten. Jetzt zählen andere Dinge: hoher **Durchsatz** (viele Anfragen pro Sekunde), kurze **Latenz** (wenig Wartezeit bis zum ersten Wort) und ein geringer Speicherverbrauch. Dafür ist Python zu langsam.

Die eigentlichen Rechenkerne laufen darum in **hardwarenahem Maschinencode** – erzeugt aus Sprachen, die vollständig in Befehle für einen bestimmten Rechnertyp übersetzt werden und danach ohne Zwischenschicht laufen:

- **C++** ist die verbreitetste Sprache für diese Schicht. Der Kern von PyTorch ist in C++ geschrieben, ebenso das Programm **llama.cpp**, das Sprachmodelle auf dem eigenen Rechner ausführt (siehe [Sprachmodell selbst betreiben](../server-einrichten/sprachmodell-selbst-betreiben.md)).
- **CUDA** ist eine Erweiterung von C++ von der Firma NVIDIA. Damit schreibt man Code, der direkt auf der **Grafikkarte** (englisch „GPU") rechnet. Eine Grafikkarte kann tausende einfache Rechnungen gleichzeitig ausführen – genau die Art Arbeit, die ein Modell verlangt.
- **Rust** ist eine jüngere hardwarenahe Sprache. Sie ist ähnlich schnell wie C++, verhindert aber schon beim Übersetzen eine ganze Klasse von Speicherfehlern. Neuere Projekte für den Modellbetrieb setzen zunehmend auf Rust.

Von dieser Schicht bekommt man als Anwender wenig zu sehen. Sie steckt fertig in den Programmen und Bausteinen der anderen beiden Schichten. Man müsste sie nur anfassen, wenn man eine Rechenbibliothek selbst verbessern will.

## Schicht 3: Integration und Orchestrierung

Die dritte Schicht baut aus dem fertigen Modell eine nutzbare Anwendung. Sie **orchestriert** – das heißt, sie steuert das Zusammenspiel vieler Teile: Sie nimmt die Anfrage eines Nutzers entgegen, holt passende Texte aus einer Datenbank dazu, schickt alles an das Modell, prüft die Antwort und gibt sie an die Benutzeroberfläche zurück.

Hier gibt es keine einzelne vorherrschende Sprache. Gewählt wird meist die, in der die übrige Anwendung schon geschrieben ist:

- **TypeScript** (und **JavaScript**): die Sprache für Weboberflächen. Ein Chatfenster im Browser und der zugehörige kleine Server sind oft in TypeScript geschrieben. Viele KI-Werkzeugkästen gibt es zuerst für diese Sprache.
- **Go**: eine Sprache von Google für Server- und Kommandozeilenwerkzeuge. Sie übersetzt zu einer eigenständigen Datei, startet schnell und kommt mit vielen gleichzeitigen Anfragen gut zurecht. Ollama selbst ist in Go geschrieben.
- **C#** und **Java**: die üblichen Sprachen in Firmen mit größerer bestehender Software. Beide haben inzwischen offizielle Bausteine, um Modelle einzubinden.
- **Python**: taugt auch für diese Schicht und wird oft gewählt, wenn das Team ohnehin aus Schicht 1 kommt.

Für dieses Buch fällt der KI-Agent Claude Code in diese Schicht: Er ruft ein Modell auf, liest und schreibt dabei selbst Dateien und führt Befehle aus (siehe [Text erstellen](./text-erstellen.md)).

## Eine ältere Linie: wissensbasierte Systeme (Lisp und Prolog)

Die drei Schichten oben beschreiben den heute üblichen Weg: ein Modell, das aus riesigen Textmengen selbst gelernt hat. Davor gab es eine andere Vorstellung davon, wie ein Computer „intelligent" werden kann – und dazu gehörten zwei eigene Programmiersprachen.

Bei einem **wissensbasierten System** lernt der Computer nichts aus Beispielen. Stattdessen schreibt ein Mensch das Wissen von Hand auf: als **Fakten** (etwa „Ein Hund ist ein Säugetier") und als **Regeln** (etwa „Wenn ein Tier Fieber und Husten hat, dann prüfe auf Lungenentzündung"). Ein zweiter Programmteil, der **Schlussfolgerer** (auch „Inferenzmaschine"), kombiniert diese Regeln und beantwortet damit Fragen. Die bekannteste Bauform heißt **Expertensystem** – ein Programm, das das Wissen eines Fachmanns in Regeln nachbildet. Der Vorteil: Jede Antwort ist nachvollziehbar, weil man genau sehen kann, welche Regeln zu ihr geführt haben.

**Lisp** (von englisch „list processing", „Listen verarbeiten") ist von 1958 und damit eine der ältesten Programmiersprachen, die noch benutzt werden. Ihre Besonderheit: Programm und Daten haben dieselbe Form – beide sind Listen. Ein Lisp-Programm kann darum andere Programme wie Daten behandeln, sie also selbst zusammenbauen und verändern. Für die frühe KI-Forschung, vor allem in den USA, war Lisp jahrzehntelang die Hauptsprache; es gab sogar Rechner, die eigens dafür gebaut waren. Heute ist Lisp eine Nische, lebt aber in Abkömmlingen wie **Clojure** und **Scheme** weiter. Manches, was Lisp früh hatte, steckt inzwischen in fast jeder Sprache: die automatische Speicherverwaltung und das zeilenweise Ausprobieren im laufenden Programm.

**Prolog** (von französisch „programmation en logique", „Programmierung in Logik") ist von 1972 und geht einen ungewöhnlichen Weg: Man beschreibt keine Arbeitsschritte, sondern nur Fakten und Regeln. Dann stellt man eine Frage, und das Programm sucht selbst nach einer Antwort, die zu den Regeln passt.

```prolog
elternteil(anna, ben).
elternteil(ben, carla).
grosselternteil(X, Z) :- elternteil(X, Y), elternteil(Y, Z).
```

Auf die Frage `grosselternteil(anna, carla)` antwortet Prolog mit „wahr" – es hat die beiden Fakten über die Regel von selbst verknüpft. Prolog war in Europa und Japan verbreitet und wurde für Expertensysteme und für die Verarbeitung von Sprache eingesetzt.

Warum trat diese Linie in den Hintergrund? Alle Regeln von Hand aufzuschreiben ist mühsam, und an den Rändern wurden die Systeme schnell unzuverlässig. Als die großen Versprechen der 1980er-Jahre ausblieben, brach die Förderung ein – diese Zeit heißt der **„KI-Winter"**. Die heutigen Sprachmodelle gehen den umgekehrten Weg: Sie lernen Muster aus vielen Texten, statt Regeln vorgesetzt zu bekommen.

Ganz verschwunden ist der Ansatz aber nicht. **Regelwerke** in Firmensoftware, **Wissensgraphen** hinter Suchmaschinen und Prüfprogramme für technische Konfigurationen arbeiten bis heute so. Und es gibt neues Interesse daran, beides zu verbinden – ein gelerntes Modell zusammen mit von Hand geschriebener Logik –, was **neuro-symbolische KI** genannt wird. Für die Arbeit an diesem Buch ist das Hintergrundwissen: Die drei Schichten oben betreffen den Modell-Weg, und weder Lisp noch Prolog braucht man dafür.

## Warum nicht eine Sprache für alles?

Die Trennung hat einen einfachen Grund: Was eine Sprache leicht macht, macht eine andere schwer.

Python ist bequem zum Ausprobieren, weil es viele technische Details verbirgt – genau das kostet aber Tempo. C++ und Rust sind schnell, weil der Programmierer jedes Detail selbst bestimmt – das macht die Arbeit langsamer und fehleranfälliger. Eine Sprache, die beides zugleich könnte, gibt es nicht.

Die Lösung ist Arbeitsteilung: Jede Schicht nutzt die Sprache, die zu ihrer Aufgabe passt, und übergibt das Ergebnis an die nächste. Die Übergabestelle zwischen Schicht 2 und Schicht 3 ist fast immer eine Schnittstelle im sogenannten **OpenAI-Format**, das sich als gemeinsame Sprache zwischen den Programmen durchgesetzt hat (siehe [Sprachmodell selbst betreiben](../server-einrichten/sprachmodell-selbst-betreiben.md)).

## Für dieses Buch

- **Nichts davon ist Pflicht.** Die Kapitel sind einfache Markdown-Dateien; zum Schreiben genügt ein Editor und ein KI-Abo (siehe [KI-Abo auswählen](./ki-abo.md)).
- **Ein eigenes Hilfsskript** (etwa zum Stapel-Prüfen von Texten) schreibt man am einfachsten in **Python** – das ist die Sprache mit den meisten fertigen KI-Bausteinen.
- **Eine Weboberfläche** oder eine Erweiterung für das Redaktionssystem entsteht in der Sprache dieses Systems – bei Drupal ist das **PHP** (siehe [Content-Management-System](../web-stack/cms.md)).
- **Ein Modell selbst betreiben** verlangt keine dieser Sprachen: Fertige Programme wie Ollama bringen die Schichten 2 und 3 schon mit (siehe [Sprachmodell selbst betreiben](../server-einrichten/sprachmodell-selbst-betreiben.md)).

## Fazit

Eine KI-Anwendung besteht aus drei Schichten mit je eigener bevorzugter Sprache. Die **Entwicklungs- und Trainingsschicht** gehört **Python**, weil sich damit schnell experimentieren lässt und alle Modellwerkzeuge dort zu Hause sind. Die **Laufzeit- und Beschleunigungsschicht** läuft in hardwarenahem Maschinencode aus **C++**, **CUDA** und zunehmend **Rust**, weil dort Tempo und Speicher zählen. Die **Integrations- und Orchestrierungsschicht** verbindet das Modell mit Datenbank, Oberfläche und Suche – meist in **TypeScript**, **Go**, **C#** oder ebenfalls Python, je nachdem, worin die übrige Anwendung geschrieben ist. Neben diesem Modell-Weg steht eine ältere Linie der KI: **wissensbasierte Systeme** mit den Sprachen **Lisp** und **Prolog**, die Wissen als von Hand geschriebene Regeln ablegen, statt es aus Daten zu lernen – heute eine Nische, aber in Regelwerken und Wissensgraphen weiter vorhanden. Für die Arbeit an diesem Buch reicht ein Editor; eigene Skripte schreibt man am besten in Python.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
