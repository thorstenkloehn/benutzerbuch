# RAW – Rohfassungen

Dieser Ordner enthält **Rohfassungen** von Buchartikeln für das „Benutzerhandbuch“.

## Was ist das?

- Hier legt der **Autor** Rohfassungen zu einem Kapitel ab — noch ungeschliffen, ohne Anspruch auf Grammatik oder Stil.
- Wichtig: Eine Rohfassung sollte der **ganze, ausformulierte Text** sein, nicht nur Stichpunkte. Aus einem vollständigen Text kann Claude einen besseren Artikel machen als aus einer Stichpunktliste.
- Diese Dateien sind **kein** Teil des fertigen Buchs; `mdbook build` liest nur aus `src/`.

## Ordnerstruktur und Sortierung

- `src/` gliedert Kapitel in Themen-Unterordnern (z. B. `src/grundlagen/`). Eine Rohfassung darf denselben Unterordner unter `RAW/` spiegeln (z. B. `RAW/grundlagen/erste_schritte.md`), muss es aber nicht — Claude legt die passende Struktur in `src/` beim Übertragen selbst an.
- Die tatsächliche Reihenfolge und Verschachtelung der Kapitel im fertigen Buch bestimmt allein `src/SUMMARY.md`, nicht die Ordnerstruktur.
- Optional lässt sich am Anfang einer Rohfassung mit einer Zeile `Nav: <Bereich> - <Elternkapitel>` angeben, wo das Kapitel in der Navigation einsortiert werden soll, z. B. `Nav: Grundlagen - Installieren` für ein Kapitel, das als Unterpunkt von „Installieren“ im Bereich „Grundlagen“ erscheinen soll. Fehlt die Zeile, entscheidet Claude selbst über eine sinnvolle Einordnung.


## Workflow: Ingest → Query → Lint

So wird aus einer Rohfassung ein fertiger Artikel in `src/`:

**1. Ingest** (Rohfassung übernehmen)
- Claude (der Agent) schreibt hier nichts hinein und ändert nichts. Nur der Autor legt Dateien an oder bearbeitet sie.
- Den ganzen Text der Rohfassung übernehmen (nicht nur Stichpunkte).
- Fehlt etwas (z. B. ein Schritt, eine Erklärung, ein Übergang), selbstständig ergänzen, sodass ein vollständiger Artikel entsteht.

**2. Query** (gegen fremde Quellen prüfen)
- Auf Urheberrechtsverletzungen prüfen: wörtliche Übernahmen, eng angelehnte Paraphrasen, übersetzte Passagen aus fremden Quellen wie Wikis.
- Auf Duplicate Content im Sinne von Google prüfen (z. B. per Websuche nach auffälligen Formulierungen), damit der Text einzigartig ist.
- Bei Auffälligkeiten überarbeiten. Erst wenn nichts mehr auffällig ist, gilt der Artikel als fertig.

**3. Lint** (Sprache und Stil)
- Grammatik prüfen und im Stil eines Praxisbuchs schreiben (klar, anwendungsorientiert): sachlich, nicht in Ich-Form, verständlich für Anfänger und 10-Jährige (einfache Sätze, keine Fachbegriffe ohne Erklärung).
- Danach darf Claude die Rohfassung hier löschen.

## Beispiel

Eine Rohfassung, z. B. `erste_schritte.md`, ist ein ganzer, aber noch unpolierter Text — optional mit einer `Nav:`-Zeile, die die Einordnung in der Navigation vorgibt:

```markdown
Kapitel: Erste Schritte
Nav: Grundlagen - Installieren

Die Installation ist eigentlich einfach, man lädt sich das Paket runter und
entpackt es irgendwo. Dann muss man noch die config datei anpassen, da steht
zum beispiel drin welcher pfad genommen wird. Der häufigste Fehler ist das
der pfad falsch eingetragen ist, das passiert echt oft und sollte man nochmal
extra erwähnen im Text. Am besten noch ein Screenshot von der config datei
dazu.
```

Daraus macht Claude beim Übertragen ein fertiges Kapitel in `src/`, z. B. `src/grundlagen/erste_schritte.md`, mit vollständigen Sätzen, korrekter Grammatik und im Praxisbuch-Stil — und trägt es passend zur `Nav:`-Angabe als Unterpunkt von „Installieren“ in `src/SUMMARY.md` ein:

```markdown
- [Installieren](./grundlagen/installieren.md)
  - [Erste Schritte](./grundlagen/erste_schritte.md)
```
