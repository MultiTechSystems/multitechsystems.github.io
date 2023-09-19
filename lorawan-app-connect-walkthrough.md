# MQTT Walkthrough

## Network Settings

Configure the gateway for the regional channel plan.

![mPower Default App Settings](/images/WALKTHROUGH-NETWORK-SETTINGS.png)


## Local Device Key Settings

Enter the device information to allow joining the network server. Enter the DevEUI/AppKey for each device to authenticate the join. AppEUI and Profiles can be assigned for each device.

![mPower Device Credentials](/images/local-key-settings.png)

### Settings
* DevEUI - End-Device EUI
* AppEUI - AppEUI to assign device record on Join
  * Must match Default App EUI setting to forward to application if set to a specific AppEUI
  * AppEUI of default app can be set to FF-FF-FF-FF-FF-FF-FF to use v1.0 protocol with any device AppEUI
  * Device AppEUI does not need to match this setting. The AppKey is matched to the DevEUI only.
* AppKey - AppKey of end-device to authenticate join and create session keys
* Device Profile - default settings to assign device on join
* Network Profile - Class and RF settings to use for device session, MAC commands are sent to update the end-device



# Gateway Settings

![mPower Default App Settings](/images/WALKTHROUGH-DEFAULT-APP-SETTINGS.png)

1. Enable Default App
2. Select protocol version 1.1
3. Set Server URL to "mqtt://test.mosquitto.org:1883"
4. Select Cloud Service as MQTT Broker
5. Select Clean Session
6. Select Enable LoRaQuery, API, Log request features as desired

End Device test with AT Commands
```
AT+DI
00-11-00-11-00-11-00-11

OK
AT+NI
6c-4e-ef-66-f4-79-86-a6
Passphrase: 'MultiTech'

OK
AT+NK
1f.33.a1.70.a5.f1.fd.a0.ab.69.7a.ae.2b.95.91.6b
Passphrase: 'MultiTech'

OK
AT+JOIN
Successfully joined network

OK
AT+SEND TESTDATA

OK
AT+SEND TESTDATA

OK
```

Testing published messages from a client connected to the same broker.
```
admin@mtcap3:~$ mosquitto_sub -v -h test.mosquitto.org -t "lorawan/#"
lorawan/8ffa106f-c751-cfcd-6300-80918d516837/init {"gateways_euis": ["00-80-00-00-00-01-b5-fc"], "time": "2023-09-13T16:49:44.960920Z", "api_version": "1.1", "app_version": "1.0.6"}
lorawan/8ffa106f-c751-cfcd-6300-80918d516837/6c-4e-ef-66-f4-79-86-a6/00-11-00-11-00-11-00-11/joined {"joineui": "6c-4e-ef-66-f4-79-86-a6", "appeui": "6c-4e-ef-66-f4-79-86-a6", "gweui": "00-80-00-00-00-01-b5-fc", "remote": false, "time": "2023-09-13T16:52:20.498194Z", "deveui": "00-11-00-11-00-11-00-11"}
lorawan/8ffa106f-c751-cfcd-6300-80918d516837/6c-4e-ef-66-f4-79-86-a6/00-11-00-11-00-11-00-11/up {"tmst":174882291,"chan":0,"rfch":0,"freq":921.4,"stat":1,"modu":"LORA","datr":"SF7BW125","codr":"4/5","lsnr":9.5,"rssi":-47,"opts":"","size":2,"fcnt":0,"cls":0,"port":1,"mhdr":"4049cd4601800000","data":"AAA=","appeui":"6c-4e-ef-66-f4-79-86-a6","joineui":"6c-4e-ef-66-f4-79-86-a6","deveui":"00-11-00-11-00-11-00-11","devaddr":"0146cd49","ack":false,"adr":true,"gweui":"00-80-00-00-00-01-b5-fc","seqn":0,"time":"2023-09-13T16:52:25.469281Z"}
lorawan/8ffa106f-c751-cfcd-6300-80918d516837/6c-4e-ef-66-f4-79-86-a6/00-11-00-11-00-11-00-11/up {"tmst":190169164,"chan":7,"rfch":1,"freq":924.6,"stat":1,"modu":"LORA","datr":"SF7BW125","codr":"4/5","lsnr":11.2,"rssi":-49,"opts":"0907030703070307030703","size":2,"fcnt":1,"cls":0,"port":1,"mhdr":"4049cd46018b0100","data":"AXM=","appeui":"6c-4e-ef-66-f4-79-86-a6","joineui":"6c-4e-ef-66-f4-79-86-a6","deveui":"00-11-00-11-00-11-00-11","devaddr":"0146cd49","ack":false,"adr":true,"gweui":"00-80-00-00-00-01-b5-fc","seqn":1,"time":"2023-09-13T16:52:40.760806Z"}
lorawan/8ffa106f-c751-cfcd-6300-80918d516837/close (null)
```
