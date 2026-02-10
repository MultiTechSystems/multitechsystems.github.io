# Device Management Schemas Reference

Complete schema documentation for device management resources extracted from mtsDeviceAPI source code.

**Source:** `/home/jreiss/Workspace/aep/mtsDeviceAPI/json/strict/`

---

## Table of Contents

- [Auto Reboot](#auto-reboot)
- [Remote Management](#remote-management)
- [Remote Access](#remote-access)
- [Reset Button](#reset-button)
- [Save and Restore](#save-and-restore)
- [Custom Applications](#custom-applications)
- [Docker](#docker)
- [Users and Roles](#users-and-roles)
- [Password Complexity Rules](#password-complexity-rules)

---

## Auto Reboot

Automatic device reboot configuration.

**Source:** `autoReboot.schema.json`

**Endpoints:**
- `GET /api?fields=autoReboot` - Get auto reboot configuration
- `PUT /api/autoReboot` - Update auto reboot configuration

### Schema

```json
{
  "mode": "DISABLED",
  "hour": 0,
  "minute": 0,
  "interval": 0
}
```

### Key Properties

| Property | Type | Description | Constraints |
|----------|------|-------------|-------------|
| `mode` | string | Auto reboot mode | enum: `DISABLED`, `TIME`, `TIMER` |
| `hour` | integer | Hour for reboot | TIME mode: 0-23, TIMER mode: min 1 |
| `minute` | integer | Minute for reboot (TIME mode only) | 0-59 |
| `interval` | integer | Random interval range in minutes | 0 or 15-120 |

### Mode-Specific Rules

**TIME Mode:**
- Reboots at a specific time of day
- `hour`: 0-23 (hour of day)
- `minute`: 0-59 (minute of hour)

**TIMER Mode:**
- Reboots after specified hours from boot
- `hour`: minimum 1 (hours since boot)
- `minute` is ignored

### Example: Schedule Daily Reboot at 3:00 AM

```bash
curl -X PUT https://device-ip/api/autoReboot \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "autoReboot": {
      "mode": "TIME",
      "hour": 3,
      "minute": 0,
      "interval": 0
    }
  }'
```

---

## Remote Management

Device Manager (DeviceHQ) integration settings.

**Source:** `remoteMgmt.schema.json`

**Endpoints:**
- `GET /api?fields=remoteMgmt` - Get remote management configuration
- `PUT /api/remoteMgmt` - Update remote management configuration

### Schema

```json
{
  "enabled": true,
  "serverName": "api.multitech.com",
  "allowFirmwareUpgrade": true,
  "allowRadioFirmwareUpgrade": true,
  "allowConfigurationUpgrade": true,
  "allowConfigurationUpload": false,
  "checkInIntervals": {
    "enabled": true,
    "interval": 3600
  },
  "checkInDayTime": {
    "enabled": false,
    "mode": "DAILY",
    "dailyTime": "",
    "customDays": [],
    "customTime": ""
  }
}
```

### Key Properties

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `enabled` | boolean | Enable Device Manager integration | true |
| `serverName` | string | Device Manager URL | api.multitech.com |
| `allowFirmwareUpgrade` | boolean | Allow firmware upgrades from DM | true |
| `allowRadioFirmwareUpgrade` | boolean | Allow radio firmware upgrades | true |
| `allowConfigurationUpgrade` | boolean | Allow config updates from DM | true |
| `allowConfigurationUpload` | boolean | Allow config upload to DM | false |

### Check-In Configuration

**Interval-based:**
```json
{
  "checkInIntervals": {
    "enabled": true,
    "interval": 3600  // seconds, minimum 300
  }
}
```

**Schedule-based:**
```json
{
  "checkInDayTime": {
    "enabled": true,
    "mode": "DAILY",        // or "CUSTOM"
    "dailyTime": "03:00:00", // HH:MM:SS format
    "customDays": [1, 3, 5], // 1-7 (Mon-Sun) for CUSTOM mode
    "customTime": "03:00:00"
  }
}
```

---

## Remote Access

HTTP/HTTPS/SSH access and security settings.

**Source:** `remoteAccess.schema.json`

**Endpoints:**
- `GET /api?fields=remoteAccess` - Get remote access configuration
- `PUT /api/remoteAccess` - Update remote access configuration

### Schema Overview

```json
{
  "timeoutSeconds": 300,
  "http": {
    "enabled": true,
    "port": 80,
    "lan": true,
    "wan": false,
    "redirectToHttps": true
  },
  "https": {
    "enabled": true,
    "port": 443,
    "lan": true,
    "wan": false
  },
  "ssh": {
    "enabled": false,
    "port": 22,
    "lan": false,
    "wan": false,
    "idleTimeoutSeconds": 300,
    "passwordAuthentication": true,
    "publicKeyAuthentication": true,
    "userLockout": {
      "enabled": true,
      "failAttempts": 3,
      "lockoutSeconds": 300
    }
  },
  "reverseSsh": {
    "enabled": false,
    "serverAddress": "",
    "serverPort": 22,
    "remotePort": 2222,
    "authMode": "PASSWORD",
    "username": "",
    "password": ""
  },
  "icmp": {
    "enabled": true,
    "respondToLan": true,
    "respondToWan": false
  },
  "snmp": {
    "lan": true,
    "wan": false
  },
  "modbusSlave": {
    "enabled": false,
    "port": 1502,
    "lan": true
  },
  "dosAttackPrevention": {
    "enabled": true,
    "limitPerMinute": 60,
    "limitBurst": 100
  },
  "pingLimit": {
    "enabled": true,
    "limitPerSecond": 10,
    "limitBurst": 30
  },
  "bruteForcePrevention": {
    "enabled": true,
    "lockoutSeconds": 300,
    "failAttempts": 3
  },
  "privateWan": false
}
```

### SSH Configuration

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `enabled` | boolean | Enable SSH access | false |
| `port` | integer | SSH port | 22 |
| `lan` | boolean | Allow SSH from LAN | false |
| `wan` | boolean | Allow SSH from WAN | false |
| `idleTimeoutSeconds` | integer | Idle timeout (0 or 60-3600) | 300 |
| `passwordAuthentication` | boolean | Allow password auth | true |
| `publicKeyAuthentication` | boolean | Allow public key auth | true |

### Reverse SSH Configuration

| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable reverse SSH tunnel |
| `serverAddress` | string | Remote SSH server address |
| `serverPort` | integer | SSH server port (default: 22) |
| `remotePort` | integer | Remote tunnel port (default: 2222) |
| `authMode` | string | `PASSWORD`, `PUBLIC_KEY`, or `PRIVATE_KEY` |

### Security Features

**DoS Attack Prevention:**
- `limitPerMinute`: Average new connections per minute (default: 60)
- `limitBurst`: Maximum burst connections (default: 100)

**Brute Force Prevention:**
- `failAttempts`: Failures before lockout (default: 3)
- `lockoutSeconds`: Lockout duration (default: 300)

---

## Reset Button

Physical reset button behavior configuration.

**Source:** `resetButton.schema.json`

**Endpoints:**
- `GET /api?fields=resetButton` - Get reset button configuration
- `PUT /api/resetButton` - Update reset button configuration

### Schema

```json
{
  "enabled": true,
  "resetToFactoryDefault": true,
  "resetToUserDefinedDefault": true
}
```

### Key Properties

| Property | Type | Description |
|----------|------|-------------|
| `enabled` | boolean | Enable reset button functionality |
| `resetToFactoryDefault` | boolean | Allow factory reset via button |
| `resetToUserDefinedDefault` | boolean | Allow user-defined default reset via button |

---

## Save and Restore

Configuration export options.

**Source:** `saveAndRestore.schema.json`

**Endpoints:**
- `GET /api?fields=saveAndRestore` - Get save/restore configuration
- `PUT /api/saveAndRestore` - Update save/restore configuration

### Schema

```json
{
  "exportPayloadManagement": true,
  "exportSshPublicKeys": true
}
```

---

## Custom Applications

Custom application management.

**Source:** `customApps.schema.json`, `customAppsConfig.schema.json`

**Endpoints:**
- `GET /api?fields=customApps` - List installed custom applications
- `GET /api?fields=customAppsConfig` - Get custom apps configuration
- `PUT /api/customAppsConfig` - Update custom apps configuration
- `POST /api/customApps/{id}/start` - Start application
- `POST /api/customApps/{id}/stop` - Stop application

### Custom App Object

```json
{
  "_id": "uuid-string",
  "name": "MyApp",
  "version": "1.0.0",
  "versionNotes": "Initial release",
  "description": "Application description",
  "status": "RUNNING",
  "info": "Application-generated info",
  "configs": {},
  "config_ids": []
}
```

### Status Values

| Status | Description |
|--------|-------------|
| `STARTED` | Application is starting |
| `STOPPED` | Application is stopped |
| `RUNNING` | Application is running |
| `FAILED` | Application failed |
| `INSTALL FAILED` | Installation failed |
| `START FAILED` | Start failed |

### Custom Apps Config

```json
{
  "allowed": true,
  "enabled": true,
  "backupOnInstall": true
}
```

| Property | Type | Description |
|----------|------|-------------|
| `allowed` | boolean | Custom apps allowed (always true on AEP) |
| `enabled` | boolean | Custom apps enabled |
| `backupOnInstall` | boolean | Backup existing app on install |

---

## Docker

Docker service configuration.

**Source:** `docker.schema.json`

**Endpoints:**
- `GET /api?fields=docker` - Get Docker configuration
- `PUT /api/docker` - Update Docker configuration

### Schema

```json
{
  "enabled": false
}
```

---

## Users and Roles

Custom user roles configuration.

**Source:** `customRoles.schema.json`

**Endpoints:**
- `GET /api?fields=customRoles` - Get custom roles
- `PUT /api/customRoles` - Update custom roles
- `POST /api/customRoles` - Create custom role
- `DELETE /api/customRoles/{name}` - Delete custom role

### Custom Role Schema

```json
{
  "name": "custom-role-name",
  "description": "Role description",
  "permissions": {
    "system": [true, true, false, false, false],
    "users": [true, false, false, false, false],
    "cellular": [true, true, true, false, false]
  }
}
```

### Permission Array Format

Each permission is an array of 5 booleans: `[read, write, create, delete, execute]`

Example permissions:
- `[true, false, false, false, false]` - Read only
- `[true, true, false, false, false]` - Read/Write
- `[true, true, true, true, false]` - Full CRUD
- `[true, true, true, true, true]` - Full access with execute

---

## Password Complexity Rules

Password requirements configuration.

**Source:** `passwordComplexityRules.schema.json`

**Endpoints:**
- `GET /api?fields=passwordComplexityRules` - Get password rules
- `PUT /api/passwordComplexityRules` - Update password rules

### Schema

```json
{
  "minPasswordLength": 8,
  "maxPasswordLength": 64,
  "charactersNotPermitted": "",
  "creditMode": false,
  "minLowerCaseChars": 0,
  "minUpperCaseChars": 0,
  "minNumeralChars": 0,
  "minOtherChars": 0,
  "minPasswordAge": 0,
  "maxPasswordAge": 0,
  "passwordHistoryLength": 0
}
```

### Standard Mode Properties

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `minPasswordLength` | integer | Minimum length | 8 (min: 6) |
| `maxPasswordLength` | integer | Maximum length | 64 |
| `minLowerCaseChars` | integer | Minimum lowercase | 0 |
| `minUpperCaseChars` | integer | Minimum uppercase | 0 |
| `minNumeralChars` | integer | Minimum numbers | 0 |
| `minOtherChars` | integer | Minimum special chars | 0 |

### Credit Mode Properties

When `creditMode: true`:

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `minCharClasses` | integer | Min different char classes | 3 |
| `lowerCaseExtraCreditCap` | integer | Extra credit for lowercase | 0 |
| `upperCaseExtraCreditCap` | integer | Extra credit for uppercase | 0 |
| `numeralExtraCreditCap` | integer | Extra credit for numbers | 0 |
| `otherExtraCreditCap` | integer | Extra credit for special | 0 |

### Password Age Settings

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `minPasswordAge` | integer | Days before can change | 0 |
| `maxPasswordAge` | integer | Days before must change | 0 |
| `passwordHistoryLength` | integer | History length (max: 400) | 0 |

---

**Last Updated:** February 10, 2026
