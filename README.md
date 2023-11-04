# Analyzing remote control signal.
## On 433 MHz.


![front img](https://github.com/sdebby/RC_433_sig_analyze/blob/main/media/IMG_7381.jpg?raw=true)

![back img](https://github.com/sdebby/RC_433_sig_analyze/blob/main/media/IMG_7382.jpg?raw=true)

![Open img](https://github.com/sdebby/RC_433_sig_analyze/blob/main/media/IMG_7380.jpg?raw=true)

## Background:
To learn and understand the transmission protocol of a remote control, transiting on 433 MHz.
Remote battery is a coin cell 2032.
Research on the internet found that this remote controll has **rolling code** function. [LINK](https://www.easygates.co.uk/product-category/remote-controls/v2-remote-controls/)

## The equipment:
In order to analyze the signal I used the following software:
* [RTL-SDR](https://www.rtl-sdr.com/) (Software Define Radio) dongle with 16 cm antenna 
* [URH](https://github.com/jopohl/urh)

## The process:
1. Opening the enclosure - found a freescale MQG4 chip
   1. This is an (MCU) 20 MHz 8 bit CPU
   2. Link to [DATASHEET](https://www.rlocman.ru/i/File/dat/Freescale/Microcontrollers_MCU/MC9S08QG4CDNE.pdf)
2. Finding the remote frequency (this was easy as it is written on the back of the remote).
3. Opening URH --> Recording signal --> and pressing on button 1 several times.
4. Repeating process with button 2.
5. Using autodetect parameters to find Center and Samples parameters.
6. Sample parameter set to 400 ms.
7. converting the bits to Hex on both samples.
8. going to "Analytics" tab,converting bits to Hex, using [Non return to zero](https://en.wikipedia.org/wiki/Non-return-to-zero) decoding and analyzing protocol.
![Analytics tab](https://github.com/sdebby/RC_433_sig_analyze/blob/main/media/Screenshot%202023-09-28%20223024.png?raw=true)

**BTN 1**

f8924920964b6c96d96cb2db64924492592592cb2c8 [Pause: 26006 samples]
f8924920964b6c96d96cb2db64924492592592cb2c8 [Pause: 371610 samples]
f8924920964b6c96d96cb2db649244b6db64965b6d8 [Pause: 25984 samples]
f8924920964b6c96d96cb2db649244b6db64965b6d8 [Pause: 410546 samples]
f8924920964b6c96d96cb2db649244b6cb2592d92c8 [Pause: 25982 samples]
f8924920964b6c96d96cb2db649244b6cb2592d92c8 [Pause: 385684 samples]
f8924920964b6c96d96cb2db649244b6d964b64b2d8 [Pause: 25969 samples]
f8924920964b6c96d96cb2db649244b6d964b64b2d8 [Pause: 352200 samples]

**BTN2**

f8924920964b6c96d96cb2db2c9244b6596d92db648 [Pause: 25951 samples]
f8924920964b6c96d96cb2db2c9244b6596d92db648 [Pause: 25932 samples]
f8924920964b6c96d96cb2db2c924492db2c964b258 [Pause: 25957 samples]
f8924920964b6c96d96cb2db2c924492db2c964b258 [Pause: 573040 samples]
f8924920964b6c96d96cb2db2c9244b65b2cb24b6d8 [Pause: 25974 samples]
f8924920964b6c96d96cb2db2c9244b65b2cb24b6d8 [Pause: 473506 samples]
f8924920964b6c96d96cb2db2c9244b6496d96d96c8 [Pause: 25976 samples]
f8924920964b6c96d96cb2db2c9244b6496d96d96c8 [Pause: 492250 samples]

## Breaking the code
- each press of the button will send the code twice (same code each time).
- The pattern describe in the tables:
  
*Button 1:*
|Sync                    |Action|Rolling Code|Stop bit|
|------------------------|------|------------|--------|
|f8924920964b6c96d96cb2db|649244|92592592cb2c|8       |
|f8924920964b6c96d96cb2db|649244|b6db64965b6d|8       |
|f8924920964b6c96d96cb2db|649244|b6cb2592d92c|8       |

  
*Button 2:*
|Sync                    |Action|Rolling Code|Stop bit|
|------------------------|------|------------|--------|
|f8924920964b6c96d96cb2db|2c9244|b6596d92db64|8       |
|f8924920964b6c96d96cb2db|2c9244|92db2c964b25|8       |
|f8924920964b6c96d96cb2db|2c9244|b65b2cb24b6d|8       |


- Sync is the unique hex agreed by the remote control and the receiver.
- Action is the operation- like open/close door.
- Rolling Code - A unique code, that follow sync algorithm, both controller and receiver use the same sync algorithm.
- Stop bit is a sign to the receiver the the transmission ends.

### Controller/Receiver sync and rolling code breakdown.
- In the picture there is a "CONTR" no. (CONTER.144), following instruction on the internet [LINK](https://www.remotecontrol-express.co.uk/remote+V2+:+PHOX2-433+-+CONTR.+17), the remote CONTR index syncs with the receiver, means that this is probably Sync index.
- Explanation on CONTR number - [LINK](https://www-telecomandiuniversali-it.translate.goog/telecomando-v2-phox-2-contr-185-433-92-mhz.html?_x_tr_sl=auto&_x_tr_tl=en&_x_tr_hl=en-US&_x_tr_pto=wapp)
- breaking down the rolling code I found a pattern: the rolling codes will start with 0x92 or 0xb6 on both buttons.
