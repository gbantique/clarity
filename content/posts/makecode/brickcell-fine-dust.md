---
title: Brickcell Fine Dust | MakeCode Microbit
url: /2023/10/02/brickcell_fine_dust/
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

Fine Dust sensor also known as Particulate Matter(PM) sensor is a device used to measure the concentration of fine particulate matter in the air. Fine particulate matter refers to tiny, airborne particles or droplets that can be inhaled into the respiratory system and can have adverse effects on human health and the environment. These particles are often categorized based on their size, with PM2.5 and Pm10 begin common size classifications.

### **Particle Detection**

The sensor contains a mechanism for detecting airborne particles. This can be done using various methods, including light scattering, laser technology, or electrical charges.

### **Data Processing**

Once particles are detected, the sensor processes the data to quantify the concentration of particulate matter in the air. It can differentiate between different particle sizes, such as PM2.5 (particles with a diameter of 2.5 micrometers or smaller) and PM10 (particles with a diameter of 10 micrometers or smaller).

### **Output**

The sensor provides output in the form of concentration measurements, often in units such as micrograms per cubic meter or other relevant units. This data can be displayed on a screen or transmitted to a monitoring system for further analysis.

## **Source Code**

Lets see how we can apply this to a MakeCode typescript programming. Essentially to create an extension to simplify block programming in <https://makecode.microbit.org/>.

### **brickcell-fine-dust**

You may directly used this extension by searching under the MakeCode extension and typing the following hyperlink:
[https://github.com/gbantique/brickcell-fine-dust/](https://github.com/gbantique/brickcell-fine-dust/)

```ts {title="brickcell-fine-dust.ts"}
//% color="#FFBF00" icon="\uf12e" weight=70
namespace Brickcell {
    // Define variables to store pin numbers for LED control and output
    let ledControl: DigitalPin;
    let analogVoltage: AnalogPin;

    /**
     * Set the Vo and LED pin of Fine Dust sensor.
     * @param outPin is pin where the analog output is connected.
     * @param ledPin is pin where the LED control is connected.
     */
    //% block="Initialize Fine Dust Sensor:|ADC Pin %outPin|Control Pin %ledPin"
    //% blockId=brickcell_fine_dust_init
    //% subcategory="fine dust"
    export function initializeFineDustSensor(outPin: AnalogPin, ledPin: DigitalPin): void {
        analogVoltage = outPin;
        ledControl = ledPin;
    }

    /**
     * Get the Fine Dust sensor reading.
     */
    //% block="Read Fine Dust"
    //% blockId=brickcell_fine_dust_read_fine_dust
    //% subcategory="fine dust"
    export function readFineDust(): number {
        // Step 1: Set the LED control pin
        pins.digitalWritePin(ledControl, 1);

        // Step 2: Wait for 300 microseconds
        control.waitMicros(300);

        // Step 3: Read the analog value of analogVoltage pin
        let measuredVoltage = pins.analogReadPin(analogVoltage);

        // Step 4: Clear the LED control pin
        pins.digitalWritePin(ledControl, 0);

        return measuredVoltage; // Return the measured voltage
    }
}

```

### **Extension Code Explanation**

_namespace Brickcell_
: creates a namespace to include the code under the Brickcell module / extension

_initializeFineDustSensor_
- is a function meant to initialized the Fine Dust sensor which takes 2 parameters: outPin (specify the pin where the analog output of the sensor is connected) and ledPin (specify the pin where the LED control for the sensor is connected).

_readFineDust_
- is a function used to read the Fine Dust sensor's data by doing the following:
    * sets the ledControl pin to '1', which presumably turns on an LED on the sensor.
    * waits for 300 microseconds.
    * reads the analog value from the analogVoltage pin. This value represents the measured voltage from the sensor.
    * returns the measured voltage as a numeric value.

_The code defines 2 custom blocks that can be used in MakeCode block programming for microbit. These blocks allow users to initialize the sensor and read Fine Dust measurements easily in block programming. The initialization function sets the pins to be used, and the reading function captures the sensor data after a brief initialization period._

### **How to try using the Brickcell Fine Dust extension**

1. Login to [https://makecode.microbit.org/](https://makecode.microbit.org/) using your Microsoft account.
2. Create a new project by clicking the "New Project" button. You may name it anything you want, I suggest to name it with descriptive name such as "fine-dust-test".
3. Click the "Extensions" block just under the "Math" block.
4. Type [https://github.com/gbantique/brickcell-fine-dust/](https://github.com/gbantique/brickcell-fine-dust/) on the search bar.
5. Select the "brickcell-fine-dust" from the search results. The "fine dust" block should appear under the "Brickcell" block.

A copy of test file could be found by selecting "Javascript", and under the Explorer > brickcell-fine-dust > test.ts. Select it and you may copy and paste the code to main.ts file. Selecting "Blocks" again will display the test code in block programming.

#### **test.ts**

```ts
Brickcell.initializeFineDustSensor(AnalogPin.P0, DigitalPin.P1)
serial.setBaudRate(BaudRate.BaudRate115200)
basic.forever(function () {
    serial.writeNumber(Brickcell.readFineDust())
    serial.writeString("" + ("\r\n"))
    basic.pause(2000)
})
```

#### **Hardware Instruction**

1. Connect the Fine Dust sensor GND pin to microbit GND pin.
2. Connect the Fine Dust sensor VCC pin to microbit VCC pin.
3. Connect the Fine Dust sensor voltage output pin (Vo) to microbit pin 0.
4. Connect the Fine Dust sensor led control (LED) to microbit pin 1.

#### **Expected result of test.ts**

The test file should make the microbit microcontroller to send the measured air particles through serial with 115200 baud rate.

The measured distance should refresh every after 2 seconds.


