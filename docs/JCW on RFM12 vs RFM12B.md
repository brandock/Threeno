This article by RCW is interesting because he contends that the RFM12B won't be damaged by 5V. 

It can be found [here](https://web.archive.org/web/20210924000509/https://jeelabs.org/2009/05/06/rfm12-vs-rfm12b-revisited/index.html).

## RFM12 vs RFM12B – revisited ##

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
