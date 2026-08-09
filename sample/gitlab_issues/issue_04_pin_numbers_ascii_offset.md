# Pin numbers encoded as ASCII offset, garbled characters instead of 95-98

# Description

EAGLE symbols for motor circuit breakers use IEC-standardized contact numbers `95`, `96` (NC) and `97`, `98` (NO) for the thermal overload release mechanism.

After importing into KiCad, these pin numbers are garbled into ASCII characters `:`, `;`, `<`, `=`, `>`, `?`, `@`. This indicates an encoding bug during symbol pin number mapping (ASCII offset of 48).

**Expected behavior:** Pin numbers `95`, `96`, `97`, `98` should be preserved as literal string pin numbers "95", "96", "97", "98".

# Steps to reproduce

1. Open KiCad Schematic Editor.
2. Import an EAGLE schematic containing motor protection switch symbols with contacts 95-98.
3. Inspect the auxiliary contact pin numbers of the motor protection switch.
4. Notice that pin numbers appear as garbled ASCII characters (e.g. `:`, `;`, `<`, `=`).

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
