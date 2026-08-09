# AC tilde notation doubled: 3~ becomes 3~~ in motor symbol

# Description

The three-phase AC notation string `3~` in EAGLE motor symbols (`M 3~`) is rendered as `3~~` after importing into KiCad.

Because the tilde (`~`) character is used as an overbar formatting prefix in KiCad text rendering, the unescaped EAGLE literal tilde `~` is duplicated or improperly escaped during conversion, resulting in `3~~`.

**Expected behavior:** The text literal `3~` should be properly escaped during import so that it renders as `3~`.

# Steps to reproduce

1. Open KiCad Schematic Editor.
2. Import an EAGLE schematic containing a 3-phase motor symbol (`M 3~`).
3. Inspect the motor symbol label.
4. Notice the text renders as `3~~` instead of `3~`.

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
