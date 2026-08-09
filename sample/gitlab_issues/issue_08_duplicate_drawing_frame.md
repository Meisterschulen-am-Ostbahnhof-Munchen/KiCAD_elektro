# Duplicate drawing frame after Eagle import (symbol + KiCad page border)

# Description

When importing an EAGLE schematic containing a drawing frame symbol (e.g. `RAHMEN_A4_8Z-19S`), the EAGLE importer performs two conflicting actions simultaneously:
1. Sets page dimensions `(paper "User" 322.88 244.27)` triggering KiCad's internal page border rendering.
2. Places the EAGLE drawing frame symbol as a normal schematic component.

As a result, KiCad renders **two overlapping frames** (KiCad's built-in page border + the imported EAGLE frame symbol).

**Expected behavior:** The importer should either suppress KiCad's default page border (reference an empty `.kicad_wks`) or populate KiCad's title block without importing the frame symbol as a graphic component.

# Steps to reproduce

1. Import an EAGLE schematic containing a drawing frame symbol into KiCad.
2. Export PDF via GUI or `kicad-cli sch export pdf`.
3. Open exported PDF or view schematic editor canvas.
4. Notice duplicate overlapping drawing frames.

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
