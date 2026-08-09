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
