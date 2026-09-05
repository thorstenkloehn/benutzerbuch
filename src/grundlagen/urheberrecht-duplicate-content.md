# Urheberrecht und Duplicate Content

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Wer eine Wissenssammlung oder eine Webseite veröffentlicht, muss zwei Dinge im Blick behalten. Das erste ist das Urheberrecht: Fremde Texte, Bilder oder Übersetzungen darf man nicht einfach übernehmen, auch nicht in leicht umgeschriebener Form. Das zweite ist der sogenannte Duplicate Content: Wenn derselbe Text fast wortgleich an mehreren Stellen im Internet steht, hält eine Suchmaschine wie Google ihn für eine Dublette und zeigt die eigene Seite schlechter an.

Beide Probleme lassen sich vor der Veröffentlichung prüfen. Dieses Kapitel zeigt, welche freien (quelloffenen) Programme es dafür gibt, wo ihre Grenzen liegen, wie man sich eine eigene Prüfung baut und wie ein KI-Agent dabei hilft. „KI-Agent" meint hier ein Programm mit einem Sprachmodell, das selbstständig Texte liest, im Internet sucht und die Ergebnisse zusammenfasst.

## Zwei verschiedene Probleme

Auch wenn beide Prüfungen ähnlich ablaufen, geht es um zwei getrennte Fragen.

- **Urheberrecht.** Hier lautet die Frage: Stammt eine Passage in Wahrheit aus einer fremden Quelle? Das kann eine wörtliche Übernahme sein, eine eng am Original entlanggeschriebene Umformulierung (eine „Paraphrase") oder eine Übersetzung aus einer anderen Sprache, etwa aus einem fremdsprachigen Wiki. Das deutsche Urheberrecht gilt als streng; ein pauschales „fair use" wie im US-Recht gibt es nicht. Ein Verstoß kann teuer werden.
- **Duplicate Content.** Hier lautet die Frage: Gibt es den eigenen Text schon fast genauso woanders im Netz? Das ist kein Rechtsproblem, sondern ein Problem für die Auffindbarkeit. Es entsteht auch ohne fremde Quelle, zum Beispiel wenn die eigene Seite unter mehreren Adressen erreichbar ist oder Textbausteine mehrfach verwendet werden.

Die Prüfungen überschneiden sich: Eine wörtlich übernommene Passage ist zugleich ein Urheberrechtsproblem und Duplicate Content. Trotzdem lohnt es sich, beide Ziele getrennt zu benennen, weil die Gegenmaßnahmen andere sind (eine Passage löschen oder neu schreiben gegen Urheberrechtsverstöße; Adressen und Textbausteine aufräumen gegen Duplicate Content).

## Warum eine reine Web-Prüfung mit offener Software schwierig ist

Um einen Text gegen „das ganze Internet" zu prüfen, braucht man ein Verzeichnis des ganzen Internets. Ein solches Verzeichnis (einen „Index") pflegen nur die Betreiber großer Suchmaschinen. Kein quelloffenes Programm bringt einen eigenen Web-Index mit.

Daraus folgt eine klare Aufteilung:

- **Freie Programme vergleichen Dokumente, die man selbst hat**, oder sie vergleichen den eigenen Text gegen eine Sammlung, die man selbst mitbringt (zum Beispiel einen heruntergeladenen Wikipedia-Auszug).
- **Für die Prüfung gegen das offene Web** gibt es zwei Wege: einen kostenpflichtigen Dienst benutzen (etwa Copyscape) oder sich mit einer Such-Schnittstelle und einem KI-Agenten selbst eine Prüfung bauen. Der zweite Weg wird weiter unten beschrieben.

## Freie Programme für den Dokumentvergleich

Diese Programme brauchen kein Internet. Sie sind stark, wenn schon ein Verdacht besteht, welche Quelle infrage kommt, oder wenn man den eigenen Bestand in sich prüfen möchte.

- **WCopyfind** — ein quelloffenes Programm, das eine Menge von Dokumenten durchsucht und gemeinsame Wortfolgen findet. Man gibt an, ab welcher Länge eine übereinstimmende Wortkette als Treffer gilt (zum Beispiel sechs Wörter am Stück). Das Ergebnis ist ein Bericht, in dem die übereinstimmenden Stellen farbig markiert sind. WCopyfind vergleicht nur Dateien, die man ihm gibt – es sucht nicht im Web. Nützlich zum Beispiel, um alle eigenen Seiten gegen einen heruntergeladenen Auszug eines fremden Wikis zu halten.
- **Sherlock** — ein Kommandozeilenprogramm, das aus Textausschnitten kurze Kennzeichen (Signaturen) berechnet und Dateien anhand dieser Kennzeichen auf Ähnlichkeit vergleicht. Es findet auch dann Übereinstimmungen, wenn einzelne Wörter geändert wurden.
- **JPlag** und **Dolos** — beide sind für Programmcode gedacht, nicht für Fließtext. Wer in seiner Sammlung auch Code-Beispiele veröffentlicht, kann damit prüfen, ob diese aus fremden Projekten stammen.

Für eng angelehnte Paraphrasen und Übersetzungen reicht der reine Wortvergleich oft nicht, weil kaum ein Wort gleich bleibt. Hier hilft die Suche nach Bedeutung: Jeder Absatz wird in eine lange Zahlenreihe umgerechnet, die seinen Inhalt beschreibt (ein „Embedding"). Absätze mit ähnlicher Bedeutung ergeben ähnliche Zahlenreihen, auch bei ganz anderem Wortlaut. Rechnet man die eigenen Absätze und die Absätze einer Vergleichssammlung in solche Zahlenreihen um und sucht die ähnlichsten Paare, findet man auch umformulierte Übernahmen. Wie man Texte lokal in solche Zahlenreihen umrechnet, steht im Kapitel [[inhalt-software-verwalten]].

## Eine eigene Web-Prüfung bauen

Die Grundidee kommerzieller Plagiatsprüfer lässt sich mit freien Bausteinen nachbauen. Der Ablauf in Schritten:

1. **Den eigenen Text zerlegen** – in Absätze und Sätze.
2. **Auffällige Wortfolgen herausziehen.** Pro Abschnitt ein paar Ketten von fünf bis acht Wörtern, am besten solche mit seltenen oder ungewöhnlichen Begriffen. Häufige Allerweltssätze taugen nicht, weil es sie überall gibt.
3. **Wörtlich im Web suchen.** Jede Wortkette als exakte Suche (in Anführungszeichen) an eine Such-Schnittstelle schicken. Selbst betreiben lässt sich zum Beispiel **SearXNG**, eine quelloffene Meta-Suchmaschine, die Anfragen an mehrere öffentliche Suchmaschinen weiterreicht und die Treffer bündelt. Alternativ gibt es kostenpflichtige Such-Schnittstellen. Bei jeder Variante gelten die Nutzungsbedingungen und Tempolimits der befragten Suchmaschinen.
4. **Treffer sammeln.** Gibt es Seiten, auf denen genau dieser Satz steht? Tauchen dieselben Seiten bei mehreren Wortketten auf?
5. **Verdächtige Seiten gegenlesen.** Die gefundenen Seiten laden und Absatz für Absatz gegen den eigenen Text halten.

Das ist im Kern genau das, was ein bezahlter Plagiatsprüfer tut. Der Aufwand steckt im Feinschliff: gute Wortketten auswählen, Fehlalarme aussortieren, das Ganze für viele Seiten automatisieren.

## Der KI-Agent: zuerst ein Gesamtdurchgang

Der Ausgangspunkt ist ein einziger, breiter Durchgang über den kompletten Bestand. Dafür braucht der KI-Agent Zugriff auf alle Inhalte und auf eine Websuche. Die Aufgabe wird ihm in einem Prompt gestellt – ein Prompt ist die Anweisung, die man dem Sprachmodell gibt.

> Überprüfe alle Inhalte dieses Wikis daraufhin, ob sie fremdes Urheberrecht verletzen – wörtliche Übernahmen, eng angelehnte Paraphrasen, übersetzte Passagen. Gib eine Liste der auffälligen Seiten mit Fundstelle, vermuteter Quelle und Schweregrad.

Das Ergebnis ist eine Liste. Wichtig: Nicht jeder Eintrag ist ein echter Verstoß. Der Agent schätzt und rät zum Teil, und er kann Quellen nennen, die es gar nicht gibt. Die Liste ist eine **Prioritätenliste** – sie sagt, wo man zuerst genauer hinschauen sollte, nicht, was schon feststeht.

Denselben Durchgang kann man mit anderem Schwerpunkt wiederholen, zum Beispiel gezielt für Duplicate Content:

> Suche für jede Seite dieses Wikis stichprobenartig drei ungewöhnliche Sätze wörtlich im Web. Liste jede Seite auf, deren Sätze fast gleich auch anderswo vorkommen, mit Adresse der Fundstelle.

## Die Treffer einzeln abarbeiten

Für jede auffällige Stelle aus der Liste folgt ein eigener, enger Durchgang. Ein paar bewährte Prompts:

> Vergleiche den folgenden Absatz mit dem Text unter [Quelle]. Markiere wörtliche Übernahmen und enge Umformulierungen. Stelle die übereinstimmenden Stellen aus beiden Texten nebeneinander.

> Ist der folgende Absatz eine Übersetzung aus [Sprache/Quelle]? Zeige Satzbau und Reihenfolge der Gedanken im Vergleich zum Original.

> Formuliere den folgenden Absatz neu. Inhalt und Fakten sollen erhalten bleiben, aber keine Formulierung aus dem Original übernommen sein. Behalte den sachlichen Ton und erkläre Fachbegriffe.

> Prüfe, ob der folgende Absatz eine eigene Leistung ist oder nur eine leicht abgewandelte Fassung einer bekannten Quelle. Begründe deine Einschätzung.

Nach einer Überarbeitung wird die betroffene Stelle noch einmal durch den Gesamtdurchgang geschickt, um zu prüfen, ob der Treffer verschwunden ist.

## Prüf-Prompts zum Selbernutzen

Die folgenden Anweisungen lassen sich direkt übernehmen. Text in eckigen Klammern jeweils ersetzen.

- **Gesamtprüfung Urheberrecht:** „Überprüfe alle Inhalte unter [Ordner/Adresse] auf fremdes Urheberrecht: wörtliche Übernahmen, enge Paraphrasen, Übersetzungen. Gib eine Tabelle mit Seite, Fundstelle, vermuteter Quelle und Schweregrad (hoch/mittel/niedrig)."
- **Gesamtprüfung Duplicate Content:** „Wähle pro Seite drei seltene Sätze und suche sie wörtlich im Web. Nenne jede Seite mit einer fast gleichen Fundstelle anderswo."
- **Einzelvergleich:** „Vergleiche [Absatz] mit [Quelle]. Zeige übereinstimmende Wortfolgen ab fünf Wörtern nebeneinander."
- **Übersetzungsverdacht:** „Prüfe, ob [Absatz] eine Übersetzung aus [Sprache] ist. Vergleiche Reihenfolge und Satzbau mit [Quelle]."
- **Neu schreiben:** „Schreibe [Absatz] so um, dass Fakten bleiben, aber keine Formulierung übernommen ist. Sachlicher Ton, einfache Sätze."
- **Gegenprüfung nach der Überarbeitung:** „Suche [überarbeiteter Absatz] wörtlich und sinngemäß im Web. Gibt es noch Übereinstimmungen?"

## Was die Prüfung nicht leistet

- Sie ersetzt keine Rechtsberatung. Ob eine Übernahme erlaubt ist (etwa als Zitat), ist eine juristische Frage. Im Zweifel hilft nur ein Anwalt.
- KI-Agenten übersehen Stellen und erfinden Quellenangaben. Jeder gemeldete Treffer muss von Hand nachgeprüft werden, bevor man daraus einen Schluss zieht.
- Bilder, Grafiken, Tabellen und Datensätze prüfen diese Verfahren nicht. Dafür braucht es eigene Schritte.
- Eine bestandene Prüfung heißt „nichts gefunden", nicht „garantiert sauber". Sicherheit gibt vor allem, von Anfang an selbst zu formulieren und Quellen nur als Beleg zu nennen, nicht als Textvorlage.

## Fazit

Für die Prüfung auf Urheberrechtsverstöße und Duplicate Content gibt es keinen fertigen quelloffenen Rundum-Dienst, weil dafür ein eigener Web-Index fehlen würde. Was es gibt, sind freie Programme für den Vergleich von Dokumenten, die man selbst besitzt (WCopyfind, Sherlock, für Code JPlag und Dolos), und die Suche nach Bedeutung über Zahlenreihen für umformulierte Passagen. Die Prüfung gegen das offene Web baut man sich aus einer selbst betriebenen Suche wie SearXNG und einem KI-Agenten zusammen: erst ein breiter Gesamtdurchgang, der eine Prioritätenliste liefert, dann ein enger Einzeldurchgang pro Treffer. Jedes Ergebnis wird von Hand geprüft. So entsteht ein Text, der rechtlich auf der sicheren Seite ist und von Suchmaschinen als eigenständig erkannt wird.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
