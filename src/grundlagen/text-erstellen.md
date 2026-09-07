# Text erstellen

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Die Kapitel dieses Buchs entstehen zuerst als grobe Rohfassung und werden danach sprachlich geglättet (siehe [Docs-as-Code](../entwicklungs-rechner/docs-as-code.md)). Bei diesem letzten Schritt hilft ein **Sprachmodell**: Es prüft Rechtschreibung und Grammatik, kürzt zu lange Sätze, erklärt Fachbegriffe und sorgt dafür, dass dasselbe Ding im ganzen Buch gleich heißt.

Ein Sprachmodell ist ein Programm, das Texte fortsetzt und umformt. Man gibt ihm eine Anweisung – einen **Prompt** – und bekommt eine Antwort zurück. Ein **KI-Agent** ist ein solches Sprachmodell mit der Fähigkeit, dabei selbst Dateien zu lesen und zu ändern (siehe [Voraussetzungen](./voraussetzungen.md)).

Dieses Kapitel zeigt zwei Arbeitsweisen – Satz für Satz und ganzer Text auf einmal –, welches Modell sich eignet (lokal auf dem eigenen Rechner oder in der Cloud), wie man die Stilregeln in einer festen Anweisung (dem **Systemprompt**) sammelt und wie man sie über die Dateien `CLAUDE.md`, `AGENTS.md` und die Einstellungen von GitHub Copilot hinterlegt, damit jeder Agent sie von allein befolgt.

## Was geprüft wird

Beim Glätten geht es um die Sprache, nicht um den Inhalt. Ein Modell darf umformulieren, aber keine Tatsachen hinzufügen, weglassen oder verändern. Geprüft wird auf:

- **Rechtschreibung und Zeichensetzung.**
- **Grammatik:** richtige Fälle, richtige Zeiten, klarer Bezug der Wörter zueinander.
- **Verständlichkeit:** kurze Sätze, keine Fachbegriffe ohne Erklärung, ein sachlicher Ton, nicht in Ich-Form. So versteht auch ein Anfänger den Text.
- **Einheitliche Begriffe:** Dasselbe Ding immer gleich benennen. Nicht einmal „Verzeichnis" und einmal „Ordner", nicht einmal „löschen" und einmal „entfernen".
- **Unverändertes:** Fakten, Zahlen, Zitate, Codebeispiele und der Transparenzhinweis in Kopf- und Fußzeile bleiben, wie sie sind.

## Zwei Arbeitsweisen

### Satz für Satz

Man gibt dem Modell einen einzelnen Satz und lässt ihn prüfen. Danach den nächsten.

- **Vorteil:** Das Modell konzentriert sich auf wenig Text und findet auch kleine Fehler. Jede vorgeschlagene Änderung lässt sich einzeln ansehen und annehmen oder ablehnen. Der Sinn kann kaum verrutschen.
- **Nachteil:** Es dauert lange und macht viel Handarbeit. Der Zusammenhang zwischen den Sätzen fehlt dem Modell – Wiederholungen über mehrere Sätze hinweg und holprige Übergänge fallen so nicht auf.

Diese Arbeitsweise passt am besten für einen einzelnen schwierigen Absatz, an dem die Formulierung noch klemmt.

> Prüfe den folgenden Satz auf Rechtschreibung und Grammatik. Ändere nur, was falsch oder unklar ist. Gib den verbesserten Satz zurück und darunter in einem Satz, was du geändert hast.

### Ganzer Text

Man gibt dem Modell den kompletten Artikel.

- **Vorteil:** Das ist schnell. Das Modell sieht den Zusammenhang, erkennt Wiederholungen und kann prüfen, ob Begriffe und Anrede im ganzen Text gleich bleiben.
- **Nachteil:** Das Modell schreibt oft mehr um als nötig, kann dabei den Sinn verschieben und übersieht kleine Fehler leichter. Viele Änderungen auf einmal sind mühsam zu prüfen.

Gegen diese Nachteile helfen drei Regeln in der Anweisung:

1. Nur das Nötigste ändern, keine Umbauten aus Geschmack.
2. Zuerst eine Liste der Änderungen mit kurzer Begründung ausgeben, dann den Text.
3. Das Ergebnis mit dem Git-Vergleich (dem *Diff*, der Zeile für Zeile zeigt, was sich geändert hat) durchgehen und jede Änderung einzeln übernehmen oder verwerfen.

> Prüfe den folgenden Artikel auf Rechtschreibung, Grammatik und Verständlichkeit. Ändere nur, was falsch oder unklar ist, und behalte alle Fakten, Zahlen und Codebeispiele bei. Achte darauf, dass gleiche Dinge im ganzen Text gleich benannt werden. Gib zuerst eine Liste der Änderungen mit je einer kurzen Begründung aus, danach den vollständigen Text.

### Empfohlene Reihenfolge

Beide Wege ergänzen sich – ähnlich wie bei der Prüfung auf [Urheberrecht und Duplicate Content](./urheberrecht-duplicate-content.md):

1. **Ein Durchgang über den ganzen Text** mit Änderungsliste. Das räumt die groben Sachen weg und sorgt für einheitliche Begriffe.
2. **Die unklaren Stellen Satz für Satz.** Wo die Liste unsicher klingt oder eine Formulierung noch hakt, den einzelnen Satz noch einmal gezielt prüfen.
3. **Den Git-Vergleich durchsehen.** Am Ende steht jede Änderung einzeln zur Entscheidung an. Nichts wird ungeprüft übernommen.

## Welches Sprachmodell?

### In der Cloud

Die großen Modelle von Anthropic (Claude), OpenAI (GPT) und Google (Gemini) laufen auf den Servern der Anbieter. Sie sind derzeit am stärksten bei deutschem Text: Sie erkennen feine Grammatikfehler und halten einen gleichmäßigen Ton über den ganzen Artikel. Jede Anfrage kostet Geld oder ist Teil eines Abos. Für die Schlussredaktion sind sie die sichere Wahl. Wer ohnehin schon mit Claude Code arbeitet (siehe [Voraussetzungen](./voraussetzungen.md)), hat ein solches Modell bereits zur Hand.

### Lokal über Ollama

**Ollama** ist ein Programm, das ein Sprachmodell auf dem eigenen Rechner ausführt – ohne Internet und ohne laufende Kosten. Nach der Installation lädt und startet ein einziger Befehl ein Modell:

```bash
ollama run gemma3
```

Für eine deutsche Sprachprüfung braucht es ein mittelgroßes Modell. Sehr kleine Modelle machen selbst Grammatikfehler. Brauchbar sind Modelle wie **Gemma**, **Qwen** oder **Llama** in einer Größe um 7 bis 14 Milliarden Parameter. Die Zahl der Parameter ist das Maß für die „Größe" eines Modells: Mehr Parameter bedeuten mehr Sprachgefühl, aber auch mehr Arbeitsspeicher und langsamere Antworten. Als Faustregel sollte das Modell in den Speicher der Grafikkarte passen, sonst wird es sehr langsam.

Es gibt auch Modelle, die eigens auf Deutsch trainiert wurden, etwa **LeoLM** oder **Teuken**. Sie sind einen Versuch wert, reichen an die großen Cloud-Modelle aber nicht heran.

| | Cloud (Claude, GPT, Gemini) | Lokal über Ollama |
| --- | --- | --- |
| Kosten | pro Anfrage oder im Abo | einmalig die Hardware, sonst frei |
| Internet | nötig | nicht nötig |
| Qualität bei Deutsch | sehr hoch | ausreichend bis gut, je nach Modellgröße |
| Datenschutz | der Text geht an den Anbieter | der Text bleibt auf dem Rechner |
| Gut für | die Schlussredaktion | viele schnelle Zwischendurchgänge |

## Die Regeln sammeln: der Systemprompt

Der **Systemprompt** ist der feste Teil der Anweisung. Er legt die Rolle des Modells und die Regeln fest und gilt bei jeder Anfrage, ohne dass man ihn wiederholt. Für dieses Buch sieht er etwa so aus:

> Du bist Lektor für ein Praxisbuch. Prüfe den Text auf Rechtschreibung, Grammatik und Verständlichkeit. Schreibe sachlich, nicht in Ich-Form, in kurzen Sätzen. Erkläre jeden Fachbegriff bei der ersten Nennung. Benenne gleiche Dinge im ganzen Text gleich. Ändere keine Fakten, Zahlen, Zitate oder Codebeispiele. Lass den Hinweis in Kopf- und Fußzeile unverändert. Gib zuerst eine Liste deiner Änderungen mit kurzer Begründung aus, danach den Text.

Denselben Absatz bei jeder Sitzung neu einzutippen ist mühsam. Besser ist es, die Regeln in eine Datei zu legen, die der Agent von allein liest.

## Die Regeln in Projektdateien hinterlegen

KI-Agenten lesen beim Start bestimmte Dateien im Projektordner und behandeln deren Inhalt wie einen Systemprompt. So gelten die Regeln für jede Sitzung und für jeden im Team – ohne Copy-and-paste.

### CLAUDE.md

Claude Code liest die Datei `CLAUDE.md` im Projektordner automatisch. Dieses Buch hat bereits eine: Darin stehen unter anderem der Praxisbuch-Stil und der Transparenzhinweis. Eine zweite `CLAUDE.md` in einem Unterordner gilt zusätzlich für die Dateien in diesem Ordner.

### AGENTS.md

`AGENTS.md` ist ein offener Standard, der nicht an einen Anbieter gebunden ist. Mehrere Agenten lesen diese Datei, darunter Codex CLI; weitere unterstützen sie zunehmend. Der Zweck ist derselbe wie bei `CLAUDE.md`. Wer mehrere Agenten nutzt, schreibt die Regeln einmal in `AGENTS.md` und verweist aus `CLAUDE.md` mit einer Zeile darauf.

### GitHub Copilot

**GitHub Copilot** vervollständigt Code und beantwortet Fragen direkt im Editor (siehe [IDE](../entwicklungs-rechner/ide.md)). Es liest die Datei `.github/copilot-instructions.md` im Projekt. Der Inhalt ist derselbe, nur die Datei heißt anders. Zusätzlich lassen sich in den Einstellungen von Visual Studio Code unter „Copilot" eigene Anweisungen für einzelne Aufgaben hinterlegen, zum Beispiel für die Texte der Git-Commits.

### Was in diese Dateien gehört

- **Der Stil:** sachlich, nicht in Ich-Form, kurze Sätze, Fachbegriffe erklären.
- **Feste Begriffe:** eine kurze Liste der Wörter, die immer gleich lauten sollen.
- **Das Unantastbare:** Fakten, Zahlen, Zitate, Codebeispiele und der Transparenzhinweis.
- **Das Ausgabeformat:** erst die Änderungsliste, dann der Text.

Die Dateien sollten kurz bleiben. Zu lange Regelwerke überliest ein Modell eher, als dass es sie befolgt.

## Für dieses Buch

- Zuerst die Rohfassung nach den Regeln im Ordner `RAW/` übernehmen, dann sprachlich glätten.
- Modell: Claude für die Schlussredaktion, da über Claude Code ohnehin vorhanden; ein lokales Modell über Ollama für schnelle Zwischendurchgänge.
- Reihenfolge: ganzer Text mit Änderungsliste, dann die unklaren Sätze einzeln, dann den Git-Vergleich prüfen.
- Die Stilregeln stehen in `CLAUDE.md`. Wer Codex oder Copilot einsetzt, legt `AGENTS.md` beziehungsweise `.github/copilot-instructions.md` mit demselben Inhalt an.
- Der Transparenzhinweis als Kopf- und Fußzeile bleibt in jedem Artikel stehen.

## Fazit

Beim Erstellen der Kapitel hilft ein Sprachmodell, die Sprache zu glätten – nicht, den Inhalt zu erfinden. Die Prüfung Satz für Satz ist gründlich, aber langsam und blind für den Zusammenhang; die Prüfung des ganzen Textes ist schnell und sieht den Zusammenhang, ändert aber oft zu viel. Bewährt hat sich beides nacheinander: erst ein Gesamtdurchgang mit Änderungsliste, dann einzelne Sätze, dann der geprüfte Git-Vergleich. Als Modell eignet sich ein großes Cloud-Modell wie Claude für die Schlussredaktion und ein lokales Modell über **Ollama** für schnelle Zwischenschritte. Die Stilregeln schreibt man einmal als Systemprompt und hinterlegt sie in `CLAUDE.md`, `AGENTS.md` oder `.github/copilot-instructions.md`, damit jeder Agent sie von allein befolgt.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
