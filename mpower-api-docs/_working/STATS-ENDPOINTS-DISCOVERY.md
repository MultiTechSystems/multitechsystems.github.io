# Stats Endpoints Discovery

**Date**: December 17, 2025  
**Gateway**: MTCAP3 (172.16.33.111)  
**Firmware**: 7.4.0-BETA2-7-g7e2e653f

## Overview

The `/api?fields=stats` endpoint provides access to real-time device statistics and status information. Unlike configuration endpoints, stats endpoints are primarily read-only and provide monitoring data.

## Stats Categories

The stats endpoint returns an array of available stat categories:

```bash
GET /api?fields=stats
```

**Available Categories**:
1. `bluetooth` - Bluetooth statistics
2. `cpu` - CPU usage and load
3. `dashboard` - Comprehensive dashboard view (most useful!)
4. `dns` - DNS server statistics
5. `docker` - Docker container statistics
6. `dualSim` - Dual SIM status
7. `extUsb` - External USB statistics
8. `flashStatus` - Flash storage status
9. `gps` - GPS location and status
10. `gre` - GRE tunnel statistics
11. `iface` - Network interface statistics
12. `ifaceHistory` - Interface history
13. `ifaceTotal` - Total interface statistics
14. `ipsec` - IPsec tunnel statistics
15. `ipsecLinkStatus` - IPsec link status
16. `lan0` - LAN interface stats
17. `lora` - LoRa gateway statistics
18. `loraNodes` - LoRa node statistics
19. `memory` - Memory usage
20. `modbus` - Modbus statistics
21. `ovpn` - OpenVPN tunnel statistics
22. `packages` - Installed packages
23. `radio` - Cellular radio statistics
24. `radioFwuStatus` - Radio firmware update status
25. `radioPdpConfEffective` - Effective PDP context configuration
26. `radioPdpConfStatic` - Static PDP context configuration
27. `radioSimStatus` - SIM card status
28. `remoteMgmt` - Remote management status
29. `rootfsInfo` - Root filesystem information
30. `saveAndRestore` - Save/restore status
31. `serial` - Serial port statistics
32. `service` - Service status
33. `storage` - Storage statistics

## Accessing Stats

### Individual Stat Category

```bash
GET /api?fields=stats/<category>
```

Example:
```bash
GET /api?fields=stats/radio
GET /api?fields=stats/memory
GET /api?fields=stats/cpu
```

### Dashboard Stats (Recommended)

The `stats/dashboard` endpoint provides a comprehensive system overview with the most commonly needed information:

```bash
GET /api?fields=stats/dashboard
```

**Dashboard Includes**:
- `system` - Full system information
- `cellular_summary` - Complete cellular configuration and status
- `ni` - All network interfaces with detailed configuration
- `stats_radio` - Current radio status
- `stats_memory` - Memory usage
- `stats_iface_ppp0` - Cellular interface status
- `stats_iface_wlan0` - WiFi client status
- `stats_dns` - DNS configuration
- `stats_gps` - GPS status
- `stats_radioPdpConfEffective` - Active APN configuration
- `waninfo_currentWan` - Currently active WAN interface
- `wifi_ap` - WiFi AP status
- `wifi_client` - WiFi client status
- `dhcp` - DHCP server status
- `ipPassthrough` - IP passthrough configuration
- `bluetooth` - Bluetooth status
- `stats_dualSim` - Dual SIM status

## Key Stats Endpoints

### 1. Radio Stats (`stats/radio`)

Real-time cellular radio information:

```json
{
  "stats_radio": {
    "carrier": "AT&T Wireless Inc.",
    "network": "AT&T",
    "model": "LE910C4-WWXD",
    "firmware": "M0F.603007",
    "imei": "016666003998671",
    "imsi": "310170410613094",
    "iccid": "89011703274106130943",
    "rssi": 25,
    "rssidBm": "-63",
    "netreg": "SEARCHING",
    "service": "Unknown",
    "roaming": false,
    "mcc": "310",
    "mnc": "260",
    "lac": "74B4",
    "cid": "4F8A",
    "channel": "512",
    "type": "LTE",
    "supportedCellularModes": "2g,3g,4g",
    "simCarrierCode": "att",
    "simMcc": "310",
    "simMnc": "170"
  }
}
```

**Key Fields**:
- `rssi` / `rssidBm` - Signal strength
- `netreg` - Network registration status
- `carrier` / `network` - Network operator
- `roaming` - Roaming status
- `lac` / `cid` - Cell tower location
- `mcc` / `mnc` - Mobile country/network code

### 2. Memory Stats (`stats/memory`)

System memory usage:

```json
{
  "stats_memory": {
    "mem": {
      "total": "483524608",
      "used": "53329920",
      "free": "372219904",
      "shared": "4263936",
      "buff/cache": "57974784",
      "available": "414412800"
    },
    "swap": {
      "total": "0",
      "used": "0",
      "free": "0"
    }
  }
}
```

### 3. Interface Stats (`stats/iface_<interface>`)

Per-interface statistics:

```bash
GET /api?fields=stats/iface_ppp0  # Cellular interface
GET /api?fields=stats/iface_eth0  # Ethernet interface
GET /api?fields=stats/iface_wlan0 # WiFi interface
```

Example for cellular (`stats/iface_ppp0`):
```json
{
  "stats_iface_ppp0": {
    "localIp": "10.123.45.67",
    "remoteIp": "10.64.64.64",
    "dnsServers": ["8.8.8.8", "8.8.4.4"],
    "cellStatus": "Connected",
    "cellStatusCode": "CONNECTED",
    "cellularMode": "LTE",
    "rssi": "25",
    "rssiDbm": "-63",
    "roaming": false,
    "tower": "4F8A",
    "areaCode": "74B4",
    "pci": "123",
    "uptime": 15234,
    "timestamp": "12/17/2025 20:40:36"
  }
}
```

### 4. CPU Stats (`stats/cpu`)

CPU usage and load:

```bash
GET /api?fields=stats/cpu
```

### 5. GPS Stats (`stats/gps`)

GPS location and status:

```bash
GET /api?fields=stats/gps
```

### 6. LoRa Stats (`stats/lora`)

LoRa gateway statistics:

```bash
GET /api?fields=stats/lora
```

### 7. Service Stats (`stats/service`)

Service status for all system services:

```bash
GET /api?fields=stats/service
```

## LoRa-Specific Endpoints

### LoRa Status

```bash
GET /api/lora/status
```

Returns process IDs for LoRa services:
```json
{
  "lora-network-server": 12811,
  "lora-packet-forwarder": [12845],
  "lora-cloud-connector": 3332,
  "lora-basic-station": 0,
  "lora-lens-server": 0
}
```

### LoRa Gateways

```bash
GET /api/lora/gateways
```

Returns registered LoRa gateways with statistics:
```json
[
  {
    "gweui": "00-80-00-00-d0-00-42-6e",
    "ip_addr": "127.0.0.1",
    "ip_port": 56078,
    "last_seen": "2025-12-17T20:39:42Z",
    "version": 2,
    "rx_count": 2501,
    "rx_ok": 2366,
    "rx_forwarded": 2496,
    "crc_errors": 130,
    "mic_fails": 0,
    "uplinks": 2370,
    "uplinks_ok": 0,
    "uplinks_nok_mic": 0,
    "uplinks_nok_unknown": 2113,
    "downlinks": 0,
    "downlinks_ack_requested": 0,
    "downlink_count": 0,
    "tx_count": 0,
    "ack_rate": 1.0,
    "total_channels": 10,
    "rx_ch_0": 325,
    "rx_ch_1": 302,
    ...
  }
]
```

### LoRa Devices

```bash
GET /api/lora/devices
```

Returns registered LoRa end devices:
```json
[
  {
    "deveui": "00-80-00-ff-00-00-00-02",
    "name": "",
    "class": "A",
    "device_profile_id": 16,
    "network_profile_id": 1,
    "created_at": "2025-09-30T15:56:29Z",
    "last_seen": "2025-12-15T22:18:21Z",
    "last_nonce": 443,
    "last_app_nonce": 20,
    "rejoin_count": 0,
    "firmware_version": "",
    "hardware_version": "",
    "product_id": "",
    "serial_number": "",
    "tags": ""
  }
]
```

## Monitoring Use Cases

### 1. Cellular Connection Monitoring

```bash
# Get comprehensive cellular status
GET /api?fields=stats/dashboard

# Extract key fields:
# - stats_radio.rssiDbm (signal strength)
# - stats_radio.netreg (registration status)
# - stats_iface_ppp0.cellStatus (connection status)
# - stats_iface_ppp0.localIp (assigned IP)
# - cellular_summary.currentProviderProfile (active profile)
```

### 2. System Health Monitoring

```bash
# Get system health metrics
GET /api?fields=stats/cpu,stats/memory,stats/storage

# Monitor:
# - CPU load
# - Memory usage
# - Disk space
```

### 3. Network Interface Monitoring

```bash
# Get all interface stats
GET /api?fields=stats/iface,stats/ifaceTotal

# Monitor:
# - Bytes sent/received
# - Packet counts
# - Error rates
```

### 4. LoRa Network Monitoring

```bash
# Get LoRa gateway and device stats
GET /api/lora/gateways
GET /api/lora/devices
GET /api?fields=stats/lora,stats/loraNodes

# Monitor:
# - Gateway uptime
# - Packet forwarding rates
# - Device activity
# - Signal quality
```

## Polling Recommendations

### High-Frequency Monitoring (Every 5-10 seconds)
- `stats/radio` - Signal strength
- `stats/iface_ppp0` - Connection status
- `stats/cpu` - CPU load

### Medium-Frequency Monitoring (Every 30-60 seconds)
- `stats/dashboard` - Overall system status
- `stats/memory` - Memory usage
- `/api/lora/gateways` - LoRa gateway stats

### Low-Frequency Monitoring (Every 5-15 minutes)
- `stats/service` - Service health
- `stats/storage` - Disk usage
- `/api/lora/devices` - Device inventory

## Important Notes

1. **Read-Only**: Stats endpoints are read-only and do not modify device configuration
2. **Session Management**: Use `inactivity=true` parameter to prevent session timeout during monitoring:
   ```bash
   GET /api?fields=stats/radio&inactivity=true
   ```
3. **Performance**: The `stats/dashboard` endpoint is optimized for comprehensive monitoring with a single request
4. **Caching**: Some stats may be cached for a few seconds to reduce system load
5. **Permissions**: Most stats endpoints require authentication but are accessible to all user roles (admin, user, guest)

## Stats vs Configuration Endpoints

| Aspect | Stats Endpoints | Config Endpoints |
|--------|----------------|------------------|
| **Access Pattern** | `GET /api?fields=stats/<category>` | `GET /api?fields=<resource>` |
| **Purpose** | Real-time monitoring | Configuration management |
| **Mutability** | Read-only | Read/write (PUT/POST/DELETE) |
| **Update Frequency** | Real-time or near-real-time | On-demand |
| **Persistence** | Not saved | Saved with `/api/command/save` |
| **Session Timeout** | Can use `inactivity=true` | Normal session timeout |

## Example Monitoring Script

```bash
#!/bin/bash
# Monitor cellular connection

GATEWAY="https://172.16.33.111"
COOKIES="/tmp/gateway-cookies.txt"

# Login
curl -k -s -X POST "${GATEWAY}/api/login" \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin2019!"}' \
  -c "${COOKIES}"

# Monitor loop
while true; do
  # Get radio stats without updating session timeout
  stats=$(curl -k -s "${GATEWAY}/api?fields=stats/radio&inactivity=true" -b "${COOKIES}")
  
  rssi=$(echo "$stats" | jq -r '.result.stats_radio.rssidBm')
  status=$(echo "$stats" | jq -r '.result.stats_radio.netreg')
  carrier=$(echo "$stats" | jq -r '.result.stats_radio.carrier')
  
  echo "$(date): Signal: ${rssi}dBm, Status: ${status}, Carrier: ${carrier}"
  
  sleep 10
done
```

## Next Steps

1. Document all stats categories in detail
2. Create monitoring dashboard examples
3. Add stats endpoints to OpenAPI specification
4. Create Grafana/Prometheus integration examples
5. Document alert thresholds and best practices
