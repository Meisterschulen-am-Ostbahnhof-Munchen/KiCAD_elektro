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
-elektro-zeichnungsrahmen, -schalter, -sicherungen, -motorschutzschalter,
-schuetze-relais, -motoren, -klemmen

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

`
/1.13B   /1.13D   /1.13E
/1.14E   /1.16D   /1.16E
/1.17D   /1.17E   /1.4D
/1.4E    /1.7D    /1.9D
`

Eagle speichert das Querverweisformat in der Schematicwurzel:
`xml
<schematic xreflabel="%F%N/%S.%C%R" xrefpart="/%S.%C%R">
`

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
`
L1/1.18A   L1/2.2A
L2/1.18A   L2/2.2A
L3/1.18A   L3/2.2A
N/1.18F    N/2.1F
PE/1.18F   PE/2.1F
`

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

## Zusammenfassung

| # | Problem | Schweregrad | KiCad-Aequivalent vorhanden? |
|---|---------|-------------|------------------------------|
| 1 | Bauteilbezeichnungen (F1, Q2...) fehlen | KRITISCH | Ja, aber nicht gemappt |
| 2 | Querverweise (/1.13E...) fehlen | KRITISCH | Nein (kein direktes Aequivalent) |
| 3 | Titelblock-Variablen nicht uebersetzt | HOCH | Teilweise (KiCad Titelblock-Felder) |
| 4 | Hilfskontakt-Nummern 95-98 beschaedigt | HOCH | Ja, Encoding-Bug |
| 5 | M 3~ wird zu 3~~ | MITTEL | Ja, Escape-Bug |
| 6 | Dateiname/Zeichnungs-Nr. fehlt | MITTEL | Ja (KiCad Titelblockfelder) |
| 7 | Netz-Label-Konkatenierung L3+N | MITTEL | Ja, Parser-Bug |
| 8 | Seitenuebergreifende Netz-Querverweise fehlen | KRITISCH | Nein (KiCad-Feature fehlt grundlegend) |
| 9 | kicad-cli expandiert ${INTERSHEET_REFS} nicht | HOCH | Nein (CLI-Bug) |
| 10 | Doppelter Zeichnungsrahmen nach Import | HOCH | Ja, Workaround: --exclude-drawing-sheet |

---

## Geplante KiCad GitLab Issues

### Issue 1 -- Component reference designators (F1, Q2, K1...) missing after Eagle import

**Modul:** eschema/importers/schematic_eagle_plugin.cpp
**Labels:** Eagle import, ug, eschema

Eagle speichert Bauteilnamen auf Layer 95 (Names). Diese werden beim Import nicht
als Referenz-Designatoren in KiCad-Symbole uebernommen.

---

### Issue 2 -- Cross-references (/%S.%C%R format) not imported from Eagle

**Modul:** eschema/importers/schematic_eagle_plugin.cpp
**Labels:** Eagle import, nhancement, eschema

Eagles xreflabel/xrefpart Querverweissystem hat kein direktes KiCad-Aequivalent.
Als Workaround sollten diese zumindest als **Textannotationen** am jeweiligen Pin
erhalten bleiben, statt komplett verloren zu gehen.

---

### Issue 3 -- Title block variables (>DATUM, >KUNDE...) not mapped to KiCad title block fields

**Modul:** eschema/importers/schematic_eagle_plugin.cpp
**Labels:** Eagle import, nhancement, eschema, 	itle block

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

**Modul:** eschema/importers/schematic_eagle_plugin.cpp
**Labels:** Eagle import, ug, eschema

IEC-Kontaktnummern 95-98 (Motorschutzschalter-Hilfskontakte) werden als
ASCII-Zeichen :;<=>?@ ausgegeben. Vermutlicher Encoding-Bug: numerische
Pin-Bezeichner werden faelschlicherweise um 48 oder 32 verschoben.

---

### Issue 5 -- AC tilde notation doubled: 3~ becomes 3~~ in motor symbol

**Modul:** eschema/importers/schematic_eagle_plugin.cpp
**Labels:** Eagle import, ug, eschema

Das Eagle-Literal ~ im Motor-Symbol (M 3~) wird nicht escaped, obwohl ~ in
KiCad als Overbar-Sonderzeichen verwendet wird. Ergebnis: 3~~ statt 3~.

---

### Issue 6 -- Multi-page net cross-references (L1/1.18A, N/2.1F...) not imported and not supported

**Modul:** eschema/importers/schematic_eagle_plugin.cpp + eschema (Feature Request)
**Labels:** Eagle import, nhancement, eschema, multi-sheet

Eagle erzeugt bei mehrseitigen Plaenen fuer jedes seitenuebergreifende Netz automatisch
Kommentare der Form NETZNAME/BLATT.SPALTEREIHE direkt am Drahtende.
Diese sind nach **DIN EN 61082 / IEC 61082** fuer Stromlaufplaene normativ gefordert.

**Konkret fehlende Querverweise** (aus sample2/, 2-blaettriger Stern-Dreieck-Plan):
`
L1/1.18A  L2/1.18A  L3/1.18A   (Blatt 1 verweist auf Blatt 2)
L1/2.2A   L2/2.2A   L3/2.2A    (Blatt 2 verweist auf Blatt 1)
N/1.18F   PE/1.18F              (N/PE Blatt 1 -> Blatt 2)
N/2.1F    PE/2.1F               (N/PE Blatt 2 -> Blatt 1)
`

Als kurzfristiger Import-Workaround sollten diese als **Text-Annotationen** am
Drahtende erhalten bleiben. Langfristig ist eine **neue KiCad-Funktion** fuer
automatische seitenuebergreifende Netz-Querverweise mit Koordinatenangabe noetig.

---

### Issue 7 -- kicad-cli sch export pdf does not expand  variable

**Modul:** kicad-cli / eschema/sch_io/kicad_sexpr/
**Labels:** kicad-cli, ug, eschema, 
egression

**Beschreibung:**

KiCad speichert seitenuebergreifende Querverweise als Property ${INTERSHEET_REFS}
in der .kicad_sch. In der GUI werden diese korrekt aufgeloest, sofern im Projekt
intersheets_ref_show: true gesetzt und (hide no) in den entsprechenden
Property-Bloecken der .kicad_sch gesetzt ist.

Der CLI-Exporter expandiert ${INTERSHEET_REFS} **nicht** -- die Felder bleiben
im exportierten PDF leer.

**Reproduktion:**

`ash
# Voraussetzungen:
# - .kicad_pro:  "intersheets_ref_show": true
# - .kicad_sch:  (property "Intersheetrefs" "" ... (hide no) ...)

kicad-cli sch export pdf --output out.pdf projekt.kicad_sch

# Ergebnis: -Felder sind leer im PDF
# Erwartet: Querverweise werden wie in der GUI aufgeloest und gedruckt
`

**Betroffene Workflows:** Alle CI/CD-Pipelines, Makefile-basierte
Dokumentationsgenerierung, automatisierte PDF-Exports.

**Workaround:** Derzeit keiner bekannt fuer reine CLI-Nutzung. GUI-Export
via KiCad-Oberflaeche funktioniert korrekt.

---

### Issue 8 -- Duplicate drawing frame after Eagle import (symbol + KiCad page border)

**Modul:** eeschema/importers/schematic_eagle_plugin.cpp
**Labels:** Eagle import, bug, eeschema

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
