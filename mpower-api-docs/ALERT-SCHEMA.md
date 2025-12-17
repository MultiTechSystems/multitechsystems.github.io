# Alert Configuration Schema

Complete schema documentation for the `alert` resource in the mPower Device API.

## Overview

The `alert` resource configures the device's notification and alerting system. It enables automated notifications via email, SMS, and SNMP traps for various system events including connectivity failures, data usage thresholds, signal strength issues, and more.

## Top-Level Structure

```json
{
  "__v": 1,
  "alerts": [
    // Array of alert configurations
  ],
  "groups": [
    // Array of recipient groups
  ]
}
```

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `__v` | integer | Yes | Schema version |
| `alerts` | array | Yes | Array of alert event configurations |
| `groups` | array | Yes | Array of notification recipient groups |

## Alert Event Types

The device supports 12 predefined alert event types:

| Event Type | Description | Notification Methods |
|------------|-------------|---------------------|
| `Data Usage` | Cellular data usage threshold exceeded | Email, SMS, SNMP |
| `Signal Strength` | Cellular signal strength below threshold | Email, SMS, SNMP |
| `Device Reboot` | Device has rebooted | Email, SMS, SNMP |
| `Ethernet Failure` | Ethernet interface failure | Email, SMS, SNMP |
| `WifiWan Failure` | WiFi WAN connection failure | Email, SMS, SNMP |
| `Cellular Failure` | Cellular connection failure | Email, SMS, SNMP |
| `Ethernet Traffic` | Periodic Ethernet traffic report | Email, SMS |
| `Wifi Traffic` | Periodic WiFi traffic report | Email, SMS |
| `Cellular Traffic` | Periodic cellular traffic report | Email, SMS |
| `WAN Failover` | WAN interface failover occurred | Email, SMS, SNMP |
| `Ping Failure` | Ping test failed | Email, SMS, SNMP |
| `Password Change` | User password changed | Email, SMS |

**Note**: Traffic events (Ethernet/Wifi/Cellular Traffic) and Password Change do not support SNMP traps.

## Alert Object Structure

Each alert in the `alerts` array has common and event-specific properties.

### Common Alert Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `event` | string | Yes | Event type (see table above) |
| `enabled` | boolean | Yes | Enable/disable this alert |
| `email` | boolean | Yes | Send notifications via email |
| `sms` | boolean | Yes | Send notifications via SMS |
| `snmp` | boolean | Conditional | Send SNMP traps (not available for traffic/password events) |
| `notifyGroup` | string | Yes | Name of recipient group to notify |

### Event-Specific Properties

#### Data Usage Alert

| Property | Type | Description |
|----------|------|-------------|
| `dataAllowedPerMonth` | integer | Data plan limit in MB (minimum 0 when enabled, -1 = unlimited) |
| `dataPlanType` | string | "Monthly" or "Custom Interval" |
| `dataPlanStartDate` | integer | Day of month when plan resets (1-31, for Monthly type) |
| `interval` | integer | Billing cycle length in days (1+, for Custom Interval type) |
| `intervalStartDate` | string | Start date for custom interval billing cycle |
| `alertOn` | integer | Alert threshold percentage (1-100%, default: 80) |

#### Signal Strength Alert

| Property | Type | Description |
|----------|------|-------------|
| `signalStrength` | integer | Signal threshold in dBm (-200 to -1, default: -60) |
| `interval` | integer | Check interval in seconds (5-604800, default: 60) |
| `minWaitTimeAfterAlarm` | integer | Hours before next alert (1-999, default: 24) |

#### Device Reboot Alert

No additional properties required. Alert is sent whenever device reboots.

#### Interface Failure Alerts (Ethernet/WiFi/Cellular)

| Property | Type | Description |
|----------|------|-------------|
| `interval` | integer | Check interval in seconds (5-604800, default: 60) |
| `minWaitTimeAfterAlarm` | integer | Hours before next alert (1-999, default: 24) |

#### Traffic Report Alerts (Ethernet/WiFi/Cellular)

| Property | Type | Description |
|----------|------|-------------|
| `periodicInterval` | integer | Report interval in hours (1-576, default: 24) |

#### WAN Failover Alert

| Property | Type | Description |
|----------|------|-------------|
| `minWaitTimeAfterStart` | integer | Ignore failovers for N seconds after startup (60-3600, default: 60) |
| `failoverEvent` | string | When to alert: "Always", "Interface went down", "Interface came up" |

#### Ping Failure Alert

| Property | Type | Description |
|----------|------|-------------|
| `pingInterval` | integer | Ping test interval in minutes (1-1440) |
| `netIface` | string | Network interface to use ("ANY" or interface name) |
| `ipAddress` | string | Target IP address or hostname to ping |
| `count` | integer | Number of ping requests to send (4-20, default: 10) |
| `failThreshold` | integer | Failed requests that trigger alert (1-20) |

#### Password Change Alert

No additional properties required. Alert is sent when any user password is changed.

## Recipient Groups

The `groups` array defines notification recipient groups.

### Group Object

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | string | Yes | Unique group name |
| `emails` | array | Yes | Array of email recipients |
| `phones` | array | Yes | Array of SMS recipients |

### Email Recipient Object

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | string | Yes | Recipient name |
| `email` | string | Yes | Email address |

### Phone Recipient Object

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | string | Yes | Recipient name |
| `phone` | string | Yes | Phone number (international format recommended) |

## Configuration Examples

### Example 1: Data Usage Alert

```json
{
  "__v": 1,
  "alerts": [
    {
      "event": "Data Usage",
      "enabled": true,
      "email": true,
      "sms": false,
      "snmp": false,
      "notifyGroup": "Admins",
      "dataAllowedPerMonth": 5000,
      "dataPlanType": "Monthly",
      "dataPlanStartDate": 1,
      "interval": 30,
      "intervalStartDate": "",
      "alertOn": 80
    }
  ],
  "groups": [
    {
      "name": "Admins",
      "emails": [
        {
          "name": "Admin",
          "email": "admin@example.com"
        }
      ],
      "phones": []
    }
  ]
}
```

### Example 2: Signal Strength Monitoring

```json
{
  "__v": 1,
  "alerts": [
    {
      "event": "Signal Strength",
      "enabled": true,
      "email": true,
      "sms": true,
      "snmp": true,
      "notifyGroup": "Operations",
      "signalStrength": -90,
      "interval": 300,
      "minWaitTimeAfterAlarm": 6
    }
  ],
  "groups": [
    {
      "name": "Operations",
      "emails": [
        {
          "name": "Ops Team",
          "email": "ops@example.com"
        }
      ],
      "phones": [
        {
          "name": "On-Call",
          "phone": "+1234567890"
        }
      ]
    }
  ]
}
```

### Example 3: Connectivity Failure Alerts

```json
{
  "__v": 1,
  "alerts": [
    {
      "event": "Cellular Failure",
      "enabled": true,
      "email": true,
      "sms": false,
      "snmp": true,
      "notifyGroup": "Network Team",
      "interval": 60,
      "minWaitTimeAfterAlarm": 1
    },
    {
      "event": "Ethernet Failure",
      "enabled": true,
      "email": true,
      "sms": false,
      "snmp": true,
      "notifyGroup": "Network Team",
      "interval": 60,
      "minWaitTimeAfterAlarm": 1
    }
  ],
  "groups": [
    {
      "name": "Network Team",
      "emails": [
        {
          "name": "Network Admin",
          "email": "netadmin@example.com"
        },
        {
          "name": "NOC",
          "email": "noc@example.com"
        }
      ],
      "phones": []
    }
  ]
}
```

### Example 4: WAN Failover Monitoring

```json
{
  "__v": 1,
  "alerts": [
    {
      "event": "WAN Failover",
      "enabled": true,
      "email": true,
      "sms": true,
      "snmp": false,
      "notifyGroup": "IT Staff",
      "minWaitTimeAfterStart": 120,
      "failoverEvent": "Interface went down"
    }
  ],
  "groups": [
    {
      "name": "IT Staff",
      "emails": [
        {
          "name": "IT Manager",
          "email": "it@example.com"
        }
      ],
      "phones": [
        {
          "name": "IT On-Call",
          "phone": "+1987654321"
        }
      ]
    }
  ]
}
```

### Example 5: Ping Failure Monitoring

```json
{
  "__v": 1,
  "alerts": [
    {
      "event": "Ping Failure",
      "enabled": true,
      "email": true,
      "sms": false,
      "snmp": true,
      "notifyGroup": "Monitoring",
      "pingInterval": 5,
      "netIface": "ppp0",
      "ipAddress": "8.8.8.8",
      "count": 10,
      "failThreshold": 8
    }
  ],
  "groups": [
    {
      "name": "Monitoring",
      "emails": [
        {
          "name": "Monitor",
          "email": "monitor@example.com"
        }
      ],
      "phones": []
    }
  ]
}
```

### Example 6: Periodic Traffic Reports

```json
{
  "__v": 1,
  "alerts": [
    {
      "event": "Cellular Traffic",
      "enabled": true,
      "email": true,
      "sms": false,
      "notifyGroup": "Reports",
      "periodicInterval": 24
    },
    {
      "event": "Ethernet Traffic",
      "enabled": true,
      "email": true,
      "sms": false,
      "notifyGroup": "Reports",
      "periodicInterval": 168
    }
  ],
  "groups": [
    {
      "name": "Reports",
      "emails": [
        {
          "name": "Management",
          "email": "reports@example.com"
        }
      ],
      "phones": []
    }
  ]
}
```

### Example 7: Comprehensive Alert Configuration

```json
{
  "__v": 1,
  "alerts": [
    {
      "event": "Data Usage",
      "enabled": true,
      "email": true,
      "sms": true,
      "snmp": false,
      "notifyGroup": "Admins",
      "dataAllowedPerMonth": 10000,
      "dataPlanType": "Monthly",
      "dataPlanStartDate": 1,
      "interval": 30,
      "intervalStartDate": "",
      "alertOn": 90
    },
    {
      "event": "Signal Strength",
      "enabled": true,
      "email": true,
      "sms": false,
      "snmp": true,
      "notifyGroup": "Operations",
      "signalStrength": -95,
      "interval": 600,
      "minWaitTimeAfterAlarm": 12
    },
    {
      "event": "Device Reboot",
      "enabled": true,
      "email": true,
      "sms": true,
      "snmp": true,
      "notifyGroup": "Admins"
    },
    {
      "event": "Cellular Failure",
      "enabled": true,
      "email": true,
      "sms": true,
      "snmp": true,
      "notifyGroup": "Operations",
      "interval": 120,
      "minWaitTimeAfterAlarm": 2
    },
    {
      "event": "WAN Failover",
      "enabled": true,
      "email": true,
      "sms": false,
      "snmp": false,
      "notifyGroup": "Network Team",
      "minWaitTimeAfterStart": 180,
      "failoverEvent": "Always"
    },
    {
      "event": "Ping Failure",
      "enabled": true,
      "email": true,
      "sms": false,
      "snmp": true,
      "notifyGroup": "Monitoring",
      "pingInterval": 5,
      "netIface": "ANY",
      "ipAddress": "8.8.8.8",
      "count": 10,
      "failThreshold": 7
    }
  ],
  "groups": [
    {
      "name": "Admins",
      "emails": [
        {
          "name": "Admin",
          "email": "admin@example.com"
        }
      ],
      "phones": [
        {
          "name": "Admin Mobile",
          "phone": "+1234567890"
        }
      ]
    },
    {
      "name": "Operations",
      "emails": [
        {
          "name": "Ops Team",
          "email": "ops@example.com"
        }
      ],
      "phones": []
    },
    {
      "name": "Network Team",
      "emails": [
        {
          "name": "Network Admin",
          "email": "netadmin@example.com"
        }
      ],
      "phones": []
    },
    {
      "name": "Monitoring",
      "emails": [
        {
          "name": "Monitor",
          "email": "monitor@example.com"
        }
      ],
      "phones": []
    }
  ]
}
```

## API Operations

### Get Alert Configuration

```bash
GET /api?fields=alert
```

### Update Alert Configuration

```bash
PUT /api/alert
Content-Type: application/json

{
  "__v": 1,
  "alerts": [...],
  "groups": [...]
}
```

### Save Configuration

```bash
POST /api/command/save
```

## Configuration Workflows

### Workflow 1: Enable Data Usage Alert

```bash
# Configure 5GB monthly data plan with 80% alert threshold
curl -X PUT http://192.168.2.1/api/alert \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "__v": 1,
    "alerts": [{
      "event": "Data Usage",
      "enabled": true,
      "email": true,
      "sms": false,
      "snmp": false,
      "notifyGroup": "Admins",
      "dataAllowedPerMonth": 5000,
      "dataPlanType": "Monthly",
      "dataPlanStartDate": 1,
      "interval": 30,
      "intervalStartDate": "",
      "alertOn": 80
    }],
    "groups": [{
      "name": "Admins",
      "emails": [{"name": "Admin", "email": "admin@example.com"}],
      "phones": []
    }]
  }'

# Save
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
```

### Workflow 2: Enable Connectivity Monitoring

```bash
# Monitor cellular and ethernet connectivity
curl -X PUT http://192.168.2.1/api/alert \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "__v": 1,
    "alerts": [
      {
        "event": "Cellular Failure",
        "enabled": true,
        "email": true,
        "sms": true,
        "snmp": true,
        "notifyGroup": "Operations",
        "interval": 60,
        "minWaitTimeAfterAlarm": 1
      },
      {
        "event": "Ethernet Failure",
        "enabled": true,
        "email": true,
        "sms": false,
        "snmp": true,
        "notifyGroup": "Operations",
        "interval": 60,
        "minWaitTimeAfterAlarm": 1
      }
    ],
    "groups": [{
      "name": "Operations",
      "emails": [{"name": "Ops", "email": "ops@example.com"}],
      "phones": [{"name": "On-Call", "phone": "+1234567890"}]
    }]
  }'

# Save
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
```

### Workflow 3: Setup Ping Monitoring

```bash
# Monitor connectivity to Google DNS
curl -X PUT http://192.168.2.1/api/alert \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "__v": 1,
    "alerts": [{
      "event": "Ping Failure",
      "enabled": true,
      "email": true,
      "sms": false,
      "snmp": false,
      "notifyGroup": "Monitoring",
      "pingInterval": 5,
      "netIface": "ANY",
      "ipAddress": "8.8.8.8",
      "count": 10,
      "failThreshold": 8
    }],
    "groups": [{
      "name": "Monitoring",
      "emails": [{"name": "Monitor", "email": "monitor@example.com"}],
      "phones": []
    }]
  }'

# Save
curl -X POST http://192.168.2.1/api/command/save -b cookies.txt
```

## Important Notes

### Prerequisites

#### For Email Alerts
- SMTP must be configured (see `smtp` resource)
- Valid SMTP server and credentials
- Test email delivery before relying on alerts

#### For SMS Alerts
- SMS must be configured (see `sms` resource)
- Device must have cellular capability
- Sufficient SMS credits/plan

#### For SNMP Traps
- SNMP traps must be configured (see `snmp` resource)
- Trap receivers must be defined
- Not available for traffic reports and password change events

### Alert Behavior

#### Notification Throttling

Most alerts have `minWaitTimeAfterAlarm` to prevent notification flooding:
- After an alert is sent, subsequent alerts are suppressed for N hours
- Prevents alert storms during extended outages
- Default is typically 24 hours

#### Startup Grace Period

Some alerts have `minWaitTimeAfterStart`:
- Ignores events for N seconds after device boot
- Prevents false alerts during startup
- Useful for WAN failover alerts

#### Periodic Reports

Traffic report alerts send periodic notifications:
- `periodicInterval` defines frequency in hours
- Sent even when no issues occur
- Useful for regular status updates

### Data Plan Types

#### Monthly
- Resets on specific day of month (`dataPlanStartDate`)
- Day 1-31
- If day doesn't exist in month (e.g., 31st in February), uses last day of month

#### Custom Interval
- Resets every N days (`interval`)
- Starts from `intervalStartDate`
- More flexible for non-monthly billing cycles

### Notification Methods

You can enable multiple notification methods per alert:
- **Email only**: `email: true, sms: false, snmp: false`
- **SMS only**: `email: false, sms: true, snmp: false`
- **All methods**: `email: true, sms: true, snmp: true`
- **None**: `email: false, sms: false, snmp: false` (alert disabled effectively)

### Group Management

- Groups can be reused across multiple alerts
- One group can have multiple email and phone recipients
- Empty arrays are allowed (e.g., no phone recipients)
- Group names must match `notifyGroup` in alerts

## Common Use Cases

### Use Case 1: Data Plan Management

Monitor cellular data usage to avoid overage charges:
- Set `dataAllowedPerMonth` to your plan limit
- Set `alertOn` to 80-90% for advance warning
- Use `dataPlanType: "Monthly"` with correct start date
- Enable email and SMS for critical alerts

### Use Case 2: Connectivity Monitoring

Monitor all WAN interfaces for failures:
- Enable alerts for Cellular, Ethernet, and WiFi failures
- Set short `interval` (60 seconds) for quick detection
- Use `minWaitTimeAfterAlarm: 1` hour for timely follow-ups
- Send to operations team via email and SNMP

### Use Case 3: Remote Site Health Check

Use ping monitoring to verify internet connectivity:
- Ping reliable target (e.g., 8.8.8.8)
- Set `pingInterval` to 5-10 minutes
- Use `failThreshold` slightly less than `count` (e.g., 8 of 10)
- Alert via email and SNMP to monitoring system

### Use Case 4: Signal Quality Monitoring

Monitor cellular signal strength in remote locations:
- Set `signalStrength` threshold (e.g., -90 dBm)
- Use longer `interval` (5-10 minutes) to avoid false positives
- Set `minWaitTimeAfterAlarm` to 6-12 hours
- Useful for identifying antenna/placement issues

### Use Case 5: Audit Trail

Track security-related events:
- Enable "Device Reboot" alert
- Enable "Password Change" alert
- Send to security/audit team
- Maintain audit trail of system changes

## Troubleshooting

### Alerts Not Being Sent

**Check:**
1. Alert is enabled: `enabled: true`
2. Notification method is enabled (email/sms/snmp)
3. Recipient group exists and has recipients
4. Prerequisites are configured:
   - SMTP for email alerts
   - SMS for SMS alerts
   - SNMP traps for SNMP alerts
5. No throttling in effect (`minWaitTimeAfterAlarm`)

### Email Alerts Not Working

**Check:**
- SMTP configuration: `GET /api?fields=smtp`
- SMTP server is reachable
- Authentication credentials are correct
- Email addresses are valid
- Check device event log for SMTP errors

### SMS Alerts Not Working

**Check:**
- SMS configuration: `GET /api?fields=sms`
- Device has cellular capability
- SIM card is active
- Phone numbers are in correct format
- SMS service is enabled

### SNMP Traps Not Received

**Check:**
- SNMP traps are configured: `GET /api?fields=snmp`
- Trap receivers are defined and enabled
- Receiver addresses are correct
- Community/credentials match
- Firewall allows UDP port 162
- Event type supports SNMP (not traffic/password events)

### Data Usage Alert Not Triggering

**Check:**
- `dataAllowedPerMonth` is not -1 (unlimited)
- Current usage is above `alertOn` threshold
- Check current usage: `GET /api?fields=stats`
- Verify billing cycle dates are correct

### Signal Strength Alert Too Sensitive

**Adjust:**
- Lower threshold (more negative): e.g., -95 instead of -85
- Increase `interval` to reduce check frequency
- Increase `minWaitTimeAfterAlarm` to reduce notification frequency

### Ping Failure False Positives

**Adjust:**
- Increase `count` (more pings per test)
- Increase `failThreshold` (allow more failures)
- Increase `pingInterval` (test less frequently)
- Choose more reliable ping target
- Verify network path to target

## Alert History

Alert events are logged in the event log:

```bash
# View alert history
GET /api?fields=eventlog
```

## Related Resources

- [SMTP Configuration](SERVICES-SCHEMAS.md) - Required for email alerts
- [SMS Configuration](SERVICES-SCHEMAS.md) - Required for SMS alerts
- [SNMP Configuration](SNMP-SCHEMA.md) - Required for SNMP trap alerts
- [Event Log](API-README.md#monitoring-resources) - Alert history
- [Main API Documentation](API-README.md)

---

**Last Updated**: December 17, 2025  
**Schema Version**: 1
