---
layout: post
title: How to clone a car key
---

Recently, a friend told me that he had bought a USB antenna to see the TV on the computer (R820T2) and that this antenna could capture traffic in a big frequency range, not only TV range. This antenna could capture frequencies of garage and car keys although it was designed to TV frequencies.

With this info I decided to buy one of this TV antennas that are very cheap, only 10$ ([R820T2 TV antenna](https://es.aliexpress.com/item/FW1S-New-USB-2-0-Digital-DVB-T-SDR-DAB-FM-HDTV-TV-Tuner-Receiver-Stick/32600825233.html?spm=a2g0s.11045068.rcmd404.2.266956a4uWrjNg&pvid=54eedbd9-24c7-42a3-9eed-ab60d3f9f07c&gps-id=detail404&scm=1007.16891.96945.0&scm-url=1007.16891.96945.0&scm_id=1007.16891.96945.0))

<p align="center">
	  <img src="/images/carkey/antenna.jpg">
</p>

To send the signal captured from the car key with the antenna I used one [Arduino One R3](https://www.dx.com/p/uno-r3-development-board-microcontroller-mega328p-atmega16u2-compat-for-arduino-blue-black-2027231#.XAvpPnVKixu) and a [433MHz Radio Frequency transmitter](https://www.dx.com/p/rf-transmitter-receiver-module-433mhz-wireless-link-kit-w-spring-antennas-for-arduino-2057011#.XAvpVnVKixu).
<p align="center">
	  <img src="/images/carkey/arduino.jpg">
	  <img src="/images/carkey/module.jpg">
</p>

I decided to install GQRX that is an open source radio software that allows us see the frequencies spectre and detect our car key frequency using our antenna.
We can install GQRX easily with:

```bash
$ sudo apt-get install gqrx
```
<p align="center">
	  <img src="/images/carkey/gqrx.jpg">
</p>

Now, with the antenna connected to the computer and GQRX running with the 433MHz frequency selected (common frequency for the car keys) and if we press any button of the car key, GQRX will capture it. In my case, the exact frecuency of my car key is 433.92MHz
<p align="center">
	  <img src="/images/carkey/wave.png">
</p>

Once checked that the car key frequency is the correct one, we can capture the "open" and "close" message to reverse them and resend to the car with our Arduino.
I will do the process for the "open" message.
I use another open source software called rtl_433 to capture the message that can be downloaded from the official [GitHub repository](https://github.com/merbanan/rtl_433)

With rtl_433 installed we can follow the repository instructions to capture the signal with this command:

```bash
$ rtl_fm -M am -f 433920000 -s 2000000 - | sox -t raw -r 2000000 -e signed-integer -b 16 -c 1 -V1 - sound.wav 
```
The -f parameter, that has the value 433920000, corresponds with the exact frequency of my car key and sound.wav is the name of the generated file with the signal captured.

I executed the command for 10 seconds and I press the key button three times to have different captures of the "open" message.
To stop the capturing process press Ctrl + C.
With the .wav file in my desktop I opened it with Audacity, a sound editing software that will allows us reverse wave captures.
If we open the .wav file with Audacity we will see the next wave:
<p align="center">
	  <img src="/images/carkey/auda1.png">
</p>

In my case it was easy to appreciate that in the signal dump, there are three zones that corresponds with the periods that I have been with the key button pressed.
If we center our attention in one of these three zones we will see the signal like this:

<p align="center">
	  <img src="/images/carkey/auda2.png">
</p>

In the previous image we can see the same pattern repeated some times. Each of this repetitions correspond to the "open" message that are sended many times when we hold the key.
With this information we will clip one of this patterns to apply reverse engineering to it.

<p align="center">
	  <img src="/images/carkey/auda3.png">
</p>

Now we have to apply reverse engineering to this signal fragment.
If we zoom in the signal, it can be appreciate that are some parts with a big amplitude and other with a smaller one. The big amplitude zones means 1 in binary and the small amplitude zones means 0 in binary.

The last thing we have to calculate is the period of the binary simbols. This means that, if we want to send a digital 0 is translated to to an analogic 0 as a signal with small amplitude during a time T. T is what we have to calculate and we can do this easily with Audacity:

<p align="center">
	  <img src="/images/carkey/auda4.png">
</p>

In my case the period T is 0.4 ms as you can see in the previous image.

With this information we can compile the code below replacing code array value with the 1/0 sequence that corresponds to our reversed signal.

In the array, 1 means a period of big amplitudes of length T and 0 means a period of small amplitudes of, also, length T.

```cpp
//Developed by j0lama
 
#include <SoftwareSerial.h>
 
#define time 400
 
#define pin 9
 
void setup() {
  Serial.begin(9600);
  while (!Serial);
  pinMode(pin, OUTPUT);
  Serial.println("The code will be sent in 2 seconds...");
}
 
void loop() {
    int code[170] = {1,0,0,1,1,0,1,1,0,1,1,0,1,0,0,1,1,0,1,1,0,1,1,0,1,1,0,1,1,0,1,1,0,1,0,0,1,1,0,1,0,0,1,1,0,1,0,0,1,0,0,0,1,1,0,1,0,0,1,0,0,1,0,0,1,1,0,1,1,0,1,0,1,0,0,1,0,0,1,0,0,1,0,0,1,1,0,1,1,0,1,0,0,1,1,0,1,0,0,1,0,0,1,0,0,1,1,0,1,1,0,1,0,0,1,1,0,1,0,0,1,1,0,1,1,0,1,0,0,1,1,0,1,1,0,1,1,0,1,1,0,1,1,0,1,1,0,1,1,0,1,1,0,1,1,0,1,0,0,1,1,0,1,1,0,1,1,0,1,1};
    int i, j;
    for(i = 0; i < 3; i++)
    {
        for(j = 0; j < 170; j++)
        {
            if(code[j] == 1)
            {
                digitalWrite(pin, HIGH);
                delayMicroseconds(time);
                digitalWrite(pin, LOW);
            }
            else
            {
                delayMicroseconds(time);
            }
        }
        Serial.println("Message sent!");
        delay(15);
    }
    Serial.println("Final");
    delay(1000);
}
```

With the code compiled and loaded in our Arduino, we have to setup the hardware following the connections below.

#### Arduino --> RF Transmitter):

- Ground to Ground
- 3.3 V pin to VCC pin
- 9 pin to ATAD pin

I tested this method on Dacia Sandero with this result:

<p align="center">
	<video width="800" height="500" controls>
	  <source type="video/mp4" src="https://raw.githubusercontent.com/j0lama/blog/master/images/carkey/car.mp4"></source>
	</video>
</p>

The car is opened!

With this small write-up I wanna show you how easily is to unlock a car (or a garage) without the key for only 15-20 $.

### Report errors
It's probably that this write-up contains a lot of mistakes so if you find any mistake, please let me know on Twitter (@j0lama).

Thanks for reading.

------