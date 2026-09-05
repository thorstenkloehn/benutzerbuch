# Freies Wissen

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).

Bevor man eine Webseite oder eine Wissenssammlung aufbaut, sollte man wissen, welche Werkzeuge es dafür überhaupt gibt. „Freies Wissen" meint hier zweierlei: Wissen, das frei zugänglich ist (jeder darf es lesen und weiterverwenden), und Programme, deren Quelltext offenliegt und die man kostenlos selbst betreiben kann (sogenannte Open-Source-Software). Dieses Kapitel ist ein geordneter Überblick über solche Programme, sortiert danach, wofür man sie einsetzt: Wissen schreiben, Daten strukturieren, offene Daten und Karten verwalten, Notizen vernetzen und über den eigenen Bestand suchen.

Der Überblick soll keine vollständige, ständig aktuelle Liste sein. Er soll helfen, die Landschaft zu kennen, bevor man sich für ein Werkzeug entscheidet.

## Wiki- und Wissensdatenbank-Software

Ein Wiki ist eine Webseite, deren Seiten viele Menschen gemeinsam im Browser bearbeiten können.

- **MediaWiki** — die Software, mit der auch die Wikipedia läuft. Sie ist auf sehr viele Seiten und viele Bearbeiter ausgelegt und speichert zu jeder Seite die komplette Änderungsgeschichte.
- **XWiki** — ebenfalls ein Wiki, das sich zusätzlich um feste Datenfelder und kleine Anwendungen erweitern lässt. Damit kann man nicht nur Fließtext ablegen, sondern auch strukturierte Einträge, etwa eine Liste von Geräten mit festen Angaben.

## Strukturierte Daten und Linked Open Data

„Strukturierte Daten" bedeutet, dass Informationen nicht als Fließtext, sondern in festen Bausteinen gespeichert werden, die auch ein Computer auswerten kann. „Linked Open Data" heißt, dass diese Bausteine offen zugänglich sind und untereinander verknüpft werden, sodass aus vielen Quellen ein Netz an Wissen entsteht.

- **Wikibase** — die Software hinter Wikidata, der Datensammlung der Wikipedia. Wissen wird hier als einzelne Objekte mit Aussagen gespeichert (zum Beispiel: Objekt „Berlin", Aussage „ist Hauptstadt von" → „Deutschland").
- **OpenRefine** — ein Werkzeug, um unordentliche Tabellen aufzuräumen: doppelte Schreibweisen zusammenführen, Fehler finden, Spalten vereinheitlichen.
- **Apache Jena** — eine Sammlung von Bausteinen für Programmierer, um mit verknüpften Daten zu arbeiten.
- **Fuseki** — der Datenbank-Server, der zu Jena gehört. Er beantwortet Anfragen an die Daten über das Internet, in der Abfragesprache SPARQL (der üblichen Sprache, um verknüpfte Daten abzufragen).
- **QLever** — ein besonders schneller Server für dieselbe Art von Anfragen. Er kann sehr große Datenmengen bis hin zur kompletten Wikidata auf einem einzelnen Rechner verarbeiten.

## Offene Daten und Karten

Offene Daten sind Datensätze, die jeder herunterladen und weiterverwenden darf, oft von Behörden oder aus der Forschung.

- **CKAN** — Software für Datenportale. Damit betreiben zum Beispiel Verwaltungen ihre offenen Datenkataloge: Jeder Datensatz bekommt eine Beschreibung, und Besucher können darin suchen.
- **OpenStreetMap** — eine freie Weltkarte, an der jeder mitarbeiten kann. Die Kartendaten sind offen und dürfen in eigenen Projekten benutzt werden.
- **Dataverse** — ein Ablageort für Forschungsdaten. Jeder Datensatz erhält einen dauerhaften Link, sodass andere ihn zitieren können.
- **DataHub** — eine Plattform, um Datensätze zu finden, zu veröffentlichen und miteinander zu teilen.

## Docs-as-Code und statische Webseiten

„Docs as Code" bedeutet: Man schreibt Texte in einfachen Textdateien (meist im Format Markdown) und lässt daraus mit einem Werkzeug automatisch eine fertige Webseite bauen. Die Texte lassen sich so genauso versionieren und prüfen wie Programmcode.

- **mdBook** — das Werkzeug, mit dem auch dieses Handbuch gebaut wird. Aus einem Ordner mit Markdown-Dateien entsteht ein durchsuchbares HTML-Buch.
- **MkDocs** — baut aus Markdown-Dateien eine Dokumentations-Webseite. „Material" ist die verbreitetste Gestaltungsvorlage dafür und bringt Suche, Navigation und ein aufgeräumtes Aussehen mit.

## Moderne Wissensdatenbanken

Neben den klassischen Wikis gibt es neuere Programme, die eher wie ein gemeinsames Notizbuch für ein Team funktionieren. Sie sind bewusst als freie Alternativen zu bekannten kommerziellen Diensten gedacht.

- **BookStack** — ordnet Inhalte in einer einfachen Gliederung aus Regalen, Büchern, Kapiteln und Seiten. Der Einstieg ist leicht, weil die Struktur vorgegeben ist.
- **Outline** — eine Wissensdatenbank für Teams mit schnellem Editor und guter Suche.
- **Docmost** — ähnlich wie Outline, mit Seiten, die sich beliebig verschachteln lassen, und gemeinsamem Bearbeiten in Echtzeit.
- **AppFlowy** — verbindet Notizen mit Tabellen und Aufgabenlisten in einem Programm.

## Vernetztes Wissen und Notizen

Diese Programme sind für einzelne Personen gedacht. Ihr gemeinsamer Gedanke: Notizen werden nicht nur in Ordnern abgelegt, sondern untereinander verlinkt, sodass mit der Zeit ein persönliches Wissensnetz entsteht.

- **Logseq** — arbeitet mit kurzen, eingerückten Stichpunkten und Verweisen zwischen ihnen. Die Notizen bleiben als normale Dateien auf dem eigenen Rechner.
- **Joplin** — ordnet Notizen in Notizbüchern, schreibt sie in Markdown und kann sie zwischen mehreren Geräten abgleichen.
- **TriliumNext Notes** — legt Notizen in einem beliebig tiefen Baum ab und eignet sich gut für eine große persönliche Wissenssammlung. Es wird von einer Gemeinschaft weiterentwickelt, nachdem das ursprüngliche Trilium Notes nur noch gepflegt, aber nicht mehr ausgebaut wird.
- **Anytype** — speichert Wissen als verknüpfte Objekte mit festen Feldern und legt alle Daten verschlüsselt auf dem eigenen Gerät ab.

## Suche über das eigene Wissen

Wenn eine Sammlung wächst, braucht man eine gute Suche. Die folgenden Programme sind Suchmaschinen, die man selbst betreibt und in eine eigene Webseite oder Anwendung einbaut.

- **OpenSearch** — eine leistungsfähige Suchmaschine für große Datenmengen. Sie ist aus der bekannten Software Elasticsearch hervorgegangen und steht unter einer freien Lizenz.
- **Meilisearch** — eine schlanke, schnelle Suchmaschine, die sich leicht einrichten lässt und schon während des Tippens Treffer anzeigt.
- **Typesense** — ähnlich wie Meilisearch: schnell, verzeiht Tippfehler und ist auf einfache Bedienung ausgelegt.

## Warum überhaupt Wissen sammeln?

Wissen ist für ein Unternehmen wie für eine einzelne Person zu einem wichtigen Gut geworden. Zwei Grundfragen stehen dabei am Anfang, und beide sollte man klären, bevor man mit dem Bauen beginnt.

Die erste Frage: Wie strukturiert man die Daten, damit man sie später wiederfindet? Und wie organisiert man sie so, dass sie über Jahre nutzbar bleiben? Wer diese Frage überspringt und einfach loslegt, muss oft alles noch einmal neu aufbauen. Eine ungünstig angelegte Struktur lässt sich später nur mühsam reparieren. Deshalb lohnt es sich, hier zuerst nachzudenken und nicht sofort die erste Lösung zu nehmen.

Die zweite Frage: Wie veröffentlicht man Inhalte, ohne gegen Gesetze zu verstoßen? Besonders wichtig ist das Urheberrecht. Das deutsche Urheberrecht gilt als vergleichsweise streng; ein pauschales „fair use" wie im US-Recht gibt es hier nicht. Fremde Texte, Bilder oder Übersetzungen dürfen nicht einfach übernommen werden, auch nicht in leicht umgeschriebener Form. Man kann diese Strenge als Nachteil sehen oder als Schutz der eigenen Arbeit; in jedem Fall muss man sie einplanen.

Viele Einzelfragen bleiben am Anfang offen: unter welcher Lizenz man die eigenen Inhalte stellt, wie man Quellen kennzeichnet, wie man KI-Unterstützung transparent macht. Das ist normal. Wichtig ist die Reihenfolge: erst Wissen sammeln, dann strukturieren, dann veröffentlichen. Ohne diese Grundlage entsteht keine wirklich gute Seite.

## Fazit

Für jede der genannten Aufgaben gibt es mehrere freie Programme, und es kommen laufend neue dazu. Man muss sie nicht alle kennen. Hilfreich ist vor allem, die Gruppen auseinanderzuhalten: Wiki für gemeinsam bearbeiteten Text, strukturierte Daten für maschinenlesbare Bausteine, Docs-as-Code für Text, der wie Code gepflegt wird, Notiz-Programme für das persönliche Wissensnetz und eine eigene Suchmaschine, sobald die Sammlung groß wird. Wer diese Einteilung im Kopf hat, kann für ein konkretes Vorhaben gezielt das passende Werkzeug auswählen, statt sich vom ersten bekannten Namen leiten zu lassen.

> Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).
