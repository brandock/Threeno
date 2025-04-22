# Threeno

**Threeno** is a family of through-hole, Arduino-compatible boards that came out of my own learning process as I dove deeper into low-power wireless sensor nodes. These boards were inspired by the amazing work of Jean-Claude Wippler (JeeLabs), Felix Russo (LowPowerLab), the OpenEnergyMonitor project, Nathan Chantrell’s TinyTx, and many others. Threeno is not meant to replace any of those designs—rather, it's a personal attempt to bring together what I’ve learned from them into a format that helped me prototype and understand more deeply. They’ve been useful to me as learning tools and prototypes—and maybe they can help others explore this ecosystem, too.

I wanted a board I could use to explore this whole ecosystem—JeeLib, OpenEnergyMonitor, RFM69 radios, and the new AVR parts supported by Spence Konde's cores—but with a familiar Arduino Uno shield footprint. I didn’t want to rely on breadboards anymore. Too often, my breadboard connections were the reason things didn’t work. So Threeno took shape as a set of hand-solderable boards using plated-through-hole parts that anyone with a soldering iron could build and experiment with.

## Objectives

These design goals shaped where and how pins are broken out on the boards:

- **Familiarity**: Match the footprint of the Arduino Uno for compatibility with Uno shields and known layouts.
- **Pin Position Parity**: Line up SPI, Serial, I²C, and other common peripherals with their usual locations on the Uno.
- **Through-hole Focus**: Use PTH parts wherever possible to keep things accessible. (HopeRF radios are technically SMD, but easily hand-soldered.)
- **Proven Components**: Include parts and patterns from designs by JeeLabs, Low Power Labs, OpenEnergyMonitor, and TinyTx—such as the MCP1702 regulator and the multi-transceiver RFM footprint.
- **Stock Arduino Tooling**: Use standard Arduino board managers—no Threeno-specific tools required. ATmega328P boards use the built-in Uno target; AVR128DB28 and ATtiny84 boards use Spence Konde’s DxCore and ATTinyCore.
- **Onboard Radios**: Including the HopeRF transceivers on the board makes prototyping wireless sensors a lot easier. These parts are somewhat notoriously difficult to breadboard, so having it soldered on in a proven way takes the guess work out of that part of your prototype. Other boards, such as Felix Russo's Moteino, also take this approach in compact and polished designs.

## Ecosystem

Threeno boards are designed to work naturally with:

- [**JeeLib**](https://github.com/jeelabs/jeelib) by Jean-Claude Wippler (JCW)
- [**OpenEnergyMonitor**](https://openenergymonitor.org/) by Glyn Hudson, Trystan Lea, Robert Wall, and many others
- [**LowPowerLab’s RFM69 Library**](https://github.com/LowPowerLab/RFM69) by Felix Russo
- [**Arduino IDE**](https://www.arduino.cc/en/software) with these board cores:
  - [**DxCore**](https://github.com/SpenceKonde/DxCore) – for AVR DB series
  - [**ATtinyCore**](https://github.com/SpenceKonde/ATTinyCore) – for ATtiny84

## Threeno Alternatives

Threeno tries to fill a specific niche: a set of through-hole, hand-solderable boards with onboard radios that integrate smoothly into the JeeLib and OpenEnergyMonitor ecosystems. But depending on your needs, one of these other boards might be a better fit:

- [**Moteino**](https://lowpowerlab.com/guide/moteino/) – Compact, preassembled, with a great set of shields and sketches. Part of an IoT ecosystem of its own, with a great user community and great support from Felix Russo.
- [**JeeNode**](https://web.archive.org/web/20201130081805/https://jeelabs.org/docs/hardware/jnclassic/) – A classic by JCW, friendly for breadboard use and educational setups.
- [**Azduino**](https://github.com/SpenceKonde/azduino) – Modern AVRs in small SMD packages from Spence Konde.
- [**ESP32/ESP8266**](https://www.espressif.com/en/products/socs) – Excellent for Wi-Fi/Bluetooth, though less ideal for ultra-low power.

Each of these brings something different to the table—Threeno just aims to make it easier for people (especially soldering hobbyists) to experiment in this space with familiar tools and components.

## Meet the Boards

### [Threeno Tx (ATmega328P)](ThreenoTx.md)
A classic 3.3V UNO-style board with onboard RFM69CW and FTDI or ISP programming.

### [Threeno Tiny Tx (ATtiny84)](ThreenoTinyTx.md)
Minimal and battery-friendly. Based on the ATtiny84, a nod to Nathan Chantrell’s TinyTx.

### [Threeno DB28 Tx (AVR128DB28)](ThreenoDB28.md)
A modern AVR with UPDI programming, MVIO, and Uno-compatible layout.

## Getting the Boards

- **Download Eagle files** and generate Gerbers
- **Order from Seeed Fusion**
- **Use the BOM** to order parts from DigiKey or Mouser

Each board page includes soldering guides and full part lists.

## Useful Tools

- [**FlexyTx Shield**](https://lowpowerlab.com/shop/flexytx) – Mount and test RFM69CW radios easily
- [**Evil Mad Scientist ISP Shield**](https://shop.evilmadscientist.com/productsmenu/652) – For burning bootloaders to raw AVRs

## Developing with Threeno

Use Spence Konde’s Arduino cores:

- [DxCore](https://github.com/SpenceKonde/DxCore) – for AVR128DB28
- [ATtinyCore](https://github.com/SpenceKonde/ATTinyCore) – for ATtiny84/85

For ATmega328P-based boards, select “Arduino Uno” in the IDE.

## License

This project is licensed under the GNU General Public License v3.0.

You may copy, distribute, and modify this software under the terms of the GPLv3.

See the full license at: https://www.gnu.org/licenses/gpl-3.0.en.html

---

This project is a thank you to the makers who built the boards I learned from.

– Brandon Baldock
