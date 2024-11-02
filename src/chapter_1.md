# Introduction

## What is it?

MDB (Multi Drop Bus) is a standard used in vending machines to define how a vending machine controller can provide power to, and communicate with a range of commonly used peripherals.
It's quite an old protocol, having been around since at least the 1990s.

The official spec is available [here](https://www.ccv.eu/wp-content/uploads/2018/05/mdb_interface_specification.pdf). 

## What is it for?

The MDB bus allows you to connect a number of devices used in a vending machine to the VMC (Vending Machine Controller).

Commonly used devices are:

* Contactless card reader (called Cashless Device)
* Coin Acceptor (takes coins and can dispense change)
* Bill validator (takes paper money - uncommon in Europe, but more common in the USA)
* Telemetry devices - used for audit purposes etc

## How does it work?

### Key points

It is a two wire, bidirectional serial bus, running 9600 baud, **9 bit**.  There is a VMC (Vending machine controller) and a number of peripherals.

The VMC is in control of the bus, and is responsible for polling the peripherals regularly.  The peripherals can only transmit on the bus in reply to a message from the master.  This avoids the risk of collisions.

The peripherals have optoisolators on their TX and RX lines, so there is galvanic isolation between the signal and power grounds at the peripheral end.

Each device has a preset address, based on what type of device it is.  For example, all coin acceptors have a base address of 0x80.   Hence, you can usually only have a maximum of 1 device of each type on the bus at any one time.

The base address may well have additional bits set to indicate the command.

So for coin acceptors, there are a range of commands, all of which have 0x80 set.  For example:

* 0x08 - reset
* 0x09 - setup data requested by VMC
* 0x0B - poll

Some commands also have sub-commands:

* 0x0F 0x00 - EXPANSION COMMAND  - IDENTIFICATION
* 0x0F 0x01 - EXPANSION COMMAND - FEATURE ENABLE.

The 9th bit is used in two ways:

* By the VMC, in the first 9-bit byte of a message, when it is transmitting the address byte of a device.
* By the device, to indicate last byte of a reply.  

There is also a checksum byte which is a wrapping addition of the sum of the bytes of the message (not including the 9th bits, or the checksum itself).  For a multibyte message, this is the last byte sent.

For a message consisting of a single byte, e.g. 0x0B, the checksum is 0x0B repeated.

For a message consisting of multiple bytes, e.g. 0x0B 0x01, the checksum would be their sum, 0x0C.

ACK/NAK/RET(retransmit) acknowledgements do **not** have the 9th bit set, nor do they have a checksum.   ACK/NAK don't get a reply from the other device.
Only the VMC may send a RET, which is a request to immediately retransmit the last message.
If a VMC replies with NAK, it does **not** give the device permission to retransmit, but it should assume the VMC did not receive its' message, so will usually retransmit it on the next poll etc.

So for example, in the following exchange:

### Get coin acceptor setup data

| Device        | Intent                 |  Message   | 9th bit?           | Checksum   |
|---------------|------------------------|------------|--------------------|------------|
| VMC           | Get coin acceptor info |  0x09      | Set on first byte  | Last byte  |
| Coinacceptor  | Reply with setup data  |  23 bytes  | Set on last byte   | Last byte  |
| VMC           | Acknowledge receipt    |  0x00 (ACK)| not set            | None       |


### Poll the coin acceptor

| Device        | Intent                 |  Message   | 9th bit?           | Checksum   |
|---------------|------------------------|------------|--------------------|------------|
| VMC           | Poll coin acceptor     |  0x0B      | Set on first byte  | Last byte  |
| Coinacceptor  | Nothing happened       |  0x00 (ACK)| Set on last byte   | None       |
| VMC           | No reply               |  None      | n/a                | N/A        |

