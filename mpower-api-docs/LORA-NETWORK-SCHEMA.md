# LoRa Network Configuration Schema

Complete schema documentation for the `loraNetwork` resource in the mPower Device API.

## Overview

The `loraNetwork` resource is one of the most complex configurations in the API, supporting multiple operational modes and extensive configuration options for LoRa/LoRaWAN network management.

## Top-Level Properties

| Property | Type | Description |
|----------|------|-------------|
| `db` | string | Database file path |
| `backupInterval` | integer | Database backup interval in seconds |
| `trimInterval` | integer | Database trim interval in seconds |
| `trimRows` | integer | Number of rows to trim |

## Main Configuration Objects

### 1. `lora` - LoRa Network Server Configuration

Core LoRa network server settings when operating in Network Server mode.

#### Basic Settings

| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable/disable LoRa network server |
| `enableAdr` | boolean | Enable Adaptive Data Rate |
| `adrNBTrans` | integer | ADR number of transmissions |
| `adrPERNBTransEnabled` | boolean | Enable ADR PER NB Trans |
| `adrPERNBTransThreshold` | number | ADR PER NB Trans threshold (0.0-1.0) |
| `basicStationMode` | boolean | Enable Basic Station mode |
| `packetForwarderMode` | boolean | Enable Packet Forwarder mode |

#### Frequency Configuration

| Property | Type | Description |
|----------|------|-------------|
| `frequencyBand` | integer | Primary frequency band |
| `frequencyBand2` | integer | Secondary frequency band (dual card) |
| `channelPlan` | string | Channel plan (e.g., "US915", "EU868") |
| `channelMask` | string | Channel mask configuration |
| `frequencySubBand` | integer | Frequency sub-band (US915/AU915) |
| `frequencySubBand2` | integer | Secondary frequency sub-band |
| `frequencyEU` | integer | EU frequency in Hz |
| `frequencyEU2` | integer | Secondary EU frequency |
| `frequencyAS` | integer | Asia frequency in Hz |
| `frequencyAS2` | integer | Secondary Asia frequency |
| `frequencyKR` | integer | Korea frequency in Hz |
| `frequencyKR2` | integer | Secondary Korea frequency |
| `frequencyIN` | integer | India frequency in Hz |
| `frequencyIN2` | integer | Secondary India frequency |
| `frequencyRU` | integer | Russia frequency in Hz |
| `frequencyRU2` | integer | Secondary Russia frequency |
| `frequencyISM2400` | integer | ISM 2.4GHz frequency |
| `frequencyISM2400_2` | integer | ISM 2.4GHz frequency (card 2) |
| `frequencyISM2400_3` | integer | ISM 2.4GHz frequency (card 3) |
| `frequencyISM2400_2_1` | integer | ISM 2.4GHz frequency (card 2, channel 1) |
| `frequencyISM2400_2_2` | integer | ISM 2.4GHz frequency (card 2, channel 2) |
| `frequencyISM2400_2_3` | integer | ISM 2.4GHz frequency (card 2, channel 3) |

#### Radio Configuration

| Property | Type | Description |
|----------|------|-------------|
| `maxTxPower` | integer | Maximum transmit power in dBm |
| `antennaGain` | integer | Antenna gain in dBi |
| `rssiOffset` | number | RSSI offset value |
| `rx1DatarateOffset` | integer | RX1 datarate offset |
| `rx2Datarate` | integer | RX2 datarate |
| `rx1Delay` | integer | RX1 delay in seconds |
| `minDatarate` | integer | Minimum datarate |
| `maxDatarate` | integer | Maximum datarate |
| `ADRStep` | integer | ADR step size |

#### Packet Forwarder Configuration

| Property | Type | Description |
|----------|------|-------------|
| `packetForwarderConfig` | string | Packet forwarder JSON configuration (max 10000 chars) |
| `packetForwarderConfig2` | string | Secondary packet forwarder configuration |
| `skipPacketForwarderFieldCheck` | boolean | Skip field validation |
| `reducedPacketUpdates` | boolean | Reduce packet update frequency |

#### Network Settings

| Property | Type | Description |
|----------|------|-------------|
| `netID` | string | Network ID |
| `deviceQueueSize` | integer | Device queue size |
| `joinDelay` | integer | Join delay in seconds |
| `maxEIRP` | integer | Maximum EIRP |
| `dwelltimeUp` | integer | Uplink dwell time |
| `dwelltimeDown` | integer | Downlink dwell time |
| `enableStrictCounterValidation` | boolean | Enable strict frame counter validation |
| `networkLeadTime` | integer | Network lead time in milliseconds |
| `maxRx2PacketSize` | integer/null | Maximum RX2 packet size |

#### Class B Beacon Configuration

| Property | Type | Description |
|----------|------|-------------|
| `beaconFrequency` | integer | Beacon frequency in Hz |
| `beaconPower` | integer | Beacon power in dBm |
| `beaconFreqHop` | boolean | Enable beacon frequency hopping |
| `beaconInterval` | integer | Beacon interval in seconds |
| `beaconInfoDesc` | integer | Beacon info descriptor |
| `beaconLatitude` | number | Beacon latitude |
| `beaconLongitude` | number | Beacon longitude |
| `pingSlotFrequency` | integer | Ping slot frequency in Hz |
| `pingSlotDatarate` | integer | Ping slot datarate |
| `pingSlotFreqHop` | boolean | Enable ping slot frequency hopping |

#### Listen Before Talk (LBT)

| Property | Type | Description |
|----------|------|-------------|
| `lbtEnabled` | boolean | Enable LBT |
| `lbtRssiOffset` | integer | LBT RSSI offset |
| `lbtRssiTarget` | integer | LBT RSSI target |

#### Hardware-Specific Settings

| Property | Type | Description |
|----------|------|-------------|
| `calAD9361` | integer | AD9361 calibration value |
| `calTempRoom` | integer | Room temperature calibration |
| `diversity` | boolean | Enable antenna diversity |
| `dspStatInterval` | integer | DSP statistics interval |
| `fskSYNC` | string | FSK sync word |
| `gpsReceiver` | boolean | Enable GPS receiver |
| `nbDSP` | integer | Number of DSP cores |
| `spi_device` | string | SPI device path |
| `spi_device1261` | string | SPI device path for SX1261 |
| `fineTimestampEnable` | boolean | Enable fine timestamp |
| `fineTimestampMode` | string | Fine timestamp mode |

#### Duty Cycle

| Property | Type | Description |
|----------|------|-------------|
| `country` | string | Country code for duty cycle |
| `dutyCycleEnabled` | boolean | Enable duty cycle |
| `dutyCyclePeriod` | integer | Duty cycle period (1-86400 seconds) |
| `dutyCycleRatio` | number | Duty cycle ratio (0.0-1.0) |

#### Advanced Settings

| Property | Type | Description |
|----------|------|-------------|
| `locked` | boolean | Configuration locked status |
| `locking` | boolean | Configuration locking in progress |
| `classCAckTimeout` | integer | Class C ACK timeout |
| `aesKey` | string | AES encryption key |
| `ftsMatchCRCError` | boolean | Fine timestamp match on CRC error |
| `ftsVersion` | integer | Fine timestamp version |

### 2. `packetForwarder` - Packet Forwarder Mode Configuration

Configuration when operating in Packet Forwarder mode (forwarding to external network server).

| Property | Type | Description |
|----------|------|-------------|
| `antennaGain` | integer | Antenna gain in dBi |
| `channelPlan` | string | Channel plan |
| `downstreamPort` | integer | Downstream UDP port |
| `downstreamPort2` | integer | Secondary downstream port |
| `upstreamPort` | integer | Upstream UDP port |
| `upstreamPort2` | integer | Secondary upstream port |
| `serverAddress` | string | Network server address (IP or domain) |
| `serverAddress2` | string | Secondary server address |
| `gwID` | string | Gateway ID (16-char hex) |
| `gwID2` | string | Secondary gateway ID |
| `gwSource` | integer | Gateway ID source |
| `public` | boolean | Public network |
| `manualMode` | boolean | Manual configuration mode |
| `path` | string | Configuration file path |
| `pathGeo` | string | Geolocation file path |
| `keepAliveInterval` | integer | Keep-alive interval in seconds |
| `pushTimeout` | integer | Push timeout in seconds |
| `statInterval` | integer | Statistics interval in seconds |
| `autoquitThreshold` | integer | Auto-quit threshold |
| `rssiOffset` | number | RSSI offset |
| `maxTxPower` | integer/null | Maximum TX power |

#### Frequency Settings (Packet Forwarder)

Same frequency properties as `lora` object above.

#### Beacon Settings (Packet Forwarder)

Same beacon properties as `lora` object above.

#### LBT Settings (Packet Forwarder)

| Property | Type | Description |
|----------|------|-------------|
| `lbtEnabled` | boolean | Enable LBT |
| `lbtDefaultChannels` | boolean | Use default LBT channels |
| `lbtFrequency0` through `lbtFrequency7` | integer | LBT channel frequencies |
| `lbtRssiOffset` | integer | LBT RSSI offset |
| `lbtRssiTarget` | integer | LBT RSSI target |
| `lbtScanTime` | integer | LBT scan time in microseconds |

#### CRC Forwarding

| Property | Type | Description |
|----------|------|-------------|
| `fwdCrcValid` | boolean | Forward packets with valid CRC |
| `fwdCrcError` | boolean | Forward packets with CRC errors |
| `fwdCrcDisabled` | boolean | Forward packets with CRC disabled |

#### Spreading Factor

| Property | Type | Description |
|----------|------|-------------|
| `spreadingFactorEnable` | array | Enabled spreading factors (card 1) |
| `spreadingFactorEnable2` | array | Enabled spreading factors (card 2) |

#### Hardware-Specific (Packet Forwarder)

Same hardware properties as `lora` object above, plus:

| Property | Type | Description |
|----------|------|-------------|
| `locked` | boolean | Configuration locked |
| `locking` | boolean | Configuration locking in progress |
| `frequencyDR6` | integer | DR6 frequency |
| `frequencyDR7` | integer | DR7 frequency |
| `lowerDefaultFrequency` | boolean | Use lower default frequency |

### 3. `basicStation` - Basic Station Mode Configuration

Configuration for Semtech Basic Station protocol.

| Property | Type | Description |
|----------|------|-------------|
| `credentials` | string | LNS credentials (token or certificate) |
| `persistance` | boolean | Enable persistence |
| `uri` | string | LNS URI (e.g., "wss://lns.example.com:443") |
| `conf` | string | Station configuration JSON |
| `trust` | string | Trusted CA certificate |
| `cert` | string | Client certificate |
| `key` | string | Client private key |
| `sign` | string | Signature |

#### Dual Card Support

| Property | Type | Description |
|----------|------|-------------|
| `credentials2` | string | Secondary LNS credentials |
| `uri2` | string | Secondary LNS URI |
| `conf2` | string | Secondary station configuration |
| `trust2` | string | Secondary trusted CA certificate |
| `cert2` | string | Secondary client certificate |
| `key2` | string | Secondary client private key |
| `sign2` | string | Secondary signature |

### 4. `chirpstack` - ChirpStack Configuration

Configuration for ChirpStack network server integration.

| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable ChirpStack mode |
| `mode` | integer | ChirpStack mode |
| `configs` | array | Array of ChirpStack configurations |

#### ChirpStack Config Object

Each config in the `configs` array contains:

**General Settings:**
```yaml
general:
  logLevel: integer (0-5)
  logToSyslog: boolean
```

**Backend Settings:**
```yaml
backend:
  type: string ("basic_station" or "semtech_udp")
  basicStation:
    bind: string (e.g., ":3001")
    region: string (e.g., "US915")
    frequencyMin: integer
    frequencyMax: integer
    frequencySubBand: integer
    frequencyEU: integer
    frequencyAS: integer
    frequencyKR: integer
    frequencyIN: integer
    frequencyRU: integer
    pingInterval: string (e.g., "1m")
    readTimeout: string (e.g., "2m")
    writeTimeout: string (e.g., "2m")
    statInterval: string (e.g., "30s")
    timesyncInternal: string (e.g., "1h")
    caCert: string
    tlsCert: string
    tlsKey: string
    concentrators:
      multi_sf:
        frequencies: array of integers
      lora_std:
        frequency: integer
        bandwidth: integer
        spreading_factor: integer
      fsk:
        frequency: integer
  semtechUdp:
    udpBind: string (e.g., "0.0.0.0:1700")
    fakeRxTime: boolean
    skipCrcCheck: boolean
```

**Integration Settings:**
```yaml
integration:
  marshaler: string ("json" or "protobuf")
  mqtt:
    auth:
      type: string ("generic" or "azure_iot_hub")
      generic:
        servers: array of strings (MQTT broker URLs)
        username: string
        password: string
        qos: integer (0-2)
        cleanSession: boolean
        clientId: string
        caCert: string
        tlsCert: string
        tlsKey: string
      azureIotHub:
        deviceConnectionString: string
        deviceId: string
        hostname: string
        sasTokenExpiration: string
        tlsCert: string
        tlsKey: string
    eventTopicTemplate: string
    commandTopicTemplate: string
    stateTopicTemplate: string
    stateRetained: boolean
    keepAlive: string (e.g., "30s")
    maxReconnectInterval: string (e.g., "1m")
    terminateOnConnectError: boolean
```

**Filters:**
```yaml
filters:
  netIds: array of strings
  joinEuis: array of strings
```

### 5. `defaultApp` - Default Application Configuration

Configuration for the built-in default application (HTTP/MQTT forwarding).

| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable default app |
| `apiVersion` | string | API version |
| `eui` | string | Application EUI |
| `url` | string | Destination URL |
| `module` | string | Module name |
| `encodeHex` | boolean | Encode payload as hex |

#### Backhaul Detection

```yaml
backhaulDetect:
  enabled: boolean
  port: integer
  payload: string
  timeout: integer
```

#### Logging

```yaml
log:
  destination: string ("file" or "syslog")
  level: integer (0-5)
  path: string
```

#### Options

```yaml
options:
  apikey: string
  username: string
  password: string
  server_cert: string
  client_cert: string
  clean_session: boolean
  check_hostname: boolean
  overrideTopicsForAllApps: boolean
  uplinkTopic: string
  downlinkTopic: string
  cloudService: string
```

#### Request Options

```yaml
requestOptions:
  api: boolean
  lora: boolean
  log: boolean
```

### 6. `network` - Network Server Settings

Local join server and network configuration.

| Property | Type | Description |
|----------|------|-------------|
| `localJoinServerEnabled` | boolean | Enable local join server |
| `public` | integer | Public network flag |
| `leasetime` | integer | Address lease time |
| `passphrase` | string | Network passphrase |
| `name` | string | Network name |
| `eui` | string | Network EUI |
| `key` | string | Network key |
| `baseKey` | string | Base key for key derivation |
| `salt` | string | Salt for key derivation |
| `joinServer` | string | External join server URL |
| `joinServerCheck` | boolean | Verify join server certificate |
| `joinServerCert` | string | Join server CA certificate |
| `joinServerCertFile` | string | Join server cert file path |
| `joinServerCertUseFile` | boolean | Use cert file instead of inline |
| `joinServerClientCert` | string | Client certificate for join server |
| `joinServerClientKey` | string | Client key for join server |
| `joinServerAppEUI` | string | Application EUI for join server |
| `defaultProfile` | string | Default device profile |
| `defaultDeviceProfile` | string | Default device profile ID |
| `gatewayTimeoutThreshold` | integer | Gateway timeout threshold |

#### LENS Integration

| Property | Type | Description |
|----------|------|-------------|
| `lensEnabled` | boolean | Enable LENS integration |
| `lensCheckinInterval` | integer | Check-in interval |
| `lensServer` | string | LENS server URL |
| `lensNetworkStats` | boolean | Send network statistics |
| `lensGatewayStats` | boolean | Send gateway statistics |
| `lensPacketPayloadData` | boolean | Send packet payload data |
| `lensPacketMetadata` | boolean | Send packet metadata |
| `lensLocalJoinMetadata` | boolean | Send local join metadata |
| `lensDeviceHQ` | boolean | Enable DeviceHQ integration |

### 7. `whitelist` - Device Whitelist

Local device credentials for OTAA joins.

| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable whitelist |
| `devices` | array | Array of device credentials |

#### Device Object

```yaml
devices:
  - name: string
    deveui: string (16-char hex)
    appeui: string (16-char hex)
    appkey: string (32-char hex)
    genappkey: string (32-char hex, optional)
    class: string ("A", "B", or "C")
    device_profile_id: string (optional)
    network_profile_id: string (optional)
```

### 8. `addressRange` - Device Address Range

ABP device address range allocation.

| Property | Type | Description |
|----------|------|-------------|
| `start` | string | Start address (8-char hex) |
| `end` | string | End address (8-char hex) |

### 9. `udp` - UDP Configuration

UDP port configuration for packet forwarder protocol.

| Property | Type | Description |
|----------|------|-------------|
| `allowPublic` | boolean | Allow public network access |
| `upstreamPort` | integer | Upstream port (default: 1700) |
| `downstreamPort` | integer | Downstream port (default: 1700) |
| `appPortUp` | integer | Application upstream port |
| `appPortDown` | integer | Application downstream port |

### 10. `mqtt` - MQTT Broker Configuration

Local MQTT broker settings.

| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable MQTT broker |
| `host` | string | Broker host (IP address) |
| `port` | integer | Broker port (default: 1883) |
| `username` | string | MQTT username |
| `password` | string | MQTT password |
| `trimEui` | boolean | Trim leading zeros from EUIs |

### 11. `trafficManager` - Traffic Manager

Filter traffic by EUI.

| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable traffic manager |
| `join_eui_filters` | array | Array of Join EUI filters |
| `dev_eui_filters` | array | Array of Device EUI filters |

### 12. `radiobridgeConsole` - RadioBridge Console

RadioBridge cloud console integration.

| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable RadioBridge console |
| `name` | string | Gateway name |
| `region` | string | Region |
| `token` | string | Authentication token (max 500 chars) |
| `validated` | boolean | Token validation status |

### 13. `redundancy` - Gateway Redundancy

High-availability configuration for dual gateway setup.

| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable redundancy |
| `role` | string | Gateway role ("primary" or "secondary") |
| `primaryAddress` | string | Primary gateway IP |
| `secondaryAddress` | string | Secondary gateway IP |
| `defaultInterval` | integer | Default sync interval |
| `failoverInterval` | integer | Failover check interval |

### 14. `log` - Logging Configuration

LoRa network server logging settings.

| Property | Type | Description |
|----------|------|-------------|
| `console` | boolean | Log to console |
| `syslog` | boolean | Log to syslog |
| `level` | integer | Log level (0-5) |
| `path` | string | Log file path |

### 15. `spectralScan` - Spectral Scan

RF spectral scanning configuration.

| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable spectral scan |
| `bandwidth` | integer | Scan bandwidth |
| `duration` | integer | Scan duration |
| `floor` | integer | Noise floor |
| `imme` | boolean | Immediate scan |
| `interval` | integer | Scan interval |
| `limit` | integer | Result limit |
| `offset` | integer | Frequency offset |
| `samples` | integer | Number of samples |
| `startAt` | string | Start time |
| `step` | integer | Frequency step |
| `stopCriteria` | integer | Stop criteria |
| `ranges` | array | Frequency ranges to scan |

#### Range Object

```yaml
ranges:
  - start: integer (frequency in Hz)
    stop: integer (frequency in Hz)
```

### 16. `test` - Test Mode Settings

Test and debug settings.

| Property | Type | Description |
|----------|------|-------------|
| `disableRxJoin1` | boolean | Disable RX1 for join |
| `disableRxJoin2` | boolean | Disable RX2 for join |
| `disableRxWindow1` | boolean | Disable RX1 window |
| `disableRxWindow2` | boolean | Disable RX2 window |
| `disableDutyCycle` | boolean | Disable duty cycle |
| `disableGPS` | boolean | Disable GPS |

## Example Configurations

### Example 1: Network Server Mode (US915)

```json
{
  "lora": {
    "enabled": true,
    "enableAdr": true,
    "packetForwarderMode": false,
    "basicStationMode": false,
    "channelPlan": "US915",
    "frequencySubBand": 2,
    "maxTxPower": 27,
    "antennaGain": 3,
    "netID": "000013"
  },
  "network": {
    "localJoinServerEnabled": true,
    "public": 1,
    "name": "My LoRaWAN Network",
    "eui": "0011223344556677",
    "key": "00112233445566778899AABBCCDDEEFF"
  },
  "whitelist": {
    "enabled": true,
    "devices": [
      {
        "name": "Sensor 1",
        "deveui": "0011223344556677",
        "appeui": "7766554433221100",
        "appkey": "00112233445566778899AABBCCDDEEFF00",
        "class": "A"
      }
    ]
  }
}
```

### Example 2: Packet Forwarder Mode

```json
{
  "lora": {
    "enabled": false,
    "packetForwarderMode": true
  },
  "packetForwarder": {
    "channelPlan": "EU868",
    "serverAddress": "lns.example.com",
    "upstreamPort": 1700,
    "downstreamPort": 1700,
    "gwID": "AA555A0000000101",
    "public": true,
    "keepAliveInterval": 10,
    "statInterval": 30
  }
}
```

### Example 3: Basic Station Mode

```json
{
  "lora": {
    "enabled": false,
    "basicStationMode": true
  },
  "basicStation": {
    "persistance": true,
    "uri": "wss://lns.example.com:443",
    "credentials": "token:ABCD1234...",
    "trust": "-----BEGIN CERTIFICATE-----\n...",
    "conf": "{\"radio_conf\": {...}}"
  }
}
```

### Example 4: ChirpStack Integration

```json
{
  "chirpstack": {
    "enabled": true,
    "mode": 0,
    "configs": [
      {
        "general": {
          "logLevel": 4,
          "logToSyslog": false
        },
        "backend": {
          "type": "semtech_udp",
          "semtechUdp": {
            "udpBind": "0.0.0.0:1700",
            "fakeRxTime": false,
            "skipCrcCheck": false
          }
        },
        "integration": {
          "marshaler": "json",
          "mqtt": {
            "auth": {
              "type": "generic",
              "generic": {
                "servers": ["tcp://localhost:1883"],
                "username": "chirpstack",
                "password": "chirpstack",
                "qos": 0,
                "cleanSession": true,
                "clientId": "chirpstack-gateway"
              }
            },
            "eventTopicTemplate": "gateway/{{ .GatewayID }}/event/{{ .EventType }}",
            "commandTopicTemplate": "gateway/{{ .GatewayID }}/command/#"
          }
        }
      }
    ]
  }
}
```

## API Operations

### Get LoRa Network Configuration

```bash
GET /api?fields=loraNetwork
```

### Update LoRa Network Configuration

```bash
PUT /api/loraNetwork
Content-Type: application/json

{
  "lora": {
    "enabled": true,
    "channelPlan": "US915",
    "frequencySubBand": 2
  }
}
```

### Update Whitelist Devices

```bash
# Add device
POST /api/loraNetwork/whitelist/devices
{
  "name": "New Sensor",
  "deveui": "0011223344556677",
  "appeui": "7766554433221100",
  "appkey": "00112233445566778899AABBCCDDEEFF00",
  "class": "A"
}

# Update device
PUT /api/loraNetwork/whitelist/devices/0
{
  "name": "Updated Sensor Name"
}

# Delete device
DELETE /api/loraNetwork/whitelist/devices/0
```

## Notes

1. **Mode Selection**: Only one mode can be active at a time:
   - Network Server mode: `lora.enabled = true`
   - Packet Forwarder mode: `lora.packetForwarderMode = true`
   - Basic Station mode: `lora.basicStationMode = true`
   - ChirpStack mode: `chirpstack.enabled = true`

2. **Dual Card Support**: Many properties have a `2` suffix for dual LoRa card configurations (e.g., `frequencyBand` and `frequencyBand2`).

3. **Channel Plans**: Supported channel plans include US915, EU868, AU915, AS923, KR920, IN865, RU864, and custom plans.

4. **Hex Format**: EUI and key fields accept hex strings with or without delimiters (`:`, `-`, `.`).

5. **Save Required**: After updating `loraNetwork`, call `POST /api/command/save` to persist changes.

6. **Service Restart**: Some changes may require restarting the LoRa service:
   ```bash
   POST /api/lora/restart
   ```

## Related Resources

- [LoRa Gateway Management](API-README.md#lora-management)
- [Quick Reference](API-QUICK-REFERENCE.md#lora-management)
- [Main API Documentation](API-README.md)
