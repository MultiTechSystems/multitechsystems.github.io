# MQTT Application

## Application configuration
    url: mqtts://test.mosquitto.org:8883
    options:
      server_cert: server certificate
      clent_cert: client certicate
      apikey: client private key
      username: MQTT username
      password: MQTT password

## MQTT Protocol
### Publishes
  * lorawan/\<APP-EUI>/\<GW-UUID>/init
    ```
    lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/init
    ```
    ```json
    {
      "gateways_euis": ["00-80-00-00-d0-00-01-1a", "00-80-00-00-d0-00-01-ff"],
      "time": "2023-03-04T23:08:26.021832Z"
    }
    ```
  * lorawan/\<APP-EUI>/\<GW-UUID>/close
    ```
    lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/close (null)
    ```
  * lorawan/\<APP-EUI>/\<GW-UUID>/disconnected
    ```
    lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/disconnected (null)
    ```
  * lorawan/\<APP-EUI>/\<DEV-EUI>/up
    ```
    lorawan/8b-6c-f0-8e-ee-df-1b-b6/00-80-00-ff-ff-00-00-03/up
    ```
    ```json
    {
      "jver": 1,
      "tmst": 561224395,
      "time": "2023-03-04T23:14:39.522787Z",
      "tmms": 1362006897523,
      "chan": 6,
      "rfch": 1,
      "freq": 903.5,
      "mid": 8,
      "stat": 1,
      "modu": "LORA",
      "datr": "SF9BW125",
      "codr": "4/5",
      "rssis": -14,
      "lsnr": 9.2,
      "foff": -2769,
      "rssi": -13,
      "opts": "03070307",
      "size": 8,
      "fcnt": 1,
      "cls": 0,
      "port": 33,
      "mhdr": "80cb80d000840100",
      "data": "dGVzdGRhdGE=",
      "appeui": "8b-6c-f0-8e-ee-df-1b-b6",
      "deveui": "00-80-00-ff-ff-00-00-03",
      "name": "JSR-DEBIAN-PC-DOT2",
      "devaddr": "00d080cb",
      "ack": false,
      "adr": true,
      "gweui": "00-80-00-00-d0-00-01-ff",
      "seqn": 1
    }
    ```
  * lorawan/\<APP-EUI>/\<DEV-EUI>/joined
    ```
    lorawan/8b-6c-f0-8e-ee-df-1b-b6/00-80-00-ff-ff-00-00-03/joined
    ```
    ```json
    {
      "joineui": "16-ea-76-f6-ab-66-3d-80",
      "name": "JSR-DEBIAN-PC-DOT2",
      "appeui": "8b-6c-f0-8e-ee-df-1b-b6",
      "gweui": "00-80-00-00-d0-00-01-ff",
      "remote": true,
      "time": "2023-03-04T23:07:11.360064Z"
    }
    ```
  * lorawan/\<GW-UUID>/\<DEV-EUI>/joined
    ```
    lorawan/029998E06156CDD44523264B523115C1/00-80-00-ff-ff-00-00-03/joined
    ```
    ```json
    {
      "joineui": "16-ea-76-f6-ab-66-3d-80",
      "name": "JSR-DEBIAN-PC-DOT2",
      "appeui": "8b-6c-f0-8e-ee-df-1b-b6",
      "gweui": "00-80-00-00-d0-00-01-ff",
      "remote": true,
      "time": "2023-03-04T23:07:11.360064Z"
    }
    ```

  * lorawan/\<APP-EUI>/\<DEV-EUI>/moved
    * Array of GW-EUIs
    * ```json
      ["00-80-00-0d-00-00-00-01"]
      ```
  * lorawan/\<GW-UUID>/\<APP-EUI>/lora_res
  * lorawan/\<GW-UUID>/\<APP-EUI>/log_res
  * lorawan/\<GW-UUID>/\<APP-EUI>/api_res

### Subscribed
Subscribed topics allow communication to the gateway to issue downlinks, clear a downlink queue or request info.
  * ***It is recomended to use topic access control to limit the clients that are able to make calls to the gateway to your back-end services***
    * [Mosquitto Dynamic Security](https://mosquitto.org/documentation/dynamic-security/)
    * [MQTT Multiple User Accounts](https://blog.jaimyn.dev/mqtt-use-acls-multiple-user-accounts/)
  * Topics to manage the downlink queue
    * [Multitech LNS MQTT Messages](https://www.multitech.net/developer/software/lora/lora-network-server/mqtt-messages/)
    * lorawan/\<APP-EUI>/\<DEV-EUI>/+
    * lorawan/\<GW-EUI>/\<DEV-EUI>/+
    * lorawan/\<GW-UUID>/\<DEV-EUI>/+
    * Supported topics
      * down - downlinks to schedule
        * Fields
          * data – base64 encoded data to be sent in downlink packet payload. If Encode Payload as HEX is selected in Default App configuration a HEX payload should be provided for downlink as well.
          * port – AppPort, 8-bit integer value to send packet, use 0 to send MAC commands.
          * ack – optional, request ACK of receipt of packet by end device. If enabled the end device should send an ACK in its next uplink packet after receiving the downlink. If the ACK is not seen in the uplink the same downlink packet may be sent again, depending on retry settings. In Class C the ACK will be expected soon after the downlink is transmitted, i.e. five seconds, otherwise a retry will be sent.
          * ack_retries – optional, number of re-transmissions to perform in sending the downlink. If network server is unable to schedule the downlink it will not be counted against the retries.
          * rx_wnd – optional, specify Rx window downlink should be scheduled for values (0, 1 or 2). If provided, and not 0, the network server will wait until the packet can be scheduled for the specified window before sending the packet to the radio for transmission. The packet will remain in the queue until scheduled. Set to 0 for Class C devices.
        ```json
        $ mosquitto_pub -t lorawan/8b-6c-f0-8e-ee-df-1b-b6/00-80-00-ff-ff-00-00-03/down -m '{"data":"QA=="}'
      * clear - clear downlinks for a device
      ```
      $ mosquitto_pub -t lorawan/8b-6c-f0-8e-ee-df-1b-b6/00-80-00-ff-ff-00-00-03/clear -m ""
      ```


## Topics to request gateway info
### LoRa Query
  * lorawan/\<APP-EUI>/\<GW-UUID>/lora_req - send request for lora-query utility
    * [Multitech LNS LoRa Query](https://www.multitech.net/developer/software/lora/lora-network-server/)
    * command - lora-query command to run
    * Example: retrieve count of device records
    ```
    $ mosquitto_pub -t lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/lora_req -m '{"command":"device count"}'
    ```
    * response
    ```
    lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/lora_res
    ```
    ```json
    {
      "count" : 9
    }
    ```
    * Example: request pages of up to 500 records
    ```
    $ mosquitto_pub -t lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/lora_req -m '{"command":"device list json page 0"}'
    ```
    * response
    ```
    lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/lora_res
    ```
    ```json
    [{
      "class" : "A",
      "created_at" : "2022-12-27T19:40:37Z",
      "deveui" : "00-80-00-00-04-00-59-04",
      "device_profile_id" : 0,
      "device_profile_updated_at" : "2022-12-27T19:40:37Z",
      "firmware_version" : "",
      "hardware_version" : "",
      "last_app_nonce" : 9,
      "last_nonce" : 14,
      "last_seen" : "",
      "name" : "",
      "network_profile_id" : 1,
      "network_profile_updated_at" : "2022-12-27T19:40:37Z",
      "product_id" : "",
      "rejoin_count" : 0,
      "serial_number" : "",
      "tags" : ""
    }]
    ```

### Log Requests
  * lorawan/\<APP-EUI>/\<GW-UUID>/log_req - send request for log file
    * lines - number of lines to returned
    * file - name of file to read from /var/log folder, only paths to /var/log are allowed.
    ```
    $ mosquitto_pub -t lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/log_req -m '{"file":"/var/log/messages","lines":100}'
    ```
    * response
    ```
    lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/log_res
    ```
    ```json
    {"result": "2023-03-05T19:55:56.913366+00:00 mtcdt lora-app-connect: Call setup MQTT App\n2023-03-05T19:55:56.934340+00:00 mtcdt lora-app-connect: Setup MQTT App\n2023-03-05T19:55:57.014137+00:00 mtcdt lora-app-connect: MQTT connect mqtt://172.16.0.222:1883\n2023-03-05T19:55:59.985408+00:00 mtcdt lora-app-connect: Start client\n2023-03-05T19:56:00.039355+00:00 mtcdt lora-app..."}"
      ```

### API Requests
  API Requests can get or change any configuration settings, restart services or reboot the gateway
  * [Multitech mPower API](https://www.multitech.net/developer/software/mtr-software/mtr-api-reference/)
  * lorawan/\<APP-EUI>/\<GW-UUID>/api_req - send request for log file
    * method - GET, PUT, POST or DELETE
    * path - API path to call
    * body - JSON object to pass to the API
    ```
    $ mosquitto_pub -t lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/api_req -m '{"method":"GET","path":"/api/loraNetwork","body":""}'
    ```
    * response
    ```
    lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/api_res
    ```
    ```json
    {
        "code" : 200,
        "result" :
        {
          "__v" : 1,
          "addressRange" :
          {
                  "end" : "FF:FF:FF:FE",
                  …
          },
          …
        }
    }
    ```
    *	Change LoRa Network Settings
    ```json
    {
      "method": "PUT",
      "path": "/api/loraNetwork/lora",
      "body": "{\"channelPlan\":\"US915\"}"
    }
    ```
    *	List device DevEUI and AppKey
    ```json
    {
      "method": "GET",
      "path": "/api/loraNetwork/whitelist/devices",
      "body": ""
    }
    ```
    *	Create device DevEUI and AppKey record
    ```json
    {
      "method": "POST",
      "path": "/api/loraNetwork/whitelist/devices",
      "body": "{\"deveui\": \"00800000FFFF0001\",\"appeui\": \"0080000000000017\",\"appkey\": \"00800000FFFF000100800000FFFF0001\",\"class\": \"A\"}"
    }
    ```
    *	Remove device DevEUI and AppKey, provide index or DEVEUI to record to delete
    ```json
    {
      "method": "DELETE",
      "path": "/api/loraNetwork/whitelist/devices/0",
      "body": ""
    }
    ```
    Or
    ```json
    {
      "method": "DELETE",
      "path": "/api/loraNetwork/whitelist/devices/00800000FFFF0001",
      "body": ""
    }
    ```
    * Restart LoRa services
    ```json
    {
      "method":"POST",
      "path":"/api/lora/restart",
      "body":""
    }
    ```

### Test brokers

* [Mosquitto Test Broker](https://test.mosquitto.org/)
  * Supports TLS1.2 server certificate, client certificate and apikey settings for authentication
  * Application configuration
    * url: mqtts://test.mosquitto.org:8883
    * options:
      * server_cert: server certificate
      * clent_cert: client certicate
      * apikey: client private key
* [FLESPI Test Broker](https://flespi.com/mqtt-api)
  * Supports authentication using username access token for authentication
  * Application configuration
    * url: mqtt://mqtt.flespi.io
    * options:
      * username: \<FLESPI-TOKEN>