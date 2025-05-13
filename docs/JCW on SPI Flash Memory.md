Some Threeno Tx boards have the option of onboard SPI flash memory, for compatibility with Moteino firmware and over-the-air programming developed by Felix Rusu of LowPowerLab.

JCW wrote in his blog about SPI flash memory, and I found it instructive, so I repost it here. You can also find it [here](https://web.archive.org/web/20221230224953/https://jeelabs.org/2015/04/15/dataflash-via-spi-2/index.html).

## Dataflash via SPI ##
_In Book on Apr 15, 2015 at 00:01_

One of the things I’m going to need at some point is additional flash memory. Since the simplest way these days to add more memory is probably via SPI, that’s what I’ll use.

This week’s episode is about connecting an SPI chip, implementing a simple driver, code re-use, the hidden dangers of solderless breadboards, and point-to-point soldering:

![image](https://github.com/user-attachments/assets/04d39e69-07b8-4a3c-9d69-74f70bbec8d6)

Hooking up a dataflash chip – Wed
Accessing SPI memory – Thu
Too much capacitance is bad – Fri
Soldering one-off SMD circuits – Sat
The end result is several megabytes of extra storage, using only 4 I/O pins. Data logger? Serial port audit? Storage for audio, video, images? Your next novel? _It’s all up to you!_

Hooking up a dataflash chip
Flash memory has taken over the world, covering an amazing range of memory needs:

- microcontrollers: _kilobytes_
- dataflash chips: _megabytes_
- USB- and (µ)SD cards: _gigabytes_
- SSDs in laptops and servers: _terabytes_

They all maintain their contents for many years and can be erased and re-written tens of thousands of times. Long gone are the days of core memory and battery backed-up RAM.

Not only that, a tiny dataflash chip such as this one from Winbond costs less than €2:

![image](https://github.com/user-attachments/assets/d090e079-9024-43e3-9348-c1b34a0d013a)

That gets you up to 16 megabytes of non-volatile storage, and the data is available faster than most most µCs can read them, with SPI clock rates up to 100 MHz. When connected in “Quad-SPI” mode, some µCs can in fact access 4 bits in parallel, giving a read-access rate of over 50 _Mbyte_ per second. That’s _random_ access: no rotational or seek delays in sight!

Writing is another matter. Usually, data can be written in units from 1 byte to one “page” (256 bytes, typically). Programming will normally take between 1 and 5 milliseconds – this is several orders of magnitude slowdown compared to the above sub-µs read timing.

It gets worse: you can’t just write anything anywhere – you can only turn a “1” bit into a “0” bit when programming any part of memory. After that, flash memory needs to be _erased_ before re-programming it further. An erase cycle resets part or all of memory to 1’s.

The bad news? Erasure can only be done in _sectors_ (or _segments_) of 4 Kb, usually (as well as a few larger units, all the way up to a full chip-erase).

The _really bad_ news? Erasing a segment can often take some 100 ms, with a full chip erase sometimes requiring several _minutes!_ – and after many erasures, these times will increase. During this time, the flash chip will be _busy_ (though there are ways to suspend/resume it).

The exact specs differ somewhat between chip families and manufacturers.

But as with the RomVars implementation described earlier, we can use tricks to make this delay less important. The basic idea is to keep one or more “pre-erased” sectors available, and to start a new erasure when the amount of erased memory gets too low. This requires keeping track of what is in use, and “garbage collecting” sectors to remove unused gaps.

But first, let’s just hook up such an SPI dataflash chip, in this case the Winbond W25Q64 – a 64 Mbit chip, i.e. 8 MB. It can erase sectors of 4, 32, and 64 KB, as well as the entire chip.

It’s easier to do this with an LPC812 than an 8-DIP LPC810 with only 6 I/O pins: with the LPC810, 4 of these will be needed for SPI, making it hard to reserve the 4 I/O pins needed for uploading as well. Besides, the LPC812 (at least some models) supports two separate SPI peripherals – which makes it much simpler to interface to both dataflash and a radio:

![image](https://github.com/user-attachments/assets/a25aac35-148d-466d-be1a-f1ab2bd8bbbc)

Note the use of a _breakout board_ again, to allow using this SMD chip on a breadboard.

The pins (numbered in counter-clockwise order) are connected as follows:

- pin 1: /cs    : PIO0_13
- pin 2: dout   : PIO0_7
- pin 3: /wprot : not connected
- pin 4: gnd    : ground
- pin 5: din    : PIO0_3
- pin 6: clk    : PIO0_2
- pin 7: /hold  : not connected
- pin 8: vcc    : 3.3V

The write-protect and hold pins are not used, and have an internal pull-up.

Note that actual GPIO pin choices are fairly arbitrary, since we can connect them all up as needed through the LPC8xx’s _switch matrix_ (except PIO0_10 and PIO0_11).

PIO0_2 and PIO0_3 are reserved for SWD hardware debugging on power-up, so the SWD functionality needs to be disabled to allow re-using them as ordinary I/O pins.

It’s probably good to point out here that most dataflash memory chips can only work with voltages up to 3.6V (and some not even that). _These chips may not be connected to 5V!_

Next, we’ll need some code to handle the SPI communication and verify that it all works.

## Accessing SPI memory ##
SPI is easy to interface to. That’s because the signalling is quite simple:

![image](https://github.com/user-attachments/assets/06bab1e1-9442-42f7-80f4-fb6ddc42f50c)

There’s a _master_ side (in this case, the µC), which is in control of the ENABLE, CLOCK, and MOSI pins, and there’s a _slave_ side (the dataflash chip), which controls the MISO pin (but only when ENABLE is low).

When ENABLE goes low, the master puts bits on the MOSI (“master out, slave in”) pin, and toggles the clock to shift each bit out to the slave. At the same time, it listens on the MISO pin (you guessed it: “master in, slave out”) and shifts the same number of bits in.

Usually the master first needs to send one or more bytes, so it ignores what comes back during those initial clock cycles. Then, it sends out 0’s and toggles the clock pin further to read back what the slave puts on the MISO pin. When done, the ENABLE pin is set to “1” again. All very simple stuff, even in software.

Unlike I2C, this is not a pure “bus”, in the sense that you can’t simply add more chips in parallel. Well, three of the pins _can_ and should be connected in parallel to all the slaves, but each slave will need a _separate_ ENABLE pin. In this example, there’s just one slave.

Most µCs have hardware support for SPI. This allows them to perform very fast signalling, much faster than software would be able to toggle and read out pins.

In the LPC8xx, the maximum speed for the master is the same as the µC’s clock speed, i.e. 12 MHz on power up, and 30 MHz when the PLL has been set up appropriately. Once set up, sending out and reading back two bytes using the LPC’s SPI hardware is trivial:

uint16_t xfer16 (uint16_t out) {
  addr()->TXDATCTL = SPI_TXDATCTL_FLEN(16-1) | SPI_TXDATCTL_EOT | out;
  while ((addr()->STAT & SPI_STAT_RXRDY) == 0)
    ;
  return addr()->RXDAT;
}
The LPC8xx hardware will set ENABLE to “0”, shift the data out and in using the CLOCK pin, and set ENABLE to “1” when done. At 12 MHz, this will all happen in less than 1.5 µS.

In the above example, three transfers can be seen: send 0x05 (result 0x00), send 0x35 (result 0x00), and send 0x9F (result 0xEF). These all access status and info registers.

SPI is a very common interconnect mechanism, and is also used by the RFM12/RFM69 wireless modules. In fact, we already have a _generic_ SPI bus driver in the embello repository on GitHub. It was designed to be easily re-used for different tasks.

And since SPI flash is a common mechanism, it’s a good idea to allow re-use of that code as well. Such an “SPI flash” driver can also be found on GitHub. Note that the SPI _bus_ driver is LPC8xx specific, but the SPI _flash_ driver built on top is not – it merely embodies the logic specific to the dataflash chip we’re using, _not_ how to talk to it for a particular µC.

With this code, we can now easily access our dataflash memory, using code such as:

#include "spi_flash.h"

SpiFlash<SpiDev0> spif;

int main () {
  ...
  spif.init();
  printf("0x%x\n", spif.identify());
  ....
  spif.eraseSector(...);
  spif.program(...);
  spif.read(...);
  ...
}

The SpiFlash<SpiDev0> is a C++ template notation, saying: define an object of the SpiFlash class, _specialised_ to use the SpiDev0 class as its bus access mechanism. Where SpiDev0 is in turn shorthand for SpiDev<0>, i.e. use the SPI0 hardware interface, of the two present in the LPC812 we’re using.

A simple test application can be found here. It erases a few sectors, programs a few pages, and then reads back the data a few times.

So, in _theory_, this stuff is trivial. We should expect this to work perfectly, right?

_Yes and no_… the “Channel 0” pin in the image above is a dangling wire, which happened to be attached to the 8-bit logic analyser (a €10 unit from eBay). Surprisingly, it returns some pulses. This is worrying – we seem to be picking up _stray_ signals from nearby wires.

_As you’ll see, there is some trouble ahead due to this and other unexpected side-effects._
