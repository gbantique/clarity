---
title: Brickcell Weight HX711 | MakeCode Microbit
url: /2023/10/04/brickcell_weight_hx711/
author: George Bantique
series:
  - MakeCode, Microbit, Brickcell
categories:
  - MakeCode
  - microbit
date: '2023-10-04T15:47:00+08:00'
tags:
  - MakeCode
  - microbit
  - weight sensor
  - load cell
  - hx711
---

## **Introduction**

The HX711 is a precision 24-bit analog-to-digital converter (ADC) designed for weigh scales and industrial control applications. It is commonly used to interface with load cells, which are sensors used to measure weight or force. The HX711 load cell amplifier provides a stable and accurate digital output of the load cell's signal, making it suitable for applications like kitchen scales, industrial weighing systems, and more.

## **Source Code**

Lets see how we can apply this to a MakeCode typescript programming. Essentially to create an extension to simplify block programming in <https://makecode.microbit.org/>.

### **brickcell-weight-hx711**

You may directly used this extension by searching under the MakeCode extension and typing the following hyperlink:
[https://github.com/gbantique/brickcell-weight-hx711/](https://github.com/gbantique/brickcell-weight-hx711/)

The above repository is a copy of [https://github.com/daferdur/pxt-myHX711/](https://github.com/daferdur/pxt-myHX711/). I modified it just a little bit for the simplicity requirements of Brickcell Development Kit. All credits goes to the original author.

```ts {title="brickcell-weight-hx711.ts"}
//% color="#FFBF00" icon="\uf12e" weight=70
namespace Brickcell {
/**
 * MakeCode editor extension for HX711 Differential 24 bit A/D for weight sensors
 * by David Ferrer - (c)2019
 * MIT License
 */

//% color="#FFBF00" icon="\uf12e" weight=70
namespace Brickcell {
    let DOUT = DigitalPin.P0;
    let PD_SCK = DigitalPin.P1;
    let GAIN: number = 0.0;
    let OFFSET: number = 0;	// used for tare weight
    let SCALE: number = 1;	// used to return weight in grams, kg, ounces, whatever

    /**
     * Set pin for DAT and SCK line is connected
     * @param pinDOUT pin at which the HX data line is connected
     * @param pinSCK pin at which the HX data line is connected
     */
    //% blockId="brickcell_weight_hx711_init" 
    //% block="Initialize HX711 DAT:%pinDOUT|SCK:%pinSCK"
    //% weight=100 blockGap=8
    //% subcategory="weight hx711"
    export function initHX711(pinDOUT: DigitalPin, pinSCK: DigitalPin): void {
        DOUT = pinDOUT;
        PD_SCK = pinSCK;
        set_gain(128) //default gain 128

        // set tare
        let sum: number = 0;
        sum = read_average(20);
        set_offset(sum);

        // set SCALE
        SCALE = 2280;
    }

    function is_ready(): boolean {
        return (pins.digitalReadPin(DOUT) == 0)
    }

    function set_gain(gain: number) {
        switch (gain) {
            case 128:		// channel A, gain factor 128
                GAIN = 1
                break
            case 64:		// channel A, gain factor 64
                GAIN = 3
                break
            case 32:		// channel B, gain factor 32
                GAIN = 2
                break
        }
        pins.digitalWritePin(PD_SCK, 0)
        read()
    }

    function shiftInSlow(bitOrder: number): number {
        let value: number = 0
        let i: number

        for (i = 0; i < 8; ++i) {
            pins.digitalWritePin(PD_SCK, 1)
            control.waitMicros(1)
            if (bitOrder == 0)
                value |= pins.digitalReadPin(DOUT) << i;
            //value = value + (pins.digitalReadPin(DOUT) * 2 ^ i)
            else
                value |= pins.digitalReadPin(DOUT) << (7 - i);
            //value = value + (pins.digitalReadPin(DOUT) * 2 ^ (7 - i))
            //value = value | (pins.digitalReadPin(DOUT) << (7 - i))
            pins.digitalWritePin(PD_SCK, 0)
            control.waitMicros(1)
        }
        return value
    }

    function read(): number {

        // Wait for the chip to become ready.
        wait_ready(0)

        // Define structures for reading data into.
        let value: number = 0
        let data: number[] = [0, 0, 0]
        let filler: number = 0x00

        // Protect the read sequence from system interrupts.  If an interrupt occurs during
        // the time the PD_SCK signal is high it will stretch the length of the clock pulse.
        // If the total pulse time exceeds 60 uSec this will cause the HX711 to enter
        // power down mode during the middle of the read sequence.  While the device will
        // wake up when PD_SCK goes low again, the reset starts a new conversion cycle which
        // forces DOUT high until that cycle is completed.
        //
        // The result is that all subsequent bits read by shiftIn() will read back as 1,
        // corrupting the value returned by read().  The ATOMIC_BLOCK macro disables
        // interrupts during the sequence and then restores the interrupt mask to its previous
        // state after the sequence completes, insuring that the entire read-and-gain-set
        // sequence is not interrupted.  The macro has a few minor advantages over bracketing
        // the sequence between `noInterrupts()` and `interrupts()` calls.

        // Pulse the clock pin 24 times to read the data.
        //LSBFIRST = 0,
        //MSBFIRST = 1
        //data[2] = shiftInSlow(DOUT, PD_SCK, MSBFIRST)
        //data[1] = shiftInSlow(DOUT, PD_SCK, MSBFIRST)
        //data[0] = shiftInSlow(DOUT, PD_SCK, MSBFIRST)

        data[2] = shiftInSlow(1)
        data[1] = shiftInSlow(1)
        data[0] = shiftInSlow(1)

        // Set the channel and the gain factor for the next reading using the clock pin.
        let i: number = 0
        for (i = 0; i < GAIN; i++) {
            pins.digitalWritePin(PD_SCK, 1)
            control.waitMicros(1)
            pins.digitalWritePin(PD_SCK, 0)
            control.waitMicros(1)
        }

        // Replicate the most significant bit to pad out a 32-bit signed integer
        if (data[2] & 0x80) {
            filler = 0xFF
        } else {
            filler = 0x00
        }
        data[2] = data[2] ^ 0x80 //shift MSB

        // Construct a 32-bit signed integer
        value = ((filler) << 24 | (data[2]) << 16 | (data[1]) << 8 | (data[0]))
        //value = ((filler * 16777216) + (data[2] * 65536) + (data[1] * 256) + (data[0]))

        return (value)
    }

    function wait_ready(delay_ms: number) {
        // Wait for the chip to become ready.
        // This is a blocking implementation and will
        // halt the sketch until a load cell is connected.
        while (!is_ready()) {

            basic.pause(delay_ms)
        }
    }

    function wait_ready_retry(retries: number, delay_ms: number): boolean {
        // Wait for the chip to become ready by
        // retrying for a specified amount of attempts
        let count: number = 0
        while (count < retries) {
            if (is_ready()) {
                return true
            }
            basic.pause(delay_ms)
            count++
        }
        return false
    }

    function wait_ready_timeout(timeout: number, delay_ms: number): boolean {
        // Wait for the chip to become ready until timeout.
        // https://github.com/bogde/HX711/pull/96
        let millisStarted: number = input.runningTime()
        while (input.runningTime() - millisStarted < timeout) {
            if (is_ready()) {
                return true
            }
            basic.pause(delay_ms)
        }
        return false
    }

    function read_average(times: number): number {
        let sum: number = 0
        let i: number = 0
        for (i = 0; i < times; i++) {
            sum += read()
            basic.pause(0)
        }
        return sum / times
    }

    function get_value(times: number): number {
        return read_average(times) - OFFSET
    }

    function get_scale(): number {
        return SCALE;
    }

    function set_offset(offset: number) {
        OFFSET = offset;
    }

    function get_offset(): number {
        return OFFSET;
    }

    function get_units(times: number): number {
        let valor: number = 0

        valor = get_value(times) / SCALE

        return valor
    }

    /**
     * Read weight from HX711
     */
    //% blockId="brickcell_weight_hx711_read_weight"
    //% block="Read weight"
    //% weight=80 blockGap=8
    //% subcategory="weight hx711"
    export function readWeight(): string {
        let valor: number = 0;
        let ceros: string = "";
        let valor_string: string = "";

        valor = get_units(1);
        if (Math.abs(Math.round((valor - Math.trunc(valor)) * 100)).toString().length == 0) {
            ceros = "00"
        } else if (Math.abs(Math.round((valor - Math.trunc(valor)) * 100)).toString().length == 1) {
            ceros = "0"
        }
        valor_string = "" + Math.trunc(valor) + "." + ceros + Math.abs(Math.round((valor - Math.trunc(valor)) * 100))
        return valor_string;
    }

    /**
     * Power down HX711
     */
    //% blockId="brickcell_weight_hx711_power_down"
    //% block="power down"
    //% weight=90 blockGap=8
    //% subcategory="weight hx711"
    export function power_down() {
        pins.digitalWritePin(PD_SCK, 0)
        pins.digitalWritePin(PD_SCK, 1)
    }

    /**
     * Power up HX711
     */
    //% blockId="brickcell_weight_hx711_power_up"
    //% block="power up"
    //% weight=90 blockGap=8
    //% subcategory="weight hx711"
    export function power_up() {
        pins.digitalWritePin(PD_SCK, 0)
    }
}

// Original: https://github.com/daferdur/pxt-myHX711/
```

### **Extension Code Explanation**

_namespace Brickcell_
: creates a namespace to include the code under the Brickcell module / extension

_This code is a MakeCode extension written in TypeScript to support the HX711 load cell weight sensor module of Brickcell Development kit._

### **How to try using the Brickcell Weight HX711 extension**

1. Login to [https://makecode.microbit.org/](https://makecode.microbit.org/) using your Microsoft account.
2. Create a new project by clicking the "New Project" button. You may name it anything you want, I suggest to name it with descriptive name such as "weight-hx711-test".
3. Click the "Extensions" block just under the "Math" block.
4. Type [https://github.com/gbantique/brickcell-weight-hx711/](https://github.com/gbantique/brickcell-weight-hx711/) on the search bar.
5. Select the "brickcell-weight-hx711" from the search results. The "weight hx711" block should appear under the "Brickcell" block.

A copy of test file could be found by selecting "Javascript", and under the Explorer > brickcell-weight-hx711 > test.ts. Select it and you may copy and paste the code to main.ts file. Selecting "Blocks" again will display the test code in block programming.

#### **test.ts**

```ts
Brickcell.initHX711(DigitalPin.P0, DigitalPin.P1)
serial.setBaudRate(BaudRate.BaudRate115200)
serial.writeLine("Setup done.")
basic.forever(function () {
    serial.writeString("Weight: ")
    serial.writeLine(Brickcell.readWeight())
    basic.pause(500)
})
```

#### **Hardware Instruction**

1. Connect the Weight sensor GND pin to microbit GND pin.
2. Connect the Weight sensor VCC pin to microbit VCC pin.
3. Connect the Weight sensor data pin (DAT) to microbit pin 0.
4. Connect the Weight sensor clock pin (SCL) to microbit pin 1.

#### **Expected result of test.ts**

The test file should make the microbit microcontroller to send the measured weight in milligrams through serial with 115200 baud rate.

The measured distance should refresh every after half seconds.


