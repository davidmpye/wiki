# Coin Acceptor

The coin acceptor is designed to handle cash in the form of coins.  

One of three things happens when a coin is inserted:

* Rejected and returned to the user 
* Identified, credited to balance and sorted into one of the coin acceptor's tubes
* Identified, credited to balance and deposited into the vending machine cash box.

When the coin acceptor is powered up, and before it is initialised by the VMC, it will not accept any coins, and any coin inserted will be returned to the user.

When the coin acceptor is initialised and told to accept coins:

* If a coin is inserted, it will be routed to a coin tube if there is an appropriate tube for it to go into, and that tube is not full.
* If a coin in inserted, but either it is not one the coin acceptor can hold in a coin tube, or its' tube is full, it will be diverted to the cash box.
* An unrecognised coin, or one that has not been programmed to be accepted will be routed through the coin reject route.

For example, in UK usage, my coin acceptor has tubes for 5p, 10p, 20p, 50p.

It can hold and give change in only those coin values, but it can accept other values eg £1 or £2.

So if a £1 is inserted, it will be credited to the balance, but the £1 will be diverted to the cash box.  If the VMC wants to return £1 to the user, it cannot return the £1 as it is no longer within the acceptor, so it would have to return other denominations (eg 2x50p, 5x20p, 10x10p) to give the user back their money.

The MDB protocol allows for the following types of coin acceptor

* Level 2
* Level 3

## Coin Acceptor Level 2

A coin acceptor meeting L2 spec can understand/respond to the following commands:

## Coin Acceptor Level 3

A coin acceptor meeting L3 spec can understand all the L2 commands, but in addition can handle the following expansion commands:

### 0x0F - Expansion commands

#### Identification

|Main   | 0x0F|
|-------|-----------|
| Subcommand | 0x00      |

Should get a 33 byte response from the acceptor:

| Bytes     | Meaning                                           |
|-----------|---------------------------------------------------|
| 0-2       | 3 byte manufacturer code in ASCII e.g. 'M','E','I'|
| 3-14      | 12 byte serial number in ASCII                    |
| 15-26     | 12 byte model number  in ASCII                    |
| 27-28     | 2 byte software vers (packed BCD)                 |
| 29-32     | 4 bytes indicating additional features            |

Of the additional features, only the lower 4 bits of the lowest byte are used.

| Bit    | Meaning  |
|------- |----------|
| 0x01   | Alterntive Payout Method supported |
| 0x02   | Extended diagnostic command supported |
| 0x04   | Controlled manual fill and payout commands supported |
| 0x08   | File transport layer (eg firmware/coin file download) supported |

#### Feature enable

|Main   | 0x0F|
|-------|-----------|
| Subcommand | 0x01     |
| Data | 4 bytes | 

Four bytes sent in the same format as above, with bits set high to indicate which supported L3 features the VMC wishes to use.

#### Payout

|Main   | 0x0F|
|-------|-----------|
| Subcommand | 0x01     |
| Data | 1 byte - payout amonut| 

Indicates the amount of money the VMC should pay out based on the scaling factor in use.

eg UK example, scaling factor of 5, a payout byte value of 0x0A would cause 50p to be dispensed in whichever coins the coin acceptor chooses to use.

#### Payout status


|Main   | 0x0F|
|-------|-----------|
| Subcommand | 0x03     |

Up to 16 byte reply - 1 byte for each coin type, in ascending value order, indicating how many of that coin have been paid out.  Unsent bytes assumed as zero.

If it is busy, it may reply with just an ACK.
Payout data is cleared after VMC replies with ACK.

#### Payout value


|Main   | 0x0F|
|-------|-----------|
| Subcommand | 0x04     |

1 byte reply: Amount of value (* scaling factor) paid out since either payout command above started, or last payout status request.

Will reply ACK if the payout is complete.

So you can use the two above commands as follows:

| Device | Message | Meaning|
|--------|---------|---------|
|VMC | 0x0F 0x01 0x0A | Dispense coins to the value of (scaling factor times 10)|
|Peripheral | ACK ||
|VMC | 0x0F 0x04 |  Request amount paid out so far |
|Peripheral | 0x05 | (scaling factor * 5) value paid out so far|
| VMC | 0x0F 0x04 | Request amount paid out so far|
|Peripheral| ACK | Payout complete |
| VMC | 0x0F 0x03 | What coins were paid out?|
| Peripheral | 0x01 0x02 0x01 | 1x highest val coin, 2x next val coin, 1x next val coin|
| VMC | ACK | Acknowledge, clears the paid out counter. |
