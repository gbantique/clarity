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
  - ntc temperature
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

## **Hardware Instruction**

Let's explore how to connect the NTC Temperature sensor to a micro:bit microcontroller and utilize MakeCode for programming.

1. Connect the NTC temperature sensor GND pin to microbit GND pin.
2. Connect the NTC temperature sensor VCC pin to microbit VCC pin.
3. Connect the NTC temperature sensor signal pin (S) to microbit pin 0.

## **Software Instruction**

Now, let's start into programming the micro:bit to utilize the NTC temperature sensor.

1. Login to [https://makecode.microbit.org/](https://makecode.microbit.org/) using your Microsoft account.
2. Create a new project by clicking the "New Project" button. You may name it anything you want, I suggest to name it with descriptive name such as "ntc-temperature-test".
3. Click the "Extensions" block just under the "Math" block.
4. Type [https://github.com/gbantique/brickcell-ntc-temperature/](https://github.com/gbantique/brickcell-ntc-temperature/) on the search bar.
5. Select the "brickcell-ntc-temperature" from the search results. The "ntc temperature" block should appear under the "Brickcell" block.
6. Copy the code provided below.

```ts
basic.forever(function () {
    serial.writeLine("" + (Brickcell.readTemperature(AnalogPin.P0)))
    basic.pause(2000)
})
```
7. Open a Serial Monitor such as Termite terminal app [https://www.compuphase.com/software_termite.htm/](https://www.compuphase.com/software_termite.htm/). Set the baud rate to 115200 bps.

## **Expected Result**

If you carefully follow the provided instruction above, you should be able to view the measured temperature in degrees Celsius on the serial terminal every 2000 milliseconds.

