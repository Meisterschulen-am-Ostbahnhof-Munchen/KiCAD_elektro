# Component reference designators (F1, Q2, K1...) missing after Eagle import

# Description

When importing an EAGLE schematic (`.sch`) into KiCad, component reference designators (F1, F2, F3, F4, Q1-Q4, K1, S1, S2, M1, X1-X3) are completely missing from the imported schematic symbols.

In EAGLE, these reference designators reside on Layer 95 (Names) and are clearly visible in the schematic and exported PDF. In KiCad, the symbol reference fields are empty or not populated from the EAGLE name layer.

**Expected behavior:** EAGLE component names (Layer 95) should be imported into the KiCad `Reference` field for each symbol.

# Steps to reproduce

1. Open KiCad Schematic Editor.
2. Import an EAGLE schematic containing named components (`File` -> `Import` -> `Non-KiCad Schematic...` -> select EAGLE `.sch`).
3. Inspect the imported symbols in the schematic editor.
4. Notice that component reference designators (F1, Q2, K1, etc.) are missing or empty.

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
