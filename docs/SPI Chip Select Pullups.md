## Pull-ups on Threeno Chip-Select Lines

On all Threeno boards you’ll find a 10 kΩ pull-up resistor permanently tied to each SPI device’s _chip-select_ (CS) line.  


## Why a permanent pull-up on CS?

### 1. Avoid SPI-bus contention during AVR ISP programming  
On our ATmega328- and ATtiny84-based Threeno boards (ThreenoTx, ThreenoTinyTx), the MOSI/MISO/SCK pins double as the ICSP interface.  Without a pull-up, a floating CS line can be driven low by noise or by the bootloader sequence, causing the radio or Flash to drive MISO at the same time your ISP programmer is talking to the MCU—and that can corrupt programming or even “brick” the chip.

Read more [here](https://lowpowerlab.com/forum/rf-range-antennas-rfm69-library/rfm69-causes-atmega328p-to-brick-in-isp-mode-(solved)/)
And [here](https://lowpowerlab.com/forum/rf-range-antennas-rfm69-library/rfm69-causes-atmega328p-to-brick-in-isp-mode-(solved)/) in the LowPowerLab forum.

### 2. Protect SPI-Flash memory from accidental writes/reads  
On Threeno boards that have SPI-Flash memory installed, the same principle applies: a 10 kΩ pull-up holds the Flash-CS line _high_ until your code actively drives it low.  This ensures:

- The Flash never drives the bus when you’re not using it.  
- Bus contention cannot occur when multiple slaves share the SPI lines. In particular, the transciever cannot corrupt the flash memory while the MCU is busy booting up or getting programmed.  

## ThreenoDB28 and Chip Select
UPDI programming never touches MOSI/MISO/SCK—so there’s no ISP-vs-radio conflict. So we only need pull-ups to prevent contention between SPI devices themselves. If there is only one (flash-only or transceiver-only, with the other omitted) the pull-up resistors can be left off.
