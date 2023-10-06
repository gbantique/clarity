---
title: 004 - Brickcell Ultrasonic HCSR04 | MakeCode Microbit
url: /2023/10/02/brickcell_ultrasonic_hcsr04/
author: George Bantique
series:
  - MakeCode, Microbit, Brickcell
categories:
  - MakeCode
  - microbit
date: '2023-10-02T10:47:00+08:00'
tags:
  - MakeCode
  - microbit
---

## **Introduction**

Ultrasonic sensors are devices that uses ultrasonic sound waves to measure distance and/or detect an object. They are a type of proximity sensor that works on the principle of echolocation, similar to how bats and dolphins navigate in the dark. These sensors emit high-frequency sound waves (ultrasonic waves) that are beyond the range of human hearing, and then listen for the echoes produced when these waves bounce off objects. By measuring the time it takes for  the sound waves to travel to the object and back, we can calculate distances with accuracy.

### **Working Principle**

The HC-SR04 sensor is consists of two main components: a transmitter and a receiver. The transmitter emits a burst of ultrasonic waves, while the receiver listens for the echoes. By measuring the time delay between sending the waves and receiving their echoes, the sensor can determine the distance to an object.

### **Measurement Range**

Typically, the HC-SR04 can accurately measure distances within a range of 2 centimeters to 400 centimeters, depending on the specific model and environmental conditions.

### **Output**

When the sensor successfully detects an object and calculates the distance, it typically provides this information as a digital signal on its output pin. The duration of the output pulse is proportional to the measured distance.

## **Source Code**

Lets see how we can apply this to a MakeCode typescript programming. Essentially to create an extension to simplify block programming in <https://makecode.microbit.org/>.

### **brickcell-ultrasonic-hcsr04**

You may directly used this extension by searching under the MakeCode extension and typing the following hyperlink:
[https://github.com/gbantique/brickcell-ultronic-hcsr04/](https://github.com/gbantique/brickcell-ultronic-hcsr04/)

```ts {title="brickcell-ultrasonic-hcsr04.ts"}
//% color="#FFBF00" icon="\uf12e" weight=70
namespace Brickcell {

    /**
    * Read Ultrasonic Distance using HC-SR04 in centimeter.
    */
    //% block="Get distance on echoPin:$echoPin by trigPin:$trigPin"
    //% blockId=brickcell_ultrasonic-hcsr04_read_distance
    //% subcategory="ultrasonic hcsr04"
    export function readDistance(trigPin: DigitalPin, echoPin: DigitalPin): number {
        let _distance_cm;

        pins.setPull(echoPin, PinPullMode.PullNone); // Set echoPin as input with no pull-up/down

        pins.digitalWritePin(trigPin, 0)
        control.waitMicros(2)
        pins.digitalWritePin(trigPin, 1)
        control.waitMicros(10)
        pins.digitalWritePin(trigPin, 0)
        _distance_cm = Math.idiv(pins.pulseIn(echoPin, PulseValue.High), 58)
        
        return _distance_cm;
    }

}
```

### **Extension Code Explanation**

_namespace Brickcell_
: creates a namespace to include the code under the Brickcell module / extension

__distance_cm_
- a variable to hold the value of calculated distance in centimeters

_The code works by triggering the ultrasonic sensor by setting (logic HIGH) the TRIG pin for 10 microseconds then clearing (logic LOW) the TRIG pin back. The duration of the pulse in ECHO pin is then measured (pulseIn) which represents the time it takes for the sound signal to bounce back to the ultrasonic sensor. A distance in centimeters is then calculated from it._

### **How to try using the Brickcell Ultrasonic HCSR04 extension**

1. Login to [https://makecode.microbit.org/](https://makecode.microbit.org/) using your Microsoft account.
2. Create a new project by clicking the "New Project" button. You may name it anything you want, I suggest to name it with descriptive name such as "ultrasonic-hcsr04-test".
3. Click the "Extensions" block just under the "Math" block.
4. Type [https://github.com/gbantique/brickcell-ultrasonic-hcsr04/](https://github.com/gbantique/brickcell-ultrasonic-hcsr04/) on the search bar.
5. Select the "brickcell-ultrasonic-hcsr04" from the search results. The "ultrasonic hcsr04" block should appear under the "Brickcell" block.

A copy of test file could be found by selecting "Javascript", and under the Explorer > brickcell-ultrasonic-hcsr04 > test.ts. Select it and you may copy and paste the code to main.ts file. Selecting "Blocks" again will display the test code in block programming.

#### **test.ts**

```ts
serial.setBaudRate(BaudRate.BaudRate115200);

basic.forever(function () {
    let _distance_cm = Brickcell.readDistance(DigitalPin.P12, DigitalPin.P13);

    serial.writeLine("" + _distance_cm);
    basic.showNumber(_distance_cm);
    basic.pause(2000);
})
```

#### **Hardware Instruction**

1. Connect the Ultrasonic sensor GND pin to microbit GND pin.
2. Connect the Ultrasonic sensor VCC pin to microbit VCC pin.
3. Connect the Ultrasonic sensor trigger pin (TRIG) to microbit pin 12.
4. Connect the Ultrasonic sensor echo pin (ECHO) to microbit pin 13.

#### **Expected result of test.ts**

The test file should make the microbit microcontroller to display the distance measured in centimeters in the 5x5 LED matrix of microbit board. It will also send the measured distance in centimeters through serial with 115200 baud rate.

The measured distance should refresh every after 2 seconds.

