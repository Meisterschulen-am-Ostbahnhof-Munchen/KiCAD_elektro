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
