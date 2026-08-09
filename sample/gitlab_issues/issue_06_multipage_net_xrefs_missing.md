# Multi-page net cross-references (L1/1.18A, N/2.1F...) not imported and not supported

# Description

For multi-page schematics, EAGLE automatically places inter-sheet net cross-references at wire ends in the format `NETNAME/SHEET.COLUMNROW` (e.g. `L1/1.18A`, `L1/2.2A`, `N/1.18F`, `PE/2.1F`). These cross-references are normatively required by DIN EN 61082 / IEC 61082 for industrial electrical schematics.

Upon importing into KiCad, these inter-sheet net cross-references are completely lost.

**Expected behavior:**
1. Short term: Import EAGLE net cross-reference labels as text annotations at wire ends.
2. Long term: KiCad feature request for automatic inter-sheet net cross-references displaying sheet number and grid coordinates.

# Steps to reproduce

1. Open KiCad Schematic Editor.
2. Import a multi-page EAGLE schematic (`.sch`) containing cross-page nets (e.g. L1, L2, L3, N, PE).
3. Inspect the wire ends crossing between sheet 1 and sheet 2.
4. Notice that cross-reference labels (e.g. `L1/1.18A`) are missing entirely.

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
