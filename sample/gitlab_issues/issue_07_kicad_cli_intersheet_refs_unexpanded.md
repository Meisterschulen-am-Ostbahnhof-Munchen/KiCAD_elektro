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
