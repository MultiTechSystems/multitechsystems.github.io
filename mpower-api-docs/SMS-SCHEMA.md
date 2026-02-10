# SMS Resource Schema

Complete documentation for the SMS resource extracted from mtsDeviceAPI source code.

**Source Files:**
- Schema: `mtsDeviceAPI/json/strict/sms.schema.json`
- Model: `mtsDeviceAPI/src/MTS_MTR_SMS.cpp`
- Controller: `mtsDeviceAPI/include/MTS_MTR_SMSController.h`

## Overview

The SMS resource provides configuration for SMS messaging capabilities including sending/receiving SMS messages and executing remote commands via SMS.

## Endpoints

### Configuration

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api?fields=sms` | Get SMS configuration |
| PUT | `/api/sms` | Update SMS configuration |

### Inbox (Received Messages)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api?fields=sms/inbox` | Get received messages |
| DELETE | `/api/sms/inbox/{guid}` | Delete specific message |
| DELETE | `/api/sms/inbox/all` | Delete all received messages |

### Outbox (Sent Messages)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api?fields=sms/outbox` | Get sent messages |
| POST | `/api/sms/outbox` | Send SMS message |
| DELETE | `/api/sms/outbox/{guid}` | Delete specific message |
| DELETE | `/api/sms/outbox/all` | Delete all sent messages |

---

## SMS Configuration Schema

### Top-Level Properties

| Property | Type | Required | Default | Min | Max | Description |
|----------|------|----------|---------|-----|-----|-------------|
| `enabled` | boolean | Yes | `false` | - | - | Enable or disable SMS service on the device |
| `sentLimit` | integer | Yes | `1000` | 0 | 1000 | Number of sent SMS messages to keep. Set to 0 to not keep any |
| `receivedLimit` | integer | Yes | `1000` | 0 | 1000 | Number of received SMS messages to keep. Set to 0 to not keep any |
| `resendLimit` | integer | Yes | `0` | 0 | 10 | Number of retry attempts for failed SMS messages. Set to 0 for no retries |
| `smsCommands` | object | Yes | - | - | - | SMS commands configuration |

### smsCommands Object

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `enabled` | boolean | Yes | `false` | Enable or disable SMS commands functionality |
| `appEnabled` | boolean | Yes | `false` | Enable `#app` command for application management |
| `rebootEnabled` | boolean | Yes | `false` | Enable `#reboot` command |
| `checkinEnabled` | boolean | Yes | `false` | Enable `#checkin` command for Device Manager |
| `rmEnabled` | boolean | Yes | `false` | Enable `#rm` command for remote management |
| `setCellularEnabled` | boolean | Yes | `false` | Enable `#setcellular` command |
| `apnEnabled` | boolean | Yes | `false` | Enable `#apn` command to get APN value |
| `cellularStatusEnabled` | boolean | Yes | `false` | Enable `#cellular` command |
| `radioEnabled` | boolean | Yes | `false` | Enable `#radio` command |
| `ethernetEnabled` | boolean | Yes | `false` | Enable `#ethernet` command |
| `wanStatusEnabled` | boolean | Yes | `false` | Enable `#wan` command |
| `wifiEnabled` | boolean | Yes | `false` | Enable `#wifi` command |
| `pingEnabled` | boolean | Yes | `false` | Enable `#ping` command |
| `serialEnabled` | boolean | Yes | `false` | Enable `#serial` command |
| `geopositionEnabled` | boolean | Yes | `false` | Enable `#geoposition` command |
| `lnsRestartEnabled` | boolean | Yes | `false` | Enable `#lnsrestart` command |
| `wanIpsEnabled` | boolean | Yes | `false` | Enable `#wanips` command |
| `getIoEnabled` | boolean | Yes | `false` | Enable `#getio` command for digital I/O |
| `setIoEnabled` | boolean | Yes | `false` | Enable `#setio` command for digital I/O |
| `password` | object | Yes | - | Password configuration for SMS commands |
| `whitelist` | object | Yes | - | Whitelist configuration for SMS commands |

### smsCommands.password Object

| Property | Type | Required | Default | Constraints | Description |
|----------|------|----------|---------|-------------|-------------|
| `enabled` | boolean | Yes | `true` | - | Enable password requirement for SMS commands |
| `useCustomPassword` | boolean | Yes | `false` | - | Use custom password instead of default |
| `customText` | string | Yes | `""` | maxLength: 250, minLength: 6 (when enabled), no spaces | Custom password text |
| `defaultText` | string | No | *dynamic* | maxLength: 250 | Default password (last 6 chars of IMEI, read-only) |
| `text` | string | No | *dynamic* | maxLength: 250 | Current active password (read-only) |

### smsCommands.whitelist Object

| Property | Type | Required | Default | Constraints | Description |
|----------|------|----------|---------|-------------|-------------|
| `enabled` | boolean | Yes | `false` | - | Enable whitelist filtering for SMS commands |
| `numbers` | array | Yes | `[]` | maxItems: 50, item maxLength: 250 | Phone numbers allowed to send commands |

---

## Business Rules

### Security Filter Requirements

When `smsCommands.enabled` is `true`, at least one security filter must be enabled:
- `smsCommands.password.enabled` = `true`, OR
- `smsCommands.whitelist.enabled` = `true`

**Error if both disabled:** `"Password or Whitelist must be enabled."`

### Custom Password Validation

When ALL of these conditions are true:
- `smsCommands.password.enabled` = `true`
- `smsCommands.password.useCustomPassword` = `true`

Then `customText` must:
- Be at least 6 characters long
- Not contain any space characters

**Error:** `"Invalid Password."`

### Default Password Generation

The default password is dynamically generated from the last 6 characters of the device's IMEI. This is set automatically when the SMS configuration is loaded.

---

## Example Requests

### Get SMS Configuration

```bash
curl -X GET "http://192.168.2.1/api?fields=sms" -b cookies.txt
```

**Response:**
```json
{
  "success": true,
  "result": {
    "sms": {
      "__v": 1,
      "enabled": true,
      "sentLimit": 1000,
      "receivedLimit": 1000,
      "resendLimit": 0,
      "smsCommands": {
        "enabled": true,
        "appEnabled": false,
        "rebootEnabled": true,
        "checkinEnabled": false,
        "rmEnabled": false,
        "setCellularEnabled": false,
        "apnEnabled": true,
        "cellularStatusEnabled": true,
        "radioEnabled": true,
        "ethernetEnabled": false,
        "wanStatusEnabled": true,
        "wifiEnabled": false,
        "pingEnabled": true,
        "serialEnabled": false,
        "geopositionEnabled": false,
        "lnsRestartEnabled": false,
        "wanIpsEnabled": true,
        "getIoEnabled": false,
        "setIoEnabled": false,
        "password": {
          "enabled": true,
          "useCustomPassword": false,
          "customText": "",
          "defaultText": "524255",
          "text": "524255"
        },
        "whitelist": {
          "enabled": false,
          "numbers": []
        }
      }
    }
  }
}
```

### Update SMS Configuration

```bash
curl -X PUT http://192.168.2.1/api/sms \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "sentLimit": 500,
    "receivedLimit": 500,
    "resendLimit": 3,
    "smsCommands": {
      "enabled": true,
      "rebootEnabled": true,
      "pingEnabled": true,
      "wanStatusEnabled": true,
      "password": {
        "enabled": true,
        "useCustomPassword": true,
        "customText": "MySecurePass123"
      },
      "whitelist": {
        "enabled": true,
        "numbers": ["+1234567890", "+0987654321"]
      }
    }
  }' \
  -b cookies.txt
```

### Enable Single Command

```bash
curl -X PUT http://192.168.2.1/api/sms \
  -H "Content-Type: application/json" \
  -d '{
    "smsCommands": {
      "pingEnabled": true
    }
  }' \
  -b cookies.txt
```

### Send SMS Message

```bash
curl -X POST http://192.168.2.1/api/sms/outbox \
  -H "Content-Type: application/json" \
  -d '{
    "recipients": ["+1234567890", "+0987654321"],
    "message": "Alert: Device rebooted at 2025-02-10 14:30:00"
  }' \
  -b cookies.txt
```

**Response:**
```json
{
  "success": true,
  "result": {
    "code": 200
  }
}
```

### Get Sent Messages (Outbox)

```bash
curl -X GET "http://192.168.2.1/api?fields=sms/outbox" -b cookies.txt
```

**Response:**
```json
{
  "success": true,
  "result": {
    "sms_outbox": [
      {
        "guid": "1923082400000058",
        "recipient": "+1234567890",
        "status": "sent",
        "timestamp": "2025/02/10 14:30:00",
        "message": "Alert: Device rebooted"
      },
      {
        "guid": "1923082400000059",
        "recipient": "+0987654321",
        "status": "sending",
        "timestamp": "2025/02/10 14:30:01",
        "message": "Alert: Device rebooted"
      }
    ]
  }
}
```

### Get Received Messages (Inbox)

```bash
curl -X GET "http://192.168.2.1/api?fields=sms/inbox" -b cookies.txt
```

**Response:**
```json
{
  "success": true,
  "result": {
    "sms_inbox": [
      {
        "guid": "1923082400000060",
        "source": "+1234567890",
        "timestamp": "2025/02/10 14:35:00",
        "message": "p 524255 #ping 8.8.8.8"
      }
    ]
  }
}
```

### Delete Specific Message

```bash
# Delete from inbox
curl -X DELETE http://192.168.2.1/api/sms/inbox/1923082400000060 -b cookies.txt

# Delete from outbox
curl -X DELETE http://192.168.2.1/api/sms/outbox/1923082400000058 -b cookies.txt
```

### Delete All Messages

```bash
# Delete all from inbox
curl -X DELETE http://192.168.2.1/api/sms/inbox/all -b cookies.txt

# Delete all from outbox
curl -X DELETE http://192.168.2.1/api/sms/outbox/all -b cookies.txt
```

---

## SMS Commands Reference

When SMS commands are enabled, the device can execute commands received via SMS. Commands must be prefixed with the password if password authentication is enabled.

### Command Format

**With password enabled:**
```
p <password> #<command> [parameters]
```

**With password disabled (whitelist only):**
```
#<command> [parameters]
```

### Available Commands

| Command | Field | Parameters | Description |
|---------|-------|------------|-------------|
| `#reboot` | `rebootEnabled` | None | Immediately reboot the device |
| `#checkin` | `checkinEnabled` | None | Trigger Device Manager check-in |
| `#rm` | `rmEnabled` | `<enable\|disable>` | Enable/disable remote management. Device reboots automatically |
| `#setcellular` | `setCellularEnabled` | `<enable\|disable> [APN]` | Enable/disable cellular connection, optionally set APN. Device reboots |
| `#ping` | `pingEnabled` | `[interface] [count] <address>` | Ping address and return result via SMS |
| `#app` | `appEnabled` | `<start\|stop\|restart> <id\|name>` | Control custom application |
| `#apn` | `apnEnabled` | None | Return current APN value via SMS |
| `#cellular` | `cellularStatusEnabled` | None | Return cellular status via SMS |
| `#radio` | `radioEnabled` | None | Return radio statistics via SMS |
| `#ethernet` | `ethernetEnabled` | None | Return ethernet status via SMS |
| `#wan` | `wanStatusEnabled` | None | Return WAN interface status via SMS |
| `#wifi` | `wifiEnabled` | None | Return WiFi status via SMS |
| `#wanips` | `wanIpsEnabled` | None | Return list of WAN IP addresses via SMS |
| `#geoposition` | `geopositionEnabled` | None | Return GPS coordinates via SMS |
| `#serial` | `serialEnabled` | None | Return serial port status via SMS |
| `#lnsrestart` | `lnsRestartEnabled` | None | Restart LoRa Network Server |
| `#getio` | `getIoEnabled` | `<di0\|do0>` | Return digital I/O state via SMS |
| `#setio` | `setIoEnabled` | `<do0> [value]` | Set digital output state |

### Command Examples

```
# Reboot device (with password "524255")
p 524255 #reboot

# Ping Google DNS
p 524255 #ping 8.8.8.8

# Ping with count on specific interface
p 524255 #ping cellular 10 8.8.8.8

# Enable remote management
p 524255 #rm enable

# Set cellular APN
p 524255 #setcellular enable broadband

# Restart custom app
p 524255 #app restart myapp

# Get WAN IPs
p 524255 #wanips

# Set digital output high
p 524255 #setio do0 1

# Get digital input state
p 524255 #getio di0
```

---

## Message Schemas

### Outbox Message

| Field | Type | Description |
|-------|------|-------------|
| `guid` | string | Unique message identifier |
| `recipient` | string | Destination phone number |
| `status` | string | `sending`, `sent`, or `failed` |
| `timestamp` | string | Send timestamp (format: `YYYY/MM/DD HH:MM:SS`) |
| `message` | string | Message content |

### Inbox Message

| Field | Type | Description |
|-------|------|-------------|
| `guid` | string | Unique message identifier |
| `source` | string | Sender phone number |
| `timestamp` | string | Receive timestamp (format: `YYYY/MM/DD HH:MM:SS`) |
| `message` | string | Message content |

### Send Request

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `recipients` | array | Yes | Non-empty | Array of phone numbers |
| `message` | string | Yes | maxLength: 160 | Message text |

---

## Validation Rules Summary

| Field | Rule |
|-------|------|
| `enabled` | Boolean only |
| `sentLimit` | Integer 0-1000 |
| `receivedLimit` | Integer 0-1000 |
| `resendLimit` | Integer 0-10 |
| `smsCommands.enabled` | Requires password OR whitelist enabled |
| `smsCommands.password.customText` | Min 6 chars, no spaces (when active) |
| `smsCommands.whitelist.numbers` | Max 50 entries, each max 250 chars |
| Send message | Max 160 characters |
| Recipient number | Valid phone number format, max 25 chars |

---

## Error Responses

### Validation Error (400)
```json
{
  "success": false,
  "error": "Password or Whitelist must be enabled.",
  "code": 400
}
```

### Invalid Password (400)
```json
{
  "success": false,
  "error": "Invalid Password.",
  "code": 400
}
```

### SMS Disabled (400)
```json
{
  "success": false,
  "error": "SMS is disabled",
  "code": 400
}
```

---

## Permissions

| Endpoint | Admin | User | Guest |
|----------|-------|------|-------|
| GET /api?fields=sms | ✅ | ✅ | ✅ (masked passwords) |
| PUT /api/sms | ✅ | ✅ | ❌ |
| GET sms/inbox | ✅ | ✅ | ✅ |
| GET sms/outbox | ✅ | ✅ | ✅ |
| POST sms/outbox | ✅ | ✅ | ❌ |
| DELETE sms/inbox | ✅ | ✅ | ❌ |
| DELETE sms/outbox | ✅ | ✅ | ❌ |

**Note:** Guest users see sensitive fields (`customText`, `defaultText`, `text`) masked as `********`.

---

## Related Documentation

- [Available Resources](AVAILABLE-RESOURCES.md)
- [Command Endpoints](COMMAND-ENDPOINTS.md)
- [Alert Schema](ALERT-SCHEMA.md) - SMS notifications for alerts

---

**Document Version:** 1.0.0  
**Source Version:** mtsDeviceAPI (analyzed February 2026)  
**Last Updated:** February 10, 2026
