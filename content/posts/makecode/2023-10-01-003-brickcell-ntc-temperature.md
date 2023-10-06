---
title: 003 - Brickcell NTC Temperature | MakeCode Microbit
url: /2023/10/01/brickcell_ntc_temperature/
author: George Bantique
series:
  - MakeCode, Microbit, Brickcell
categories:
  - MakeCode
  - microbit
date: '2023-10-01T18:50:00+08:00'
tags:
  - MakeCode
  - microbit
---

## **Introduction**

A Negative Temperature Coefficient (NTC) temperature sensor is a type of resistor that exhibits a decrease in electrical resistance as its temperature increases. In other words, the resistance of an NTC thermistor decreases as the temperature rises, and it increases as the terperature decreases. This behavior makes NTC thermistors suitable for various temperature-sensing applications.

From this, we can read an analog value from the signal pin of the NTC sensor. And using the Steinhart-Hart equation we can relate the resistance to its equivalent temperature.

The Steinhart-Hart equation is as follows:

1 / T = (1 / Tnominal) + [ (1 / β) * ln(R / Rnominal)]

where:
- T is the temperature in Kelvin (K)
- Tnominal is the temperature at nominal resistance (Rnominal) in Kelvin
- β is the beta coefficient (B value) of the thermistor
- R is the resistance of the thermistor measured at it signal pin.
- Rnominal is the nominal resistance of the thermistor at a specific temperature (usually 25 degrees Celsius)

## **Source Code**

Lets see how we can apply this to a MakeCode typescript programming. Essentially to create an extension to simplify block programming in <https://makecode.microbit.org/>.

### **brickcell-ntc-temperature**

You may directly used this extension by searching under the MakeCode extension and typing the following hyperlink:
[https://github.com/gbantique/brickcell-ntc-temperature/](https://github.com/gbantique/brickcell-ntc-temperature/)

```ts {title="brickcell-ntc-temperature.ts"}
//% color="#FFBF00" icon="\uf12e" weight=70
namespace Brickcell {
    let samplingRate = 10;          // Get 10 samples for averaging
    let nominalResistance = 10000;  // 10 Kilo Ohms
    let betaValue = 3950;           // The beta coefficient or the B value of the thermistor 
                                    // (usually 3000-4000) check the datasheet for the accurate value.

    /**
    * Read temperature using NTC thermistor
    */
    //% block="Read temperature on pin $ntcPin"
    //% blockId=brickcell_ntc_read_temperature
    //% subcategory="ntc-temperaturae"
    export function readTemperature(ntcPin: AnalogPin): number {
        let _temperature;

        // Read analog value and calculate approximate temperature
        let samples = 0;
        for (let i = 0; i < samplingRate; i++) {
            samples += pins.analogReadPin(ntcPin);
            basic.pause(10);
        }
        let average = samples / samplingRate;
        let resistance = 1023 / average - 1;
        resistance = nominalResistance / resistance;

        // temperature in *C is calculated using the Steinhart-Hart equation
        _temperature = 1 / (Math.log(resistance / nominalResistance) / betaValue + 1 / (25 + 273.15)) - 273.15;
        _temperature = Math.round(_temperature);

        return _temperature;
    }
}

```

### **Extension Code Explanation**

_namespace Brickcell_
: creates a namespace to include the code under the Brickcell module / extension

_samplingRate_
- a variable used for averaging

_nominalResistance_
- resistance value of NTC thermistor on 25 degrees Celsius.

_betaValue_
- the beta coefficient of the NTC thermistor
- usual beta value is between 3000 to 4000. Consult the datasheet for accuracy.

_The code works by getting the average resistance from 10 samples of analog values from the ntc pin. And by using the Steinhart-hart equation, the temperature in degrees Celsius is calculated._

### **How to try using the Brickcell NTC Temperature extension**

1. Login to [https://makecode.microbit.org/](https://makecode.microbit.org/) using your Microsoft account.
2. Create a new project by clicking the "New Project" button. You may name it anything you want, I suggest to name it with descriptive name such as "ntc-temperature-test".
3. Click the "Extensions" block just under the "Math" block.
4. Type [https://github.com/gbantique/brickcell-ntc-temperature/](https://github.com/gbantique/brickcell-ntc-temperature/) on the search bar.
5. Select the "brickcell-ntc-temperature" from the search results. The "ntc temperature" block should appear under the "Brickcell" block.

A copy of test file could be found by selecting "Javascript", and under the Explorer > brickcell-ntc-temperature > test.ts. Select it and you may copy and paste the code to main.ts file. Selecting "Blocks" again will display the test code in block programming.

#### **test.ts**

```ts
basic.forever(function () {
    serial.writeLine("" + (Brickcell.readTemperature(AnalogPin.P0)))
    basic.pause(2000)
})
```

#### **Hardware Instruction**

1. Connect the NTC temperature sensor GND pin to microbit GND pin.
2. Connect the NTC temperature sensor VCC pin to microbit VCC pin.
3. Connect the NTC temperature sensor signal pin (S) to microbit pin 0.

#### **Expected result of test.ts**

The test file should make the microbit microcontroller to send the measured temperature in degrees Celsius through serial with 115200 baud rate.

The measured temperature should refresh every after 2 seconds.

