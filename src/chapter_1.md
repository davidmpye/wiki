# MDB - Multi drop bus

## What is it?

MDB (Multi Drop Bus) is a standard used in vending machines to define how a vending machine controller can provide power to, and communicate with a range of commonly used peripherals.
It's quite an old protocol, having been around since at least the 1990s.

## What is it for?

The MDB bus allows you to connect a number of devices used in a vending machine to the VMC (Vending Machine Controller).

Commonly used devices are:

* Contactless card reader (called Cashless Device)
* Coin Acceptor (takes coins and can dispense change)
* Bill validator (takes paper money - uncommon in Europe, but more common in the USA)
* Telemetry devices - used for audit purposes etc

## How does it work?

The specification defines the pinout, and the connector used for MDB:

It is a serial bus, with optoisolators on each device.   It runs at 9600 baud, but the key unusual thing is that it is a 9-bit bus, rather than the more common 8 bits.

Each device has a preset address, based on what type of device it is.  For example, all coin acceptors have a base address of 0x80. 

The 9th bit is used in two ways:

* By the VMC, in the first 9-bit byte of a message, to indicate it is addressing a device.
* By the device, to indicate last byte of a reply

# Hardware interface

## Description

## Schematic

