# Inter-sheet net references show all pages instead of only connected pages

# Description

KiCad's `${INTERSHEET_REFS}` variable lists **every other sheet** in the project where a label with the same net name exists, regardless of wire topology or whether the net actually enters or exits that sheet.

In a 6-page project where a common net (such as PE or N) is present on all sheets, page 1 displays `2 3 4 5 6`, page 2 displays `1 3 4 5 6`, etc. This creates information clutter rather than clear navigational references.

**Expected behavior (EAGLE model):** Net cross-references should perform topological filtering, showing only sheets with direct wire connections along with grid coordinates (`NETNAME/SHEET.COLUMNROW`).

# Steps to reproduce

1. Create or import a multi-sheet project (e.g. 6 sheets) with common net labels (e.g. PE, N) present on every sheet.
2. Enable `intersheets_ref_show: true` and make `${INTERSHEET_REFS}` visible on net labels.
3. Inspect net labels on sheet 1.
4. Notice that page references list every sheet (`2 3 4 5 6`) indiscriminately.

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
