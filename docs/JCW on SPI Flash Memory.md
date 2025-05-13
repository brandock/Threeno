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

```c#
uint16_t xfer16 (uint16_t out) {
  addr()->TXDATCTL = SPI_TXDATCTL_FLEN(16-1) | SPI_TXDATCTL_EOT | out;
  while ((addr()->STAT & SPI_STAT_RXRDY) == 0)
    ;
  return addr()->RXDAT;
}
```

The LPC8xx hardware will set ENABLE to “0”, shift the data out and in using the CLOCK pin, and set ENABLE to “1” when done. At 12 MHz, this will all happen in less than 1.5 µS.

In the above example, three transfers can be seen: send 0x05 (result 0x00), send 0x35 (result 0x00), and send 0x9F (result 0xEF). These all access status and info registers.

SPI is a very common interconnect mechanism, and is also used by the RFM12/RFM69 wireless modules. In fact, we already have a _generic_ SPI bus driver in the embello repository on GitHub. It was designed to be easily re-used for different tasks.

And since SPI flash is a common mechanism, it’s a good idea to allow re-use of that code as well. Such an “SPI flash” driver can also be found on GitHub. Note that the SPI _bus_ driver is LPC8xx specific, but the SPI _flash_ driver built on top is not – it merely embodies the logic specific to the dataflash chip we’re using, _not_ how to talk to it for a particular µC.

With this code, we can now easily access our dataflash memory, using code such as:

```c#
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
```

The SpiFlash<SpiDev0> is a C++ template notation, saying: define an object of the SpiFlash class, _specialised_ to use the SpiDev0 class as its bus access mechanism. Where SpiDev0 is in turn shorthand for SpiDev<0>, i.e. use the SPI0 hardware interface, of the two present in the LPC812 we’re using.

A simple test application can be found here. It erases a few sectors, programs a few pages, and then reads back the data a few times.

So, in _theory_, this stuff is trivial. We should expect this to work perfectly, right?

_Yes and no_… the “Channel 0” pin in the image above is a dangling wire, which happened to be attached to the 8-bit logic analyser (a €10 unit from eBay). Surprisingly, it returns some pulses. This is worrying – we seem to be picking up _stray_ signals from nearby wires.

_As you’ll see, there is some trouble ahead due to this and other unexpected side-effects._

## Too much capacitance is bad ##

The test code for the SPI flash erases sectors 0..9 (i.e. the first 40 KB), and then programs some values into the first 10 _pages_ (i.e. bytes 0..2559). Each page gets three values: N * 11, N * 22, and N * 33 – some of these values will end up different due to 8-bit truncation.

This is what a _good_ read-back of the first 11 pages should look like:

    #0: 0,0,0 @ 491 ms
    #1: 11,22,33 @ 493 ms
    #2: 22,44,66 @ 495 ms
    #3: 33,66,99 @ 496 ms
    #4: 44,88,132 @ 498 ms
    #5: 55,110,165 @ 501 ms
    #6: 66,132,198 @ 503 ms
    #7: 77,154,231 @ 505 ms
    #8: 88,176,8 @ 507 ms
    #9: 99,198,41 @ 509 ms
    #10: 255,255,255 @ 511 ms

The last value on each line is the clock time since startup (SPI is not running at max speed). Also, the last line shown above comes from a page which has been erased but not written to, and as you can see, it’s all 0xFF’s, i.e. all “1” bits.

Unfortunately, the _real_ test results are not consistent at all on the breadboard setup:

    #0: 0,128,128 @ 608 ms
    #1: 0,128,128 @ 610 ms
    #2: 0,128,128 @ 612 ms
    #3: 33,66,227 @ 614 ms
    #4: 44,216,132 @ 616 ms
    #5: 55,110,165 @ 618 ms
    #6: 66,132,198 @ 621 ms
    #7: 77,154,231 @ 623 ms
    #8: 88,176,136 @ 625 ms
    #9: 99,198,169 @ 627 ms
    #10: 255,255,255 @ 629 ms

Keep in mind that this is serial data, and for some reason, MISO is receiving all 0’s, or a 1 followed by 0’s on those first three pages. So some signal levels seem to be _wrong!_

The problem must be either a voltage or a timing issue, since the chip is clearly working, i.e. receiving and returning data “more or less” as expected. The software too must be doing its thing – again, we’d never be able to erase / program / read back _anything_ if it weren’t!

This is a typical hardware issue. In fact, it’s a fundamental electrical problem. Digital logic is all about 1’s and 0’s, and we’re not seeing the values we expect. At least not consistently.

An intermittent problem with voltage / timing (in reality, it’s _both_) points to an issue with “signal integrity”. What _should_ be a clean pulse on the wire, isn’t. It can be quite tricky to analyse such problems – even just hooking up an oscilloscope probe might affect the signal.

But there’s one easy way to find out if we’re looking in the right direction: build the same circuit again with a substantially different electrical configuration and see how it compares:

![image](https://github.com/user-attachments/assets/de8d06a4-31ce-4090-8160-623ccb918900)

As you can see, this dataflash chip was soldered directly onto the back of an LPC812 board (from [eBay](https://web.archive.org/web/20230401203300/http://www.ebay.com/itm/151628723373)). And sure enough: it works flawlessly, at full 12 MHz speed. _Bingo!_

_So what’s going on here?_

It turns out that the culprit is the combination of a high-speed dataflash chip and our solderless breadboard. From the Winbond W25Q64 datasheet:

![image](https://github.com/user-attachments/assets/f37383aa-25c8-4ec6-bc7d-b1eadd3fd707)

The maximum specified load capacitance is only 30 pF: these SPI slave chips are meant to be placed very close to the SPI master, on a PCB with short and straight traces.

A solderless breadboard is a horrible way to connect high-speed signals (both analog and digital), due to the _parasitic_ capacitance. Think about it: lots of little electrical connectors, all running in parallel and right next to each other – what a great way to create capacitors!

Add to that the fairly long jumper wires, their parasitic inductance, and you end up with a signal path that really messes up the leading and falling edges of these signals. These chips are intended to run at up to 100 MHz, i.e. with pulse edges within _under 5 nanoseconds!_

By having to charge / discharge capacitors along the way and by going through wires which act as little inductances, we’re really messing with the signal shape, introducing “ringing” and “overshoot” on all its edges. Unintended signal reflections will make it even worse.

Here’s another hint that these dataflash chips can only generate very weak signals:

![image](https://github.com/user-attachments/assets/e589f9b1-70d4-4f55-82c3-c0649bd6b729)

The output voltage levels are specified at currents of only 100 µA. That’s not a lot to overcome any _parasitic capacitance_, which our setup must be adding to the circuit.

There are two conclusions to be drawn from all this:

1. solderless breadboards are not suitable for _every_ circuit, even simple ones  
2. we don’t always need high-end equipment to diagnose and fix such issues

Note that even with a 50..200 MHz oscilloscope it’s not always easy to figure this out. Such very high-speed signals will not render accurately, but worse still is that the probe itself is likely to affect these results: a standard scope probe adds 10..15 pF to the circuit! A high-end low-capacitance active FET probe would be needed, costing more than the scope itself.

_But the good news is that we’ve identified the issue and found a first workaround._

## Soldering one-off SMD circuits ##

Now that we’re running up against the limitations of solderless breadboards, it’s time to find alternative ways of constructing electronics circuits. And in this day and age, there’s really only one mainstream approach worth pursuing: _point-to-point soldered wires._

There are many ways to solder together components. It’s all about addressing the issue of mechanical stability, as well as taking care of the electrical connections, evidently:

![image](https://github.com/user-attachments/assets/b27516c8-86b5-4bd8-a4c7-91009dfb440f)

This impressive image comes from [a Czech website](https://web.archive.org/web/20230401203408/http://www.valachnet.cz/lvanek/diy/old_story/index.html). It shows how to make a Z80-based computer, as built in the 1980’s. Point-to-point soldering has been around for a while.

We’re not going to be quite as ambitious, but we can re-use the same technique today to construct a small board with an SPI dataflash chip on it, piggy-backed onto an LPC810.

The first step is to mechanically fix the chip in place – using the VCC (8) and GND (4) pins:

![image](https://github.com/user-attachments/assets/a0695e51-e73f-47f7-8f57-23b36353730c)

The rest of the pins have been bent up a bit, to avoid touching the pads which are a bit too large for this SMD component. Then we use “Kynar wire-wrap” wire for all connections:

https://web.archive.org/web/20241027051047im_/https://jeelabs.org/wp-content/uploads/2015/04/DSC_50201.jpg

This type of wire is available in different size spools at a very reasonable cost. The core is 0.01″ in diameter and has a resistance of about 0.3 Ω/m. Some important benefits are that the insulation is easy to remove, the wire solders very easily, and perhaps most important of all: the insulation can withstand brief contact with a hot soldering iron tip without immediately melting away.

Some links to Conrad’s Dutch web-shop, also available in most other European countries:

- 15 m of wire-wrap wire, it’s available in many colours – [606632](https://web.archive.org/web/20230401203408/https://www.conrad.nl/nl/wikkeldraad-wrapping-1-x-005-mm-zwart-conrad-93014c351-15-m-606632.html?insertCode=89)
- side-strip pliers, suitable for these thin types of insulated wires – [1312388](https://web.archive.org/web/20230401203408/https://www.conrad.nl/nl/ck-t3893-1312388.html)
- another kind of wire stripper, mostly suitable for end-stripping – [821746](https://web.archive.org/web/20230401203408/https://www.conrad.nl/nl/jokari-pws-plus-001-micro-striptang-pws-plus-001-012-040-mm-awg-36-26-kabels-met-een-buitenmantel-van-pvc-teflon-kynar-tefzel-mylar-40024-821746.html)

You might think that stripping such short bits of wire is difficult, but here is a video of an interesting approach. It strips the insulation in the middle and then shifts it around to make precisely controlled pieces of wire – while still being easy to hold and solder:

- [https://youtu.be/7a3dA4r8rxc?t=400](https://web.archive.org/web/20230401203408/https://youtu.be/7a3dA4r8rxc?t=400)

No need to watch all 10 minutes, the first 1:30 min at the time linked to above is enough.

As for the pcb material to use for this: in the above images, a low-cost single-sided paper-based “perf-board” was used, mainly because it’s very easy to break and cut into small pieces as needed. The trouble with single-sided copper plating on this type of material is that the pads will easily come loose when heated too much, or when pushed on.

A much better option is FR4 with “plated-through” holes and pads: first of all, it supports soldering on both sides, i.e. placing components on one side and soldering wires on the other side, but also such plated-through holes are a lot sturdier to solder on and to handle considerable amounts of mechanical force. This is most useful for headers, switches, etc.

A drawback of FR4, which is commonly used for PCBs, is that it’s much harder to cut. A small hacksaw could be used, or – with some patience – you can make a series of cuts with a Stanley knife on _both_ sides, and then wiggle and break the board in half with flat pliers:

![DSC_5023](https://web.archive.org/web/20230401203408im_/https://jeelabs.org/wp-content/uploads/2015/04/DSC_5023.jpg)

This option is more durable, and will withstand lots of soldering and re-soldering.

For [another](https://web.archive.org/web/20230401203408/http://elm-chan.org/docs/wire/wiring_e.html) – _astonishing_ – technique, check out this [demo board](https://web.archive.org/web/20230401203408/http://elm-chan.org/docs/wire/wcd.jpeg) and [10-minute video](https://web.archive.org/web/20230401203408/https://www.youtube.com/watch?v=i5MNLTc7YhY). This uses “magnet wire” with a clear insulation that melts _and_ fluxes the wire at ≈ 400°C.

_Who says one-off electronic designs have to end up as quick-and-dirty builds?_

