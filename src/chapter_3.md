# UART interfacing

There are several possibilities for interfacing to the 9 bit bus using the schematics in Chapter 2.

* Microcontroller with 9 bit UART hardware support (eg some of the Atmel/ST series)
* Raspberry Pi PICO with programmable PIO devices which can be programmed to act as a 9 bit UART
* Bit-banging on a MCU that does not natively support 9 bit.
* Bit banging on an IO line using a device like a Raspberry PI


### MCU with 9 bit support

If using an MCU with native 9 bit hardware UART support, the interfacing should be straightforward, though beware - some libraries you may wish to use, e.g. Arduino Serial, may not support 9 bit mode, as it is a bit unusual.

Be sure to find out where the 9th bit value is read from/written to - in some devices it is a separate register.

### Pi Pico with PIO

Rasperry Pi PICO microcontrollers are equipped with a PIO interface - this is a piece of hardware that can be designed to perform IO tasks, and is programmed in a simple assembler language.  It runs separately from the MCU and is able to emulate a UART (and many other things).

I have developed a PIO crate for Rust which provides 9 bit support based on an existing 8 bit UART crate.
[PIO 9-bit Crate](https://github.com/davidmpye/pio-uart-9bit)

Porting back to C should not be especially difficult - the PIO ASM can remain unchanged but the example C shim from the Raspberry Pi foundation would need some adjustments to hand back/receive the 9th bit.

### Bit banging on an MCU without native 9 bit serial

There are a number of libraries eg SoftSerial, that provide bitbanged UART support on MCUs.  However, to receive, this requires attention to ensure polling is regular enough to not miss bits, and people have often encountered timing/data errors relating to this, so overall, best avoided.

### Bit banging on a Pi

It is in theory possible to use a Raspberry Pi IO pin to send/receive 9k6 softserial, but again, it will be challenging to get the timing accurate enough to receive/transmit reliably due to the various demands a multitasking OS puts on the hardware.  