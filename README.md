# KiCAD_elektro

KiCad symbol libraries for German/IEC 60617-style electrical schematics (terminals, switches, contactors, relays, fuses, motors, motor protection switches, PLC symbols, drawing frames, etc.).

## Origin

These libraries were originally distributed as EAGLE `.lbr` files under the `elektro` library set:

- https://github.com/zumbik/Eagle-Libraries/tree/master/elektro
- https://gitlab.ethz.ch/PLL/eagle/-/tree/b5ed0982e245ca465ad94a335dc35cb3c885401d/lbr/MyLibrary/Original/elektro

They were taken from the sources above and converted to KiCad's native `.kicad_sym` format.

## Library Overview

Each library has its own folder with a `README.md` listing all symbols as rendered SVG images:

| Library | Symbols |
|---|---|
| [e-elektro-zeichnungsrahmen](e-elektro-zeichnungsrahmen/README.md) | Drawing frames |
| [e-elektromechanische-antriebe](e-elektromechanische-antriebe/README.md) | Electromechanical drives |
| [e-halbleiter](e-halbleiter/README.md) | Semiconductors |
| [e-klemmen](e-klemmen/README.md) | Terminals |
| [e-kondensatoren](e-kondensatoren/README.md) | Capacitors |
| [e-lampen-signalisation](e-lampen-signalisation/README.md) | Lamps and signalling |
| [e-messfuehler](e-messfuehler/README.md) | Sensors |
| [e-messinstrumente](e-messinstrumente/README.md) | Measuring instruments |
| [e-motoren](e-motoren/README.md) | Motors |
| [e-motorschutzschalter](e-motorschutzschalter/README.md) | Motor protection switches |
| [e-schalter](e-schalter/README.md) | Switches |
| [e-schuetze-relais](e-schuetze-relais/README.md) | Contactors and relays |
| [e-sicherungen](e-sicherungen/README.md) | Fuses |
| [e-sps](e-sps/README.md) | PLCs |
| [e-spulen-transformatoren](e-spulen-transformatoren/README.md) | Coils and transformers |
| [e-steckverbinder](e-steckverbinder/README.md) | Connectors |
| [e-stromversorgungselemente](e-stromversorgungselemente/README.md) | Power supply elements |
| [e-symbole](e-symbole/README.md) | General symbols |
| [eib-busch-jaeger](eib-busch-jaeger/README.md) | EIB / Busch-Jaeger |

## Credits

- **Original Author:** librarian@cadsoft.de — likely [Alfred Zaffran](https://www.az-cad.de/) at the time
- **Original Publisher:**
  CadSoft Computer GmbH
  Pleidolfweg 15
  84568 Pleiskirchen
  Deutschland
- **Conversion to KiCad:** Franz Höpfinger ([franz-ms-muc](https://github.com/franz-ms-muc))

## Compatibility Note

The original EAGLE libraries also load in Fusion 360's EAGLE integration, but the special "elektro" schematic features (e.g. cross-references between a contactor coil and its contacts, documented in `elektro-tutorial.pdf` under `EAGLE-7.7.0/doc`) are missing there — reportedly removed to push users toward AutoCAD Electrical. For the full feature set, use **EAGLE 7.7.0**, the last standalone version before the Autodesk acquisition, which also runs under the **EAGLE Express** license tier. The original license terms from that time continue to apply.

## Eagle → KiCad Import Analysis

During the conversion of Eagle schematics to KiCad format, a number of issues
were identified and documented. The analysis is based on a star-delta motor
starter schematic converted from Eagle 7.7.0 to KiCad 10.0.5.

| Language | Document |
|----------|----------|
| 🇩🇪 Deutsch | [EAGLE-to-KiCad-Import-Issues_DE.md](sample/EAGLE-to-KiCad-Import-Issues_DE.md) |
| 🇬🇧 English | [EAGLE-to-KiCad-Import-Issues_EN.md](sample/EAGLE-to-KiCad-Import-Issues_EN.md) |

**10 official KiCad GitLab issues** ([#25189](https://gitlab.com/kicad/code/kicad/-/work_items/25189) to [#25198](https://gitlab.com/kicad/code/kicad/-/work_items/25198)) submitted to the KiCad tracker covering:
missing reference designators, lost cross-references, title block variable
mapping, pin encoding bugs, tilde escaping, net label concatenation,
missing inter-sheet cross-references, duplicate drawing frames, intersheet refs topology, and CLI multi-sheet export.

## License

The original EAGLE library terms are reproduced verbatim in [`DESCRIPTION`](DESCRIPTION). See that file for the original author's licensing/disclaimer notice.
