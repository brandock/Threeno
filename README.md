# Threeno

> **Hand‑solderable • UNO‑shield‑compatible footprints • 3 V 3 native • RFM69/95 onboard**  
> A three‑board ecosystem that feels like classic Arduino, plays nicely with JeeLib,
> OpenEnergyMonitor and LowPowerLabs, and never makes you hunt for a 0.4 mm stencil.

| Board | MCU | Radio | Why you’d pick it |
|-------|-----|-------|-------------------|
| **Threeno Tx** | ATmega328P | RFM69CW | Moteino‑compatible 8‑bit workhorse |
| **Threeno Tiny Tx** | ATtiny84 | RFM69CW | Ultra‑tiny sensor node (JeeNode/TinyTx spirit) |
| **Threeno DB28 Tx** | AVR128DB28 | RFM69CW/RFM95 | Modern AVRx with MVIO tricks & 5 V / 3 V 3 split |

## Philosophy

* **Hand solder first** – 0603 passives, SOIC or TQFP MCUs, 2‑54 mm headers.
* **Uno muscle memory** – Shield footprint, D0‑D13/A0‑A5 exactly where you expect.
* **Ecosystem glue** – Runs *unmodified* JeeLib, Felis’ RFM69, OEM sketches.

Jump straight to the boards:

* [`boards/ThreenoTx`](boards/ThreenoTx) – ATmega328P
* [`boards/ThreenoTinyTx`](boards/ThreenoTinyTx) – ATtiny84
* [`boards/ThreenoDB28Tx`](boards/ThreenoDB28Tx) – AVR128DB28

Need hardware?  See **docs/GettingBoardsAndParts.md**  
Need extra gadgets?  Check **tools/**.

---

© 2025 <your name> • Licensed under CERN‑OHL‑S v2.
