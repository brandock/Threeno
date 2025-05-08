This might be the earliest ancestor of the Threeno board. From JCW on December 11, 2008.

Also available here: https://web.archive.org/web/20221230225609/https://jeelabs.org/2008/12/11/good-rf-with-rfm12b/index.html

## Good RF with RFM12B
_In AVR, Software on Dec 11, 2008 at 15:38_

Yippie. Got a simple blinking LED demo running on RFM12B’s after lots of head-scratching. Had to understand ATmega’s SPI mechanism to get there, but these two boards now talk to each other over the air, uni-directionally:

![image](https://github.com/user-attachments/assets/06b91c6b-e3d0-42ac-b3c3-e089719d2ad6)


That’s an Arduino Mini Pro (which runs at 3.3V internally, as does the RFM12B). Power comes from the FTDI cable, but with a special hack to drive the raw power input from it, not the default VCC connection. This way, the on-board 3.3V regulator is used to lower the voltage for both the MCU and the wireless module.

The other board is a bit of a mess, has the same Mini Pro, but some other stuff which is partly a left-over from earlier experiments:

![image](https://github.com/user-attachments/assets/6042b0f1-310a-4edb-bac4-6c9739561446)


Here a USB interface is used to provide 3.3V power. It’s functionally identical to the first board: with RFM12B’s the same setup can be used as transmitter or as receiver.

The top board is the receiver – its picture shows the green LED on, i.e. correct packet reception!

The hookup is very simple, just 5 wires between RF module and MCU, and 2 power pins / 1 pull-up / 1 antenna wire on the RF module is all it takes. The 5 wires are standard SPI plus an IRQ line:

- MOSI (MCU) –> SDI (RFM)
- MISO (MCU) <– SDO (RFM)
- SCK (MCU) –> SCK (RFM)
- SS (MCU) –> nSEL (RFM)
- PD2 (MCU) <– nIRQ (RFM)
  
It took some work to get the software working. The trouble with this stuff is that until you get both transmitter and receiver working properly, it all remains a bit hard to debug. And there are quite a few registers to set up on the RFM12B to make it do its magic.

But after some tweaking, the rfm12xmit and rfm12recv sketches for the Arduino IDE started working. First with bit-banged SPI coding, but then also with the ATmega168’s built-in SPI hardware. Which is nice, because all exchanges now take place at 1..2 MHz (i.e. 8..16 µsec per 16-bit command). All I/O with the RF module uses simple busy polling for now.

The RF range and quality of reception with the RFM12B is considerably better than my earlier 868 MHz AM setup. Packets now easily get across 3 layers of stone/concrete and all of them appear to be arriving properly.

**Update** – here’s the C source code for the xmit and recv sides.

**Update 2** – the connections for MISO and MISO as listed above were _reversed_ – many thanks to José Xavier for pointing this out. The text above has been fixed.
