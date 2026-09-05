# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Was das ist

Ein mdBook-Projekt ("Benutzerhandbuch"). Inhalte werden als Markdown unter `src/` geschrieben und von mdBook zu einem statischen HTML-Buch gebaut.

## Befehle

- `mdbook build` — baut das Buch nach `book/` (im Git ignoriert, generierte Ausgabe).
- `mdbook serve` — dient das Buch lokal mit Live-Reload beim Bearbeiten.
- `mdbook test` — führt eingebettete Rust-Codeblöcke aus dem Markdown als Doctests aus.

## Struktur

- `src/SUMMARY.md` — das Inhaltsverzeichnis; jede Kapiteldatei muss hier verlinkt sein, sonst nimmt mdBook sie nicht in den Build auf.
- `src/*.md` — Kapitelinhalte, in der von `SUMMARY.md` festgelegten Reihenfolge.
- `book.toml` — Buch-Metadaten (Titel, Autoren, Sprache, Quellverzeichnis).
- `RAW/` — Rohfassungen von Buchartikeln, die noch nicht als Kapitel nach `src/` übernommen wurden; nicht Teil des gebauten Buchs. Details und Regeln dazu stehen in `RAW/README.md`.

## Workflow: Ingest → Query → Lint

Eine Rohfassung wird in drei Schritten zu einem fertigen Artikel in `src/`:

**1. Ingest** (Rohfassung übernehmen)
- Dateien in `RAW/` schreibt ausschließlich der Autor. Claude darf dort nichts anlegen oder ändern (siehe `RAW/README.md`).
- Den ganzen Text der Rohfassung übernehmen (nicht nur Stichpunkte).
- Fehlt etwas (z. B. ein Schritt, eine Erklärung, ein Übergang), selbstständig ergänzen, sodass ein vollständiger Artikel entsteht.

**2. Query** (gegen fremde Quellen prüfen)
- Auf Urheberrechtsverletzungen prüfen: wörtliche Übernahmen, eng angelehnte Paraphrasen, übersetzte Passagen aus fremden Quellen (z. B. Wikis).
- Auf Duplicate Content im Sinne von Google prüfen (z. B. per Websuche nach auffälligen Formulierungen), damit der Text einzigartig ist.
- Bei Auffälligkeiten eine Liste mit Fundstelle, vermuteter Quelle und Schweregrad erstellen und den Artikel überarbeiten. Ein Artikel gilt erst als fertig, wenn Query keine Verletzungen und keinen Duplicate Content mehr findet.

**3. Lint** (Sprache und Stil)
- Grammatik prüfen und im Stil eines Praxisbuchs schreiben (klar, anwendungsorientiert, keine reine Theorie): sachlich, nicht in Ich-Form, verständlich für Anfänger und 10-Jährige (einfache Sätze, keine Fachbegriffe ohne Erklärung).
- Jedem Artikel folgenden Hinweis als Kopf- und Fußzeile hinzufügen (identischer Text an beiden Stellen):
  > Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
- Danach darf Claude die Rohfassung in `RAW/` löschen.
