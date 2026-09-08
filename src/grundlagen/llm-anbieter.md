# LLM-Anbieter

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Ein **LLM** (englisch „large language model", also „großes Sprachmodell") ist ein Programm, das Text fortsetzt und umformt (siehe [Text erstellen](./text-erstellen.md)). Für die Arbeit an diesem Buch braucht es Zugang zu einem solchen Modell. Diesen Zugang verkaufen viele verschiedene Firmen, und sie tun das auf sehr unterschiedliche Weise – vom fertigen Chatprogramm bis zum leeren Rechner mit Grafikkarte, auf dem man das Modell selbst startet.

Dieses Kapitel ordnet die Anbieter in fünf Gruppen und nennt je Gruppe einige Beispiele, sortiert von günstig nach teuer. Zwei verwandte Kapitel gehen tiefer: [KI-Abo auswählen](./ki-abo.md) vergleicht die festen Monats-Abos, [Sprachmodell selbst betreiben](../server-einrichten/sprachmodell-selbst-betreiben.md) beschreibt den Betrieb auf eigener oder gemieteter Hardware.

## Zwei Abrechnungsarten

Bevor es um die Gruppen geht, hilft ein Blick auf die Abrechnung. Es gibt im Wesentlichen zwei Wege:

- **Abo:** ein fester Betrag pro Monat, dafür eine bestimmte Nutzungsmenge. Gut planbar. Beschrieben in [KI-Abo auswählen](./ki-abo.md).
- **Pro Nutzung** (englisch „pay as you go"): Abgerechnet wird nach der Menge des verarbeiteten Textes. Die Einheit heißt **Token** – ein Token ist ein Wortstück, grob vier Buchstaben. Preise stehen meist als Betrag „pro eine Million Token", getrennt für den eingegebenen Text (die Eingabe) und den erzeugten Text (die Ausgabe). Die Ausgabe ist fast immer teurer.

Für den Zugang über ein Programm oder einen Agenten braucht es einen **API-Schlüssel** – eine lange Zeichenkette, die das Konto ausweist und die Kosten zuordnet. „API" (englisch „application programming interface") ist die Schnittstelle, über die zwei Programme miteinander reden.

Preise ändern sich häufig. Die hier genannten Größenordnungen dienen nur der groben Einordnung; maßgeblich ist immer die Seite des jeweiligen Anbieters.

## Gruppe 1: GPU-Server und GPU-vServer

Hier mietet man keinen Modellzugang, sondern einen **Rechner mit Grafikkarte** (englisch „GPU") und startet das Sprachmodell selbst darauf (siehe [Sprachmodell selbst betreiben](../server-einrichten/sprachmodell-selbst-betreiben.md)). Ein **vServer** ist ein virtueller Server – ein abgeteiltes Stück eines größeren Rechners; ein dedizierter Server ist eine ganze Maschine allein für einen selbst.

Das lohnt sich, wenn viel Text anfällt, die Inhalte das Haus nicht verlassen sollen oder ein bestimmtes Modell gebraucht wird. Abgerechnet wird nach Laufzeit, meist pro Stunde. Wichtig: Eine ungenutzt laufende Grafikkarte kostet trotzdem Geld – man schaltet den Server ab, wenn er nicht gebraucht wird.

Von günstig nach teuer:

- **Marktplätze für freie Rechenzeit** wie Vast.ai oder RunPod (im „Community"-Angebot): Privatleute und kleine Rechenzentren vermieten hier freie Grafikkarten. Am billigsten, dafür ohne Garantie auf Verfügbarkeit und mit schwankender Qualität.
- **Spezialisierte GPU-Vermieter** wie RunPod (reguläres Angebot), Lambda, Paperspace oder DataCrunch: fester Preis, gute Auswahl an Karten, auf KI zugeschnitten.
- **Mittelgroße Cloud-Anbieter** wie Hetzner, OVHcloud oder Scaleway: solide Technik zu Preisen unter den ganz Großen, aber weniger und oft ältere Kartenmodelle.
- **Große Cloud-Plattformen** (englisch „Hyperscaler") wie Amazon Web Services, Google Cloud und Microsoft Azure: die höchsten Stundenpreise, dafür jederzeit verfügbar, weltweit und mit vielen Zusatzdiensten.

## Gruppe 2: Cloud-KI-Anbieter

Die großen Cloud-Plattformen verkaufen nicht nur leere Rechner, sondern auch **fertigen Modellzugang als Dienst**. Man muss dann nichts selbst betreiben: Der Anbieter hält die Modelle bereit, man schickt Anfragen und zahlt pro Token.

Der Vorteil ist die Einbindung in eine schon genutzte Cloud – gleiche Anmeldung, gleiche Rechnung, gleicher Datenschutzvertrag. Oft liegen mehrere Modelle verschiedener Hersteller unter einer Schnittstelle.

- **Microsoft Azure** („Azure AI Foundry", früher „Azure OpenAI"): vor allem die Modelle von OpenAI, dazu weitere.
- **Google Cloud** („Vertex AI"): die Gemini-Modelle von Google und eine Auswahl fremder Modelle.
- **Amazon Web Services** („Bedrock"): Modelle von Anthropic, Meta, Mistral und anderen sowie Amazons eigene.
- **IBM** („watsonx") und **Oracle**: eher für Firmenkunden mit bestehenden Verträgen.

Die Token-Preise entsprechen meist ungefähr denen der Modellhersteller; teurer wird es durch Mindestumsätze, Zusatzdienste und den Aufwand der Einrichtung. Deshalb steht diese Gruppe hier hinter den direkten Wegen.

## Gruppe 3: Multi-LLM-Provider (Aggregatoren)

Ein **Aggregator** (von lateinisch „aggregare", „ansammeln") bündelt viele Modelle vieler Hersteller hinter **einer einzigen Schnittstelle und einem Schlüssel**. Man meldet sich einmal an, lädt Guthaben auf und kann dann zwischen Hunderten Modellen wechseln, ohne bei jedem Hersteller ein eigenes Konto zu haben.

Das ist praktisch zum Vergleichen und um nicht von einem einzelnen Anbieter abhängig zu sein. Manche Aggregatoren betreiben die offenen Modelle auf eigener Hardware besonders günstig; andere reichen die Anfrage nur weiter und nehmen eine kleine Gebühr.

Von günstig nach teuer:

- **Günstig-Betreiber** wie DeepInfra, Novita oder Hyperbolic: fahren offene Modelle (etwa Llama, Qwen, DeepSeek) zu sehr niedrigen Token-Preisen.
- **Leistungs-Betreiber** wie Together AI, Fireworks AI oder Groq: ähnliche Modellauswahl, Schwerpunkt auf hohem Tempo, etwas höhere Preise.
- **Weitervermittler** wie OpenRouter: leiten die Anfrage an den jeweiligen Hersteller oder Betreiber weiter, meist zum dortigen Preis plus einem kleinen Aufschlag beim Aufladen. Eine Schnittstelle für praktisch alle Modelle, auch die geschlossenen.
- **Baukasten-Dienste** wie Replicate oder Hugging Face: neben Sprachmodellen auch Bild- und Tonmodelle, Abrechnung teils pro Sekunde Rechenzeit. Bequem, aber selten die günstigste Wahl.

## Gruppe 4: Native KI-Anbieter

„Nativ" heißt hier: direkt beim **Modellhersteller** selbst, über dessen eigenes Chatprogramm und dessen Abo – ohne Zwischenhändler. Das sind die Firmen, die die Modelle bauen und trainieren.

Für die reine Schreib- und Recherchearbeit an diesem Buch ist das der übliche Weg; die Abos sind ausführlich in [KI-Abo auswählen](./ki-abo.md) beschrieben. Hier nur die Einordnung nach dem Preis der Standard-Abos (meist rund 20 Euro im Monat, mit Ausnahmen nach unten und oben):

- **DeepSeek** (China): eigene App, sehr niedriger Preis, in Grundzügen kostenlos nutzbar.
- **Google** („Gemini", im Abo „Google AI Pro"): breite Einbindung in Gmail, Docs und Drive.
- **Mistral** (Frankreich, „Le Chat"): europäischer Anbieter, etwas günstigeres Abo, Server in der EU.
- **OpenAI** („ChatGPT Plus"): Standard-Abo im mittleren Bereich.
- **xAI** („Grok"): Standard-Abo etwas oberhalb der anderen.
- **Anthropic** („Claude Pro"): gleiches Preisniveau wie OpenAI, bei langem deutschem Text derzeit besonders zuverlässig; ein größeres Abo („Max") kostet ein Mehrfaches.

## Gruppe 5: APIs der KI-Anbieter

Dieselben Modellhersteller bieten neben App und Abo auch eine **API** an – den Zugang pro Token für eigene Programme und Agenten. Das ist der Weg, wenn ein Skript, ein KI-Agent (siehe [KI-Agenten auf dem Server](../server-einrichten/ki-agent.md)) oder ein Redaktionssystem wie Drupal (siehe [Content-Management-System](../web-stack/cms.md)) das Modell aufrufen soll.

Jeder Hersteller hat meist mehrere Modellstufen: ein kleines, schnelles und billiges Modell und ein großes, langsames und teures. Die Reihenfolge unten meint grob die mittlere Stufe.

Von günstig nach teuer:

- **DeepSeek:** mit Abstand am billigsten, oft ein Bruchteil der anderen.
- **Google (Gemini API):** die „Flash"-Stufen sind sehr günstig, „Pro" liegt im Mittelfeld. Ein kostenloses Kontingent zum Ausprobieren ist vorhanden.
- **Mistral:** günstige offene Modelle, Server in Europa.
- **OpenAI (GPT):** die „mini"-Modelle sind preiswert, die großen Modelle im oberen Bereich.
- **xAI (Grok):** mittleres bis oberes Preisniveau.
- **Anthropic (Claude):** die kleinen Modelle („Haiku") sind günstig, die mittleren und großen („Sonnet", „Opus") gehören zu den teureren am Markt.

Ein Kostenvorteil der API gegenüber dem Abo besteht nur bei geringer Nutzung. Wer täglich viel arbeitet, fährt mit einem Abo oft günstiger und vor allem planbarer.

## Für dieses Buch

- **Schreiben und Recherche:** ein natives Abo, in der Regel **Claude Pro** (siehe [KI-Abo auswählen](./ki-abo.md)).
- **Agenten, Skripte, Drupal:** ein **API-Schlüssel** beim selben Hersteller oder ein **Aggregator** wie OpenRouter, um Modelle zu vergleichen.
- **Günstige Zwischenschritte und Datenschutz:** ein lokales Modell über Ollama; bei größerem Bedarf ein **GPU-vServer** (siehe [Sprachmodell selbst betreiben](../server-einrichten/sprachmodell-selbst-betreiben.md)).
- **Ausprobieren ohne Kosten:** das kostenlose Kontingent der Gemini-API oder ein kleines Modell bei einem Günstig-Betreiber.

## Fazit

Zugang zu einem großen Sprachmodell gibt es auf fünf Wegen. Ein **GPU-Server** ist der leere Rechner zum Selbstbetrieb – am günstigsten über Marktplätze, am teuersten bei den großen Cloud-Plattformen. **Cloud-KI-Anbieter** liefern fertigen Modellzugang innerhalb einer Cloud, bequem, aber selten am billigsten. **Aggregatoren** bündeln viele Modelle hinter einem Schlüssel und sind gut zum Vergleichen. **Native Anbieter** sind die Modellhersteller selbst mit App und Abo – der richtige Weg fürs Schreiben. Deren **API** rechnet pro Token ab und ist die Wahl für Agenten und eigene Programme. Quer über alle Gruppen gilt: DeepSeek und die kleinen Google-Modelle sind am günstigsten, Anthropic und die großen Modelle am oberen Ende – und jeder Preis kann sich morgen ändern.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
