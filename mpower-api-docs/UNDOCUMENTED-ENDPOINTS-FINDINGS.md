# Undocumented API Endpoints - Discovery Report

**Date**: December 17, 2025  
**Gateway**: MTCAP3 (172.16.33.111)  
**Firmware**: 7.4.0-BETA2-7-g7e2e653f  
**API Version**: 7.2.0-86-g5ae2b66c

## Summary

During live testing against an MTCAP3 gateway, **19 previously undocumented API endpoints** were discovered. All endpoints are accessible via `GET /api?fields=<endpoint>` and support standard CRUD operations where applicable.

## Newly Discovered Endpoints

### 1. `autoReboot` - Automatic Reboot Scheduler

**Purpose**: Configure automatic device reboots on a schedule

**Example Response**:
```json
{
  "autoReboot": {
    "hour": 0,
    "minute": 0,
    "interval": 0,
    "mode": "DISABLED"
  }
}
```

**Fields**:
- `mode`: Reboot mode - `DISABLED`, `DAILY`, `WEEKLY`, `INTERVAL`
- `hour`: Hour for scheduled reboot (0-23)
- `minute`: Minute for scheduled reboot (0-59)
- `interval`: Interval in minutes for interval-based reboots

**Use Cases**:
- Scheduled maintenance reboots
- Periodic device refresh
- Memory leak mitigation

---

### 2. `backOffTimers` - Cellular Retry Backoff Timers

**Purpose**: Configure exponential backoff timers for cellular connection retries

**Example Response**:
```json
{
  "backOffTimers": {
    "enabled": true,
    "timers": [
      {
        "carrierName": "AT&T",
        "data": [60, 60, 60, 60, 600, 900, 1800, 3600],
        "dataReset": [60, 60, 60, 60, 600, 900, 1800, 3600],
        "modemReset": [300, 300, 300, 300, 600, 1800, 3600],
        "sms": [30, 30, 30, 30, 900, 900, 1800, 3600],
        "readOnly": true
      }
    ]
  }
}
```

**Fields**:
- `enabled`: Enable backoff timers
- `timers[]`: Array of carrier-specific timer configurations
  - `carrierName`: Carrier name (AT&T, Verizon, Rogers, default)
  - `data[]`: Data connection retry intervals (seconds)
  - `dataReset[]`: Data reset retry intervals (seconds)
  - `modemReset[]`: Modem reset retry intervals (seconds)
  - `sms[]`: SMS retry intervals (seconds)
  - `readOnly`: Whether timer is read-only

**Use Cases**:
- Optimize cellular reconnection behavior
- Carrier-specific retry strategies
- Reduce network congestion during outages

---

### 3. `bootloader` - Bootloader Configuration

**Purpose**: Configure bootloader console access and security

**Example Response**:
```json
{
  "bootloader": {
    "console": {
      "access": true,
      "silent": false
    },
    "lock": {
      "enabled": false,
      "supported": false
    }
  }
}
```

**Fields**:
- `console.access`: Allow console access during boot
- `console.silent`: Silent boot mode
- `lock.enabled`: Enable bootloader lock
- `lock.supported`: Whether device supports bootloader locking

**Use Cases**:
- Secure boot configuration
- Prevent unauthorized bootloader access
- Debug boot issues

---

### 4. `brand` - White-Label Branding Configuration

**Purpose**: Configure custom branding for OEM/white-label deployments

**Example Response**:
```json
{
  "brand": {
    "__v": 2,
    "companyName": "",
    "address1": "",
    "address2": "",
    "city": "",
    "state": "",
    "country": "",
    "zipCode": "",
    "phoneNumbers": [],
    "fax": "",
    "website": "",
    "logo": "",
    "icon": "",
    "image": "",
    "buttonColor": "",
    "buttonFontColor": "",
    "buttonHighlightColor": "",
    "buttonHighlightFontColor": "",
    "showOnDashboard": false,
    "links": []
  }
}
```

**Fields**:
- `companyName`: OEM company name
- `address1`, `address2`, `city`, `state`, `country`, `zipCode`: Company address
- `phoneNumbers[]`: Contact phone numbers
- `website`: Company website URL
- `logo`, `icon`, `image`: Custom branding images (base64 or URLs)
- `buttonColor`, `buttonFontColor`: UI button customization
- `buttonHighlightColor`, `buttonHighlightFontColor`: UI highlight colors
- `showOnDashboard`: Display branding on dashboard
- `links[]`: Custom navigation links

**Use Cases**:
- OEM/white-label deployments
- Custom UI branding
- Partner integrations

---

### 5. `cellTimeSync` - Cellular Time Synchronization

**Purpose**: Synchronize device time from cellular network

**Example Response**:
```json
{
  "cellTimeSync": {
    "enabled": false,
    "pollingTime": 120
  }
}
```

**Fields**:
- `enabled`: Enable cellular time sync
- `pollingTime`: Polling interval in seconds

**Use Cases**:
- Time sync when NTP unavailable
- Cellular-only deployments
- Accurate timekeeping without internet

---

### 6. `ddns` - Dynamic DNS Configuration

**Purpose**: Configure Dynamic DNS for devices with changing IP addresses

**Example Response**:
```json
{
  "ddns": {
    "enabled": false,
    "mode": "PROVIDER",
    "server": "dyndns.org",
    "domain": "",
    "authentication": {
      "username": "",
      "password": ""
    },
    "updateInterval": 5,
    "checkIpPeriod": 15,
    "checkIp": {
      "enabled": true,
      "mode": "DEFAULT",
      "custom": {
        "server": "checkip.dyndns.org",
        "port": 80,
        "path": "/",
        "ssl": false
      }
    },
    "custom": {
      "server": "",
      "port": 443,
      "path": "/nic/update?hostname=%h",
      "ssl": true
    }
  }
}
```

**Fields**:
- `enabled`: Enable DDNS
- `mode`: `PROVIDER` or `CUSTOM`
- `server`: DDNS provider (dyndns.org, no-ip.com, etc.)
- `domain`: Domain name to update
- `authentication`: Username/password for DDNS service
- `updateInterval`: Update interval in minutes
- `checkIpPeriod`: IP check period in minutes
- `checkIp`: IP detection configuration
- `custom`: Custom DDNS server configuration

**Use Cases**:
- Remote access with dynamic IPs
- Cloud service integration
- DNS-based device discovery

---

### 7. `ipPassthrough` - IP Passthrough Configuration

**Purpose**: Configure IP passthrough mode (bridge mode for cellular/WAN IP)

**Example Response**:
```json
{
  "ipPassthrough": {
    "ipMode": "IPv4",
    "ip": "192.168.2.1",
    "mask": "255.255.255.0",
    "gateway": "",
    "publicIpMask": "255.255.255.255",
    "ipv6": {
      "ip": "",
      "prefixlen": ""
    }
  }
}
```

**Fields**:
- `ipMode`: `IPv4`, `IPv6`, or `DUAL_STACK`
- `ip`: LAN IP address
- `mask`: LAN subnet mask
- `gateway`: Gateway address
- `publicIpMask`: Public IP mask for passthrough
- `ipv6`: IPv6 configuration

**Use Cases**:
- Bridge mode operation
- Pass public IP to downstream device
- Transparent cellular gateway

---

### 8. `lldp` - Link Layer Discovery Protocol

**Purpose**: Configure LLDP for network device discovery

**Example Response**:
```json
{
  "lldp": {
    "enabled": false,
    "sysName": "",
    "sysDescr": "",
    "txInterval": 30,
    "txHold": 4
  }
}
```

**Fields**:
- `enabled`: Enable LLDP
- `sysName`: System name advertised via LLDP
- `sysDescr`: System description
- `txInterval`: Transmission interval (seconds)
- `txHold`: Hold time multiplier

**Use Cases**:
- Network topology discovery
- Integration with network management systems
- Automated device inventory

---

### 9. `mdns` - Multicast DNS (mDNS/Bonjour)

**Purpose**: Configure mDNS for local network service discovery

**Example Response**:
```json
{
  "mdns": {
    "enabled": false,
    "allowInterfaces": []
  }
}
```

**Fields**:
- `enabled`: Enable mDNS
- `allowInterfaces[]`: Interfaces allowed for mDNS

**Use Cases**:
- Zero-configuration networking
- Local service discovery
- Apple Bonjour integration

---

### 10. `radius` - RADIUS Authentication

**Purpose**: Configure RADIUS server for centralized authentication

**Example Response**:
```json
{
  "radius": {
    "authEnabled": false,
    "authPort": 1812,
    "authType": "PAP",
    "acctEnabled": false,
    "acctPort": 1813,
    "host": "",
    "secondaryHost": "",
    "secondaryAuthPort": 1812,
    "secondaryAcctPort": 1813,
    "secret": "",
    "timeout": 2,
    "retries": 3,
    "checkServer": true,
    "anonymousEnabled": false,
    "anonymousId": "anonymous"
  }
}
```

**Fields**:
- `authEnabled`: Enable RADIUS authentication
- `authPort`: Authentication port (default 1812)
- `authType`: Authentication type (`PAP`, `CHAP`, `MSCHAP`, `MSCHAPv2`)
- `acctEnabled`: Enable RADIUS accounting
- `acctPort`: Accounting port (default 1813)
- `host`: Primary RADIUS server
- `secondaryHost`: Secondary RADIUS server
- `secret`: RADIUS shared secret
- `timeout`: Request timeout (seconds)
- `retries`: Number of retries
- `anonymousEnabled`: Enable anonymous outer identity
- `anonymousId`: Anonymous identity string

**Use Cases**:
- Enterprise authentication
- Centralized user management
- 802.1X authentication

---

### 11. `remoteAccess` - Remote Access Security

**Purpose**: Configure remote access security settings (HTTP, HTTPS, SSH, SNMP)

**Example Response**:
```json
{
  "remoteAccess": {
    "__v": 1,
    "http": {
      "enabled": true,
      "port": 80,
      "lan": true,
      "wan": true,
      "redirectToHttps": true
    },
    "https": {
      "enabled": true,
      "port": 443,
      "lan": true,
      "wan": true
    },
    "ssh": {
      "enabled": true,
      "port": 22,
      "lan": true,
      "wan": true,
      "passwordAuthentication": true,
      "publicKeyAuthentication": true,
      "idleTimeoutSeconds": 3600,
      "userLockout": {
        "enabled": true,
        "failAttempts": 3,
        "lockoutSeconds": 300
      }
    },
    "snmp": {
      "lan": true,
      "wan": false
    },
    "icmp": {
      "enabled": true,
      "respondToLan": true,
      "respondToWan": false
    },
    "modbusSlave": {
      "enabled": false,
      "port": 1502,
      "lan": true
    },
    "reverseSsh": {
      "enabled": false,
      "serverAddress": "",
      "serverPort": 22,
      "remotePort": 2222,
      "username": "",
      "password": "",
      "privateKey": "",
      "authMode": "PASSWORD"
    },
    "bruteForcePrevention": {
      "enabled": true,
      "failAttempts": 3,
      "lockoutSeconds": 300
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
    "timeoutSeconds": 3000,
    "privateWan": false
  }
}
```

**Fields**:
- `http`: HTTP server configuration
- `https`: HTTPS server configuration
- `ssh`: SSH server configuration with lockout protection
- `snmp`: SNMP access control
- `icmp`: ICMP/ping response settings
- `modbusSlave`: Modbus slave interface
- `reverseSsh`: Reverse SSH tunnel for remote support
- `bruteForcePrevention`: Brute force attack protection
- `dosAttackPrevention`: DoS attack rate limiting
- `pingLimit`: ICMP rate limiting
- `timeoutSeconds`: Session timeout
- `privateWan`: Treat WAN as private network

**Use Cases**:
- Security hardening
- Remote management access control
- Attack prevention
- Reverse SSH for NAT traversal

---

### 12. `remoteMgmt` - Remote Management (Cloud)

**Purpose**: Configure Multi-Tech cloud management integration

**Example Response**:
```json
{
  "remoteMgmt": {
    "enabled": true,
    "serverName": "api.multitech.com",
    "checkInIntervals": {
      "enabled": true,
      "interval": 14400
    },
    "checkInDayTime": {
      "enabled": false,
      "mode": "DAILY",
      "dailyTime": "",
      "customTime": "",
      "customDays": []
    },
    "allowFirmwareUpgrade": true,
    "allowRadioFirmwareUpgrade": true,
    "allowConfigurationUpgrade": true,
    "allowConfigurationUpload": true
  }
}
```

**Fields**:
- `enabled`: Enable remote management
- `serverName`: Cloud management server
- `checkInIntervals`: Periodic check-in configuration
- `checkInDayTime`: Scheduled check-in times
- `allowFirmwareUpgrade`: Allow cloud firmware updates
- `allowRadioFirmwareUpgrade`: Allow cloud radio firmware updates
- `allowConfigurationUpgrade`: Allow cloud config updates
- `allowConfigurationUpload`: Allow uploading config to cloud

**Use Cases**:
- Fleet management
- Remote firmware updates
- Centralized configuration management
- Device monitoring

---

### 13. `resetButton` - Physical Reset Button Configuration

**Purpose**: Configure physical reset button behavior

**Example Response**:
```json
{
  "resetButton": {
    "enabled": true,
    "resetToFactoryDefault": true,
    "resetToUserDefinedDefault": false
  }
}
```

**Fields**:
- `enabled`: Enable reset button
- `resetToFactoryDefault`: Reset to factory defaults
- `resetToUserDefinedDefault`: Reset to user-defined defaults

**Use Cases**:
- Device recovery
- Field deployment reset
- Security (disable button in production)

---

### 14. `smtp` - SMTP Email Configuration

**Purpose**: Configure SMTP for email notifications

**Example Response**:
```json
{
  "smtp": {
    "enabled": false,
    "serverAddress": "",
    "serverPort": 465,
    "sourceEmailAddress": "",
    "authEnabled": false,
    "username": "",
    "password": "",
    "tls": {
      "enabled": true,
      "startTls": false,
      "certCheck": false
    },
    "maxNumberOfEntries": 50
  }
}
```

**Fields**:
- `enabled`: Enable SMTP
- `serverAddress`: SMTP server hostname/IP
- `serverPort`: SMTP port (25, 465, 587)
- `sourceEmailAddress`: From email address
- `authEnabled`: Enable SMTP authentication
- `username`, `password`: SMTP credentials
- `tls`: TLS/SSL configuration
- `maxNumberOfEntries`: Max email queue size

**Use Cases**:
- Alert notifications
- Event reporting
- System status emails

---

### 15. `sntp` - SNTP/NTP Time Synchronization

**Purpose**: Configure NTP time synchronization

**Example Response**:
```json
{
  "sntp": {
    "enabled": true,
    "servers": ["time.nist.gov"],
    "pool": {
      "enabled": true,
      "server": "north-america.pool.ntp.org"
    },
    "minpoll": 6,
    "maxpoll": 10,
    "timeZone": "UTC",
    "timeZones": ["Africa/Abidjan", "America/New_York", ...]
  }
}
```

**Fields**:
- `enabled`: Enable NTP
- `servers[]`: NTP server list
- `pool`: NTP pool configuration
- `minpoll`: Minimum poll interval (2^n seconds)
- `maxpoll`: Maximum poll interval (2^n seconds)
- `timeZone`: Device timezone
- `timeZones[]`: Available timezones (read-only)

**Use Cases**:
- Accurate timekeeping
- Log timestamp accuracy
- Certificate validation

---

### 16. `waninfo` - WAN Interface Status

**Purpose**: Get real-time WAN interface status (read-only)

**Example Response**:
```json
{
  "waninfo": {
    "wans": [
      {
        "name": "eth0",
        "interface": "eth0",
        "type": "ETHERNET",
        "enabled": true,
        "available": true,
        "current": true,
        "ip": "172.16.33.111",
        "subnet": "255.255.0.0",
        "gateway": ""
      },
      {
        "name": "ppp0",
        "interface": "ppp0",
        "type": "CELLULAR",
        "enabled": true,
        "available": true,
        "current": false,
        "ip": "",
        "subnet": "",
        "gateway": ""
      }
    ]
  }
}
```

**Fields**:
- `wans[]`: Array of WAN interfaces
  - `name`: Interface name
  - `interface`: System interface name
  - `type`: `ETHERNET`, `CELLULAR`, `WIFI`
  - `enabled`: Interface enabled
  - `available`: Interface available/connected
  - `current`: Currently active WAN
  - `ip`, `subnet`, `gateway`: Network configuration

**Use Cases**:
- WAN failover monitoring
- Connection status dashboard
- Network diagnostics

---

### 17. `wanmngr` - WAN Manager (Failover/Load Balancing)

**Purpose**: Configure WAN failover and load balancing

**Example Response**:
```json
{
  "wanmngr": {
    "mode": "FAILOVER",
    "wans": [
      {
        "interface": "eth0",
        "priority": 1,
        "weight": 1,
        "monitor": {
          "mode": "ACTIVE",
          "checkInterval": 60,
          "active": {
            "type": "ICMP",
            "hostname": "www.google.com",
            "icmpCount": 5,
            "tcpPort": 80
          }
        }
      }
    ]
  }
}
```

**Fields**:
- `mode`: `FAILOVER` or `LOAD_BALANCE`
- `wans[]`: WAN interface configurations
  - `interface`: Interface name
  - `priority`: Failover priority (1=highest)
  - `weight`: Load balancing weight
  - `monitor`: Health check configuration
    - `mode`: `ACTIVE` or `PASSIVE`
    - `checkInterval`: Check interval (seconds)
    - `active.type`: `ICMP` or `TCP`
    - `active.hostname`: Target hostname/IP
    - `active.icmpCount`: Ping count
    - `active.tcpPort`: TCP port for checks

**Use Cases**:
- WAN failover
- Load balancing across multiple WANs
- Connection health monitoring
- Automatic failback

---

### 18. `ipPipes` - IP Pipes (Port Forwarding/Tunneling)

**Purpose**: Configure IP pipe tunnels for advanced routing

**Example Response**:
```json
{
  "ipPipes": []
}
```

**Fields**:
- Array of IP pipe configurations (device-specific)

**Use Cases**:
- Advanced port forwarding
- IP tunneling
- Custom routing scenarios

---

### 19. `notificationEventGroup` - Event Notification Groups

**Purpose**: Configure event notifications via email, SMS, and SNMP

**Example Response**:
```json
{
  "notificationEventGroup": {
    "groups": [],
    "eventsGroups": [
      {
        "eventGroup": "Authentication",
        "enabled": false,
        "email": false,
        "sms": false,
        "snmp": false,
        "notifyGroup": ""
      },
      {
        "eventGroup": "Chassis",
        "enabled": false,
        "email": false,
        "sms": false,
        "snmp": false,
        "notifyGroup": ""
      },
      {
        "eventGroup": "Configuration",
        "enabled": false,
        "email": false,
        "sms": false,
        "snmp": false,
        "notifyGroup": ""
      },
      {
        "eventGroup": "Link",
        "enabled": false,
        "email": false,
        "sms": false,
        "snmp": false,
        "notifyGroup": ""
      },
      {
        "eventGroup": "Security",
        "enabled": false,
        "email": false,
        "sms": false,
        "snmp": false,
        "notifyGroup": ""
      },
      {
        "eventGroup": "Wan",
        "enabled": false,
        "email": false,
        "sms": false,
        "snmp": false,
        "notifyGroup": ""
      }
    ]
  }
}
```

**Fields**:
- `groups[]`: Notification recipient groups
- `eventsGroups[]`: Event group configurations
  - `eventGroup`: Event category (Authentication, Chassis, Configuration, Link, Security, Wan)
  - `enabled`: Enable notifications for this group
  - `email`: Send email notifications
  - `sms`: Send SMS notifications
  - `snmp`: Send SNMP traps
  - `notifyGroup`: Recipient group name

**Use Cases**:
- Alert management
- Event-driven notifications
- Integration with monitoring systems
- Security event tracking

---

### 20. `customAppsConfig` - Custom Applications Configuration

**Purpose**: Configure custom application installation settings

**Example Response**:
```json
{
  "customAppsConfig": {
    "enabled": true,
    "allowed": true,
    "backupOnInstall": true
  }
}
```

**Fields**:
- `enabled`: Enable custom apps
- `allowed`: Allow custom app installation
- `backupOnInstall`: Backup config before app install

**Use Cases**:
- Custom application deployment
- Third-party integrations
- User-developed applications

---

## Additional Endpoints to Investigate

The following endpoints from the manifest were not fully tested but may contain additional functionality:

- `btCommand` - Bluetooth command interface
- `btDevices` - Bluetooth device management
- `extIo` - External I/O configuration
- `extUsb` - External USB configuration
- `log` - System log configuration
- `saveAndRestore` - Save/restore configuration
- `stateIo` - State I/O configuration
- `help` - API help/documentation endpoint
- `cloudConnector` - Cloud connector configuration (returned null)

## Schema Availability

All discovered endpoints have corresponding JSON schemas in:
- `/mtsDeviceAPI/json/permissive/<endpoint>.schema.json`
- `/mtsDeviceAPI/json/strict/<endpoint>.schema.json`

## Testing Notes

- All endpoints tested on MTCAP3 firmware 7.4.0-BETA2
- All endpoints return HTTP 200 with valid session
- All endpoints support standard GET operations
- PUT/POST/DELETE operations not fully tested
- Some endpoints may be device-model specific

## Recommendations

1. **Update API Documentation**: Add all 19+ endpoints to the main API documentation
2. **Schema Validation**: Review JSON schemas for complete field documentation
3. **Permission Testing**: Test each endpoint with different user roles (admin/user/guest)
4. **CRUD Testing**: Test PUT/POST/DELETE operations where applicable
5. **Device Compatibility**: Test on other device models (MTCDT, Conduit, etc.)
6. **Command Endpoints**: Check for associated command endpoints (e.g., `/api/command/...`)

## Next Steps

1. Generate OpenAPI specs for new endpoints
2. Create Postman collection examples
3. Document permission requirements per endpoint
4. Add to quick reference guide
5. Test modification operations (PUT/POST)
6. Verify on multiple device models
