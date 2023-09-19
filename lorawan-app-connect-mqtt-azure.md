# MQTT Application

## Application configuration
    url: mqtts://test.mosquitto.org:8883
    eui: filter end-devices joined to AppNet EUI
    encodeHex: encode payloads to and from hex instead of default base64
    options:
      check_hostname: enable hostname check for tls connection
      client_id: client id
      server_cert: server certificate
      client_cert: client certicate
      apikey: client private key
      username: MQTT username
      password: MQTT password
      uplinkTopic: override topic for uplinks to publish to
      downlinkTopic: override topic to subscribe to for downlinks
      overrideTopicsForAllApps: use custom topics for all apps
    requestOptions:
      api: enable api requests
      log: enable log requests
      lora: enable lora query requests
    backhaulDetect:
      enabled: turn backhaul detect messages on/off
      payload: hex payload of downlink packet to send to end-device when backhaul is down
      port: port to send downlink packet to end-device
      timeout: scheduling timeout limit the number of downlinks scheduled, default one per 5 minutes

## Azure
Azure MQTT messages are expected to use the following topics where '%(client_id)s' is the id of the gateway in the Azure cloud.
Messages on these topics will match the those shown in the MQTT protocol for uplinks and downlinks. Azure SAS tokens can be used as shown below, SAS tokens can be created using Azure CLI, VS Code Extension or Azure IoT Explorer. TLS Certificates and Keys can also be used instead of SAS tokens. TLS security is recommended for production systems.

![mPower Default App Settings](/images/DEFAULT-APP-TOP.png)


  * username: HUBNAME.azure-devices.net/DEVICEID/?api-version=2021-04-12
  * password: SharedAccessSignature sr=HUBNAME.azure-devices.net%2Fdevices%2FDEVICEID3&sig=6qNZSUClW0dv...BFn%2BdQ%3D&se=1710875213
  * uplinkTopic: devices/%(client_id)s/messages/events/"
  * downlinkTopic: devices/%(client_id)s/messages/devicebound/#"

![mPower Default App Settings](/images/DEFAULT-APP-BOTTOM.png)


See [Azure IoT Hub MQTT Support](https://learn.microsoft.com/en-us/azure/iot-hub/iot-hub-mqtt-support) for protocol connection details.
More info on SAS tokens can be found here [Azure IoT Hub SAS](https://learn.microsoft.com/en-us/azure/iot-hub/iot-hub-dev-guide-sas?tabs=node).

  ### Downlinks using Cloud-to-device messages
  Send a C2D message containing the appeui, deveui and data to send in a downlink. Optional fields: port, rx_wnd, ack, ack_retries see details in downlinks section of subscribed MQTT messages.
  ```json
  { "appeui": "8b-6c-f0-8e-ee-df-1b-b6", "deveui": "00-80-00-ff-ff-00-00-03", "data": "QA==" }
  ```

  ### Direct Methods

  Azure direct methods can be used to access gateway configuration and status info. The AppEUI and GwUUID need to be provided in the request. The payload for method requests and responses is a JSON document up to 128 KB.

  * LoRa Query Request
    * Direct method name "lora_req"
    * Direct method payload
    ```json
    { "command":"device list json" }
    ```

  * Log Request
    * Direct method name "log_req"
    * Direct method payload
    ```json
    { "file":"/var/log/messages", "filter": "lora-net", "lines":10 }
    ```

  * API Request
    * Direct method name "api_req"
    * Direct method payload
    ```json
    { "method":"GET", "path":"/api/system", "body":"" }
    ```

  * Downlink
    * Queue a downlink to a device, see details in downlinks section of subscribed MQTT messages for optional parameters.
    * Direct method name "downlink"
    * Direct method payload
    ```json
    { "deveui": "00-80-00-ff-ff-00-00-03", "data": "QA==" }
    ```
