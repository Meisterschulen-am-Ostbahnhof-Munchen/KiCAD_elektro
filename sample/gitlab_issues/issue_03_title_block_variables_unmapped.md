# Title block variables (>DATUM, >KUNDE...) not mapped to KiCad title block fields

# Description

EAGLE title block symbols use placeholder attributes of the form `>VARIABLENAME` (e.g. `>DATUM`, `>KUNDE`, `>FUNKTION`, `>ZEICHNUNGS_NR`, `>ERSTELLER`). In EAGLE, these placeholders are dynamically replaced by schematic attribute values.

Upon importing into KiCad, these literal strings (`>DATUM`, `>BEARBEITET`, etc.) are left as raw unexpanded text strings rather than being mapped to KiCad title block properties (`Date`, `Revision`, `Title`, `Company`, `Comment 1`).

**Expected behavior:** Common EAGLE title block variables should be mapped to KiCad title block fields:
- `>DATUM` -> `Date`
- `>ZEICHNUNGS_NR` -> `Revision`
- `>FUNKTION` / `>KUNDE` -> `Title` / `Company`
- `>ERSTELLER` / `>BEARBEITET` -> `Comment 1`

# Steps to reproduce

1. Open KiCad Schematic Editor.
2. Import an EAGLE schematic containing a title block symbol with `>VARIABLE` placeholders.
3. Inspect the imported title block.
4. Notice that raw text strings like `>DATUM` or `>BEARBEITET` appear as unexpanded text in the schematic.

# KiCad Version

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
