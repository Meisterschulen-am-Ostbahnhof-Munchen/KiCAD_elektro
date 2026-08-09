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

| # | Problem | Severity | KiCad equivalent available? | GitLab Issue |
|---|---------|----------|-----------------------------|--------------|
| 1 | Component reference designators (F1, Q2...) missing | CRITICAL | Yes, but not mapped | [#25189](https://gitlab.com/kicad/code/kicad/-/work_items/25189) |
| 2 | Cross-references (/1.13E...) missing | CRITICAL | No (no direct equivalent) | [#25190](https://gitlab.com/kicad/code/kicad/-/work_items/25190) |
| 3 | Title block variables not translated | HIGH | Partial (KiCad title block fields) | [#25191](https://gitlab.com/kicad/code/kicad/-/work_items/25191) |
| 4 | Auxiliary contact numbers 95-98 garbled | HIGH | Yes, encoding bug | [#25192](https://gitlab.com/kicad/code/kicad/-/work_items/25192) |
| 5 | M 3~ becomes 3~~ | MEDIUM | Yes, escape bug | [#25193](https://gitlab.com/kicad/code/kicad/-/work_items/25193) |
| 6 | Multi-page net cross-references missing | CRITICAL | No (missing KiCad feature) | [#25194](https://gitlab.com/kicad/code/kicad/-/work_items/25194) |
| 7 | kicad-cli does not expand ${INTERSHEET_REFS} | HIGH | No (CLI bug) | [#25195](https://gitlab.com/kicad/code/kicad/-/work_items/25195) |
| 8 | Duplicate drawing frame after import | HIGH | Yes, workaround: empty.kicad_wks | [#25196](https://gitlab.com/kicad/code/kicad/-/work_items/25196) |
| 9 | Inter-sheet refs without hierarchy (all-to-all) | HIGH | No (fundamental KiCad design issue) | [#25197](https://gitlab.com/kicad/code/kicad/-/work_items/25197) |
| 10 | kicad-cli ignores flat top-level sheets, Ghostscript workaround needed | HIGH | No (CLI architecture bug) | [#25198](https://gitlab.com/kicad/code/kicad/-/work_items/25198) |

---

## Planned KiCad GitLab Issues (Ready to Copy)

### Issue 1 -- Component reference designators (F1, Q2, K1...) missing after Eagle import

**Module:** `eeschema/importers/schematic_eagle_plugin.cpp`  
**Labels:** `Eagle import`, `bug`, `eeschema`
**GitLab Issue:** [#25189](https://gitlab.com/kicad/code/kicad/-/work_items/25189)

## Description

When importing an EAGLE schematic (`.sch`) into KiCad, component reference designators (F1, F2, F3, F4, Q1-Q4, K1, S1, S2, M1, X1-X3) are completely missing from the imported schematic symbols.

In EAGLE, these reference designators reside on Layer 95 (Names) and are clearly visible in the schematic and exported PDF. In KiCad, the symbol reference fields are empty or not populated from the EAGLE name layer.

**Expected behavior:** EAGLE component names (Layer 95) should be imported into the KiCad `Reference` field for each symbol.

## Steps to reproduce

1. Open KiCad Schematic Editor.
2. Import an EAGLE schematic containing named components (`File` -> `Import` -> `Non-KiCad Schematic...` -> select EAGLE `.sch`).
3. Inspect the imported symbols in the schematic editor.
4. Notice that component reference designators (F1, Q2, K1, etc.) are missing or empty.

## KiCad Version

```
Application: KiCad x64 on x64

Version: 10.0.5, release build

Libraries:
	wxWidgets 3.3.2 
	FreeType 2.13.3
	HarfBuzz 12.3.0
	FontConfig 2.17.1

Platform: Windows 11 (Erzeugungsversion 26200), 64-bit Edition, 64 bit, Little endian, wxMSW
OpenGL: Intel, Intel(R) Graphics, 4.6.0 - Build 32.0.101.8801

	wxWidgets: 3.3.2 (wchar_t,STL containers)
	Boost: 1.90.0
	OCC: 7.9.2
	Curl: 8.18.0
	ngspice: 46
	Compiler: Visual C++ 1944 without C++ ABI
	KICAD_IPC_API=ON
	KICAD_USE_PCH=OFF

Locale: 
	Lang: de_DE
	Enc: UTF-8
	Num: 1.234,5
	Encoded кΩ丈: D0BACEA9E4B888 (sys), D0BACEA9E4B888 (utf8)
```

---

### Issue 2 -- Cross-references (/%S.%C%R format) not imported from Eagle

**Module:** `eeschema/importers/schematic_eagle_plugin.cpp`  
**Labels:** `Eagle import`, `enhancement`, `eeschema`
**GitLab Issue:** [#25190](https://gitlab.com/kicad/code/kicad/-/work_items/25190)

## Description

EAGLE automatically generates cross-references in the format `/%S.%C%R` (sheet.column+row) for multi-page schematics and split symbols (e.g. contactor coil on sheet 1, auxiliary contacts on sheet 2). EAGLE stores this cross-reference format in the schematic root: `<schematic xreflabel="%F%N/%S.%C%R" xrefpart="/%S.%C%R">`.

During EAGLE schematic import into KiCad, these cross-reference strings (such as `/1.13B`, `/1.13D`, `/1.16D`) are completely ignored and lost.

**Expected behavior:** EAGLE cross-references should at minimum be imported as text annotations near the respective symbol pins so that historical schematic references are preserved.

## Steps to reproduce

1. Open KiCad Schematic Editor.
2. Import an EAGLE schematic (`.sch`) containing split symbols (e.g. contactor coils & contacts) with EAGLE cross-references.
3. Inspect the imported symbols.
4. Notice that cross-reference annotations (e.g. `/1.13B`) are completely missing.

## KiCad Version

```
Application: KiCad x64 on x64

Version: 10.0.5, release build

Libraries:
	wxWidgets 3.3.2 
	FreeType 2.13.3
	HarfBuzz 12.3.0
	FontConfig 2.17.1

Platform: Windows 11 (Erzeugungsversion 26200), 64-bit Edition, 64 bit, Little endian, wxMSW
OpenGL: Intel, Intel(R) Graphics, 4.6.0 - Build 32.0.101.8801

	wxWidgets: 3.3.2 (wchar_t,STL containers)
	Boost: 1.90.0
	OCC: 7.9.2
	Curl: 8.18.0
	ngspice: 46
	Compiler: Visual C++ 1944 without C++ ABI
	KICAD_IPC_API=ON
	KICAD_USE_PCH=OFF

Locale: 
	Lang: de_DE
	Enc: UTF-8
	Num: 1.234,5
	Encoded кΩ丈: D0BACEA9E4B888 (sys), D0BACEA9E4B888 (utf8)
```

---

### Issue 3 -- Title block variables (>DATUM, >KUNDE...) not mapped to KiCad title block fields

**Module:** `eeschema/importers/schematic_eagle_plugin.cpp`  
**Labels:** `Eagle import`, `enhancement`, `eeschema`, `title block`
**GitLab Issue:** [#25191](https://gitlab.com/kicad/code/kicad/-/work_items/25191)

## Description

EAGLE title block symbols use placeholder attributes of the form `>VARIABLENAME` (e.g. `>DATUM`, `>KUNDE`, `>FUNKTION`, `>ZEICHNUNGS_NR`, `>ERSTELLER`). In EAGLE, these placeholders are dynamically replaced by schematic attribute values.

Upon importing into KiCad, these literal strings (`>DATUM`, `>BEARBEITET`, etc.) are left as raw unexpanded text strings rather than being mapped to KiCad title block properties (`Date`, `Revision`, `Title`, `Company`, `Comment 1`).

**Expected behavior:** Common EAGLE title block variables should be mapped to KiCad title block fields:
- `>DATUM` -> `Date`
- `>ZEICHNUNGS_NR` -> `Revision`
- `>FUNKTION` / `>KUNDE` -> `Title` / `Company`
- `>ERSTELLER` / `>BEARBEITET` -> `Comment 1`

## Steps to reproduce

1. Open KiCad Schematic Editor.
2. Import an EAGLE schematic containing a title block symbol with `>VARIABLE` placeholders.
3. Inspect the imported title block.
4. Notice that raw text strings like `>DATUM` or `>BEARBEITET` appear as unexpanded text in the schematic.

## KiCad Version

```
Application: KiCad x64 on x64

Version: 10.0.5, release build

Libraries:
	wxWidgets 3.3.2 
	FreeType 2.13.3
	HarfBuzz 12.3.0
	FontConfig 2.17.1

Platform: Windows 11 (Erzeugungsversion 26200), 64-bit Edition, 64 bit, Little endian, wxMSW
OpenGL: Intel, Intel(R) Graphics, 4.6.0 - Build 32.0.101.8801

	wxWidgets: 3.3.2 (wchar_t,STL containers)
	Boost: 1.90.0
	OCC: 7.9.2
	Curl: 8.18.0
	ngspice: 46
	Compiler: Visual C++ 1944 without C++ ABI
	KICAD_IPC_API=ON
	KICAD_USE_PCH=OFF

Locale: 
	Lang: de_DE
	Enc: UTF-8
	Num: 1.234,5
	Encoded кΩ丈: D0BACEA9E4B888 (sys), D0BACEA9E4B888 (utf8)
```

---

### Issue 4 -- Pin numbers encoded as ASCII offset, garbled characters instead of 95-98

**Module:** `eeschema/importers/schematic_eagle_plugin.cpp`  
**Labels:** `Eagle import`, `bug`, `eeschema`
**GitLab Issue:** [#25192](https://gitlab.com/kicad/code/kicad/-/work_items/25192)

## Description

EAGLE symbols for motor circuit breakers use IEC-standardized contact numbers `95`, `96` (NC) and `97`, `98` (NO) for the thermal overload release mechanism.

After importing into KiCad, these pin numbers are garbled into ASCII characters `:`, `;`, `<`, `=`, `>`, `?`, `@`. This indicates an encoding bug during symbol pin number mapping (ASCII offset of 48).

**Expected behavior:** Pin numbers `95`, `96`, `97`, `98` should be preserved as literal string pin numbers "95", "96", "97", "98".

## Steps to reproduce

1. Open KiCad Schematic Editor.
2. Import an EAGLE schematic containing motor protection switch symbols with contacts 95-98.
3. Inspect the auxiliary contact pin numbers of the motor protection switch.
4. Notice that pin numbers appear as garbled ASCII characters (e.g. `:`, `;`, `<`, `=`).

## KiCad Version

```
Application: KiCad x64 on x64

Version: 10.0.5, release build

Libraries:
	wxWidgets 3.3.2 
	FreeType 2.13.3
	HarfBuzz 12.3.0
	FontConfig 2.17.1

Platform: Windows 11 (Erzeugungsversion 26200), 64-bit Edition, 64 bit, Little endian, wxMSW
OpenGL: Intel, Intel(R) Graphics, 4.6.0 - Build 32.0.101.8801

	wxWidgets: 3.3.2 (wchar_t,STL containers)
	Boost: 1.90.0
	OCC: 7.9.2
	Curl: 8.18.0
	ngspice: 46
	Compiler: Visual C++ 1944 without C++ ABI
	KICAD_IPC_API=ON
	KICAD_USE_PCH=OFF

Locale: 
	Lang: de_DE
	Enc: UTF-8
	Num: 1.234,5
	Encoded кΩ丈: D0BACEA9E4B888 (sys), D0BACEA9E4B888 (utf8)
```

---

### Issue 5 -- AC tilde notation doubled: 3~ becomes 3~~ in motor symbol

**Module:** `eeschema/importers/schematic_eagle_plugin.cpp`  
**Labels:** `Eagle import`, `bug`, `eeschema`
**GitLab Issue:** [#25193](https://gitlab.com/kicad/code/kicad/-/work_items/25193)

## Description

The three-phase AC notation string `3~` in EAGLE motor symbols (`M 3~`) is rendered as `3~~` after importing into KiCad.

Because the tilde (`~`) character is used as an overbar formatting prefix in KiCad text rendering, the unescaped EAGLE literal tilde `~` is duplicated or improperly escaped during conversion, resulting in `3~~`.

**Expected behavior:** The text literal `3~` should be properly escaped during import so that it renders as `3~`.

## Steps to reproduce

1. Open KiCad Schematic Editor.
2. Import an EAGLE schematic containing a 3-phase motor symbol (`M 3~`).
3. Inspect the motor symbol label.
4. Notice the text renders as `3~~` instead of `3~`.

## KiCad Version

```
Application: KiCad x64 on x64

Version: 10.0.5, release build

Libraries:
	wxWidgets 3.3.2 
	FreeType 2.13.3
	HarfBuzz 12.3.0
	FontConfig 2.17.1

Platform: Windows 11 (Erzeugungsversion 26200), 64-bit Edition, 64 bit, Little endian, wxMSW
OpenGL: Intel, Intel(R) Graphics, 4.6.0 - Build 32.0.101.8801

	wxWidgets: 3.3.2 (wchar_t,STL containers)
	Boost: 1.90.0
	OCC: 7.9.2
	Curl: 8.18.0
	ngspice: 46
	Compiler: Visual C++ 1944 without C++ ABI
	KICAD_IPC_API=ON
	KICAD_USE_PCH=OFF

Locale: 
	Lang: de_DE
	Enc: UTF-8
	Num: 1.234,5
	Encoded кΩ丈: D0BACEA9E4B888 (sys), D0BACEA9E4B888 (utf8)
```

---

### Issue 6 -- Multi-page net cross-references (L1/1.18A, N/2.1F...) not imported and not supported

**Module:** `eeschema/importers/schematic_eagle_plugin.cpp` + `eeschema` (Feature Request)  
**Labels:** `Eagle import`, `enhancement`, `eeschema`, `multi-sheet`
**GitLab Issue:** [#25194](https://gitlab.com/kicad/code/kicad/-/work_items/25194)

## Description

For multi-page schematics, EAGLE automatically places inter-sheet net cross-references at wire ends in the format `NETNAME/SHEET.COLUMNROW` (e.g. `L1/1.18A`, `L1/2.2A`, `N/1.18F`, `PE/2.1F`). These cross-references are normatively required by DIN EN 61082 / IEC 61082 for industrial electrical schematics.

Upon importing into KiCad, these inter-sheet net cross-references are completely lost.

**Expected behavior:**
1. Short term: Import EAGLE net cross-reference labels as text annotations at wire ends.
2. Long term: KiCad feature request for automatic inter-sheet net cross-references displaying sheet number and grid coordinates.

## Steps to reproduce

1. Open KiCad Schematic Editor.
2. Import a multi-page EAGLE schematic (`.sch`) containing cross-page nets (e.g. L1, L2, L3, N, PE).
3. Inspect the wire ends crossing between sheet 1 and sheet 2.
4. Notice that cross-reference labels (e.g. `L1/1.18A`) are missing entirely.

## KiCad Version

```
Application: KiCad x64 on x64

Version: 10.0.5, release build

Libraries:
	wxWidgets 3.3.2 
	FreeType 2.13.3
	HarfBuzz 12.3.0
	FontConfig 2.17.1

Platform: Windows 11 (Erzeugungsversion 26200), 64-bit Edition, 64 bit, Little endian, wxMSW
OpenGL: Intel, Intel(R) Graphics, 4.6.0 - Build 32.0.101.8801

	wxWidgets: 3.3.2 (wchar_t,STL containers)
	Boost: 1.90.0
	OCC: 7.9.2
	Curl: 8.18.0
	ngspice: 46
	Compiler: Visual C++ 1944 without C++ ABI
	KICAD_IPC_API=ON
	KICAD_USE_PCH=OFF

Locale: 
	Lang: de_DE
	Enc: UTF-8
	Num: 1.234,5
	Encoded кΩ丈: D0BACEA9E4B888 (sys), D0BACEA9E4B888 (utf8)
```

---

### Issue 7 -- kicad-cli sch export pdf does not expand ${INTERSHEET_REFS} variable

**Module:** `kicad-cli` / `eeschema/sch_io/kicad_sexpr/`  
**Labels:** `kicad-cli`, `bug`, `eeschema`, `regression`
**GitLab Issue:** [#25195](https://gitlab.com/kicad/code/kicad/-/work_items/25195)

## Description

KiCad stores inter-sheet cross-references as property `${INTERSHEET_REFS}` in `.kicad_sch` files. In the KiCad GUI, these variables are correctly expanded and printed when `intersheets_ref_show: true` is set in the project file.

However, the CLI exporter (`kicad-cli sch export pdf`) does **not** expand `${INTERSHEET_REFS}` -- the fields remain blank in the exported PDF output.

**Expected behavior:** `kicad-cli sch export pdf` should expand `${INTERSHEET_REFS}` variables identically to GUI PDF export.

## Steps to reproduce

1. Prepare a multi-sheet project with `intersheets_ref_show: true` in `.kicad_pro` and `(hide no)` on `Intersheetrefs` properties in `.kicad_sch`.
2. Run command line export:
   `kicad-cli sch export pdf --output out.pdf project.kicad_sch`
3. Open `out.pdf` and check inter-sheet reference fields.
4. Notice that `${INTERSHEET_REFS}` fields are empty in the CLI-exported PDF, whereas GUI export displays them correctly.

## KiCad Version

```
Application: KiCad x64 on x64

Version: 10.0.5, release build

Libraries:
	wxWidgets 3.3.2 
	FreeType 2.13.3
	HarfBuzz 12.3.0
	FontConfig 2.17.1

Platform: Windows 11 (Erzeugungsversion 26200), 64-bit Edition, 64 bit, Little endian, wxMSW
OpenGL: Intel, Intel(R) Graphics, 4.6.0 - Build 32.0.101.8801

	wxWidgets: 3.3.2 (wchar_t,STL containers)
	Boost: 1.90.0
	OCC: 7.9.2
	Curl: 8.18.0
	ngspice: 46
	Compiler: Visual C++ 1944 without C++ ABI
	KICAD_IPC_API=ON
	KICAD_USE_PCH=OFF

Locale: 
	Lang: de_DE
	Enc: UTF-8
	Num: 1.234,5
	Encoded кΩ丈: D0BACEA9E4B888 (sys), D0BACEA9E4B888 (utf8)
```

---

### Issue 8 -- Duplicate drawing frame after Eagle import (symbol + KiCad page border)

**Module:** `eeschema/importers/schematic_eagle_plugin.cpp`  
**Labels:** `Eagle import`, `bug`, `eeschema`
**GitLab Issue:** [#25196](https://gitlab.com/kicad/code/kicad/-/work_items/25196)

## Description

When importing an EAGLE schematic containing a drawing frame symbol (e.g. `RAHMEN_A4_8Z-19S`), the EAGLE importer performs two conflicting actions simultaneously:
1. Sets page dimensions `(paper "User" 322.88 244.27)` triggering KiCad's internal page border rendering.
2. Places the EAGLE drawing frame symbol as a normal schematic component.

As a result, KiCad renders **two overlapping frames** (KiCad's built-in page border + the imported EAGLE frame symbol).

**Expected behavior:** The importer should either suppress KiCad's default page border (reference an empty `.kicad_wks`) or populate KiCad's title block without importing the frame symbol as a graphic component.

## Steps to reproduce

1. Import an EAGLE schematic containing a drawing frame symbol into KiCad.
2. Export PDF via GUI or `kicad-cli sch export pdf`.
3. Open exported PDF or view schematic editor canvas.
4. Notice duplicate overlapping drawing frames.

## KiCad Version

```
Application: KiCad x64 on x64

Version: 10.0.5, release build

Libraries:
	wxWidgets 3.3.2 
	FreeType 2.13.3
	HarfBuzz 12.3.0
	FontConfig 2.17.1

Platform: Windows 11 (Erzeugungsversion 26200), 64-bit Edition, 64 bit, Little endian, wxMSW
OpenGL: Intel, Intel(R) Graphics, 4.6.0 - Build 32.0.101.8801

	wxWidgets: 3.3.2 (wchar_t,STL containers)
	Boost: 1.90.0
	OCC: 7.9.2
	Curl: 8.18.0
	ngspice: 46
	Compiler: Visual C++ 1944 without C++ ABI
	KICAD_IPC_API=ON
	KICAD_USE_PCH=OFF

Locale: 
	Lang: de_DE
	Enc: UTF-8
	Num: 1.234,5
	Encoded кΩ丈: D0BACEA9E4B888 (sys), D0BACEA9E4B888 (utf8)
```

---

### Issue 9 -- Inter-sheet net references show all pages instead of only connected pages

**Module:** `eeschema` / net navigation  
**Labels:** `eeschema`, `enhancement`, `multi-sheet`
**GitLab Issue:** [#25197](https://gitlab.com/kicad/code/kicad/-/work_items/25197)

## Description

KiCad's `${INTERSHEET_REFS}` variable lists **every other sheet** in the project where a label with the same net name exists, regardless of wire topology or whether the net actually enters or exits that sheet.

In a 6-page project where a common net (such as PE or N) is present on all sheets, page 1 displays `2 3 4 5 6`, page 2 displays `1 3 4 5 6`, etc. This creates information clutter rather than clear navigational references.

**Expected behavior (EAGLE model):** Net cross-references should perform topological filtering, showing only sheets with direct wire connections along with grid coordinates (`NETNAME/SHEET.COLUMNROW`).

## Steps to reproduce

1. Create or import a multi-sheet project (e.g. 6 sheets) with common net labels (e.g. PE, N) present on every sheet.
2. Enable `intersheets_ref_show: true` and make `${INTERSHEET_REFS}` visible on net labels.
3. Inspect net labels on sheet 1.
4. Notice that page references list every sheet (`2 3 4 5 6`) indiscriminately.

## KiCad Version

```
Application: KiCad x64 on x64

Version: 10.0.5, release build

Libraries:
	wxWidgets 3.3.2 
	FreeType 2.13.3
	HarfBuzz 12.3.0
	FontConfig 2.17.1

Platform: Windows 11 (Erzeugungsversion 26200), 64-bit Edition, 64 bit, Little endian, wxMSW
OpenGL: Intel, Intel(R) Graphics, 4.6.0 - Build 32.0.101.8801

	wxWidgets: 3.3.2 (wchar_t,STL containers)
	Boost: 1.90.0
	OCC: 7.9.2
	Curl: 8.18.0
	ngspice: 46
	Compiler: Visual C++ 1944 without C++ ABI
	KICAD_IPC_API=ON
	KICAD_USE_PCH=OFF

Locale: 
	Lang: de_DE
	Enc: UTF-8
	Num: 1.234,5
	Encoded кΩ丈: D0BACEA9E4B888 (sys), D0BACEA9E4B888 (utf8)
```

---

### Issue 10 -- kicad-cli sch export pdf ignores flat top-level sheets in project

**Module:** `kicad-cli` / `eeschema`  
**Labels:** `kicad-cli`, `enhancement`, `eeschema`, `multi-sheet`
**GitLab Issue:** [#25198](https://gitlab.com/kicad/code/kicad/-/work_items/25198)

## Description

Running `kicad-cli sch export pdf --output out.pdf project_page1.kicad_sch` exports only the specified schematic file. If a project consists of multiple independent ("flat") top-level sheets (the standard structure produced by the EAGLE importer), all other project sheets are ignored.

In contrast, the KiCad GUI Page Navigator recognizes all project sheets and plots them into a single multi-page PDF natively.

**Expected behavior:** `kicad-cli` should recognize all top-level sheets in a project (via `.kicad_pro`) and export them into a single multi-page PDF.

## Steps to reproduce

1. Import a multi-page EAGLE project (producing `page1.kicad_sch`, `page2.kicad_sch` under `project.kicad_pro`).
2. Run `kicad-cli sch export pdf --output out.pdf page1.kicad_sch`.
3. Open `out.pdf`.
4. Notice that `out.pdf` contains only page 1 instead of all project sheets.

## KiCad Version

```
Application: KiCad x64 on x64

Version: 10.0.5, release build

Libraries:
	wxWidgets 3.3.2 
	FreeType 2.13.3
	HarfBuzz 12.3.0
	FontConfig 2.17.1

Platform: Windows 11 (Erzeugungsversion 26200), 64-bit Edition, 64 bit, Little endian, wxMSW
OpenGL: Intel, Intel(R) Graphics, 4.6.0 - Build 32.0.101.8801

	wxWidgets: 3.3.2 (wchar_t,STL containers)
	Boost: 1.90.0
	OCC: 7.9.2
	Curl: 8.18.0
	ngspice: 46
	Compiler: Visual C++ 1944 without C++ ABI
	KICAD_IPC_API=ON
	KICAD_USE_PCH=OFF

Locale: 
	Lang: de_DE
	Enc: UTF-8
	Num: 1.234,5
	Encoded кΩ丈: D0BACEA9E4B888 (sys), D0BACEA9E4B888 (utf8)
```

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

