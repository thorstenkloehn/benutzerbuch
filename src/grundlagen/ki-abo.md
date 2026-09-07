# KI-Abo auswählen

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Für die Arbeit an diesem Buch braucht es einen bezahlten Zugang zu einem Sprachmodell (siehe [Voraussetzungen](./voraussetzungen.md)). Ein **Abo** ist dabei ein fester Monatsbetrag, für den man den Dienst nutzen darf – bis zu einer bestimmten Menge pro Zeitraum. Das ist der Gegenentwurf zur Abrechnung pro Anfrage über einen Programmierschnittstellen-Schlüssel (einen **API-Schlüssel**), bei der jede einzelne Nutzung Geld kostet.

Vier Abos kommen für die hier beschriebene Arbeit infrage:

- **ChatGPT Plus** von OpenAI
- **Claude Pro** von Anthropic
- **Google AI Pro** von Google
- **GitHub Copilot** von GitHub (einer Tochterfirma von Microsoft)

Dieses Kapitel beschreibt, was die vier Abos können, wie sie zu den typischen Aufgaben passen – Texte und Quellen für Wiki- und Blog-Artikel schreiben, Inhalte in einem Redaktionssystem wie Drupal pflegen, Material sammeln und auswerten, am Server und am Code arbeiten – und welche Wahl sich für dieses Buch bewährt hat.

## Die Aufgaben

Bevor man ein Abo wählt, hilft es, die eigene Arbeit in Gruppen zu ordnen. Für dieses Buch fallen vier Arten von Aufgaben an:

- **Texte schreiben und glätten:** Rohfassungen zu fertigen Artikeln ausformulieren, Grammatik und Stil prüfen (siehe [Text erstellen](./text-erstellen.md)). Hier zählt vor allem gutes Deutsch über längere Abschnitte hinweg.
- **Quellen sammeln und auswerten:** Mehrere Dokumente, Webseiten oder Notizen zusammentragen und daraus eine Übersicht gewinnen.
- **Inhalte pflegen:** Artikel in ein Redaktionssystem wie **Drupal** einpflegen, ordnen und aktuell halten (siehe [Drupal einrichten](../web-stack/drupal.md)).
- **Am Server und am Code arbeiten:** Dateien auf dem eigenen Rechner und auf dem Server anlegen und ändern, Befehle ausführen, Fehler suchen. Dafür braucht es einen **KI-Agenten** – ein Sprachmodell, das selbst Dateien lesen und schreiben darf.

## Die vier Abos im Überblick

### ChatGPT Plus (OpenAI)

ChatGPT Plus ist das Standard-Abo von OpenAI. Es öffnet den Zugang zu den stärkeren Modellen, zur Bilderzeugung, zur Sprachein- und -ausgabe und zur Auswertung hochgeladener Dateien. Zum Abo gehört außerdem **Codex CLI**, der KI-Agent von OpenAI fürs Programmieren im Terminal (siehe [Voraussetzungen](./voraussetzungen.md)). ChatGPT Plus ist ein Alleskönner für den Alltag und deckt alle vier Aufgabengruppen ab.

### Claude Pro (Anthropic)

Claude Pro ist das Standard-Abo von Anthropic. Mit einem einzigen Preis deckt es drei Dinge ab: das Chatprogramm **Claude**, den KI-Agenten **Claude Code** fürs Arbeiten an Dateien und am Server und den Desktop-Modus **Cowork** (siehe [Desktop-Agenten](./desktop-agenten.md)). Die enthaltene Nutzungsmenge teilen sich diese drei – wer viel über Claude Code arbeitet, hat entsprechend weniger für das Chatprogramm übrig. Claude gilt bei längeren deutschen Texten derzeit als besonders sicher: Es hält einen gleichmäßigen Ton und befolgt feste Stilregeln zuverlässig.

### Google AI Pro (Google)

Google AI Pro schaltet das stärkere **Gemini**-Modell frei und bindet es in die Google-Programme ein: in **Gmail**, in **Google Docs**, in **Tabellen** und in **Drive**. Dazu gehören **NotebookLM** – ein Werkzeug, das aus einer Sammlung eigener Quellen Antworten und Zusammenfassungen erzeugt – und ein größerer Online-Speicher. Fürs Programmieren gibt es **Antigravity CLI** beziehungsweise die **Gemini CLI**. Dieses Abo lohnt sich vor allem, wenn die Inhalte ohnehin in Google Docs und Drive entstehen.

### GitHub Copilot (GitHub)

GitHub Copilot ist kein allgemeines Chatprogramm, sondern eine Hilfe direkt im Editor (siehe [IDE](../entwicklungs-rechner/ide.md)). Es vervollständigt Code beim Tippen, beantwortet Fragen zum geöffneten Projekt und kann als Agent kleine Aufgaben über mehrere Dateien hinweg erledigen. Es ist eng mit **Visual Studio Code**, den **JetBrains**-Programmen und der Plattform GitHub verbunden. Für reines Schreiben von Fließtext ist es nicht gedacht. Es gibt eine kostenlose Stufe mit begrenztem Umfang und darüber ein günstiges Bezahl-Abo.

## Welches Abo für welche Aufgabe

| Aufgabe | ChatGPT Plus | Claude Pro | Google AI Pro | GitHub Copilot |
| --- | --- | --- | --- | --- |
| Texte schreiben und glätten | gut | sehr gut bei langem Deutsch | gut, stark in Google Docs | nicht dafür gedacht |
| Quellen sammeln und auswerten | gut, Datei-Upload | gut, große Textmengen | sehr gut mit NotebookLM | nein |
| Inhalte in Drupal pflegen | mit Codex CLI | mit Claude Code | mit Antigravity CLI | im Editor, begrenzt |
| Am Server und am Code arbeiten | mit Codex CLI | mit Claude Code | mit Antigravity CLI | im Editor, begrenzt |

Zum Preis: Die drei großen Chat-Abos – ChatGPT Plus, Claude Pro und Google AI Pro – liegen aktuell bei rund 20 Euro im Monat. GitHub Copilot ist deutlich günstiger und in der kleinsten Stufe kostenlos. Bei jährlicher Zahlung sind die großen Abos meist etwas billiger. Preise und enthaltene Mengen ändern sich häufig; maßgeblich ist immer die Seite des jeweiligen Anbieters.

## Ein Abo genügt selten allein

Die vier Abos schließen sich nicht aus. Üblich ist eine Kombination aus einem großen Chat-Abo für Text und Auswertung und einem günstigen Copilot-Abo für die Arbeit im Editor. Wer knapp rechnet, beginnt mit den kostenlosen Stufen und den enthaltenen Agenten und bucht erst dazu, was wirklich fehlt.

Ein Hinweis zum Datenschutz: Bei den Bezahl-Abos nutzen die Anbieter die eingegebenen Texte in der Regel nicht zum Training ihrer Modelle. Sicherheitshalber sollte man diese Einstellung im eigenen Konto einmal prüfen (siehe auch [Datenschutz](../Datenschutz.md)).

## Für dieses Buch

Die Arbeit an diesem Handbuch besteht zu großen Teilen aus deutschem Fließtext und zu einem kleineren Teil aus Arbeit am Server und am Code. Für diese Mischung fällt die Wahl auf **Claude Pro**:

- Es glättet lange deutsche Texte zuverlässig und hält die Stilregeln aus `CLAUDE.md` ein (siehe [Text erstellen](./text-erstellen.md)).
- Mit **Claude Code** deckt dasselbe Abo die Arbeit an Dateien, an Drupal und am Server ab, ohne dass ein zweites großes Abo nötig wird.
- Der Rest des Buchs geht ohnehin von Claude Code aus (siehe [Voraussetzungen](./voraussetzungen.md)).

Ergänzend lohnt sich **GitHub Copilot** in der kostenlosen oder der günstigen Stufe für die Code-Vervollständigung direkt im Editor. **Google AI Pro** ist eine sinnvolle Zusatzwahl, wenn viele Inhalte in Google Docs entstehen – vor allem wegen NotebookLM für die Quellensammlung. **ChatGPT Plus** ist die gleichwertige Alternative zu Claude Pro für alle, die lieber mit den Modellen von OpenAI arbeiten; der Leistungsumfang ist ähnlich, bei langem deutschem Text gilt Claude derzeit als etwas stärker, bei der Bilderzeugung ChatGPT.

## Fazit

Es gibt keine allgemein „beste" Wahl – sie hängt davon ab, wofür man das Werkzeug hauptsächlich braucht. **ChatGPT Plus** und **Claude Pro** sind breite Alleskönner mit je einem eingebauten Coding-Agenten; Claude liegt bei langem deutschem Text vorn. **Google AI Pro** spielt seine Stärke aus, wenn die Inhalte in den Google-Programmen entstehen. **GitHub Copilot** ist keine Schreibhilfe, sondern eine günstige Ergänzung für die Arbeit im Editor. Für die Mischung aus viel Text und etwas Server- und Code-Arbeit, wie sie dieses Buch verlangt, deckt **Claude Pro** mit dem enthaltenen **Claude Code** beide Seiten unter einem Preis ab; ein kostenloses oder günstiges **GitHub Copilot** kommt für den Editor dazu.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
