# Command Endpoints Reference

This document provides comprehensive documentation for all command endpoints in the mPower Edge Intelligence platform.

## Overview

Commands are executed via `POST /api/command/<command>` and perform actions on the device such as saving configuration, restarting services, sending messages, and running diagnostics.

```bash
# Basic command execution
POST /api/command/<command>

# Command with parameters
POST /api/command/<command>
Content-Type: application/json
{ "param1": "value1", "param2": "value2" }
```

## Table of Contents

- [Configuration Commands](#configuration-commands)
- [System Commands](#system-commands)
- [Firmware Commands](#firmware-commands)
- [Network Commands](#network-commands)
- [Radio Commands](#radio-commands)
- [SMS Commands](#sms-commands)
- [User Commands](#user-commands)
- [LoRa Commands](#lora-commands)
- [Certificate Commands](#certificate-commands)
- [Diagnostic Commands](#diagnostic-commands)

---

## Configuration Commands

### `save`

Save the current runtime configuration to persistent storage.

**Endpoint:** `POST /api/command/save`

**Parameters:** None

**Description:**  
Changes made via PUT requests only modify the runtime configuration. Use this command to persist changes so they survive a reboot.

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
```

**Response:**
```json
{
  "success": true,
  "result": {
    "message": "Configuration saved"
  }
}
```

**Usage Pattern:**
```bash
# 1. Make configuration changes
curl -X PUT http://192.168.2.1/api/cellular \
  -H "Content-Type: application/json" \
  -d '{"enabled": true}' \
  -b cookies.txt

# 2. Check if there are unsaved changes
curl -X GET "http://192.168.2.1/api?fields=system/dbDirty" -b cookies.txt

# 3. Save configuration
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
```

---

### `revert`

Revert to the last saved configuration, discarding unsaved changes.

**Endpoint:** `POST /api/command/revert`

**Parameters:** None

**Description:**  
Discards all runtime configuration changes and reverts to the last saved (persisted) configuration.

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/revert -b cookies.txt
```

**Response:**
```json
{
  "success": true,
  "result": {
    "message": "Configuration reverted"
  }
}
```

---

### `save_apply`

Save configuration and apply changes that require service restart.

**Endpoint:** `POST /api/command/save_apply`

**Parameters:** None

**Description:**  
Saves the configuration and restarts any services that need to be restarted for the changes to take effect. This is a convenience command that combines save with selective service restarts.

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/save_apply -b cookies.txt
```

---

### `upload_config`

Upload a configuration file to the device.

**Endpoint:** `POST /api/command/upload_config`

**Content-Type:** `multipart/form-data`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file` | file | Yes | Configuration file (.json or .tar.gz) |

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/upload_config \
  -H "Content-Type: multipart/form-data" \
  -F "file=@/path/to/config.json" \
  -b cookies.txt
```

---

### `download_config`

Download the current device configuration.

**Endpoint:** `POST /api/command/download_config`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `format` | string | No | Output format: `json` or `tar.gz` (default: json) |

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/download_config \
  -H "Content-Type: application/json" \
  -d '{"format": "json"}' \
  -b cookies.txt \
  -o device_config.json
```

---

### `factory_reset`

Reset device to factory defaults.

**Endpoint:** `POST /api/command/factory_reset`

**Parameters:** None

**Warning:** This command will erase all configuration and restore factory defaults. The device will reboot.

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/factory_reset -b cookies.txt
```

---

## System Commands

### `restart`

Restart the device.

**Endpoint:** `POST /api/command/restart`

**Parameters:** None

**Description:**  
Initiates a graceful device reboot. Ensure configuration is saved before restarting.

**Example:**
```bash
# Save configuration first
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt

# Restart device
curl -X POST http://192.168.2.1/api/command/restart -b cookies.txt
```

**Response:**
```json
{
  "success": true,
  "result": {
    "message": "Device restarting"
  }
}
```

---

### `reset_modem`

Reset the cellular modem.

**Endpoint:** `POST /api/command/reset_modem`

**Parameters:** None

**Description:**  
Power cycles the cellular modem without rebooting the entire device. Useful for recovering from cellular connection issues.

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/reset_modem -b cookies.txt
```

---

### `reset_wifi`

Reset the WiFi module.

**Endpoint:** `POST /api/command/reset_wifi`

**Parameters:** None

**Description:**  
Restarts the WiFi module without rebooting the device.

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/reset_wifi -b cookies.txt
```

---

### `reset_bluetooth`

Reset the Bluetooth module.

**Endpoint:** `POST /api/command/reset_bluetooth`

**Parameters:** None

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/reset_bluetooth -b cookies.txt
```

---

### `reset_gps`

Reset the GPS module.

**Endpoint:** `POST /api/command/reset_gps`

**Parameters:** None

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/reset_gps -b cookies.txt
```

---

## Firmware Commands

### `firmware_check`

Check for available firmware updates.

**Endpoint:** `POST /api/command/firmware_check`

**Parameters:** None

**Description:**  
Contacts the update server to check if newer firmware is available.

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/firmware_check -b cookies.txt
```

**Response:**
```json
{
  "success": true,
  "result": {
    "updateAvailable": true,
    "currentVersion": "6.3.0",
    "availableVersion": "6.4.0",
    "releaseNotes": "Bug fixes and security updates"
  }
}
```

---

### `firmware_upgrade`

Upgrade device firmware from uploaded file.

**Endpoint:** `POST /api/command/firmware_upgrade`

**Content-Type:** `multipart/form-data`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file` | file | Yes | Firmware image file (.bin) |

**Description:**  
Uploads and installs new firmware. The device will reboot after successful installation.

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/firmware_upgrade \
  -H "Content-Type: multipart/form-data" \
  -F "file=@/path/to/firmware-6.4.0.bin" \
  -b cookies.txt
```

**Response:**
```json
{
  "success": true,
  "result": {
    "message": "Firmware upgrade started. Device will reboot."
  }
}
```

---

### `radio/firmware_upgrade`

Upgrade cellular radio firmware.

**Endpoint:** `POST /api/command/radio/firmware_upgrade`

**Content-Type:** `multipart/form-data`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file` | file | Yes | Radio firmware file |

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/radio/firmware_upgrade \
  -H "Content-Type: multipart/form-data" \
  -F "file=@/path/to/radio_firmware.bin" \
  -b cookies.txt
```

---

## Network Commands

### `ping`

Execute a ping test.

**Endpoint:** `POST /api/command/ping`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `host` | string | Yes | Target hostname or IP address |
| `count` | integer | No | Number of ping packets (default: 4) |
| `size` | integer | No | Packet size in bytes (default: 56) |
| `interface` | string | No | Source interface (eth0, ppp0, etc.) |

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/ping \
  -H "Content-Type: application/json" \
  -d '{
    "host": "8.8.8.8",
    "count": 4
  }' \
  -b cookies.txt
```

**Response:**
```json
{
  "success": true,
  "result": {
    "host": "8.8.8.8",
    "transmitted": 4,
    "received": 4,
    "loss": 0,
    "min": 10.5,
    "avg": 15.2,
    "max": 22.1,
    "output": "PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data..."
  }
}
```

---

### `continuous_ping_start`

Start a continuous ping test.

**Endpoint:** `POST /api/command/continuous_ping_start`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `host` | string | Yes | Target hostname or IP address |
| `interval` | integer | No | Seconds between pings (default: 1) |
| `interface` | string | No | Source interface |

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/continuous_ping_start \
  -H "Content-Type: application/json" \
  -d '{
    "host": "8.8.8.8",
    "interval": 5
  }' \
  -b cookies.txt
```

---

### `continuous_ping_stop`

Stop the continuous ping test.

**Endpoint:** `POST /api/command/continuous_ping_stop`

**Parameters:** None

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/continuous_ping_stop -b cookies.txt
```

---

### `traceroute`

Execute a traceroute to a destination.

**Endpoint:** `POST /api/command/traceroute`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `host` | string | Yes | Target hostname or IP address |
| `maxHops` | integer | No | Maximum number of hops (default: 30) |

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/traceroute \
  -H "Content-Type: application/json" \
  -d '{"host": "google.com"}' \
  -b cookies.txt
```

---

### `dns_lookup`

Perform a DNS lookup.

**Endpoint:** `POST /api/command/dns_lookup`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `host` | string | Yes | Hostname to resolve |
| `type` | string | No | Record type: A, AAAA, MX, etc. (default: A) |

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/dns_lookup \
  -H "Content-Type: application/json" \
  -d '{"host": "google.com", "type": "A"}' \
  -b cookies.txt
```

---

## Radio Commands

### `radio/check_sim_status`

Check the SIM card status.

**Endpoint:** `POST /api/command/radio/check_sim_status`

**Parameters:** None

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/radio/check_sim_status -b cookies.txt
```

**Response:**
```json
{
  "success": true,
  "result": {
    "simPresent": true,
    "simLocked": false,
    "iccid": "89012345678901234567",
    "imsi": "310260123456789"
  }
}
```

---

### `radio/unlock_sim_card`

Unlock a PIN-locked SIM card.

**Endpoint:** `POST /api/command/radio/unlock_sim_card`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `pin` | string | Yes | SIM PIN code |

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/radio/unlock_sim_card \
  -H "Content-Type: application/json" \
  -d '{"pin": "1234"}' \
  -b cookies.txt
```

---

### `radio/cmd`

Execute an AT command on the cellular modem.

**Endpoint:** `POST /api/command/radio/cmd`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `command` | string | Yes | AT command to execute |

**Description:**  
Sends a raw AT command to the cellular modem. Use with caution as incorrect commands may disrupt connectivity.

**Example - Get Manufacturer:**
```bash
curl -X POST http://192.168.2.1/api/command/radio/cmd \
  -H "Content-Type: application/json" \
  -d '{"command": "AT+CGMI"}' \
  -b cookies.txt
```

**Response:**
```json
{
  "success": true,
  "result": {
    "response": "Telit\r\nOK"
  }
}
```

**Common AT Commands:**
| Command | Description |
|---------|-------------|
| `AT+CGMI` | Get manufacturer |
| `AT+CGMM` | Get model |
| `AT+CGMR` | Get firmware revision |
| `AT+CGSN` | Get IMEI |
| `AT+CSQ` | Get signal strength |
| `AT+COPS?` | Get current operator |
| `AT+CREG?` | Get network registration status |

---

### `radio/force_cellular_scan`

Force a cellular network scan.

**Endpoint:** `POST /api/command/radio/force_cellular_scan`

**Parameters:** None

**Description:**  
Initiates a scan for available cellular networks. Results can be retrieved with `radio/get_cellular_scan`.

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/radio/force_cellular_scan -b cookies.txt
```

---

### `radio/get_cellular_scan`

Get cellular network scan results.

**Endpoint:** `POST /api/command/radio/get_cellular_scan`

**Parameters:** None

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/radio/get_cellular_scan -b cookies.txt
```

**Response:**
```json
{
  "success": true,
  "result": {
    "networks": [
      {
        "operator": "AT&T",
        "mcc": "310",
        "mnc": "410",
        "technology": "LTE",
        "status": "available"
      },
      {
        "operator": "T-Mobile",
        "mcc": "310",
        "mnc": "260",
        "technology": "LTE",
        "status": "available"
      }
    ]
  }
}
```

---

## SMS Commands

See **[SMS Schema](SMS-SCHEMA.md)** for complete SMS documentation.

### Send SMS via Outbox

Send an SMS message using the SMS outbox endpoint.

**Endpoint:** `POST /api/sms/outbox`

**Parameters:**
| Parameter | Type | Required | Constraints | Description |
|-----------|------|----------|-------------|-------------|
| `recipients` | array | Yes | Non-empty, max 25 chars per number | Array of destination phone numbers |
| `message` | string | Yes | Max 160 characters | Message text |

**Prerequisites:**
- Cellular modem must be connected
- SMS must be enabled (`sms.enabled: true`)
- Valid SIM card with SMS capability

**Example:**
```bash
curl -X POST http://192.168.2.1/api/sms/outbox \
  -H "Content-Type: application/json" \
  -d '{
    "recipients": ["+1234567890", "+0987654321"],
    "message": "Alert: Device gateway-01 has rebooted"
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

**Error Response (SMS Disabled):**
```json
{
  "success": false,
  "error": "SMS is disabled",
  "code": 400
}
```

### Get Outbox (Sent Messages)

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
      }
    ]
  }
}
```

### Get Inbox (Received Messages)

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

### Delete Messages

```bash
# Delete specific inbox message
curl -X DELETE http://192.168.2.1/api/sms/inbox/{guid} -b cookies.txt

# Delete all inbox messages
curl -X DELETE http://192.168.2.1/api/sms/inbox/all -b cookies.txt

# Delete specific outbox message
curl -X DELETE http://192.168.2.1/api/sms/outbox/{guid} -b cookies.txt

# Delete all outbox messages
curl -X DELETE http://192.168.2.1/api/sms/outbox/all -b cookies.txt
```

### SMS Configuration

Configure SMS settings including remote SMS commands:

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
        "useCustomPassword": false
      },
      "whitelist": {
        "enabled": true,
        "numbers": ["+1234567890"]
      }
    }
  }' \
  -b cookies.txt
```

### SMS Remote Commands

When SMS commands are enabled, the device can execute commands received via SMS. Commands require password and/or whitelist security.

**Command Format (with password):**
```
p <password> #<command> [parameters]
```

**Available Commands:**
| Command | Description |
|---------|-------------|
| `#reboot` | Reboot the device |
| `#checkin` | Trigger Device Manager check-in |
| `#rm <enable\|disable>` | Enable/disable remote management |
| `#setcellular <enable\|disable> [APN]` | Control cellular connection |
| `#ping [interface] [count] <address>` | Ping and return result |
| `#app <start\|stop\|restart> <id\|name>` | Control custom application |
| `#apn` | Get current APN value |
| `#cellular` | Get cellular status |
| `#radio` | Get radio statistics |
| `#ethernet` | Get ethernet status |
| `#wan` | Get WAN interface status |
| `#wifi` | Get WiFi status |
| `#wanips` | Get WAN IP addresses |
| `#geoposition` | Get GPS coordinates |
| `#serial` | Get serial port status |
| `#lnsrestart` | Restart LoRa Network Server |
| `#getio <di0\|do0>` | Get digital I/O state |
| `#setio <do0> [value]` | Set digital output state |

**Example SMS Commands:**
```
p 524255 #reboot
p 524255 #ping 8.8.8.8
p 524255 #wanips
p 524255 #setcellular enable broadband
```

### SMS Workflow Example

Complete SMS workflow for sending alerts:

```bash
# 1. Login
curl -X POST http://192.168.2.1/api/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "admin"}' \
  -c cookies.txt

# 2. Check cellular status
curl -X GET "http://192.168.2.1/api?fields=stats/radio" -b cookies.txt

# 3. Verify SMS is enabled
curl -X GET "http://192.168.2.1/api?fields=sms" -b cookies.txt

# 4. Send SMS alert
curl -X POST http://192.168.2.1/api/sms/outbox \
  -H "Content-Type: application/json" \
  -d '{
    "recipients": ["+1234567890"],
    "message": "Gateway status: Online, Signal: -65 dBm"
  }' \
  -b cookies.txt

# 5. Check sent message status
curl -X GET "http://192.168.2.1/api?fields=sms/outbox" -b cookies.txt

# 6. Logout
curl -X POST http://192.168.2.1/api/logout -b cookies.txt
```

---

## User Commands

### `passwd`

Change a user's password.

**Endpoint:** `POST /api/command/passwd`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `username` | string | No | Username (admin only, defaults to current user) |
| `oldPassword` | string | Yes | Current password |
| `newPassword` | string | Yes | New password |

**Example - Change Own Password:**
```bash
curl -X POST http://192.168.2.1/api/command/passwd \
  -H "Content-Type: application/json" \
  -d '{
    "oldPassword": "currentpass",
    "newPassword": "newSecurePass123!"
  }' \
  -b cookies.txt
```

**Example - Admin Changes Another User's Password:**
```bash
curl -X POST http://192.168.2.1/api/command/passwd \
  -H "Content-Type: application/json" \
  -d '{
    "username": "engineer1",
    "oldPassword": "adminpass",
    "newPassword": "newUserPass123!"
  }' \
  -b cookies.txt
```

---

## LoRa Commands

### `lora/restart`

Restart the LoRa service.

**Endpoint:** `POST /api/lora/restart`

**Parameters:** None

**Description:**  
Restarts the LoRa network server or packet forwarder service. Use after making LoRa configuration changes.

**Example:**
```bash
# Update LoRa configuration
curl -X PUT http://192.168.2.1/api/loraNetwork \
  -H "Content-Type: application/json" \
  -d '{"network": {"name": "Updated Network"}}' \
  -b cookies.txt

# Restart LoRa service
curl -X POST http://192.168.2.1/api/lora/restart -b cookies.txt
```

---

### `upload_lora_license`

Upload a LoRa network server license.

**Endpoint:** `POST /api/command/upload_lora_license`

**Content-Type:** `multipart/form-data`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file` | file | Yes | License file |

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/upload_lora_license \
  -H "Content-Type: multipart/form-data" \
  -F "file=@/path/to/lora_license.json" \
  -b cookies.txt
```

---

### `upload_lora_fota_file`

Upload a LoRa FOTA (Firmware Over The Air) file.

**Endpoint:** `POST /api/command/upload_lora_fota_file`

**Content-Type:** `multipart/form-data`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file` | file | Yes | FOTA firmware file |

**Description:**  
Uploads firmware for over-the-air updates to LoRa end devices.

See [LoRa Operations Workflow](LORA-OPERATIONS-WORKFLOW.md) for complete FOTA procedures.

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/upload_lora_fota_file \
  -H "Content-Type: multipart/form-data" \
  -F "file=@/path/to/device_firmware.bin" \
  -b cookies.txt
```

---

## Certificate Commands

### `upload_certificate`

Upload a device certificate.

**Endpoint:** `POST /api/command/upload_certificate`

**Content-Type:** `multipart/form-data`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file` | file | Yes | Certificate file (.pem, .crt) |
| `type` | string | No | Certificate type |

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/upload_certificate \
  -H "Content-Type: multipart/form-data" \
  -F "file=@/path/to/certificate.pem" \
  -b cookies.txt
```

---

### `upload_ca_certificate`

Upload a CA certificate.

**Endpoint:** `POST /api/command/upload_ca_certificate`

**Content-Type:** `multipart/form-data`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file` | file | Yes | CA certificate file (.pem, .crt) |

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/upload_ca_certificate \
  -H "Content-Type: multipart/form-data" \
  -F "file=@/path/to/ca_certificate.pem" \
  -b cookies.txt
```

---

### `generate_csr`

Generate a Certificate Signing Request.

**Endpoint:** `POST /api/command/generate_csr`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `commonName` | string | Yes | Common name (CN) |
| `organization` | string | No | Organization (O) |
| `organizationalUnit` | string | No | Organizational unit (OU) |
| `country` | string | No | Country code (C) |
| `state` | string | No | State/Province (ST) |
| `locality` | string | No | City/Locality (L) |

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/generate_csr \
  -H "Content-Type: application/json" \
  -d '{
    "commonName": "gateway.example.com",
    "organization": "My Company",
    "country": "US"
  }' \
  -b cookies.txt
```

---

## Diagnostic Commands

### `debug_report`

Generate a diagnostic report.

**Endpoint:** `POST /api/command/debug_report`

**Parameters:** None

**Description:**  
Generates a comprehensive diagnostic report including system logs, configuration, and status information. Useful for troubleshooting.

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/debug_report \
  -b cookies.txt \
  -o debug_report.tar.gz
```

---

### `speedtest`

Run an internet speed test.

**Endpoint:** `POST /api/command/speedtest`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `interface` | string | No | Network interface to test |

**Example:**
```bash
curl -X POST http://192.168.2.1/api/command/speedtest \
  -H "Content-Type: application/json" \
  -d '{"interface": "ppp0"}' \
  -b cookies.txt
```

---

## Command Response Format

All commands return a consistent JSON response format:

### Success Response
```json
{
  "success": true,
  "result": {
    // Command-specific result data
  }
}
```

### Error Response
```json
{
  "success": false,
  "error": "Error description",
  "code": 400
}
```

### Common Error Codes
| Code | Description |
|------|-------------|
| 400 | Bad request - Invalid parameters |
| 401 | Unauthorized - Not logged in |
| 403 | Forbidden - Insufficient permissions |
| 404 | Not found - Unknown command |
| 409 | Conflict - Device in commissioning mode |
| 500 | Internal server error |
| 503 | Service unavailable - Required service not running |

---

## Command Permissions

Commands require appropriate user permissions:

| Role | Allowed Commands |
|------|------------------|
| **Admin** | All commands |
| **User (Engineer)** | Most commands except user management, security |
| **Guest (Monitor)** | Read-only - no command execution |

---

## Best Practices

### 1. Always Save After Configuration Changes
```bash
# Make changes
curl -X PUT http://192.168.2.1/api/cellular -d '...' -b cookies.txt

# Save changes
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
```

### 2. Check Status Before Destructive Operations
```bash
# Check for unsaved changes before restart
curl -X GET "http://192.168.2.1/api?fields=system/dbDirty" -b cookies.txt

# If dbDirty is true, save first
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt

# Then restart
curl -X POST http://192.168.2.1/api/command/restart -b cookies.txt
```

### 3. Handle Long-Running Commands
Some commands (firmware upgrade, network scan) may take time. Check status periodically:

```bash
# Start firmware upgrade
curl -X POST http://192.168.2.1/api/command/firmware_upgrade \
  -F "file=@firmware.bin" -b cookies.txt

# Poll for completion
while true; do
  status=$(curl -s "http://192.168.2.1/api?fields=system/upgradeStatus" -b cookies.txt)
  # Check status and break when complete
  sleep 5
done
```

### 4. Error Handling
Always check the `success` field in responses:

```bash
response=$(curl -s -X POST http://192.168.2.1/api/command/ping \
  -H "Content-Type: application/json" \
  -d '{"host": "8.8.8.8"}' \
  -b cookies.txt)

if echo "$response" | jq -e '.success == true' > /dev/null; then
  echo "Ping successful"
else
  echo "Ping failed: $(echo "$response" | jq -r '.error')"
fi
```

---

## Related Documentation

- [Available Resources](AVAILABLE-RESOURCES.md) - Resource configuration reference
- [API Quick Reference](API-QUICK-REFERENCE.md) - Concise command reference
- [API README](API-README.md) - Getting started guide
- [LoRa Operations Workflow](LORA-OPERATIONS-WORKFLOW.md) - FOTA and multicast operations

---

**Document Version**: 1.0.0  
**Last Updated**: February 2026
