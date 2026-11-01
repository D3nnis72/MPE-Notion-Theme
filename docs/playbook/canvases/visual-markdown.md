# Visual Markdown — Brainstorming Canvas

> **Status:** Approved — basis for specs
>
> Living map of this brainstorm. Rewrite it when the direction changes.
> Specs and plans are distilled from this file. Not canonical product behavior.
> After shipping, the README and skills own shipped truth; leftovers hold
> remaining work only, then delete when nothing remains.
>
> Größe M, Plan Lite: ein zusammenhängender Prototyp, kein Paket, das sich getrennt prüfen ließe.

## Ziel

Ein Markdown-Format, das im Preview von Markdown Preview Enhanced (MPE) für Menschen richtig gut aussieht und für LLMs trotzdem schlank bleibt. Dazu ein Skill, mit dem Agents darin illustrieren und arbeiten. Kollegen können es installieren, extern geht es als PDF raus.

## Leitidee

Die Datei beschreibt, was da ist. Der Renderer entscheidet, wie es aussieht. Tokens kostet nur die Datei, alles Visuelle steckt in `style.less` und `parser.js`, die kein LLM beim Lesen mitlädt.

Gegenüber gut geschriebenem Markdown sparen Bausteine kaum Tokens, gegenüber HTML sehr viel. Ihr eigentlicher Gewinn ist, dass die Struktur explizit wird: Der Renderer weiß, dass etwas eine Entscheidung mit Status ist, und kann sie so zeigen und verlinken. Echte Ersparnis kommt von dem, was aus der Datei verschwindet: Standardtexte, die der Renderer ergänzt, und Daten, auf die verwiesen statt kopiert wird.

## Regeln für das Format

- Bausteine sind Codeblöcke mit einem Typ. Darin stehen ein paar Zeilen YAML oder, bei `steps`, eine erweiterte Checkliste. `parser.js` macht daraus HTML, das Aussehen kommt aus dem gemeinsamen Theme. Rohes HTML bleibt Notausgang für Einzelfälle.
- Wo GitHub schon eine Syntax hat, nehmen wir die statt eines eigenen Blocks, etwa bei Callouts. Die rendert GitHub selbst, und LLMs kennen sie.
- Jeder Block ist ohne Renderer verständlich. Namen und Felder sprechen für sich, damit ein lesendes LLM keinen Skill braucht. Das Vokabular braucht nur, wer schreibt. Auf GitHub erscheint der Block als lesbarer Codeblock.
- Jeder Baustein hat ein Schema mit seinen Feldern, Typen, erlaubten Werten und Pflichtfeldern. Renderer, Check-Skript und die Referenz im Skill lesen alle daraus, damit Beschreibung und Verhalten nicht auseinanderlaufen.
- Das Vokabular bleibt klein. Projekte können eigene Blöcke mit eigenem Schema anmelden, statt das gemeinsame Vokabular aufzublähen.
- Viele Zeilen mit kurzen Feldern bleiben eine normale Tabelle, weil YAML die Feldnamen bei jedem Eintrag wiederholt. Navigationstabellen, Datei- und Feldlisten und Fließtext bleiben normales Markdown, Flussdiagramme Mermaid, Formeln KaTeX.
- Daten und Blöcke werden über `src:` eingebunden statt kopiert. Ein Block, der nur `src: datei.md` enthält, zeigt den gleichartigen Block aus der anderen Datei, `#abschnitt` wählt bei mehreren den unter dieser Überschrift. Ein lesendes LLM sieht dort eine Zeile. Kein Inline-SVG und kein Base64 in der Quelldatei.
- Erst statisch. JavaScript im Preview braucht `enableScriptExecution` und kommt erst bei einem konkreten Bedarf.
- Ein kaputter Block zeigt im Preview einen Fehlerkasten mit Zeilennummer. Das Check-Skript prüft alle Blöcke gegen ihr Schema, Agents lassen es nach dem Schreiben laufen.
- Gilt für Dokumente, die Menschen anschauen: Canvasse, Statusseiten, Entscheidungen, Fragensammlungen, Berichte. Knowledge-Docs, die vor allem Agents lesen und schreiben, bleiben schlichtes Markdown.
- Erfolgsmaß: Ein nachgebautes Canvas braucht nicht mehr Tokens als die schlichte Version, und man erfasst den Stand beim Draufschauen in zehn Sekunden.

## Bausteine

| Baustein | Wo es heute vorkommt | Was der Renderer zeigt | Runde |
| --- | --- | --- | --- |
| `status` | Kopf jedes Canvas und jeder Statusseite: Status, Stand-Datum, fester Erklärtext | Farbiges Band mit Status und Datum, den Standardtext ergänzt der Renderer | 1 |
| `steps` | Schritt-Tabellen in Canvassen, „Was als Nächstes passiert“ auf Statusseiten, Checkboxen und Prosa | Fortschritt, Zustand je Schritt, „Jetzt möglich“, parallele Wellen, Engpass und Graph | 1 |
| `decision` | Entscheidungen mit ID: entschieden, warum, ersetzt, erhalten, offen | Karte mit Status-Badge, ersetzte Entscheidungen als Link | 1 |
| `question` | Fragensammlung: an wen, was wir sehen, was wir brauchen, was wir sonst tun | Karte mit Empfänger und Stand (offen, teilweise beantwortet, geklärt) | 1 |
| `checks` | „Prüfung, Ergebnis, Grenze“ in Findings, Testläufe und „durchgefallen, wenn“ in LioCare | Liste mit bestanden, offen oder durchgefallen als Badge, Grenze klar abgesetzt | 1 |
| `chart` | Heute fast nur Zahlentabellen: Anteile und Spend je Lieferant, KPI-Werte über Wochen, Gewichte in LioCare | Balken, gestapelte Balken, Linien und Histogramme über Vega-Lite | 1 |
| Callouts | Beispielfälle, Annahmen, Grenzen, Hinweise | Kasten je Art | 1 |
| Badges | Data Status in LioCare, Richtungen, Zustände wie offen und entschieden | Gefärbte Tabellenzellen in der ganzen Datei | 1 |
| `scale` | Schwellen und Bereiche, etwa die Health States in LioCare | Farbiges Band über den ganzen Bereich mit den Grenzen darauf | 2 |
| `gallery` | Screenshots, Referenzbilder, vorher und nachher, Dark und Light in LioCare | Bilder nebeneinander mit Bildunterschrift | 2 |
| `compare` | Zwei bis drei Ansätze mit Vor- und Nachteilen beim Brainstorming | Spalten nebeneinander, Empfehlung markiert | 2 |
| Glossar | Begriffe, die in vielen Docs neu erklärt werden | Erklärung beim Hovern, die nur einmal im Glossar steht | 2 |

Die Details stehen unten nur für die Bausteine, bei denen die Tabelle nicht reicht.

### `steps`

Der wichtigste Baustein: die Übersicht, wo wir stehen, was fehlt und was als Nächstes dran ist, zum Abarbeiten für Menschen und Agents. Heute steht dieser Stand verteilt und geht unter.

- Format ist eine erweiterte Checkliste. Das Kästchen trägt den Zustand: `[x]` fertig, `[~]` läuft, `[!]` wartet, `[ ]` offen. Jeder Schritt hat ein Kürzel. Nach einem Trennstrich stehen `nach: <Kürzel>` für interne Abhängigkeiten, `wartet auf: <Partei>` für externe Blocker, optional `fertig wenn:`, `seit:`, `schreibt: <Dateien>` und ein Link auf die Step-Datei.
- Der Renderer zeigt oben den Fortschritt und berechnet „Jetzt möglich“: alle offenen Schritte, deren Vorgänger fertig sind und die auf niemanden warten. Kein Prosatext zum Stand, der veraltet.
- Parallele Wellen rechnet der Renderer aus den Abhängigkeiten. Niemand pflegt sie von Hand. Engpässe und die längste Kette bis zum Ziel sind hervorgehoben. Eine Graph-Ansicht erzeugt `parser.js` als Mermaid-Diagramm mit den Theme-Klassen.
- Unterpunkte sind normale Checklisten, der Fortschritt eines Schritts zählt aus den Häkchen.
- Statusseiten und `00_overview.md` binden die `steps` eines Canvas über `src:` ein, statt eigene Schritt-Tabellen zu pflegen. Dadurch können sie nicht veralten.
- Das Check-Skript findet Zyklen, unbekannte Kürzel und parallele Schritte, die dieselben Dateien schreiben. Es gibt die Wellen als Text aus, damit ein ausführender Agent sie nicht herleiten muss.

````markdown
```steps
- [x] gt: Ground Truth und Batch-Ablauf
- [~] rb: Rechnungs-Batches übernehmen | nach: gt
- [!] fb: Feedback zur Startauswahl | wartet auf: Accenture
- [ ] ag: Vorgeschlagene Artikel gegenlesen
- [ ] lp: Lieferantenpakete neu bauen | nach: rb
- [ ] lm: Lieferantenmails vorbereiten | nach: fb, lp
```
````

```text
Batch 1                                       1 von 6 fertig  ██░░░░░░░░░░
● Ground Truth und Batch-Ablauf                              fertig
◐ Rechnungs-Batches übernehmen                               läuft
⏸ Feedback zur Startauswahl                                  wartet auf Accenture
○ Vorgeschlagene Artikel gegenlesen                          kann starten
○ Lieferantenpakete neu bauen                                nach Rechnungs-Batches
○ Lieferantenmails vorbereiten                               nach Feedback und Paketen

Jetzt möglich: Vorgeschlagene Artikel gegenlesen
```

### `chart`

`parser.js` übersetzt den Block vor dem Rendern in Vega-Lite, das MPE selbst zeichnet. Gestapelte Balken gehen auch auf 100 %. Pflichtfelder sind `says`, die Aussage in einem Satz, und `basis`, worauf die Zahlen beruhen. Damit versteht ein LLM das Diagramm, ohne die Daten zu laden. Farben kommen aus dem Theme, die Palette beim Bauen mit dem Skill `dataviz` festlegen. Große Rohdaten verdichtet vorher ein Skript zu einer kleinen CSV. Für Einzelfälle bleibt ein roher `vega-lite`-Block.

````markdown
```chart
type: histogram
src: SLB/enriched/textlaenge-nach-quelle.csv
x: textlaenge
color: spend_source
says: SAP-Texte stauen sich bei genau 40 Zeichen, Cameron-Texte nicht.
basis: PO-Daten 2025, Zeilen im Scope
```
````

### Callouts und Badges

Callouts schreiben wir in der GitHub-Syntax `> [!NOTE]` oder `> [!WARNING]`. Für Beispielfälle, Annahmen und Grenzen kommen eigene Arten wie `> [!EXAMPLE]` und `> [!ASSUMPTION]` dazu, die der Renderer gestaltet. Ohne Renderer bleibt ein lesbares Zitat.

Badges sind ein Mechanismus statt eines Blocks. Eine Zeile im Frontmatter ordnet feste Wörter Farben zu, etwa Da, Berechenbar, In Prod und Fehlt beim Data Status in LioCare. Der Renderer färbt passende Tabellenzellen in der ganzen Datei.

## Später

- Weitere Chart-Typen wie Boxplot, Streudiagramm und Heatmap bei Bedarf.
- Projektspezifische Blöcke, etwa ein Grundriss aus den Raumtabellen der LioCare-Office-Docs.

## Zuhause, Verteilung, Export

- Zuhause ist dieses Repo. Der lokale Ordner heißt schon `Visual-Markdown`, das GitHub-Repo noch `MPE-Notion-Theme`. Es hat schon Theme, Mermaid-Konfiguration, `install.sh` und den Skill `mermaid-theme`, der nach demselben Prinzip arbeitet. Neu kommen `parser.js`, die Schemas der Bausteine, ein Skill für die Bausteine und das Check-Skript dazu.
- Kollegen installieren MPE, klonen das Repo und führen `install.sh` aus. Alternativ ein `.crossnote/`-Ordner im Projekt-Repo, dann reicht MPE allein.
- Externes Teilen läuft als statischer PDF-Export. Verwiesene Daten und Diagramme werden eingebacken, relative Links auf Repo-Dateien aufgelöst oder entfernt.
- Langfristig wird der Renderer ein eigenständiges Modul, damit dieselben Dateien auch ohne VS Code rendern, etwa als gehostete Seite mit Link. Deshalb trennt `parser.js` von Anfang an zwei Stufen: Das Einlesen macht aus jedem Block eine gegen das Schema geprüfte Datenstruktur, die Ausgabe macht daraus HTML. Weitere Ausgaben kommen dann ohne Umbau dazu.

## Reihenfolge

```steps
- [ ] proto: Prototyp mit `steps`, `chart`, `decision` und Badges an einem echten Canvas, Schemas und zwei Stufen von Anfang an
- [ ] check: Prüfpunkte aus dem Prototyp klären (siehe offene Fragen) | nach: proto
- [ ] rest: Übrige Bausteine der ersten Runde | nach: check
- [ ] skill: Skill für die Bausteine und Check-Skript | nach: rest
- [ ] share: GitHub-Repo umbenennen, `install.sh` erweitern, PDF-Export prüfen | nach: rest
- [ ] plans: Entscheiden, ob `steps` die Tabelle in `writing-plans` ersetzt | nach: proto
```

## Offene Fragen

- Im Prototyp zu prüfen: Laufen die `parser.js`-Hooks auch beim PDF-Export? Hat ein `.crossnote/`-Ordner im Workspace Vorrang vor der globalen Config? Kann `parser.js` für `src:` andere Dateien lesen, Daten wie Blöcke, oder geht das nur über MPEs `@import` beziehungsweise eine Daten-URL in Vega-Lite? Erscheinen Vega-Lite-Diagramme im PDF? Ab welcher Datenmenge muss vorverdichtet werden? Zeigt MPE GitHub-Callouts schon selbst an, oder übernimmt das `parser.js`?
- Endgültigen Namen bestätigen, bevor das GitHub-Repo umbenannt wird.
- Wo verläuft die Grenze zwischen `steps` und den persönlichen Aufgaben in Notion, damit nichts doppelt gepflegt wird?
- Ersetzt `steps` später die Tabelle „Execution Schedule“ in `writing-plans`? Das wäre eine Änderung im öffentlichen Playbook-Repo und betrifft auch `subagent-driven-development`.
- Wie verhindern wir, dass Kundendaten ungewollt in einem PDF landen?
- LioCare hat eine eigene Kopie des Notion-Themes im Repo-Root. Ziehen Projekte das Theme künftig nur noch aus diesem Repo?

## Nicht jetzt

- Eigene Bausteine `split` und `callout`: Ein einzelner gestapelter Balken in `chart` und die Callout-Syntax von GitHub decken sie ab.
- MDX (Markdown mit React-Komponenten): braucht Build und React-Laufzeit, rendert weder in MPE noch auf GitHub, und ein einziger Syntaxfehler bricht die ganze Datei. Passt erst, falls wir eine gehostete Doku-Seite bauen. Dann können die Bausteine dorthin übersetzt werden.
- Markdoc als Syntax, samt Variablen und Bedingungen: Seine Tags erscheinen ohne Renderer als Rohtext, und bei Variablen im Fließtext sähe ein lesendes LLM nur den Namen statt der Zahl. Übernommen sind das Schema je Baustein, das Einbinden und die getrennten Stufen.
- ReadMe als Plattform: gehostete API-Doku mit OpenAPI-Referenz, Analytics und Browser-Editor, eigene Komponenten über MDX. Übernommen sind Callouts als Zitatblock, das Einbinden und die Glossar-Idee.
- HTML-Export und interaktiver Export.
- Eigener Viewer oder eigene VS-Code-Extension.
- Frei verschiebbares Whiteboard wie Miro.
- JavaScript-Interaktivität im Preview.
