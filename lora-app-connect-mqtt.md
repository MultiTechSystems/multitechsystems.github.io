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

* Publishes
  * lorawan/\<APP-EUI>/\<GW-UUID>/init
    ```json
    lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/init
    {
      "gateways_euis": ["00-80-00-00-d0-00-01-1a", "00-80-00-00-d0-00-01-ff"],
      "time": "2023-03-04T23:08:26.021832Z"
    }
    ```
  * lorawan/\<APP-EUI>/\<GW-UUID>/close
    ```json
    lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/close (null)
    ```
  * lorawan/\<APP-EUI>/\<GW-UUID>/disconnected
    ```json
    lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/disconnected (null)
    ```
  * lorawan/\<APP-EUI>/\<DEV-EUI>/up
    ```json
    lorawan/8b-6c-f0-8e-ee-df-1b-b6/00-80-00-ff-ff-00-00-03/up
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
    ```json
    lorawan/8b-6c-f0-8e-ee-df-1b-b6/00-80-00-ff-ff-00-00-03/joined
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
    ```json
    lorawan/029998E06156CDD44523264B523115C1/00-80-00-ff-ff-00-00-03/joined
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

* Subscribed
  * Topics to manage the downlink queue
    * https://www.multitech.net/developer/software/lora/lora-network-server/mqtt-messages/
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
      ```json
      $ mosquitto_pub -t lorawan/8b-6c-f0-8e-ee-df-1b-b6/00-80-00-ff-ff-00-00-03/clear -m ""
      ```
  * Topics to request gateway info
    * lorawan/\<APP-EUI>/\<GW-UUID>/lora_req - send request for lora-query utility
      * https://www.multitech.net/developer/software/lora/lora-network-server/
      * command - lora-query command to run
      ```json
      $ mosquitto_pub -t lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/lora_req -m '{"command":"device count"}'
      ```
      * response
      ```json
      lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/lora_res
      {
        "count" : 9
      }
      ```
      * request pages of up to 500 records
      ```json
      $ mosquitto_pub -t lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/lora_req -m '{"command":"device list json page 0"}'
      ```
      * response
      ```json
      lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/lora_res
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
      },
      ...
      ]
      ```
    * lorawan/\<APP-EUI>/\<GW-UUID>/log_req - send request for log file
      * lines - number of lines to returned
      * file - name of file to read from /var/log folder, only paths to /var/log are allowed.
      ```json
      $ mosquitto_pub -t lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/log_req -m '{"file":"/var/log/messages","lines":100}'
      ```
      * response
      ```json
      lorawan/8b-6c-f0-8e-ee-df-1b-b6/029998E06156CDD44523264B523115C1/log_res
      {"result": "2023-03-05T19:55:56.913366+00:00 mtcdt lora-app-connect: Call setup MQTT App\n2023-03-05T19:55:56.934340+00:00 mtcdt lora-app-connect: Setup MQTT App\n2023-03-05T19:55:57.014137+00:00 mtcdt lora-app-connect: MQTT connect mqtt://172.16.0.222:1883\n2023-03-05T19:55:59.985408+00:00 mtcdt lora-app-connect: Start client\n2023-03-05T19:56:00.039355+00:00 mtcdt lora-app..."}"
        ```

### Test brokers

* https://test.mosquitto.org/
  * Supports TLS1.2 server certificate, client certificate and apikey settings for authentication
  * Application configuration
    * url: mqtts://test.mosquitto.org:8883
    * options:
      * server_cert: server certificate
      * clent_cert: client certicate
      * apikey: client private key
* https://flespi.com/mqtt-api
  * Supports authentication using username access token for authentication
  * Application configuration
    * url: mqtt://mqtt.flespi.io
    * options:
      * username: \<FLESPI-TOKEN>