und da heißtange# Benutzerhandbuch

Ein mdBook-Projekt. Artikel entstehen in zwei Schritten: erst eine Rohfassung, dann ein fertiger Artikel.

## Schritt 1: Rohfassung erstellen

Der Autor legt eine neue Datei in `RAW/` an, z. B. `RAW/erste_schritte.md`, mit dem ganzen, ausformulierten Text (nicht nur Stichpunkte) — siehe `RAW/README.md` für Details und ein Beispiel.

## Schritt 2: Fertigen Artikel erstellen lassen

Wenn die Rohfassung fertig gespeichert ist, Claude Code mit einem Prompt wie diesem beauftragen:

```
Übernimm RAW/erste_schritte.md nach src/.
```

Claude durchläuft dabei den Workflow **Ingest → Query → Lint** (siehe `CLAUDE.md`):

1. **Ingest** — den ganzen Text übernehmen, Lücken selbstständig ergänzen.
2. **Query** — auf Urheberrechtsverletzungen und Duplicate Content prüfen, bis nichts mehr auffällig ist.
3. **Lint** — Grammatik prüfen, im Praxisbuch-Stil schreiben (sachlich, keine Ich-Form, verständlich für Anfänger und 10-Jährige).

Danach liegt der fertige Artikel in `src/` und ist in `src/SUMMARY.md` eingetragen; die Rohfassung in `RAW/` wird gelöscht.
