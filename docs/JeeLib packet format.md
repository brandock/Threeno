What is now known as the JeeLib Classic format was developed by JCW and discussed in his blog as early as 2008.

Since his blog is no longer available at jeelabs.org, here is a reprint of his article on the packet format of JeeLib RF12 from June 9, 2011. 

It can currently be found at https://web.archive.org/web/20230603171646/https://jeelabs.org/2011/06/09/rf12-packet-format-and-design/index.html.

## RF12 packet format and design
_In Software on Jun 9, 2011 at 00:01_

The RF12 library contains the code to support the RFM12B wireless module in an Arduino-like environment. It’s used for JeeNodes but also in several projects by others.

Here’s the general structure of a packet, as supported by the RF12 driver:
![image](https://github.com/user-attachments/assets/d2cefa72-c329-4858-a35e-9560d6773c52)

I made quite a few design decisions in the RF12 driver. One of the goals was to make the communication work in the background, so the driver is fully interrupt-driven. An important choice was to limit the packet size to 66 bytes of payload, to keep RAM use low and still allow just over 64 bytes of payload, enough to send data in 64-byte chunks with a _teeny_ bit of additional info.

Another major design decision was to support absolutely minimal packet sizes. This directly affects the power consumption, because longer packets take more time, and the longer the receiver and transmitter are on, the more precious battery power they will consume. For this same reason, the transmission bit rate is set fairly high (about 50 kbits/sec) – a higher rate means the same message can be sent in less time. A higher rate also makes it harder for the receiver to still pick up a good packet, so I didn’t want to push this to the limit.

This is how the RF12 driver can support really short packets:

- the network group is one byte and also doubles as second SYN byte
- the node ID is small enough (5 bits) to allow a few more header bits in the same byte
- there are only three header bits, as described in more detail in tomorrow’s post
- there is only room for _either_ the source node ID _or_ the destination node ID

That last decision is a bit unusual. It means an incoming packet can _only_ inform the receiver where it came from, _or_ define which receiver the packet is intended for – not both.

This may seem like a severe limitation, but it really isn’t: just add the missing info in the payload and agree on a convention so that the receiver can pick it up. All the RF12 does is enforce a truly minimal design, you can add any info you like as payload.

As a result, a minimal packet has the following format:

![image](https://github.com/user-attachments/assets/88191ee9-0529-4010-a0b3-8fdf41561eaf)

That’s 9 bytes, i.e. 72 bits – which means that a complete (but empty) packet can be sent out in less than 1.5 ms.

_Tomorrow, I’ll describe the exact logic used in the HDR byte, and how to use broadcasts and ACKs._

