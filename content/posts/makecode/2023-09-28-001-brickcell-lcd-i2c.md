---
title: 001 - Brickcell LCD I2C | MakeCode Microbit
url: /2023/09/28/brickcell_fine_dust/
author: George Bantique
series:
  - MakeCode, Microbit, Brickcell
categories:
  - MakeCode
  - microbit
date: '2023-09-28T11:25:00+08:00'
tags:
  - MakeCode
  - microbit
---


## **Introduction**

Brickcell Development Kit comes with an LCD module which is a 16x2 LCD module with an I2C interface.

It has an alphanumeric dot matrix display screen, an HD44780 display controller developed by Hitachi, and a PCF8574T I2C to GPIO expansion IC which converts parallel IO to I2C. The LCD module can be access through `0x20` I2C address.

### **Working Details**
- The I2C interface, can only work with 4-bit mode (no 8-bit mode support).
- Register Select (RS pin):
    * Data register - is a register that contains what is displayed in the LCD
    * Instruction register - is a register that contains instruction for the display controller.
- Read/Write (R/W pin):
    * Reading mode
    * Writing mode
- Enable pin
- Data bus (8 bits, bi-directional) pin

## **Source Code**

Lets see how we can apply this to a MakeCode typescript programming. Essentially to create an extension to simplify block programming in <https://makecode.microbit.org/>. For us not re-invent the wheel, we will use an existing extension and modify it a little bit to adapt to our own needs. The original extension can be found on [https://github.com/makecode-extensions/i2cLCD1602/](https://github.com/makecode-extensions/i2cLCD1602/). Credits to the original author for sharing his work. 

### **brickcell-lcd-i2c**

You may directly used this extension by searching under the MakeCode extension and typing the following hyperlink:
[https://github.com/gbantique/brickcell-lcd-i2c/](https://github.com/gbantique/brickcell-lcd-i2c/)

```ts {title="brickcell-lcd-i2c.ts"}
//% color="#FFBF00" icon="\uf12e" weight=70
namespace Brickcell {
    let i2cAddr: number // 0x3F: PCF8574A, 0x27: PCF8574, 0x20: PCF8574T
    let BK: number      // backlight control
    let RS: number      // command/data

    // set LCD reg
    function setreg(d: number) {
        pins.i2cWriteNumber(i2cAddr, d, NumberFormat.Int8LE)
        basic.pause(1)
    }

    // send data to I2C bus
    function set(d: number) {
        d = d & 0xF0
        d = d + BK + RS
        setreg(d)
        setreg(d + 4)
        setreg(d)
    }

    // send command
    function cmd(d: number) {
        RS = 0
        set(d)
        set(d << 4)
    }

    // send data
    function dat(d: number) {
        RS = 1
        set(d)
        set(d << 4)
    }

    // auto get LCD address
    function AutoAddr() {
        let k = true
        let addr = 0x20
        let d1 = 0, d2 = 0
        while (k && (addr < 0x28)) {
            pins.i2cWriteNumber(addr, -1, NumberFormat.Int32LE)
            d1 = pins.i2cReadNumber(addr, NumberFormat.Int8LE) % 16
            pins.i2cWriteNumber(addr, 0, NumberFormat.Int16LE)
            d2 = pins.i2cReadNumber(addr, NumberFormat.Int8LE)
            if ((d1 == 7) && (d2 == 0)) k = false
            else addr += 1
        }
        if (!k) return addr

        addr = 0x38
        while (k && (addr < 0x40)) {
            pins.i2cWriteNumber(addr, -1, NumberFormat.Int32LE)
            d1 = pins.i2cReadNumber(addr, NumberFormat.Int8LE) % 16
            pins.i2cWriteNumber(addr, 0, NumberFormat.Int16LE)
            d2 = pins.i2cReadNumber(addr, NumberFormat.Int8LE)
            if ((d1 == 7) && (d2 == 0)) k = false
            else addr += 1
        }
        if (!k) return addr
        else return 0

    }

    /**
     * initial LCD, set I2C address.
     * @param Addr is i2c address for LCD, 0 is auto find address
     */
    //% blockId="brickcell_lcd_i2c_init"
    //% block="Set LCD to %addr I2C Address "
    //% weight=100 blockGap=8
    //% subcategory="lcd i2c"
    export function init(Addr: number) {
        if (Addr == 0) i2cAddr = AutoAddr()
        else i2cAddr = Addr
        BK = 8
        RS = 0
        cmd(0x33)       // set 4-bit mode
        basic.pause(5)
        set(0x30)       // still 4-bit mode
        basic.pause(5)
        set(0x20)       // still 4-bit mode
        basic.pause(5)
        cmd(0x28)       // set 5x8 character size mode
        cmd(0x0C)       // turns on the display without showing the cursor
        cmd(0x06)       // sets the cursor to move to the right after each character is displayed.
        cmd(0x01)       // clear
    }

    /**
     * show a number in LCD at given position
     * @param n is number will be show, eg: 10, 100, 200
     * @param x is LCD column position, eg: 0
     * @param y is LCD row position, eg: 0
     */
    //% blockId="brickcell_lcd_i2c_show_number" 
    //% block="show number %n|at x %x|y %y"
    //% weight=90 blockGap=8
    //% x.min=0 x.max=15
    //% y.min=0 y.max=1
    //% subcategory="lcd i2c"
    export function ShowNumber(n: number, x: number, y: number): void {
        let s = n.toString()
        ShowString(s, x, y)
    }

    /**
     * Show numbers in LCD at given position
     * @param numbers is an array of numbers to be displayed, eg: [10, 100, 200]
     * @param x is LCD column position, eg: 0
     * @param y is LCD row position, eg: 0
     */
    //% blockId="brickcell_lcd_i2c_show_numbers" 
    //% block="show numbers %numbers|at x %x|y %y"
    //% weight=90 blockGap=8
    //% x.min=0 x.max=15
    //% y.min=0 y.max=1
    //% subcategory="lcd i2c"
    export function ShowNumbers(numbers: number[], x: number, y: number): void {
        let numStrings: string[] = [];

        // Convert numbers to strings
        for (let num of numbers) {
            numStrings.push(num.toString());
        }

        // Join the number strings with a space separator
        let combinedString = numStrings.join(" ");

        // Display the combined string on the LCD
        ShowString(combinedString, x, y);
    }



    /**
     * show a string in LCD at given position
     * @param s is string will be show, eg: "Hello"
     * @param x is LCD column position, [0 - 15], eg: 0
     * @param y is LCD row position, [0 - 1], eg: 0
     */
    //% blockId="brickcell_lcd_i2c_show_string"
    //% block="show string %s|at x %x|y %y"
    //% weight=90 blockGap=8
    //% x.min=0 x.max=15
    //% y.min=0 y.max=1
    //% subcategory="lcd i2c"
    export function ShowString(s: string, x: number, y: number): void {
        let a: number

        if (y > 0)
            a = 0xC0
        else
            a = 0x80
        a += x
        cmd(a)

        for (let i = 0; i < s.length; i++) {
            dat(s.charCodeAt(i))
        }
    }

    /**
     * turn on LCD
     */
    //% blockId="brickcell_lcd_i2c_on"
    //% block="turn on LCD"
    //% weight=81 blockGap=8
    //% subcategory="lcd i2c"
    export function on(): void {
        cmd(0x0C)
    }

    /**
     * turn off LCD
     */
    //% blockId="brickcell_lcd_i2c_off"
    //% block="turn off LCD"
    //% weight=80 blockGap=8
    //% parts=LCD1602_I2C trackArgs=0
    //% subcategory="lcd i2c"
    export function off(): void {
        cmd(0x08)
    }

    /**
     * clear all display content
     */
    //% blockId="brickcell_lcd_i2c_clear"
    //% block="clear LCD"
    //% weight=85 blockGap=8
    //% subcategory="lcd i2c"
    export function clear(): void {
        cmd(0x01)
    }

    /**
     * turn on LCD backlight
     */
    //% blockId="brickcell_lcd_i2c_backlight_on"
    //% block="turn on backlight"
    //% weight=71 blockGap=8
    //% subcategory="lcd i2c"
    export function BacklightOn(): void {
        BK = 8
        cmd(0)
    }

    /**
     * turn off LCD backlight
     */
    //% blockId="brickcell_lcd_i2c_backlight_off"
    //% block="turn off backlight"
    //% weight=70 blockGap=8
    //% subcategory="lcd i2c"
    export function BacklightOff(): void {
        BK = 0
        cmd(0)
    }

    /**
     * shift left
     */
    //% blockId="brickcell_lcd_i2c_shift_left"
    //% block="Shift Left"
    //% weight=61 blockGap=8
    //% subcategory="lcd i2c"
    export function shl(): void {
        cmd(0x18)
    }

    /**
     * shift right
     */
    //% blockId="brickcell_lcd_i2c_shift_right"
    //% block="Shift Right"
    //% weight=60 blockGap=8
    //% subcategory="lcd i2c"
    export function shr(): void {
        cmd(0x1C)
    }
}

// Original copy: https://github.com/makecode-extensions/i2cLCD1602/
```

### **Extension Code Explanation**

_The code works by communicating through the 0x20 I2C address. The LCD is configured as an LCD with 2 lines of 16 characters each. The LCD's backlight is also turned ON. You may configure it during runtime using the turn off backlight block under the lcd i2c submodule blocks._ 

### **How to try using the Brickcell LCD I2C extension**

1. Login to [https://makecode.microbit.org/](https://makecode.microbit.org/) using your Microsoft account.
2. Create a new project by clicking the "New Project" button. You may name it anything you want, I suggest to name it with descriptive name such as "lcd-i2c-test".
3. Click the "Extensions" block just under the "Math" block.
4. Type [https://github.com/gbantique/brickcell-lcd-i2c/](https://github.com/gbantique/brickcell-lcd-i2c/) on the search bar.
5. Select the "brickcell-lcd-i2c" from the search results. The "lcd i2c" block should appear under the "Brickcell" block.

A copy of test file could be found by selecting "Javascript", and under the Explorer > brickcell-lcd-i2c > test.ts. Select it and you may copy and paste the code to main.ts file. Selecting "Blocks" again will display the test code in block programming.

#### **test.ts**

```ts
let loopCount = 0
Brickcell.init(0)
Brickcell.ShowString("Brickcell", 0, 0)
basic.forever(function () {
    loopCount += 1
    Brickcell.ShowNumber(loopCount, 0, 1)
    basic.pause(1000)
})
```

#### **Hardware Instruction**

1. Connect the LCD GND pin to microbit GND pin.
2. Connect the LCD VCC pin to microbit VCC pin.
3. Connect the LCD SDA pin to microbit pin 20.
4. Connect the LCD SCL pin to microbit pin 19.

#### **Expected result of test.ts**

The test file should display a counter value that increases by 1 every 1 second on the LCD.


---
## **References And Credits**

1. [https://docs.arduino.cc/learn/electronics/lcd-displays/](https://docs.arduino.cc/learn/electronics/lcd-displays/)

2. [https://www.playembedded.org/blog/hd44780-lcdii-and-chibioshal/](https://www.playembedded.org/blog/hd44780-lcdii-and-chibioshal/)

3. [http://www.matidavid.com/pic/LCD%20interfacing/introduction.htm/](http://www.matidavid.com/pic/LCD%20interfacing/introduction.htm/)
