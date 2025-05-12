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

microcontrollers: kilobytes
dataflash chips: megabytes
USB- and (µ)SD cards: gigabytes
SSDs in laptops and servers: terabytes
They all maintain their contents for many years and can be erased and re-written tens of thousands of times. Long gone are the days of core memory and battery backed-up RAM.

Not only that, a tiny dataflash chip such as this one from Winbond costs less than €2:

![image](https://github.com/user-attachments/assets/d090e079-9024-43e3-9348-c1b34a0d013a)

That gets you up to 16 megabytes of non-volatile storage, and the data is available faster than most most µCs can read them, with SPI clock rates up to 100 MHz. When connected in “Quad-SPI” mode, some µCs can in fact access 4 bits in parallel, giving a read-access rate of over 50 Mbyte per second. That’s random access: no rotational or seek delays in sight!

Writing is another matter. Usually, data can be written in units from 1 byte to one “page” (256 bytes, typically). Programming will normally take between 1 and 5 milliseconds – this is several orders of magnitude slowdown compared to the above sub-µs read timing.

It gets worse: you can’t just write anything anywhere – you can only turn a “1” bit into a “0” bit when programming any part of memory. After that, flash memory needs to be erased before re-programming it further. An erase cycle resets part or all of memory to 1’s.

The bad news? Erasure can only be done in sectors (or segments) of 4 Kb, usually (as well as a few larger units, all the way up to a full chip-erase).

The really bad news? Erasing a segment can often take some 100 ms, with a full chip erase sometimes requiring several _minutes!_ – and after many erasures, these times will increase. During this time, the flash chip will be _busy_ (though there are ways to suspend/resume it).

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
