# KiCAD_elektro

KiCad symbol libraries for German/IEC 60617-style electrical schematics (terminals, switches, contactors, relays, fuses, motors, motor protection switches, PLC symbols, drawing frames, etc.).

## Origin

These libraries were originally distributed as EAGLE `.lbr` files under the `elektro` library set:

- https://github.com/zumbik/Eagle-Libraries/tree/master/elektro
- https://gitlab.ethz.ch/PLL/eagle/-/tree/b5ed0982e245ca465ad94a335dc35cb3c885401d/lbr/MyLibrary/Original/elektro

They were taken from the sources above and converted to KiCad's native `.kicad_sym` format.

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

## License

The original EAGLE library terms are reproduced verbatim in [`DESCRIPTION`](DESCRIPTION). See that file for the original author's licensing/disclaimer notice.
