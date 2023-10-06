---
title: Brickcell Pressure BMP280 | MakeCode Microbit
url: /2023/10/05/brickcell_pressure_bmp280/
author: George Bantique
series:
  - MakeCode, Microbit, Brickcell
categories:
  - MakeCode
  - microbit
date: '2023-10-05T16:20:00+08:00'
tags:
  - MakeCode
  - microbit
  - pressure sensor
  - bmp280
---

## **Introduction**

The BMP280 is a popular digital pressure sensor manufactured by Bosch Sensortec. It is commonly used to measure barometric pressure and temperature. 

It is capable of measuring pressure ranging from 300 hPa to 1100 hPa.

While the sensor is mainly designed to measure barometric pressure, it also comes with a built-in temperature sensor that can give you accurate temperature readings, typically in degrees Celsius (°C).

The Brickcell Development Kit comes 1 piece of BMP280 pressure sensor module which can be access through 0x76 I2C address.

## **Source Code**

Lets see how we can apply this to a MakeCode typescript programming. Essentially to create an extension to simplify block programming in <https://makecode.microbit.org/>.

### **brickcell-pressure-bmp280**

You may directly used this extension by searching under the MakeCode extension and typing the following hyperlink:
[https://github.com/gbantique/brickcell-pressure-bmp280/](https://github.com/gbantique/brickcell-pressure-bmp280/)

The above repository is a copy of [https://github.com/makecode-extensions/BMP280/](https://github.com/makecode-extensions/BMP280/). I modified it just a little bit for the simplicity requirements of Brickcell Development Kit. All credits goes to the original author.

```ts {title="brickcell-pressure-bmp280.ts"}
/**
 * makecode BMP280 digital pressure sensor Package.
 * From microbit/micropython Chinese community.
 * http://www.micropython.org.cn
 */

enum BMP280_I2C_ADDRESS {
    //% block="0x76"
    ADDR_0x76 = 0x76,
    //% block="0x77"
    ADDR_0x77 = 0x77
}
/**
 * Brickcell block
 */
//% color="#FFBF00" icon="\uf12e" weight=70
namespace Brickcell {
    let BMP280_I2C_ADDR = BMP280_I2C_ADDRESS.ADDR_0x76

    function setreg(reg: number, dat: number): void {
        let buf = pins.createBuffer(2);
        buf[0] = reg;
        buf[1] = dat;
        pins.i2cWriteBuffer(BMP280_I2C_ADDR, buf);
    }

    function getreg(reg: number): number {
        pins.i2cWriteNumber(BMP280_I2C_ADDR, reg, NumberFormat.UInt8BE);
        return pins.i2cReadNumber(BMP280_I2C_ADDR, NumberFormat.UInt8BE);
    }

    function getUInt16LE(reg: number): number {
        pins.i2cWriteNumber(BMP280_I2C_ADDR, reg, NumberFormat.UInt8BE);
        return pins.i2cReadNumber(BMP280_I2C_ADDR, NumberFormat.UInt16LE);
    }

    function getInt16LE(reg: number): number {
        pins.i2cWriteNumber(BMP280_I2C_ADDR, reg, NumberFormat.UInt8BE);
        return pins.i2cReadNumber(BMP280_I2C_ADDR, NumberFormat.Int16LE);
    }

    let dig_T1 = getUInt16LE(0x88)
    let dig_T2 = getInt16LE(0x8A)
    let dig_T3 = getInt16LE(0x8C)
    let dig_P1 = getUInt16LE(0x8E)
    let dig_P2 = getInt16LE(0x90)
    let dig_P3 = getInt16LE(0x92)
    let dig_P4 = getInt16LE(0x94)
    let dig_P5 = getInt16LE(0x96)
    let dig_P6 = getInt16LE(0x98)
    let dig_P7 = getInt16LE(0x9A)
    let dig_P8 = getInt16LE(0x9C)
    let dig_P9 = getInt16LE(0x9E)
    setreg(0xF4, 0x2F)
    setreg(0xF5, 0x0C)
    let T = 0
    let P = 0

    function get(): void {
        let adc_T = (getreg(0xFA) << 12) + (getreg(0xFB) << 4) + (getreg(0xFC) >> 4)
        let var1 = (((adc_T >> 3) - (dig_T1 << 1)) * dig_T2) >> 11
        let var2 = (((((adc_T >> 4) - dig_T1) * ((adc_T >> 4) - dig_T1)) >> 12) * dig_T3) >> 14
        let t = var1 + var2
        T = Math.idiv(((t * 5 + 128) >> 8), 100)
        var1 = (t >> 1) - 64000
        var2 = (((var1 >> 2) * (var1 >> 2)) >> 11) * dig_P6
        var2 = var2 + ((var1 * dig_P5) << 1)
        var2 = (var2 >> 2) + (dig_P4 << 16)
        var1 = (((dig_P3 * ((var1 >> 2) * (var1 >> 2)) >> 13) >> 3) + (((dig_P2) * var1) >> 1)) >> 18
        var1 = ((32768 + var1) * dig_P1) >> 15
        if (var1 == 0)
            return; // avoid exception caused by division by zero
        let adc_P = (getreg(0xF7) << 12) + (getreg(0xF8) << 4) + (getreg(0xF9) >> 4)
        let _p = ((1048576 - adc_P) - (var2 >> 12)) * 3125
        _p = Math.idiv(_p, var1) * 2;
        var1 = (dig_P9 * (((_p >> 3) * (_p >> 3)) >> 13)) >> 12
        var2 = (((_p >> 2)) * dig_P8) >> 13
        P = _p + ((var1 + var2 + dig_P7) >> 4)
    }

    /**
     * get pressure
     */
    //% blockId="BMP280_GET_PRESSURE"
    //% block="get pressure"
    //% weight=80 blockGap=8
    //% subcategory="pressure bmp280"
    export function pressure(): number {
        get();
        return P;
    }

    /**
     * get temperature
     */
    //% blockId="BMP280_GET_TEMPERATURE"
    //% block="get temperature"
    //% weight=80 blockGap=8
    //% subcategory="pressure bmp280"
    export function temperature(): number {
        get();
        return T;
    }

    /**
     * power on
     */
    //% blockId="BMP280_POWER_ON"
    //% block="Power On"
    //% weight=61 blockGap=8
    //% subcategory="pressure bmp280"
    export function PowerOn() {
        setreg(0xF4, 0x2F)
    }

    /**
     * power off
     */
    //% blockId="BMP280_POWER_OFF"
    //% block="Power Off"
    //% weight=60 blockGap=8
    //% subcategory="pressure bmp280"
    export function PowerOff() {
        setreg(0xF4, 0)
    }

    /**
     * set I2C address
     */
    //% blockId="BMP280_SET_ADDRESS"
    //% block="set address %addr"
    //% weight=50 blockGap=8
    //% subcategory="pressure bmp280"
    export function Address(addr: BMP280_I2C_ADDRESS) {
        BMP280_I2C_ADDR = addr
    }
}
// Original: https://github.com/makecode-extensions/BMP280/
```


### **Extension Code Explanation**

_namespace Brickcell_
: creates a namespace to include the code under the Brickcell module / extension

_This code is a MakeCode extension written in TypeScript to support the BMP280 Pressure sensor module of Brickcell Development kit._

### **How to try using the Brickcell Pressure BMP280 extension**

1. Login to [https://makecode.microbit.org/](https://makecode.microbit.org/) using your Microsoft account.
2. Create a new project by clicking the "New Project" button. You may name it anything you want, I suggest to name it with descriptive name such as "pressure-bmp280-test".
3. Click the "Extensions" block just under the "Math" block.
4. Type [https://github.com/gbantique/brickcell-pressure-bmp280/](https://github.com/gbantique/brickcell-pressure-bmp280/) on the search bar.
5. Select the "brickcell-pressure-bmp280" from the search results. The "pressure bmp280" block should appear under the "Brickcell" block.

A copy of test file could be found by selecting "Javascript", and under the Explorer > brickcell-pressure-bmp280 > test.ts. Select it and you may copy and paste the code to main.ts file. Selecting "Blocks" again will display the test code in block programming.

#### **test.ts**

```ts
serial.setBaudRate(BaudRate.BaudRate115200)
let pressure = 0
let temperature = 0
basic.forever(function () {
    pressure = Brickcell.pressure()
    temperature = Brickcell.temperature()
    serial.writeNumber(pressure)
    serial.writeString(" hpa, ")
    serial.writeNumber(temperature)
    serial.writeLine(" C.")
    basic.pause(1000)
})
```

#### **Hardware Instruction**

1. Connect the Pressure sensor GND pin to microbit GND pin.
2. Connect the Pressure sensor VCC pin to microbit VCC pin.
3. Connect the Pressure sensor serial data pin (SDA) to microbit pin 20.
4. Connect the Pressure sensor serial clock pin (SCL) to microbit pin 19.

#### **Expected result of test.ts**

The test file should make the microbit microcontroller to send the measured pressure in hecto Pascal and temperature in degrees Celsius through serial with 115200 baud rate.

The measured distance should refresh every after 1 second.

