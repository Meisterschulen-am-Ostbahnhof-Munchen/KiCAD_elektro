# Eagle -> KiCad Import: Analyse der Konvertierungsverluste

> **Testfall:** Stern-Dreieck-Anlauf-Schaltplan (einseitig + mehrseitig)
> **Eagle-Version:** 7.7.0
> **KiCad-Version:** 10.0.5
> **Analysedatum:** 2026-08-09
> **Analysemethode:** PDF-Textvergleich (pdfminer) + Auswertung der .sch-Quelldatei

---

## Kontext

Dieser Schaltplan ist ein klassischer **industrieller Stromlaufplan** (Hauptstrom- und
Steuerstromkreis, Stern-Dreieck-Anlauf) mit folgenden Bauteilen:

| Bezeichnung | Typ | Bibliothek |
|-------------|-----|-----------|
| Q1 | Lasttrennschalter 3-pol | e-schalter |
| F1 | Sicherung 3-pol NH 63A | e-sicherungen |
| F2 | Sicherung 3-pol NH 35A | e-sicherungen |
| F3 | Motorschutzschalter 3-pol 16,3A (mit Hilfskontakten) | e-motorschutzschalter |
| F4 | Leitungsschutzschalter 10A | e-sicherungen |
| Q2 | Hauptschuetz 3-pol (HK 13-14) | e-schuetze-relais |
| Q3 | Sternschuetz 3-pol (HK 11-12) | e-schuetze-relais |
| Q4 | Dreieckschuetz 3-pol (HK 11-12 + 13-14) | e-schuetze-relais |
| K1 | Hilfsschuetz, zeitverzoegert oeffnend (HK 15-16) | e-schuetze-relais |
| S1 | Taster Oeffner (AUS) | e-schalter |
| S2 | Eintaster (EIN) | e-schalter |
| M1 | Motor 3-Phasen 10 kW, Stern-Dreieck | e-motoren |
| X1, X2, X3 | Reihenklemmen | e-klemmen |
| EINSPEISUNG1 | Einspeisung 3/N/PE | e-klemmen |

**Verwendete Eagle-Bibliotheken (eigene, elektrotechnische Spezial-Libs):**
e-elektro-zeichnungsrahmen, e-schalter, e-sicherungen, e-motorschutzschalter,
e-schuetze-relais, e-motoren, e-klemmen

---

## Analyseergebnis: Was geht beim Import verloren?

### Problem 1 -- Bauteilbezeichnungen verschwinden (F1, Q2, K1 ...)

**Schweregrad:** KRITISCH

Alle Bauteilbezeichnungen (Referenzen) aus dem Schaltplan fehlen im konvertierten PDF
vollstaendig. Im Eagle-PDF sind F1, F2, F3, F4, Q1-Q4, K1, S1, S2,
M1, X1-X3 klar lesbar. Im KiCad-PDF tauchen diese Namen nicht auf.

**Ursache (Vermutung):** Die Referenz-Texte liegen in Eagle auf Layer 95 (Names).
Der Importer uebertraegt diese Layer-Information moeglicherweise nicht korrekt auf
KiCad-Textfelder.

---

### Problem 2 -- Querverweise (/1.13E, /1.16D ...) gehen verloren

**Schweregrad:** KRITISCH

Eagle generiert bei mehrseitigen Plaenen und bei aufgeteilten Symbolen (z. B. Schuetz-Spule
auf Blatt A, Hilfskontakte auf Blatt B) automatisch **Querverweise** im Format /%S.%C%R
(Blatt.Spalte+Reihe).

Im Testplan fehlen nach der Konvertierung:

```
/1.13B   /1.13D   /1.13E
/1.14E   /1.16D   /1.16E
/1.17D   /1.17E   /1.4D
/1.4E    /1.7D    /1.9D
```

Eagle speichert das Querverweisformat in der Schematicwurzel:
```xml
<schematic xreflabel="%F%N/%S.%C%R" xrefpart="/%S.%C%R">
```

KiCad hat kein direktes Aequivalent fuer dieses automatische Querverweissystem.
Die Informationen werden beim Import schlicht ignoriert.

---

### Problem 3 -- Titelblock-Variablen (>DATUM, >KUNDE ...) werden nicht uebersetzt

**Schweregrad:** HOCH

Eagle verwendet in Titelblock-Symbolen Platzhalter der Form >VARIABLENNAME, die zur
Laufzeit durch ttribute-Werte ersetzt werden. Im Testplan z. B.:

| Eagle-Variable | Wert im Testplan |
|----------------|-----------------|
| >FUNKTION | Foerderband |
| >HERSTELLER | Siemens |
| >DATUM | (nicht gesetzt) |
| >KUNDE | (nicht gesetzt) |
| >AUFTRAGS_NR | (nicht gesetzt) |
| >ERSTELLER | (nicht gesetzt) |
| >WERKS_NR | (nicht gesetzt) |
| >ZEICHNUNGS_NR | (nicht gesetzt) |

Im konvertierten KiCad-Plan erscheinen die Literaltexte >DATUM, >BEARBEITET etc.
als rohe Strings, statt die gesetzten Werte zu enthalten oder auf KiCad-Titelblockfelder
gemappt zu werden.

---

### Problem 4 -- Hilfskontakt-Nummern (95/96/97/98) beschaedigt

**Schweregrad:** HOCH

Eagle-Symbole fuer Motorschutzschalter verwenden die IEC-normierten Kontaktbezeichnungen
95, 96 (Oeffner), 97, 98 (Schliesser) fuer den Thermischen Ausloser.
Nach der Konvertierung erscheinen diese Nummern nicht mehr korrekt -- stattdessen tauchen
ASCII-Zeichen (:, ;, <, =, >, ?, @) auf, was auf ein
**Encoding-Problem im Symbol-Mapping** hindeutet (ASCII-Offset von 48).

---

### Problem 5 -- Motor-Symbol M 3~ wird zu 3~~

**Schweregrad:** MITTEL

Das Drehstromsymbol 3~ wird in KiCad als 3~~ ausgegeben.
Die Tilde (~) ist in KiCad ein Overbar-Marker; beim Import wird das Eagle-Literal ~
nicht escaped, sodass es als Sonderzeichen statt als Text interpretiert wird.

---

### Problem 6 -- Dateiname / Zeichnungsnummer aus Titelblock fehlt

**Schweregrad:** MITTEL

Der Dateiname stern-dreieck-anlauf (aus Eagle-Attribut) und die Zeichnungsnummer
erscheinen nicht im KiCad-Titelblock. Stattdessen generiert KiCad einen generischen
Eintrag mit leerem Title:, Date:, Rev:.

---

### Problem 7 -- L3 + N werden zu L3 N konkateniert

**Schweregrad:** MITTEL

Im Eagle-Plan sind L3 und N separate Netze/Labels. Im KiCad-Output erscheint
3 N als ein zusammengesetzter String -- Parser-Problem bei nahe beieinander liegenden
Netz-Labels oder Pins.

---

### Problem 8 -- Seitenuebergreifende Netz-Querverweise fehlen komplett

**Schweregrad:** KRITISCH

Bei mehrseitigen Schaltplaenen erzeugt Eagle automatisch **seitenuebergreifende
Netz-Querverweise** fuer jedes Netz das auf mehreren Blaettern vorkommt.
Format: NETZNAME/BLATT.SPALTEREIHE -- z. B.:

| Eagle-Querverweis | Bedeutung |
|-------------------|-----------|
| L1/1.18A | Netz L1, weiter auf Blatt 1, Position Spalte 18, Reihe A |
| L1/2.2A | Netz L1, kommt von Blatt 2, Position Spalte 2, Reihe A |
| N/1.18F | Netz N, weiter auf Blatt 1, Position 18F |
| PE/2.1F | Netz PE, kommt von Blatt 2, Position 1F |

**Nachweis aus dem mehrseitigen Testplan** (sample2/, 2 Blaetter):

Im Eagle-Original vorhanden, in KiCad komplett fehlend:
```
L1/1.18A   L1/2.2A
L2/1.18A   L2/2.2A
L3/1.18A   L3/2.2A
N/1.18F    N/2.1F
PE/1.18F   PE/2.1F
```

Diese Angaben sind in industriellen Stromlaufplaenen nach **DIN EN 61082 / IEC 61082**
normativ gefordert. Ohne sie ist ein mehrseitiger Plan fuer den Elektrotechniker
**nicht lesbar** -- er kann nicht erkennen, woher ein Netz kommt oder wohin es gefuehrt wird.

**Ursache:** KiCad hat Net Labels und Hierarchical Labels/Ports -- aber kein automatisches
Querverweissystem das Blattnummer + Koordinate anzeigt. Dies ist ein **fehlendes
KiCad-Feature**, nicht nur ein Import-Bug.

**Verifiziert (2026-08-09):** KiCad zeigt in der GUI die **Seitennummer** des
verweisenden Blatts (z. B. "2"), aber **nicht** die Spalte/Reihe (Row/Col).
Eagle zeigt das vollstaendige Format NETZNAME/BLATT.SPALTEREIHE (z. B. L1/2.2A).
Das KiCad-Aequivalent ist damit nur ein Teilergebnis -- fuer normgerechte
Stromlaufplaene nach DIN EN 61082 nicht ausreichend.

---

### Problem 9 -- kicad-cli expandiert ${INTERSHEET_REFS} nicht

**Schweregrad:** HOCH | **Status: VERIFIZIERT**

KiCad speichert seitenuebergreifende Querverweise als Property ${INTERSHEET_REFS}
in der .kicad_sch-Datei. In der KiCad-GUI werden diese korrekt aufgeloest und
angezeigt (sofern intersheets_ref_show: true im Projekt gesetzt ist).

Der **CLI-Exporter** (kicad-cli sch export pdf) expandiert ${INTERSHEET_REFS}
jedoch **nicht** -- die Felder bleiben leer. Verifiziert mit:

```bash
# intersheets_ref_show: true im .kicad_pro
# (hide no) in allen Intersheetrefs-Bloecken der .kicad_sch
kicad-cli sch export pdf stern-dreieck-anlauf_multipage.kicad_sch
# Ergebnis: ${INTERSHEET_REFS} = leer im PDF
```

Dies betrifft **alle** Workflows die kicad-cli zur automatisierten PDF-Erzeugung
verwenden (CI/CD, Makefile, Skripte).

**Workaround:** PDF ueber die KiCad GUI exportieren (Datei -> Schaltplan plotten -> PDF).
Die GUI loest ${INTERSHEET_REFS} korrekt auf.

---

### Problem 11 -- Seitenreferenzen ohne Hierarchie: jede Seite verweist auf alle anderen

**Schweregrad:** HOCH | **Status: VERIFIZIERT**

Bei einem mehrseitigen Schaltplan mit einem Netz das auf mehreren Seiten vorkommt
zeigt KiCad's ${INTERSHEET_REFS} fuer jede Seite **alle anderen Seiten** auf
denen dieses Netz einen Label hat -- ohne Filterung, ohne Hierarchie,
ohne Koordinatenangabe.

**Beobachtetes Verhalten** (6-seitiger Plan, Netz erscheint auf allen Seiten):

| Seite | KiCad zeigt | Eagle wuerde zeigen |
|-------|-------------|---------------------|
| 1 | 2 3 4 5 6 | nur Seiten mit direkter Verbindung + Koordinate |
| 2 | 1 3 4 5 6 | nur Seiten mit direkter Verbindung + Koordinate |
| 3 | 1 2 4 5 6 | nur Seiten mit direkter Verbindung + Koordinate |

In der Praxis entsteht dadurch eine **Informationsflut** statt einer
nuetzlichen Navigation: Ein Netz wie PE oder N das auf allen 6 Seiten
vorkommt verweist von jeder Seite auf alle anderen 5 -- der Elektrotechniker
kann nicht erkennen **wohin** das Netz gefuehrt wird.

**Eagle-Verhalten (korrekt):** Die Querverweise zeigen ausschliesslich die
Stellen wo das Netz tatsaechlich ein- oder austritt (Drahtende an Seite),
mit vollstaendiger Koordinate (NETZNAME/BLATT.SPALTEREIHE).

**Ursache:** KiCad's ${INTERSHEET_REFS} macht eine einfache Namensuche ueber
alle Blaetter -- kein topologisches Routing, keine Unterscheidung zwischen
"Netz durchlaeuft diese Seite" und "Netz endet/beginnt hier".

---

### Problem 12 -- kicad-cli folgt nur der uebergebenen Datei, nicht allen "flat" Top-Level-Sheets

**Schweregrad:** HOCH | **Status: VERIFIZIERT**

`kicad-cli sch export pdf` exportiert **nur die Seitenhierarchie ab der
uebergebenen Datei**. Bei Projekten mit mehreren unabhaengigen ("flat")
Top-Level-Sheets -- wie sie beim Eagle-Import entstehen -- wird nur die
eine uebergebene Seite exportiert, die anderen Seiten werden **ignoriert**.

**Auswirkung fuer mehrseitige Eagle-Importe:**

Da Eagle alle Blaetter als gleichwertige Top-Level-Sheets speichert (kein
Hierarchie-Root wie bei nativen KiCad-Projekten), muss jede Seite einzeln
exportiert und anschliessend zusammengefuehrt werden:

```bash
# Notwendiger Workaround: Jede Seite einzeln, dann merge mit Ghostscript
kicad-cli sch export pdf --output seite1.pdf seite1.kicad_sch
kicad-cli sch export pdf --output seite2.pdf seite2.kicad_sch
gs -dBATCH -dNOPAUSE -sDEVICE=pdfwrite -sOutputFile=gesamt.pdf seite1.pdf seite2.pdf
```

**Qualitaetsproblem:** Der Ghostscript-Merge-Schritt (Fonts/Vektoren neu
einbetten) kann **sichtbare Qualitaetsunterschiede** im Vergleich zum
nativen GUI-Export erzeugen.

**GUI-Vorteil:** Die KiCad-GUI kennt ueber den Page Navigator alle Seiten
des Projekts und kann sie in einem **einzigen nativen Plot-Durchgang**
ohne Zwischenschritt ausgeben -- daher das sauberere Ergebnis.

**Vergleich:**

| Methode | Alle Seiten | Qualitaet | INTERSHEET_REFS |
|---------|-------------|-----------|----------------|
| kicad-cli (direkt) | Nein (nur uebergebene Datei) | Nativ | Leer |
| kicad-cli + Ghostscript | Ja (Workaround) | Qualitaetsverlust moeglich | Leer |
| KiCad GUI Plotten | Ja | Nativ, beste Qualitaet | Korrekt aufgeloest |

---

## Zusammenfassung

| # | Problem | Schweregrad | KiCad-Aequivalent vorhanden? | GitLab Issue |
|---|---------|-------------|------------------------------|--------------|
| 1 | Bauteilbezeichnungen (F1, Q2...) fehlen | KRITISCH | Ja, aber nicht gemappt | [#25189](https://gitlab.com/kicad/code/kicad/-/work_items/25189) |
| 2 | Querverweise (/1.13E...) fehlen | KRITISCH | Nein (kein direktes Aequivalent) | [#25190](https://gitlab.com/kicad/code/kicad/-/work_items/25190) |
| 3 | Titelblock-Variablen nicht uebersetzt | HOCH | Teilweise (KiCad Titelblock-Felder) | [#25191](https://gitlab.com/kicad/code/kicad/-/work_items/25191) |
| 4 | Hilfskontakt-Nummern 95-98 beschaedigt | HOCH | Ja, Encoding-Bug | [#25192](https://gitlab.com/kicad/code/kicad/-/work_items/25192) |
| 5 | M 3~ wird zu 3~~ | MITTEL | Ja, Escape-Bug | [#25193](https://gitlab.com/kicad/code/kicad/-/work_items/25193) |
| 6 | Multi-page Netz-Querverweise fehlen | KRITISCH | Nein (KiCad-Feature fehlt grundlegend) | [#25194](https://gitlab.com/kicad/code/kicad/-/work_items/25194) |
| 7 | kicad-cli expandiert ${INTERSHEET_REFS} nicht | HOCH | Nein (CLI-Bug) | [#25195](https://gitlab.com/kicad/code/kicad/-/work_items/25195) |
| 8 | Doppelter Zeichnungsrahmen nach Import | HOCH | Ja, Workaround: empty.kicad_wks | [#25196](https://gitlab.com/kicad/code/kicad/-/work_items/25196) |
| 9 | Seitenreferenzen ohne Hierarchie (alle auf alle) | HOCH | Nein (grundlegendes KiCad-Designproblem) | [#25197](https://gitlab.com/kicad/code/kicad/-/work_items/25197) |
| 10 | kicad-cli ignoriert flat Top-Level-Sheets | HOCH | Nein (CLI-Architektur-Bug) | [#25198](https://gitlab.com/kicad/code/kicad/-/work_items/25198) |

---

## Geplante KiCad GitLab Issues

### Issue 1 -- Component reference designators (F1, Q2, K1...) missing after Eagle import

**Modul:** eeschema/importers/schematic_eagle_plugin.cpp
**Labels:** Eagle import, ug, eeschema
**GitLab Issue:** [#25189](https://gitlab.com/kicad/code/kicad/-/work_items/25189)

Eagle speichert Bauteilnamen auf Layer 95 (Names). Diese werden beim Import nicht
als Referenz-Designatoren in KiCad-Symbole uebernommen.

---

### Issue 2 -- Cross-references (/%S.%C%R format) not imported from Eagle

**Modul:** eeschema/importers/schematic_eagle_plugin.cpp
**Labels:** Eagle import, enhancement, eeschema
**GitLab Issue:** [#25190](https://gitlab.com/kicad/code/kicad/-/work_items/25190)

Eagles xreflabel/xrefpart Querverweissystem hat kein direktes KiCad-Aequivalent.
Als Workaround sollten diese zumindest als **Textannotationen** am jeweiligen Pin
erhalten bleiben, statt komplett verloren zu gehen.

---

### Issue 3 -- Title block variables (>DATUM, >KUNDE...) not mapped to KiCad title block fields

**Modul:** eeschema/importers/schematic_eagle_plugin.cpp
**Labels:** Eagle import, enhancement, eeschema, 	itle block
**GitLab Issue:** [#25191](https://gitlab.com/kicad/code/kicad/-/work_items/25191)

Eagle-Titelblock-Platzhalter >VARIABLENNAME sollten auf KiCad-Titelblockfelder
gemappt werden:

| Eagle-Variable | KiCad-Titelblockfeld |
|----------------|----------------------|
| >DATUM | Date |
| >ERSTELLER / >BEARBEITET | Comment 1 |
| >ZEICHNUNGS_NR | Rev |
| >FUNKTION / >KUNDE | Title / Company |

---

### Issue 4 -- Pin numbers encoded as ASCII offset, garbled characters instead of 95-98

**Modul:** eeschema/importers/schematic_eagle_plugin.cpp
**Labels:** Eagle import, ug, eeschema
**GitLab Issue:** [#25192](https://gitlab.com/kicad/code/kicad/-/work_items/25192)

IEC-Kontaktnummern 95-98 (Motorschutzschalter-Hilfskontakte) werden als
ASCII-Zeichen :;<=>?@ ausgegeben. Vermutlicher Encoding-Bug: numerische
Pin-Bezeichner werden faelschlicherweise um 48 oder 32 verschoben.

---

### Issue 5 -- AC tilde notation doubled: 3~ becomes 3~~ in motor symbol

**Modul:** eeschema/importers/schematic_eagle_plugin.cpp
**Labels:** Eagle import, ug, eeschema
**GitLab Issue:** [#25193](https://gitlab.com/kicad/code/kicad/-/work_items/25193)

Das Eagle-Literal ~ im Motor-Symbol (M 3~) wird nicht escaped, obwohl ~ in
KiCad als Overbar-Sonderzeichen verwendet wird. Ergebnis: 3~~ statt 3~.

---

### Issue 6 -- Multi-page net cross-references (L1/1.18A, N/2.1F...) not imported and not supported

**Modul:** eeschema/importers/schematic_eagle_plugin.cpp + eeschema (Feature Request)
**Labels:** Eagle import, enhancement, eeschema, multi-sheet
**GitLab Issue:** [#25194](https://gitlab.com/kicad/code/kicad/-/work_items/25194)

Eagle erzeugt bei mehrseitigen Plaenen fuer jedes seitenuebergreifende Netz automatisch
Kommentare der Form NETZNAME/BLATT.SPALTEREIHE direkt am Drahtende.
Diese sind nach **DIN EN 61082 / IEC 61082** fuer Stromlaufplaene normativ gefordert.

**Konkret fehlende Querverweise** (aus sample2/, 2-blaettriger Stern-Dreieck-Plan):
```
L1/1.18A  L2/1.18A  L3/1.18A   (Blatt 1 verweist auf Blatt 2)
L1/2.2A   L2/2.2A   L3/2.2A    (Blatt 2 verweist auf Blatt 1)
N/1.18F   PE/1.18F              (N/PE Blatt 1 -> Blatt 2)
N/2.1F    PE/2.1F               (N/PE Blatt 2 -> Blatt 1)
```

Als kurzfristiger Import-Workaround sollten diese als **Text-Annotationen** am
Drahtende erhalten bleiben. Langfristig ist eine **neue KiCad-Funktion** fuer
automatische seitenuebergreifende Netz-Querverweise mit Koordinatenangabe noetig.

---

### Issue 7 -- kicad-cli sch export pdf does not expand ${INTERSHEET_REFS} variable

**Modul:** `kicad-cli` / `eeschema/sch_io/kicad_sexpr/`  
**Labels:** `kicad-cli`, `bug`, `eeschema`, `regression`
**GitLab Issue:** [#25195](https://gitlab.com/kicad/code/kicad/-/work_items/25195)

**Beschreibung:**

KiCad speichert seitenuebergreifende Querverweise als Property `${INTERSHEET_REFS}`
in der `.kicad_sch`. In der GUI werden diese korrekt aufgeloest, sofern im Projekt
`intersheets_ref_show: true` gesetzt und `(hide no)` in den entsprechenden
Property-Bloecken der `.kicad_sch` gesetzt ist.

Der CLI-Exporter expandiert `${INTERSHEET_REFS}` **nicht** -- die Felder bleiben
im exportierten PDF leer.

**Reproduktion:**

```bash
# Voraussetzungen:
# - .kicad_pro:  "intersheets_ref_show": true
# - .kicad_sch:  (property "Intersheetrefs" "" ... (hide no) ...)

kicad-cli sch export pdf --output out.pdf projekt.kicad_sch

# Ergebnis: ${INTERSHEET_REFS}-Felder sind leer im PDF
# Erwartet: Querverweise werden wie in der GUI aufgeloest und gedruckt
```

**Betroffene Workflows:** Alle CI/CD-Pipelines, Makefile-basierte
Dokumentationsgenerierung, automatisierte PDF-Exports.

**Workaround:** PDF ueber die KiCad GUI exportieren (Datei -> Schaltplan plotten -> PDF).
Die GUI loest `${INTERSHEET_REFS}` korrekt auf.

---

### Issue 8 -- Duplicate drawing frame after Eagle import (symbol + KiCad page border)

**Modul:** eeschema/importers/schematic_eagle_plugin.cpp
**Labels:** Eagle import, bug, eeschema
**GitLab Issue:** [#25196](https://gitlab.com/kicad/code/kicad/-/work_items/25196)

**Beschreibung:**

Der Eagle-Importer setzt beim Import gleichzeitig:

1. Das Papierformat (`paper "User" ...`) auf die Masse des Eagle-Zeichnungsblatts
2. Das Eagle-Zeichnungsrahmen-Symbol (z. B. `RAHMEN_A4_8Z-19S`) als normales
   Schaltplanbauteil

Dadurch zeichnet KiCad beim PDF-Export **zwei Rahmen uebereinander**:
- Den KiCad-internen Seitenrahmen (aus dem `paper`-Format)
- Den importierten Eagle-Rahmen (als Symbol)

**Reproduktion:**

```
1. Eagle .sch mit Zeichnungsrahmen-Symbol importieren
2. kicad-cli sch export pdf projekt.kicad_sch
3. Ergebnis: Doppelter Rahmen im PDF sichtbar
```

**Erwartetes Verhalten:** Der Importer sollte entweder:
- Option A: Das `paper`-Format ohne KiCad-Standardrahmen setzen
  (leere `.kicad_wks` referenzieren), oder
- Option B: Das Zeichnungsrahmen-Symbol nicht importieren und stattdessen
  den KiCad-Titelblock befuellen

**Workaround (verifiziert):** Leere `.kicad_wks`-Datei anlegen und im Projekt eintragen.

Datei `empty.kicad_wks` (4 Zeilen, kein Rahmen, kein Titelblock):

```
(kicad_wks
	(version 20210606)
	(generator "pl_editor")
)
```

In der `.kicad_pro` unter `"schematic"` eintragen:

```json
"page_layout_descr_file": "empty.kicad_wks"
```

Ergebnis: KiCad-Standardrahmen verschwindet **sowohl in der GUI als auch beim
kicad-cli PDF-Export** -- nur der importierte Eagle-Rahmen bleibt sichtbar.

**Alternativer Workaround (nur CLI):** `--exclude-drawing-sheet` Flag:

```bash
kicad-cli sch export pdf --exclude-drawing-sheet --output out.pdf projekt.kicad_sch
```

---

### Issue 9 -- Inter-sheet net references show all pages instead of only connected pages

**Modul:** eeschema / net navigation
**Labels:** eeschema, enhancement, multi-sheet
**GitLab Issue:** [#25197](https://gitlab.com/kicad/code/kicad/-/work_items/25197)

**Beschreibung:**

KiCad's ${INTERSHEET_REFS} zeigt fuer jeden Net-Label **alle anderen Seiten**
auf denen ein Label mit demselben Netzname existiert. Es wird keine Unterscheidung
getroffen ob das Netz auf der jeweiligen Seite tatsaechlich ein- oder austritt
(Drahtende) oder ob es sich nur um ein weiteres Label desselben Netzes handelt.

**Reproduktion:**

Mehrseitiger Schaltplan mit einem Netz (z. B. PE, N, L1) das auf allen Seiten
als Label vorhanden ist:

- Seite 1 zeigt: `2 3 4 5 6`
- Seite 2 zeigt: `1 3 4 5 6`
- usw.

**Erwartetes Verhalten (Eagle-Vorbild):**

Querverweise zeigen ausschliesslich Seiten mit **direkter Drahtverbindung**,
vollstaendig im Format `NETZNAME/BLATT.SPALTEREIHE` (z. B. `L1/3.4B`).

**Auswirkung:** Bei grossen Plaenen (6+ Seiten) erzeugt das aktuelle Verhalten
Informationsflut statt nuetzlicher Navigation und verletzt die Anforderungen
der **DIN EN 61082 / IEC 61082** fuer normgerechte Stromlaufplaene.

**Vorgeschlagene Loesung:**
- Topologische Auswertung: Nur Seiten anzeigen wo das Netz physisch
  ein- oder austritt (Drahtende an Seite)
- Optional: Koordinate (Spalte/Reihe) zusaetzlich zur Seitennummer anzeigen

---

### Issue 10 -- kicad-cli sch export pdf ignoriert flat Top-Level-Sheets in mehrseitigen Projekten

**Modul:** `kicad-cli` / `eeschema`  
**Labels:** `kicad-cli`, `enhancement`, `eeschema`, `multi-sheet`
**GitLab Issue:** [#25198](https://gitlab.com/kicad/code/kicad/-/work_items/25198)

**Beschreibung:**

Beim Aufruf von `kicad-cli sch export pdf --output out.pdf projekt_seite1.kicad_sch`
wird ausschliesslich die uebergebene Schaltplandatei exportiert. Wenn ein Projekt
aus mehreren unabhaengigen ("flat") Top-Level-Sheets besteht -- wie beim Import aus
Eagle ueblich -- werden alle weiteren Dateien des Projekts ignoriert.

**Reproduktion:**

1. Mehrseitiges Eagle-Projekt importieren (z. B. 2 Seiten: `seite1.kicad_sch` und `seite2.kicad_sch`)
2. `kicad-cli sch export pdf --output out.pdf seite1.kicad_sch`
3. Ergebnis: `out.pdf` enthaelt nur Seite 1.

**Erwartetes Verhalten:**

`kicad-cli` sollte -- analog zum Page Navigator in der KiCad GUI -- alle zum Projekt
gehoerenden Schaltplaene erkennen und in ein einziges mehrseitiges PDF exportieren.

**Workaround & Nachteile:**

Derzeit muss jede Seite einzeln via `kicad-cli` exportiert und mit externen Tools
(z. B. Ghostscript / `pdfunite`) zusammengefuegt werden. Dies fuehrt zu:
- Potentiellen Qualitaetsverlusten durch erneutes Re-Embedding von Schriften/Vektoren
- Leeren/unvollstaendigen `${INTERSHEET_REFS}`-Variablen

---

## Hinweis: Querverweise als Grundfunktion in der Elektrotechnik

Eagles Querverweissystem (%F%N/%S.%C%R) ist fuer industrielle Stromlaufplaene
nach **DIN EN 61082** fundamental. Es gibt zwei Arten, die beide fehlen:

1. **Bauteil-Querverweise** (aufgeteilte Symbole): Schuetz-Spule auf Blatt 1,
   Hilfskontakte auf Blatt 2 -- mit automatischem Rueckverweis /2.13E am Kontakt.

2. **Netz-Querverweise** (seitenuebergreifende Netze): L1/2.2A am Drahtende
   zeigt: *Netz L1 kommt von Blatt 2, Spalte 2, Reihe A.*

Ohne diese Querverweise ist ein mehrseitiger Schaltplan **nicht normgerecht** und
in der Praxis kaum verwendbar.

---

*Analyse erstellt mit: Python 3.14, pdfminer.six 20260107*
*Repository: [Meisterschulen-am-Ostbahnhof-Munchen/KiCAD_elektro](https://github.com/Meisterschulen-am-Ostbahnhof-Munchen/KiCAD_elektro)*
