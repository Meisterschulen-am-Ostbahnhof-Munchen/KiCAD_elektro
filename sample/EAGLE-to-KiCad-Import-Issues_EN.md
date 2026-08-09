# Eagle -> KiCad Import: Analysis of Conversion Losses

> **Test case:** Star-Delta motor starter schematic (single-page + multi-page)
> **Eagle version:** 7.7.0
> **KiCad version:** 10.0.5
> **Analysis date:** 2026-08-09
> **Analysis method:** PDF text comparison (pdfminer) + evaluation of .sch source file

---

## Context

This schematic is a classic **industrial circuit diagram** (main circuit and control
circuit, star-delta motor starter) with the following components:

| Reference | Type | Library |
|-----------|------|---------|
| Q1 | 3-pole load disconnect switch | e-schalter |
| F1 | 3-pole NH fuse 63A | e-sicherungen |
| F2 | 3-pole NH fuse 35A | e-sicherungen |
| F3 | Motor circuit breaker 3-pole 16.3A (with auxiliary contacts) | e-motorschutzschalter |
| F4 | Circuit breaker 10A | e-sicherungen |
| Q2 | Main contactor 3-pole (NO 13-14) | e-schuetze-relais |
| Q3 | Star contactor 3-pole (NO 11-12) | e-schuetze-relais |
| Q4 | Delta contactor 3-pole (NO 11-12 + 13-14) | e-schuetze-relais |
| K1 | Auxiliary contactor, time-delayed opening (NO 15-16) | e-schuetze-relais |
| S1 | NC pushbutton (STOP) | e-schalter |
| S2 | NO pushbutton (START) | e-schalter |
| M1 | 3-phase motor 10 kW, star-delta | e-motoren |
| X1, X2, X3 | Terminal blocks | e-klemmen |
| EINSPEISUNG1 | Supply 3/N/PE | e-klemmen |

**Eagle libraries used (custom electrical engineering special libs):**
e-elektro-zeichnungsrahmen, e-schalter, e-sicherungen, e-motorschutzschalter,
e-schuetze-relais, e-motoren, e-klemmen

---

## Analysis Results: What Is Lost During Import?

### Problem 1 -- Component reference designators disappear (F1, Q2, K1 ...)

**Severity:** CRITICAL

All component reference designators are completely missing from the converted PDF.
In the Eagle PDF, F1, F2, F3, F4, Q1-Q4, K1, S1, S2, M1, X1-X3 are clearly
readable. In the KiCad PDF these names do not appear at all.

**Root cause (suspected):** Reference texts reside on Eagle Layer 95 (Names).
The importer likely does not correctly transfer this layer information to
KiCad text fields.

---

### Problem 2 -- Cross-references (/1.13E, /1.16D ...) are lost

**Severity:** CRITICAL

Eagle automatically generates **cross-references** in the format `/%S.%C%R`
(sheet.column+row) for multi-page schematics and for split symbols (e.g. contactor
coil on sheet A, auxiliary contacts on sheet B).

Missing from the converted test schematic:

```
/1.13B   /1.13D   /1.13E
/1.14E   /1.16D   /1.16E
/1.17D   /1.17E   /1.4D
/1.4E    /1.7D    /1.9D
```

Eagle stores the cross-reference format in the schematic root:
```xml
<schematic xreflabel="%F%N/%S.%C%R" xrefpart="/%S.%C%R">
```

KiCad has no direct equivalent for this automatic cross-reference system.
The information is simply ignored during import.

---

### Problem 3 -- Title block variables (>DATUM, >KUNDE ...) are not translated

**Severity:** HIGH

Eagle uses placeholders of the form `>VARIABLENAME` in title block symbols,
replaced at runtime by attribute values. In the test schematic for example:

| Eagle Variable | Value in test schematic |
|----------------|------------------------|
| >FUNKTION | Foerderband |
| >HERSTELLER | Siemens |
| >DATUM | (not set) |
| >KUNDE | (not set) |
| >AUFTRAGS_NR | (not set) |
| >ERSTELLER | (not set) |
| >WERKS_NR | (not set) |
| >ZEICHNUNGS_NR | (not set) |

In the converted KiCad schematic the literal strings >DATUM, >BEARBEITET etc.
appear as raw strings instead of containing the set values or being mapped
to KiCad title block fields.

---

### Problem 4 -- Auxiliary contact numbers (95/96/97/98) garbled

**Severity:** HIGH

Eagle symbols for motor circuit breakers use the IEC-standardised contact
designations 95, 96 (NC), 97, 98 (NO) for the thermal overload relay.
After conversion these numbers no longer appear correctly -- instead
ASCII characters (`:`, `;`, `<`, `=`, `>`, `?`, `@`) appear, indicating an
**encoding bug in the symbol mapping** (ASCII offset of 48).

---

### Problem 5 -- Motor symbol M 3~ becomes 3~~

**Severity:** MEDIUM

The three-phase AC symbol `3~` is rendered in KiCad as `3~~`.
The tilde (`~`) is an overbar marker in KiCad; during import the Eagle
literal `~` is not escaped, so it is interpreted as a special character
instead of plain text.

---

### Problem 6 -- Filename / drawing number from title block missing

**Severity:** MEDIUM

The filename `stern-dreieck-anlauf` (from Eagle attribute) and the drawing
number do not appear in the KiCad title block. Instead KiCad generates a
generic entry with empty `Title:`, `Date:`, `Rev:`.

---

### Problem 7 -- L3 + N are concatenated to "L3 N"

**Severity:** MEDIUM

In the Eagle schematic `L3` and `N` are separate nets/labels. In the KiCad
output `3 N` appears as a combined string -- a parser issue with closely
spaced net labels or pins.

---

### Problem 8 -- Multi-page net cross-references missing entirely

**Severity:** CRITICAL

For multi-page schematics Eagle automatically generates **inter-sheet net
cross-references** for every net that appears on more than one sheet.
Format: `NETNAME/SHEET.COLUMNROW` -- e.g.:

| Eagle cross-reference | Meaning |
|-----------------------|---------|
| L1/1.18A | Net L1, continues on sheet 1, column 18, row A |
| L1/2.2A | Net L1, comes from sheet 2, column 2, row A |
| N/1.18F | Net N, continues on sheet 1, position 18F |
| PE/2.1F | Net PE, comes from sheet 2, position 1F |

**Evidence from the multi-page test schematic** (sample2/, 2 sheets):

Present in Eagle original, completely absent in KiCad:
```
L1/1.18A   L1/2.2A
L2/1.18A   L2/2.2A
L3/1.18A   L3/2.2A
N/1.18F    N/2.1F
PE/1.18F   PE/2.1F
```

These annotations are **normatively required** for industrial circuit diagrams
per **DIN EN 61082 / IEC 61082**. Without them a multi-page schematic is
**not readable** for an electrical engineer -- they cannot determine where a
net comes from or where it goes.

**Root cause:** KiCad has Net Labels and Hierarchical Labels/Ports -- but no
automatic cross-reference system that displays sheet number + coordinate.
This is a **missing KiCad feature**, not just an import bug.

**Verified (2026-08-09):** KiCad does show the **sheet number** of the
referenced sheet in the GUI (e.g. "2"), but **not** the column/row coordinate.
Eagle shows the full format `NETNAME/SHEET.COLUMNROW` (e.g. `L1/2.2A`).
KiCad's equivalent is therefore only a partial result -- not sufficient for
standards-compliant circuit diagrams per DIN EN 61082.

---

### Problem 9 -- kicad-cli does not expand ${INTERSHEET_REFS}

**Severity:** HIGH | **Status: VERIFIED**

KiCad stores inter-sheet cross-references as property `${INTERSHEET_REFS}`
in the `.kicad_sch` file. In the KiCad GUI these are correctly resolved and
displayed (provided `intersheets_ref_show: true` is set in the project).

The **CLI exporter** (`kicad-cli sch export pdf`) does **not** expand
`${INTERSHEET_REFS}` -- the fields remain empty. Verified with:

```bash
# intersheets_ref_show: true in .kicad_pro
# (hide no) in all Intersheetrefs blocks of .kicad_sch
kicad-cli sch export pdf stern-dreieck-anlauf_multipage.kicad_sch
# Result: ${INTERSHEET_REFS} = empty in PDF
```

This affects **all** workflows using kicad-cli for automated PDF generation
(CI/CD pipelines, Makefile-based documentation, automated exports).

**Workaround:** Export PDF via the KiCad GUI (File -> Plot -> PDF).
The GUI resolves `${INTERSHEET_REFS}` correctly.

---

### Problem 10 -- Duplicate drawing frame after Eagle import

**Severity:** HIGH

The Eagle importer simultaneously sets:

1. The paper format (`paper "User" ...`) to the dimensions of the Eagle drawing sheet
2. The Eagle drawing frame symbol (e.g. `RAHMEN_A4_8Z-19S`) as a normal schematic component

As a result KiCad renders **two overlapping frames** in the PDF export:
- The KiCad internal page border (from the `paper` format)
- The imported Eagle frame symbol

**Root cause:** The importer should either:
- **Option A:** Set the `paper` format without the KiCad default border
  (by referencing an empty `.kicad_wks`), or
- **Option B:** Not import the drawing frame symbol and instead populate
  the KiCad title block

**Workaround (verified):** Create an empty `.kicad_wks` file and reference
it in the project.

File `empty.kicad_wks` (4 lines, no frame, no title block):

```
(kicad_wks
	(version 20210606)
	(generator "pl_editor")
)
```

Add to `.kicad_pro` under `"schematic"`:

```json
"page_layout_descr_file": "empty.kicad_wks"
```

Result: The KiCad default border disappears **both in the GUI and in the
kicad-cli PDF export** -- only the imported Eagle frame remains visible.

**Alternative workaround (CLI only):** `--exclude-drawing-sheet` flag:

```bash
kicad-cli sch export pdf --exclude-drawing-sheet --output out.pdf project.kicad_sch
```

---

### Problem 11 -- Inter-sheet references without hierarchy: every page references all others

**Severity:** HIGH | **Status: VERIFIED**

In a multi-page schematic where a net appears on multiple sheets, KiCad's
`${INTERSHEET_REFS}` shows **every other sheet** on which that net has a label --
without filtering, without hierarchy, without any coordinate information.

**Observed behaviour** (6-sheet schematic, net appears on all sheets):

| Sheet | KiCad shows | Eagle would show |
|-------|-------------|------------------|
| 1 | 2 3 4 5 6 | only sheets with a direct connection + coordinate |
| 2 | 1 3 4 5 6 | only sheets with a direct connection + coordinate |
| 3 | 1 2 4 5 6 | only sheets with a direct connection + coordinate |

In practice this creates **information overload** rather than useful navigation:
a net such as PE or N that appears on all 6 sheets generates a reference
from every sheet to all other 5 -- the engineer cannot determine
**where** the net actually goes.

**Eagle behaviour (correct):** Cross-references show only the locations where
the net actually enters or exits a sheet (wire end at page boundary),
with full coordinates (`NETNAME/SHEET.COLUMNROW`).

**Root cause:** KiCad's `${INTERSHEET_REFS}` performs a simple name search
across all sheets -- no topological routing, no distinction between
"net passes through this sheet" and "net terminates/originates here".

---

### Problem 12 -- kicad-cli only exports from the given file, ignoring flat top-level sheets

**Severity:** HIGH | **Status: VERIFIED**

`kicad-cli sch export pdf` exports **only the sheet hierarchy rooted at the
given file**. For projects with multiple independent ("flat") top-level sheets
-- as produced by the Eagle importer -- only the one passed file is exported;
all other sheets are **silently ignored**.

**Impact for multi-page Eagle imports:**

Since Eagle stores all sheets as equal top-level sheets (no hierarchy root
as in native KiCad projects), each sheet must be exported individually
and then merged:

```bash
# Required workaround: each sheet separately, then merge with Ghostscript
kicad-cli sch export pdf --output page1.pdf page1.kicad_sch
kicad-cli sch export pdf --output page2.pdf page2.kicad_sch
gs -dBATCH -dNOPAUSE -sDEVICE=pdfwrite -sOutputFile=combined.pdf page1.pdf page2.pdf
```

**Quality issue:** The Ghostscript merge step (re-embedding fonts/vectors)
can introduce **visible quality differences** compared to a native GUI export.

**GUI advantage:** The KiCad GUI knows all project sheets via the Page Navigator
and can plot them in a **single native pass** without any intermediate step --
hence the cleaner result.

**Comparison:**

| Method | All sheets | Quality | INTERSHEET_REFS |
|--------|------------|---------|----------------|
| kicad-cli (direct) | No (given file only) | Native | Empty |
| kicad-cli + Ghostscript | Yes (workaround) | Possible quality loss | Empty |
| KiCad GUI Plot | Yes | Native, best quality | Correctly resolved |

---

## Summary

| # | Problem | Severity | KiCad equivalent available? |
|---|---------|----------|-----------------------------|
| 1 | Component reference designators (F1, Q2...) missing | CRITICAL | Yes, but not mapped |
| 2 | Cross-references (/1.13E...) missing | CRITICAL | No (no direct equivalent) |
| 3 | Title block variables not translated | HIGH | Partial (KiCad title block fields) |
| 4 | Auxiliary contact numbers 95-98 garbled | HIGH | Yes, encoding bug |
| 5 | M 3~ becomes 3~~ | MEDIUM | Yes, escape bug |
| 6 | Filename/drawing number missing | MEDIUM | Yes (KiCad title block fields) |
| 7 | Net label concatenation L3+N | MEDIUM | Yes, parser bug |
| 8 | Multi-page net cross-references missing | CRITICAL | No (missing KiCad feature) |
| 9 | kicad-cli does not expand ${INTERSHEET_REFS} | HIGH | No (CLI bug) |
| 10 | Duplicate drawing frame after import | HIGH | Yes, workaround: empty.kicad_wks |
| 11 | Inter-sheet refs without hierarchy (all-to-all) | HIGH | No (fundamental KiCad design issue) |
| 12 | kicad-cli ignores flat top-level sheets, Ghostscript workaround needed | HIGH | No (CLI architecture bug) |

---

## Planned KiCad GitLab Issues

### Issue 1 -- Component reference designators (F1, Q2, K1...) missing after Eagle import

**Module:** `eeschema/importers/schematic_eagle_plugin.cpp`
**Labels:** `Eagle import`, `bug`, `eeschema`

Eagle stores component names on Layer 95 (Names). These are not transferred
as reference designators to KiCad symbols during import.

---

### Issue 2 -- Cross-references (/%S.%C%R format) not imported from Eagle

**Module:** `eeschema/importers/schematic_eagle_plugin.cpp`
**Labels:** `Eagle import`, `enhancement`, `eeschema`

Eagle's xreflabel/xrefpart cross-reference system has no direct KiCad equivalent.
As a workaround these should at minimum be preserved as **text annotations**
at the respective pin instead of being lost entirely.

---

### Issue 3 -- Title block variables (>DATUM, >KUNDE...) not mapped to KiCad title block fields

**Module:** `eeschema/importers/schematic_eagle_plugin.cpp`
**Labels:** `Eagle import`, `enhancement`, `eeschema`, `title block`

Eagle title block placeholders `>VARIABLENAME` should be mapped to KiCad
title block fields:

| Eagle Variable | KiCad Title Block Field |
|----------------|------------------------|
| >DATUM | Date |
| >ERSTELLER / >BEARBEITET | Comment 1 |
| >ZEICHNUNGS_NR | Rev |
| >FUNKTION / >KUNDE | Title / Company |

---

### Issue 4 -- Pin numbers encoded as ASCII offset, garbled characters instead of 95-98

**Module:** `eeschema/importers/schematic_eagle_plugin.cpp`
**Labels:** `Eagle import`, `bug`, `eeschema`

IEC contact numbers 95-98 (motor circuit breaker auxiliary contacts) are
rendered as ASCII characters `:;<=>?@`. Suspected encoding bug: numeric pin
designators are incorrectly shifted by 48 or 32.

---

### Issue 5 -- AC tilde notation doubled: 3~ becomes 3~~ in motor symbol

**Module:** `eeschema/importers/schematic_eagle_plugin.cpp`
**Labels:** `Eagle import`, `bug`, `eeschema`

The Eagle literal `~` in the motor symbol (`M 3~`) is not escaped, even though
`~` is used as an overbar special character in KiCad. Result: `3~~` instead of `3~`.

---

### Issue 6 -- Multi-page net cross-references (L1/1.18A, N/2.1F...) not imported and not supported

**Module:** `eeschema/importers/schematic_eagle_plugin.cpp` + `eeschema` (Feature Request)
**Labels:** `Eagle import`, `enhancement`, `eeschema`, `multi-sheet`

Eagle automatically generates annotations of the form `NETNAME/SHEET.COLUMNROW`
at every wire end for each net that crosses sheets.
These are **normatively required** per **DIN EN 61082 / IEC 61082**.

**Specific missing cross-references** (from sample2/, 2-sheet star-delta schematic):
```
L1/1.18A  L2/1.18A  L3/1.18A   (sheet 1 references sheet 2)
L1/2.2A   L2/2.2A   L3/2.2A    (sheet 2 references sheet 1)
N/1.18F   PE/1.18F              (N/PE sheet 1 -> sheet 2)
N/2.1F    PE/2.1F               (N/PE sheet 2 -> sheet 1)
```

As a short-term import workaround these should be preserved as **text annotations**
at the wire end. Long-term, a **new KiCad feature** for automatic inter-sheet net
cross-references with coordinate display is needed.

---

### Issue 7 -- kicad-cli sch export pdf does not expand ${INTERSHEET_REFS} variable

**Module:** `kicad-cli` / `eeschema/sch_io/kicad_sexpr/`
**Labels:** `kicad-cli`, `bug`, `eeschema`, `regression`

**Description:**

KiCad stores inter-sheet cross-references as property `${INTERSHEET_REFS}`
in the `.kicad_sch`. In the GUI these are correctly resolved, provided
`intersheets_ref_show: true` is set in the project and `(hide no)` is set
in the corresponding property blocks of the `.kicad_sch`.

The CLI exporter does **not** expand `${INTERSHEET_REFS}` -- the fields
remain empty in the exported PDF.

**Reproduction:**

```bash
# Prerequisites:
# - .kicad_pro:  "intersheets_ref_show": true
# - .kicad_sch:  (property "Intersheetrefs" "" ... (hide no) ...)

kicad-cli sch export pdf --output out.pdf project.kicad_sch

# Result: ${INTERSHEET_REFS} fields are empty in the PDF
# Expected: cross-references are resolved and printed as in the GUI
```

**Affected workflows:** All CI/CD pipelines, Makefile-based documentation
generation, automated PDF exports.

**Workaround:** Export PDF via the KiCad GUI (File -> Plot -> PDF).
The GUI resolves `${INTERSHEET_REFS}` correctly.

---

### Issue 8 -- Duplicate drawing frame after Eagle import (symbol + KiCad page border)

**Module:** `eeschema/importers/schematic_eagle_plugin.cpp`
**Labels:** `Eagle import`, `bug`, `eeschema`

**Description:**

The Eagle importer simultaneously:

1. Sets the paper format (`paper "User" ...`) to the dimensions of the Eagle drawing sheet
2. Places the Eagle drawing frame symbol (e.g. `RAHMEN_A4_8Z-19S`) as a normal
   schematic component

As a result KiCad renders **two overlapping frames** in the PDF export:
- The KiCad internal page border (from the `paper` format)
- The imported Eagle frame (as a symbol)

**Reproduction:**

```
1. Import an Eagle .sch with a drawing frame symbol
2. kicad-cli sch export pdf project.kicad_sch
3. Result: duplicate frame visible in PDF
```

**Expected behaviour:** The importer should either:
- Option A: Set the `paper` format without the KiCad default border
  (reference an empty `.kicad_wks`), or
- Option B: Not import the drawing frame symbol and instead populate
  the KiCad title block

**Workaround (verified):** Create an empty `.kicad_wks` file and reference
it in the project under `"schematic"` -> `"page_layout_descr_file"`.
This suppresses the KiCad default border **both in the GUI and in the
kicad-cli PDF export**.

**Alternative workaround (CLI only):** `--exclude-drawing-sheet` flag:

```bash
kicad-cli sch export pdf --exclude-drawing-sheet --output out.pdf project.kicad_sch
```

---

### Issue 9 -- Inter-sheet net references show all pages instead of only connected pages

**Module:** `eeschema` / net navigation
**Labels:** `eeschema`, `enhancement`, `multi-sheet`

**Description:**

KiCad's `${INTERSHEET_REFS}` shows **all other sheets** on which a label with
the same net name exists, regardless of whether the net actually enters or
exits (wire end) that sheet or merely has another label there.

**Reproduction:**

Multi-page schematic with a net (e.g. PE, N, L1) that has labels on all sheets:

- Sheet 1 shows: `2 3 4 5 6`
- Sheet 2 shows: `1 3 4 5 6`
- etc.

**Expected behaviour (Eagle model):**

Cross-references show only sheets with a **direct wire connection**,
fully formatted as `NETNAME/SHEET.COLUMNROW` (e.g. `L1/3.4B`).

**Impact:** On large schematics (6+ sheets) the current behaviour creates
information overload instead of useful navigation and violates the requirements
of **DIN EN 61082 / IEC 61082** for standards-compliant circuit diagrams.

**Proposed fix:**
- Topological evaluation: only show sheets where the net physically
  enters or exits (wire end at sheet boundary)
- Optionally: display column/row coordinate in addition to the sheet number

---

### Issue 10 -- kicad-cli sch export pdf ignores flat top-level sheets in project

**Module:** `kicad-cli` / `eeschema`  
**Labels:** `kicad-cli`, `enhancement`, `eeschema`, `multi-sheet`

**Description:**

When running `kicad-cli sch export pdf --output out.pdf project_page1.kicad_sch`,
only the specified schematic file is exported. If a project consists of multiple
independent ("flat") top-level sheets -- as is standard when imported from Eagle --
all other sheets in the project are silently ignored.

**Reproduction:**

1. Import a multi-page Eagle project (e.g. 2 sheets: `page1.kicad_sch` and `page2.kicad_sch`)
2. Run `kicad-cli sch export pdf --output out.pdf page1.kicad_sch`
3. Result: `out.pdf` contains only page 1.

**Expected behaviour:**

`kicad-cli` should -- similar to the Page Navigator in the KiCad GUI -- recognize all
schematics belonging to the project and plot them into a single multi-page PDF.

**Workaround & Drawbacks:**

Currently, each sheet must be exported individually via `kicad-cli` and merged using
external tools (e.g. Ghostscript / `pdfunite`). This leads to:
- Potential quality degradation due to font/vector re-embedding
- Unexpanded `${INTERSHEET_REFS}` variables

---

## Note: Cross-References as a Core Function in Electrical Engineering

Eagle's cross-reference system (`%F%N/%S.%C%R`) is fundamental for industrial
circuit diagrams per **DIN EN 61082**. There are two types, both of which are missing:

1. **Component cross-references** (split symbols): Contactor coil on sheet 1,
   auxiliary contacts on sheet 2 -- with automatic back-reference `/2.13E` at the contact.

2. **Net cross-references** (inter-sheet nets): `L1/2.2A` at a wire end means:
   *Net L1 comes from sheet 2, column 2, row A.*

Without these cross-references a multi-page schematic is **not standards-compliant**
and barely usable in practice.

---

*Analysis created with: Python 3.14, pdfminer.six 20260107*
*Repository: [Meisterschulen-am-Ostbahnhof-Munchen/KiCAD_elektro](https://github.com/Meisterschulen-am-Ostbahnhof-Munchen/KiCAD_elektro)*
