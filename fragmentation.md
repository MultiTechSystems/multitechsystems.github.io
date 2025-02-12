# Fragmentation for FUOTA

The fragmentation implementation is based LoRa Alliance [Fragmented Data Block Transport Protocol](https://resources.lora-alliance.org/technical-specifications/lorawan-fragmented-data-block-transport-specification-v1-0-0)


# Dot Configuration

This example is using Dot v4.0.0 firmware and mPower 5.3.0 on Conduit


## AT Commands

### Output format
Enable addition of Type, DevAddr, FCnt and Port to HEX output of the data. Use AT+RXO=2 for ascii output instead of hexadeicmal.

```
AT+RXO=3
```
Packet output will contain a set of fields, Type is 1 byte, DevAddr is 4 bytes, FCnt is 4 bytes and Port is 1 byte. DevAddr and FCnt are presented in little endian order. The Data field is provided as received.
```
012f49daf00500000001090909

Type  DevAddr   FCnt      Port  Data
01    2f49daf0  05000000  01    090909
```


Enable output of unsolicited messages during the AT command session.
```
AT+URC=1
```
Received packets will appear in the command output without needing to issue commands.

```
RECV 012f49daf00500000001090909
```

The last packet can be displayed with the AT+RECV command.
```
at+recv
012f49daf00500000001090909

OK
```


# FOTA Setup

![Multicast Operation](/images/FOTA-OPERATION.png)

## Setup Time

This setting controls the when the Session Setup packet will be queued for each end-device. Class A end-devices must send an uplink after this packet is queued to create the session needed for to authenticate and decrypt the multicast message payload.

## Launch Time

This setting controls the time allowed for end-devices to retrieve the Session Setup before the multicast payload will be sent.

# Session Progress

![Multicast Operation Progress](/images/FOTA-OPERATION-PROGRESS.png)

After setup time has expired and until launch time the Operation will be in Setup phase. After the setup phase ends the session will transfer to the Broadcasting phase and send the scheduled packet. The progress page shows the Operation and corresponding Multicast EUI.

The Session Setup can be seen below in the Class A uplink response. Port 200 (0xC8) is the multicast setup protocol port. There are two commands in this packet.

```
at+rxo=3

OK
at+urc=1

OK
at+send
01d4482f0002000000c802002657daf02710fa55811ddca35b3bcf307a149f37000000000000000004004e4fab4c0d68e28c0a

OK
RECV 01d4482f0003000000ca0303
at+send
01d4482f0004000000c902012204ef00a200000000

OK
RECV 012657daf001000000c908010000800020b5680008e1680008fd57000803580008095800080f58000800000000000000000000000000000000e1680008e168000800000000e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e168000800000000e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e1680008e16800
RECV 012657daf002000000c908020008e168000800000000e1680008e16800080000000000000000e1680008e1680008e1680008e1680008e168000800000000e168000800000000000000000000000000000000000000005ff808f110b5054c237833b9044b13b10448aff300800123237010bdf80200200000000050cc000808b5034b1bb103490348aff3008008bd00000000fc02002050cc0008154b002b08bf134b9d46a3f5803a00218b460f461348144a121a0af0a1fd0f4b002b00d098470e4b002b00d098470020002104000d000d48002802d00c4805f027ff0af0adfc2000290006f04ff906f0cbf800bf0000080000800020000000002164
RECV 012657daf003000000c90803000008f8020020281700200d6000080000000000207047002070470120002170474ff48050002170474ff4805000217047ff20704700f19801d1e90001704700487047acb500082de9f0471e46dde9087a036815469b6a0446984702460b46284631460bf01cf8134302d00020bde8f087236820469b6a984702460b46384651460bf00df81343f0d1236815eb070820469b6b46eb0a099847494508bf40452cbf01200020e2e72de9f04701681746dde9089a096b98460646884702460b46384641460af0ecff134302d00020bde8f087336817eb090448eb0a0514f1ff32196b304645f1ff33884702460b46204629
RECV 012657daf004000000c9080400460af0d5ff1343e8d1336830469b6b9847a94208bfa0422cbf01200020dee72de9f0471e46dde9087a036815465b6a0446984702460b46284631460af0b8ff134302d00020bde8f087236820465b6a984702460b46384651460af0a9ff1343f0d1236815eb070820469b6b46eb0a099847494508bf40452cbf01200020e2e737b505460446114845f8040b4ff0ff300090284604f0f6f804f178031c22184600210af0b5fcbdf9181004f0e9fd00220023c4e92623d4f88030a26f284607991a6004f02af8204603b030bdc0b500082de9f0471d46836f1646d0f884201b0400f10408044613604046089f4ff00009

```

The multicast packet is shown broken in to each command and corresponding fields.

```
Type  DevAddr  FCnt     Port  Payload
01    d4482f00 02000000 c8    02002657daf02710fa55811ddca35b3bcf307a149f37000000000000000004004e4fab4c0d68e28c0a

MC Group Setup Req
MC Index DevAddr  Encrypted Key                    FCnt Start  FCnt End
02 00    2657daf0 2710fa55811ddca35b3bcf307a149f37 00000000 0000000004004e4fab4c0d68e28c0a
02 00    2f49daf0 11fe3c120a78c11b4b769c52e45d2116 00000000    00000000

MC Class C Session Req
MC Index Start Time   Timeout Freq   DR
04 00    11a2a84c     0f      68e28c 0a
```
## Fragmentation Setup

```
RECV 01d4482f0003000000ca0303
at+send
01d4482f0004000000c902012204ef00a200000000

OK
```
```
Type  DevAddr  FCnt     Port  Payload
01    d4482f00 03000000 ca    0303

Frag Session Delete
CMD
03  03

Type  DevAddr  FCnt     Port  Payload
01    d4482f00 04000000 c9    02012204ef00a200000000

Frag Setup Req
CMD FragIndex  NbFrag FragSize  Control  Padding  Descriptor
02  01           2204   ef        00       a2       00000000
```

```
c9 0200
```

## Fragment Frames

```
RECV 012657daf002000000c908020008e168000800000000e1680008e16800080000000000000000e1680008e1680008e1680008e1680008e168000800000000e168000800000000000000000000000000000000000000005ff808f110b5054c237833b9044b13b10448aff300800123237010bdf80200200000000050cc000808b5034b1bb103490348aff3008008bd00000000fc02002050cc0008154b002b08bf134b9d46a3f5803a00218b460f461348144a121a0af0a1fd0f4b002b00d098470e4b002b00d098470020002104000d000d48002802d00c4805f027ff0af0adfc2000290006f04ff906f0cbf800bf0000080000800020000000002164

Type  DevAddr  FCnt     Port  Payload
01    2657daf0 02000000 c9    08020008e168000800000000e1680008e16800080000000000000000
                              e1680008e1680008e1680008e1680008e168000800000000e1680008
                              00000000000000000000000000000000000000005ff808f110b5054c
                              237833b9044b13b10448aff300800123237010bdf802002000000000
                              50cc000808b5034b1bb103490348aff3008008bd00000000fc020020
                              50cc0008154b002b08bf134b9d46a3f5803a00218b460f461348144a
                              121a0af0a1fd0f4b002b00d098470e4b002b00d09847002000210400
                              0d000d48002802d00c4805f027ff0af0adfc2000290006f04ff906f0
                              cbf800bf0000080000800020000000002164
ID  Index&N Fragment
08  0200    08e168000800000000e1680008e16800080000000000000000
            e1680008e1680008e1680008e1680008e168000800000000e1680008
            00000000000000000000000000000000000000005ff808f110b5054c
            237833b9044b13b10448aff300800123237010bdf802002000000000
            50cc000808b5034b1bb103490348aff3008008bd00000000fc020020
            50cc0008154b002b08bf134b9d46a3f5803a00218b460f461348144a
            121a0af0a1fd0f4b002b00d098470e4b002b00d09847002000210400
            0d000d48002802d00c4805f027ff0af0adfc2000290006f04ff906f0
            cbf800bf0000080000800020000000002164
```

## Parity Frames

After all fragments have been transmitted parity frames will be sent to allow any missed packets to be recovered using the received packets.

```
RECV 012657daf029040000c90829041a805f134cafa5a481c531f6028941cb6003e99528b558a057594b4734927c50d7e888e8d4aca4f39011f10d4e2cf627986dcd1fbfd973f8b4cc06ab6844fee73c835456048586587f734435ae0236a3463fa5788c7d0e3f6eeb6ca2be46a925fce44664cce8e3074631ab934e9c3829d140cdd725a8c897ac03e8e048a5fa9e8972e44a648bf200722c8d51081b025c655704f6f505a7d8c9dd685076a7588d64abae39dcf798096872a1dcfd7d0915812f2bbfe618a6918c26c2a15651f24fc467963327fdeafe6487004f0db9940407c348eb2d35d7a5f8077bb130ce34ece63f82ef59285fee973a46aabb438c

Type  DevAddr  FCnt     Port    Payload
01    2657daf0 29040000 c9   0829041a805f134cafa5a481c531f6028941cb6003e99528b558a05
                                7594b4734927c50d7e888e8d4aca4f39011f10d4e2cf627986dcd1f
                                bfd973f8b4cc06ab6844fee73c835456048586587f734435ae0236a
                                3463fa5788c7d0e3f6eeb6ca2be46a925fce44664cce8e3074631ab
                                934e9c3829d140cdd725a8c897ac03e8e048a5fa9e8972e44a648bf
                                200722c8d51081b025c655704f6f505a7d8c9dd685076a7588d64ab
                                ae39dcf798096872a1dcfd7d0915812f2bbfe618a6918c26c2a1565
                                1f24fc467963327fdeafe6487004f0db9940407c348eb2d35d7a5f8
                                077bb130ce34ece63f82ef59285fee973a46aabb438c

ID  Index&N Fragment
08  2904    1a805f134cafa5a481c531f6028941cb6003e99528b558a05
            7594b4734927c50d7e888e8d4aca4f39011f10d4e2cf627986dcd1f
            bfd973f8b4cc06ab6844fee73c835456048586587f734435ae0236a
            3463fa5788c7d0e3f6eeb6ca2be46a925fce44664cce8e3074631ab
            934e9c3829d140cdd725a8c897ac03e8e048a5fa9e8972e44a648bf
            200722c8d51081b025c655704f6f505a7d8c9dd685076a7588d64ab
            ae39dcf798096872a1dcfd7d0915812f2bbfe618a6918c26c2a1565
            1f24fc467963327fdeafe6487004f0db9940407c348eb2d35d7a5f8
            077bb130ce34ece63f82ef59285fee973a46aabb438c

```

## Status Request

```
RECV 012657daf02a040000c90100
```

### Status Answer Uplink

```
Port CMD  Received MissingFrag  Status
c9   01   0000     FF           01
```

## CRC Verification


CRC-64 verification uplink, http://download.redis.io/redis-stable/src/crc64.c
```
Port CMD  Index  CRC Value
c9   80   00     1C57CAE8353843B2
```

CRC verification response, 0x01 expected for a matching CRC. Otherwise 0x00 will be received and the file was not assembled correctly.
```
RECV 01187950000c000000c98001

Port CMD  Status
c9   80  01
```

## Session Complete

After receiving the CRC valid response the device will send session complete and reset to install the firmware image.
```
Port CMD Index
c9   90  00
```

