# Schema Analysis Working Document

This document tracks the progress of extracting accurate API schema information from the mtsDeviceAPI and mtsDeviceUI source code.

**Source Locations:**
- Schema files: mtsDeviceAPI `json/strict/`
- API controllers: mtsDeviceAPI `src/`
- UI components: mtsDeviceUI `src/`
- API tests: mtsDeviceAPITests

---

## Analysis Progress

### Core Resources (API-SCHEMAS-REFERENCE.md)

| Resource | Schema | Doc Status |
|----------|--------|------------|
| system | ✅ | ✅ Complete |
| users | ✅ | ✅ Complete |
| cellular | ✅ | ✅ Complete |
| wifi | ✅ | ✅ Complete |
| dns | ✅ | ✅ Complete |
| dhcp | ✅ | ✅ Complete |
| routes | ✅ | ✅ Complete |
| firewall | ✅ | ✅ Complete |
| nat | ✅ | ✅ Complete |
| gps | ✅ | ✅ Complete |
| serial | ✅ | ✅ Complete |
| alert | ✅ | ✅ Complete |
| sms | ✅ | ✅ Complete (SMS-SCHEMA.md) |

### VPN Resources (VPN-SCHEMAS.md)

| Resource | Schema | Doc Status |
|----------|--------|------------|
| ovpnTunnels | ✅ | ✅ Complete |
| ipsecTunnels | ✅ | ✅ Complete |
| greTunnels | ✅ | ✅ Complete |

### Network Services (NETWORK-SERVICES-SCHEMAS.md)

| Resource | Schema | Doc Status |
|----------|--------|------------|
| ni | ✅ | ✅ Complete |
| snmp | ✅ | ✅ Complete |
| sntp | ✅ | ✅ Complete |
| smtp | ✅ | ✅ Complete |
| loraNetwork | ✅ | ✅ Complete (complex) |

### Device Management (DEVICE-MANAGEMENT-SCHEMAS.md)

| Resource | Schema | Doc Status |
|----------|--------|------------|
| autoReboot | ✅ | ✅ Complete |
| remoteMgmt | ✅ | ✅ Complete |
| remoteAccess | ✅ | ✅ Complete |
| resetButton | ✅ | ✅ Complete |
| saveAndRestore | ✅ | ✅ Complete |
| customApps | ✅ | ✅ Complete |
| customAppsConfig | ✅ | ✅ Complete |
| customRoles | ✅ | ✅ Complete |
| docker | ✅ | ✅ Complete |
| passwordComplexityRules | ✅ | ✅ Complete |

### Connectivity (CONNECTIVITY-SCHEMAS.md)

| Resource | Schema | Doc Status |
|----------|--------|------------|
| bluetooth | ✅ | ✅ Complete |
| bluetoothLowEnergy | ✅ | ✅ Complete |
| btDevices | ✅ | ✅ Complete |
| ipPassthrough | ✅ | ✅ Complete |
| ipPipes | ✅ | ✅ Complete |
| ddns | ✅ | ✅ Complete |
| wanmngr | ✅ | ✅ Complete |
| waninfo | ✅ | ✅ Complete |
| backOffTimers | ✅ | ✅ Complete |
| cellTimeSync | ✅ | ✅ Complete |

### Advanced Features (ADVANCED-FEATURES-SCHEMAS.md)

| Resource | Schema | Doc Status |
|----------|--------|------------|
| filters | ✅ | ✅ Complete |
| scada | ✅ | ✅ Complete |
| mqttBroker | ✅ | ✅ Complete |
| radius | ✅ | ✅ Complete |
| secureProtocols | ✅ | ✅ Complete |
| trustedIp | ✅ | ✅ Complete |
| notificationEventGroup | ✅ | ✅ Complete |

### Logging and Status (LOGGING-STATUS-SCHEMAS.md)

| Resource | Schema | Doc Status |
|----------|--------|------------|
| eventlog | ✅ | ✅ Complete |
| syslog | ✅ | ✅ Complete |
| status | ✅ | ✅ Complete |
| lldp | ✅ | ✅ Complete |
| mdns | ✅ | ✅ Complete |
| policy | ✅ | ✅ Complete |
| extIo | ✅ | ✅ Complete |
| extUsb | ✅ | ✅ Complete |
| databaseInfo | ✅ | ✅ Complete |

### Additional Resources

| Resource | Schema | Doc Status |
|----------|--------|------------|
| brand | ✅ | ✅ Complete (DEVICE-MANAGEMENT-SCHEMAS.md) |

### Internal/Low Priority

| Resource | Schema | Doc Status |
|----------|--------|------------|
| legacyDefaults | ✅ | ⏭️ Skipped (internal use only) |

---

## Completed Analysis

### SMS Resource (Complete)

**Source Files Analyzed:**
- Schema: `mtsDeviceAPI/json/strict/sms.schema.json`
- Model: `mtsDeviceAPI/src/MTS_MTR_SMS.cpp`
- Controller: `mtsDeviceAPI/include/MTS_MTR_SMSController.h`
- Tests: `mtsDeviceAPITests/tests/sms/sms.test.js`
- UI Settings: `mtsDeviceUI/src/pages/sms/Settings.vue`
- UI Send/Receive: `mtsDeviceUI/src/pages/sms/SendAndReceive.vue`
- Types: `mtsDeviceUI/src/types/Sms.ts`

**Endpoints Verified:**
- `GET /api?fields=sms` - Get SMS configuration
- `PUT /api/sms` - Update SMS configuration
- `GET /api?fields=sms/inbox` - Get received messages
- `GET /api?fields=sms/outbox` - Get sent messages
- `POST /api/sms/outbox` - Send SMS message
- `DELETE /api/sms/inbox/{guid}` - Delete received message
- `DELETE /api/sms/inbox/all` - Delete all received messages
- `DELETE /api/sms/outbox/{guid}` - Delete sent message
- `DELETE /api/sms/outbox/all` - Delete all sent messages

**Schema (from source):**

```json
{
  "enabled": false,
  "sentLimit": 1000,
  "receivedLimit": 1000,
  "resendLimit": 0,
  "smsCommands": {
    "enabled": false,
    "appEnabled": false,
    "rebootEnabled": false,
    "checkinEnabled": false,
    "rmEnabled": false,
    "setCellularEnabled": false,
    "apnEnabled": false,
    "cellularStatusEnabled": false,
    "radioEnabled": false,
    "ethernetEnabled": false,
    "wanStatusEnabled": false,
    "wifiEnabled": false,
    "pingEnabled": false,
    "serialEnabled": false,
    "geopositionEnabled": false,
    "lnsRestartEnabled": false,
    "wanIpsEnabled": false,
    "getIoEnabled": false,
    "setIoEnabled": false,
    "password": {
      "enabled": true,
      "useCustomPassword": false,
      "customText": "",
      "defaultText": "<last 6 chars of IMEI>",
      "text": "<current password>"
    },
    "whitelist": {
      "enabled": false,
      "numbers": []
    }
  }
}
```

**Field Constraints (from schema):**
- `enabled`: boolean, default: false
- `sentLimit`: integer, min: 0, max: 1000, default: 1000
- `receivedLimit`: integer, min: 0, max: 1000, default: 1000
- `resendLimit`: integer, min: 0, max: 10, default: 0
- `smsCommands.password.customText`: string, maxLength: 250, minLength: 6 when enabled+useCustomPassword, no spaces allowed
- `smsCommands.whitelist.numbers`: array of strings, maxLength: 250 per number

**Business Rules (from code):**
- When `smsCommands.enabled` is true, either `password.enabled` OR `whitelist.enabled` must be true
- When `password.enabled` and `useCustomPassword` are both true, `customText` must be at least 6 characters with no spaces
- Default password is dynamically set to last 6 characters of device IMEI
- Whitelist can contain up to 50 numbers (UI validation)

**SMS Commands Available:**
| Command | Field | Description |
|---------|-------|-------------|
| `#reboot` | rebootEnabled | Reboot the device |
| `#checkin` | checkinEnabled | Check in to Device Manager |
| `#rm <enable\|disable>` | rmEnabled | Enable/disable remote management |
| `#setcellular <enable\|disable> [APN]` | setCellularEnabled | Enable/disable cellular, optionally set APN |
| `#ping [interface] [count] <address>` | pingEnabled | Ping an address |
| `#app <start\|stop\|restart> <id\|name>` | appEnabled | Control custom applications |
| `#apn` | apnEnabled | Get current APN value |
| `#cellular` | cellularStatusEnabled | Get cellular status |
| `#radio` | radioEnabled | Get radio statistics |
| `#ethernet` | ethernetEnabled | Get ethernet status |
| `#wan` | wanStatusEnabled | Get WAN interface status |
| `#wifi` | wifiEnabled | Get WiFi status |
| `#wanips` | wanIpsEnabled | Get WAN IP addresses |
| `#geoposition` | geopositionEnabled | Get GPS coordinates |
| `#serial` | serialEnabled | Get serial port status |
| `#lnsrestart` | lnsRestartEnabled | Restart LoRa Network Server |
| `#setio <do0> [value]` | setIoEnabled | Set digital output state |
| `#getio <di0\|do0>` | getIoEnabled | Get digital I/O state |

**Send SMS Request Format:**
```json
{
  "recipients": ["1234567890", "0987654321"],
  "message": "Message text (max 160 chars)"
}
```

**Outbox Message Format:**
```json
{
  "guid": "1923082400000058",
  "recipient": "1234567890",
  "status": "sending|sent|failed",
  "timestamp": "2025-02-10 14:30:00",
  "message": "Message text"
}
```

**Inbox Message Format:**
```json
{
  "guid": "1923082400000059",
  "source": "1234567890",
  "timestamp": "2025-02-10 14:35:00",
  "message": "Message text"
}
```

---

## OpenAPI/Swagger Documentation

**Status:** ✅ Complete (February 10, 2026)

The OpenAPI specification has been restructured into modular files:

### Directory Structure
```
mpower-api-docs/openapi/
├── api-documentation.yaml    # Main spec with $ref imports
├── schemas/                  # 14 schema files by domain
└── paths/                    # 14 path files by domain
```

### Generated Files
- `api-documentation-bundled.yaml` - Bundled single-file YAML
- `api-documentation-bundled.json` - Bundled single-file JSON (271 KB)

### Validation
- ✅ Validated with swagger-cli
- All schemas extracted from source `.schema.json` files
- Covers 100+ endpoints across 58+ resources

### Schema Files
| File | Contents |
|------|----------|
| common.yaml | Auth, responses, common types |
| system.yaml | System configuration |
| users.yaml | User management, roles, password rules |
| cellular.yaml | Cellular, SIM profiles, backoff timers |
| sms.yaml | SMS configuration and commands |
| network.yaml | WiFi, DNS, DHCP, routes, WAN, DDNS |
| vpn.yaml | OpenVPN, IPsec, GRE tunnels |
| firewall.yaml | Firewall, filters, NAT, trusted IP |
| lora.yaml | LoRa network server and devices |
| services.yaml | SNMP, NTP, SMTP, syslog, LLDP, mDNS, RADIUS |
| device-management.yaml | Auto reboot, remote access, Docker, apps |
| connectivity.yaml | Bluetooth, BLE, GPS, serial, IP pipes |
| advanced.yaml | SCADA, MQTT, alerts, I/O |
| status.yaml | Status, event log, policy |

---

## Notes

### Data Sources Priority
1. JSON Schema files (strict/) - Authoritative for structure and validation rules
2. C++ Model/Controller code - Authoritative for business logic and defaults
3. API Test files - Authoritative for actual working request/response examples
4. UI Vue components - Authoritative for field labels, help text, UI constraints

### Common Issues Found in Existing Docs
- Made-up field names not in schema
- Missing required fields
- Incorrect data types
- Missing validation constraints
- Business logic rules not documented

---

**Last Updated:** February 10, 2026
