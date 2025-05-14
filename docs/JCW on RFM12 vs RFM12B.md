This article by RCW is interesting because he contends that the RFM12B won't be damaged by 5V. 

It can be found [here](https://web.archive.org/web/20210924000509/https://jeelabs.org/2009/05/06/rfm12-vs-rfm12b-revisited/index.html).

It is a follow-up to an initial article (also below) that can be found [here](https://web.archive.org/web/20210923233828/https://jeelabs.org/2009/04/29/rfm12-vs-rfm12b/index.html).

----------------


## RFM12 vs RFM12B – revisited ##
_In AVR, Hardware on May 6, 2009 at 00:01_

Now that the RFM12 is [working](https://jeelabs.org/2009/04/29/rfm12-vs-rfm12b/index.html), here’s the list of differences between RFM12 and RFM12B I’ve come across:

- The RFM12 works up to 5V, the RFM12B only up to 3.8V (but it won’t be damaged by 5V).
- The RFM12 needs a pull-up resistor of 10..100 KΩ on the FSK/DATA/nFFS pin, the RFM12B doesn’t.
- The RFM12 can only sync on the hex “2DD4” 2-byte pattern, the RFM12B can sync on any value for the second byte, not just D4.

More differences, found by Reinhard Max:

- The B version can be set to a single sync byte.
- The “PLL setting command” (CCxx), which only exists on the RFM12B.
- Different values for RX sensitivity and TX output power.
- A slightly different formula to calculate the time of the Wake-Up Timer.

Here is the back of the module, RFM12 on the left, RFM12B on the right:

![image](https://github.com/user-attachments/assets/dde167d0-00c7-4ab8-a0d0-13ca853740ba)

_(these also happen to be for different frequency bands, as you can see)_

And here is the front view, with RFM12 on the _right_ and RFM12B on the _left_ this time:

![image](https://github.com/user-attachments/assets/34d18dd6-4a3a-4d26-bb6d-9ae15c634072)


The RFM12 unit on the right has a pull-up resistor added on top – note also the exposed pads at the top.

Note how the alignment of the solder pads on the JeeNode PCB is a bit off – these are v1 PCB’s, the v2 PCB’s have better alignment.


## RFM12 vs RFM12B ##
_In AVR, Hardware on Apr 29, 2009 at 00:01_

Here’s an RFM12 433 MHz module from Pollin, hooked up to an Arduino Duemilanove:

![image](https://github.com/user-attachments/assets/c69f86c7-00c5-4c74-bd9a-30051a92a2ad)


One major difference between the RFM12 and the RFM12B is that the RFM12 can run at 5V, whereas the maximum operating voltage for the RFM12B is 3.8V (it can withstand up to 6V, which is good to know).

Alas, there seems to be some other difference which eludes me, because the RFM12 hooked up in the above picture only seems to be able to send. The RF12 demo code, which works fine for send and receive on RFM12B’s, seems to do something wrong. Same behavior with another RFM12 module – so it looks like this is not due to a broken module. Send works, but nothing is coming in other than occasional garbage data.

The transmit part works fine when sending to an RFM12B, also in OOK mode: the RFM12 successfully controls both 433 and 868 MHz units (KAKU and FS20, respectively). _But as packet receiver for other RFM12 or RFM12B modules … no joy so far._

Weird. Tips, anyone? Please let me know.

**Update** – Thanks to R. Max’s comment below, the problem has been solved: the RFM12 does not support arbitrary 2-byte sync patterns, _it has to be 0x2D + 0xD4_ – then it works fine!

**Update 2** – _See also the [RFM12 vs RFM12B revisited](https://jeelabs.org/2009/05/06/rfm12-vs-rfm12b-revisited/index.html) page for a list of differences._
