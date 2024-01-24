# Multicast Messaging with Multitech Conduit and Dot

The multicast implementation is based LoRa Alliance [Multicast Setup Protocol](https://resources.lora-alliance.org/technical-specifications/lorawan-remote-multicast-setup-specification-v1-0-0)

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


# Multicast Session Setup

![Multicast Operation](/images/MULTICAST-OPERATION.png)

## Setup Time

This setting controls the when the Session Setup packet will be queued for each end-device. Class A end-devices must send an uplink after this packet is queued to create the session needed for to authenticate and decrypt the multicast message payload.

## Launch Time

This setting controls the time allowed for end-devices to retrieve the Session Setup before the multicast payload will be sent.

# Session Progress

![Multicast Operation Progress](/images/MULTICAST-OPERATION-PROGRESS.png)

After setup time has expired and until launch time the Operation will be in Setup phase. After the setup phase ends the session will transfer to the Broadcasting phase and send the scheduled packet. The progress page shows the Operation and corresponding Multicast EUI.

## Session Setup Req

The Session Setup can be seen below in the Class A uplink response. Port 200 (0xC8) is the multicast setup protocol port. There are two commands in this packet.

```
at+rxo
0

OK
at+rxo=3

OK
at+urc=1

OK
at+send
01231400000a000000c802002f49daf011fe3c120a78c11b4b769c52e45d21160000000000000000040011a2a84c0f68e28c0a

OK
at+send

OK
RECV 012f49daf003000000017777
RECV 012f49daf004000000018888
RECV 012f49daf00500000001090909
```

The multicast packet is shown broken in to each command and corresponding fields.

```
Type  DevAddr  FCnt     Port  Payload
01    23140000 0a000000 c8    02002f49daf011fe3c120a78c11b4b769c52e45d21160000000000000000040011a2a84c0f68e28c0a

MC Group Setup Req
MC ID DevAddr  Encrypted Key                    FCnt Start  FCnt End
02 00 2f49daf0 11fe3c120a78c11b4b769c52e45d2116 00000000    00000000

MC Class C Session Req
MC ID Start Time   Timeout Freq   DR
04 00 11a2a84c     0f      68e28c 0a
```

Example session key derivation

```python
from Crypto.Cipher import AES

encrypted_key = bytearray.fromhex('1996988aa74237bf48b970a5d67e4317')

nonce = bytearray.fromhex("00000000000000000000000000000000")
appkey = bytearray.fromhex("00000000000000000000000000000000")

cipher = AES.new(appkey, AES.MODE_ECB)
rootkey = cipher.encrypt(nonce)

cipher1 = AES.new(rootkey, AES.MODE_ECB)
mcKEKey = cipher1.encrypt(nonce)

cipher2 = AES.new(mcKEKey, AES.MODE_ECB)
mcKey = cipher2.encrypt(encrypted_key)

keygen_cipher = AES.new(mcKey, AES.MODE_ECB)

# McAppSKey = aes128_encrypt(McKey, 0x01 | McAddr | pad16)
# McNetSKey = aes128_encrypt(McKey, 0x02 | McAddr | pad16)

print("MCKey: " + mcKey.encode("hex"))
print("AppSKey: " + keygen_cipher.encrypt(bytearray.fromhex("018b9adaf00000000000000000000000")).encode("hex"))
print("NwkSKey: " + keygen_cipher.encrypt(bytearray.fromhex("028b9adaf00000000000000000000000")).encode("hex"))

```


## Session Setup Ans
```
c8 020004000D0000
```

# More Messages

Additional messages can be sent to the end-devices in the session using the downlink queue. Queue packets to the Multicast EUI.

![Multicast Downlink](/images/MULTICAST-DOWNLINK.png)

![Multicast Downlink Queue](/images/MULTICAST-DOWNLINK-QUEUE.png)
