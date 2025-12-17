# WAN Failover Configuration Guide
## MTS Device API - High Availability Networking

### 📋 Table of Contents
1. [Overview](#overview)
2. [Quick Start](#quick-start)
3. [Configuration Modes](#configuration-modes)
4. [Health Check Strategies](#health-check-strategies)
5. [Configuration Examples](#configuration-examples)
6. [Monitoring](#monitoring)
7. [Troubleshooting](#troubleshooting)
8. [Best Practices](#best-practices)

---

## Overview

The MTS Device WAN Manager provides automatic failover and load balancing across multiple WAN interfaces, ensuring continuous connectivity even when primary connections fail.

### Supported Interfaces
- **Ethernet**: eth0, eth1, eth2
- **WiFi**: wlan0
- **Cellular**: ppp0 (LTE/4G/3G)

### Key Features
- **Automatic Failover**: Seamless switchover when primary WAN fails
- **Load Balancing**: Distribute traffic across multiple WANs
- **Active Health Monitoring**: ICMP ping or TCP port checks
- **Priority-Based Routing**: Control which WAN is preferred
- **Weight-Based Distribution**: Fine-tune load balancing ratios

### API Endpoints

**Configuration**:
```bash
GET  /api?fields=wanmngr     # Get WAN manager configuration
PUT  /api/wanmngr            # Update WAN manager configuration
```

**Status**:
```bash
GET  /api?fields=waninfo     # Get current WAN status
```

---

## Quick Start

### 1. Check Current WAN Status

```bash
# Login
curl -k -X POST https://gateway-ip/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"your-password"}' \
  -c cookies.txt

# Get WAN status
curl -k -s "https://gateway-ip/api?fields=waninfo" -b cookies.txt | jq .
```

**Example Response**:
```json
{
  "code": 200,
  "status": "success",
  "result": {
    "waninfo": {
      "wans": [
        {
          "interface": "eth0",
          "name": "eth0",
          "type": "ETHERNET",
          "ip": "172.16.33.111",
          "subnet": "255.255.0.0",
          "gateway": "",
          "enabled": true,
          "available": true,
          "current": true
        },
        {
          "interface": "ppp0",
          "name": "ppp0",
          "type": "CELLULAR",
          "ip": "",
          "enabled": true,
          "available": true,
          "current": false
        }
      ]
    }
  }
}
```

**Key Fields**:
- `enabled`: Interface is configured for WAN
- `available`: Interface passed health checks
- `current`: Interface is currently active for routing

### 2. Get Current Configuration

```bash
curl -k -s "https://gateway-ip/api?fields=wanmngr" -b cookies.txt | jq .
```

### 3. Configure Failover

See [Configuration Examples](#configuration-examples) below.

---

## Configuration Modes

### FAILOVER Mode (Recommended)

**Use Case**: High availability with automatic failover to backup WAN

**Behavior**:
- Uses highest priority available WAN
- Automatically switches to next priority when primary fails
- Switches back to primary when it recovers

**Configuration**:
```json
{
  "mode": "FAILOVER",
  "wans": [
    {
      "interface": "eth0",
      "priority": 1,
      "monitor": { ... }
    },
    {
      "interface": "ppp0",
      "priority": 2,
      "monitor": { ... }
    }
  ]
}
```

**Priority Rules**:
- Lower number = higher priority
- Priority 1 is always preferred
- If priority 1 fails, switches to priority 2
- If priority 1 recovers, switches back

### LOADBALANCE Mode

**Use Case**: Distribute traffic across multiple WANs for increased bandwidth

**Behavior**:
- Distributes connections across all available WANs
- Uses weight to control distribution ratio
- Removes failed WANs from pool

**Configuration**:
```json
{
  "mode": "LOADBALANCE",
  "wans": [
    {
      "interface": "eth0",
      "weight": 3,
      "monitor": { ... }
    },
    {
      "interface": "ppp0",
      "weight": 1,
      "monitor": { ... }
    }
  ]
}
```

**Weight Rules**:
- Higher weight = more connections
- Weight 3:1 means eth0 gets 75% of connections
- Only applies to new connections (not bandwidth)

---

## Health Check Strategies

### Active Monitoring (Recommended)

Actively tests connectivity by sending probes.

#### ICMP Ping (Default)

**Best for**: General internet connectivity

```json
{
  "monitor": {
    "mode": "ACTIVE",
    "checkInterval": 60,
    "active": {
      "type": "ICMP",
      "hostname": "www.google.com",
      "icmpCount": 5
    }
  }
}
```

**Parameters**:
- `hostname`: Target to ping (IP or domain)
- `icmpCount`: Number of pings per check (default: 5)
- `checkInterval`: Seconds between checks (default: 60)

**Health Criteria**:
- ✅ Healthy: At least 1 ping succeeds
- ❌ Unhealthy: All pings fail

**Recommendations**:
- Use reliable targets (8.8.8.8, 1.1.1.1, www.google.com)
- Increase `icmpCount` for cellular (10+) due to higher latency
- Use shorter intervals (30s) for critical applications

#### TCP Port Check

**Best for**: Specific service availability

```json
{
  "monitor": {
    "mode": "ACTIVE",
    "checkInterval": 60,
    "active": {
      "type": "TCP",
      "hostname": "api.example.com",
      "tcpPort": 443
    }
  }
}
```

**Parameters**:
- `hostname`: Target server
- `tcpPort`: Port to check (80, 443, etc.)

**Health Criteria**:
- ✅ Healthy: TCP connection succeeds
- ❌ Unhealthy: Connection fails or times out

**Use Cases**:
- Verify specific API endpoint is reachable
- Check VPN server availability
- Ensure cloud service connectivity

### Passive Monitoring

**Not Recommended**: Only checks if interface has IP address, doesn't verify actual connectivity.

```json
{
  "monitor": {
    "mode": "PASSIVE"
  }
}
```

---

## Configuration Examples

### Example 1: Ethernet Primary, Cellular Backup

**Scenario**: Office gateway with wired ethernet primary and LTE backup

```bash
curl -k -X PUT https://gateway-ip/api/wanmngr \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
  "mode": "FAILOVER",
  "wans": [
    {
      "interface": "eth0",
      "priority": 1,
      "weight": 1,
      "monitor": {
        "mode": "ACTIVE",
        "checkInterval": 30,
        "active": {
          "type": "ICMP",
          "hostname": "8.8.8.8",
          "icmpCount": 3
        }
      }
    },
    {
      "interface": "ppp0",
      "priority": 2,
      "weight": 1,
      "monitor": {
        "mode": "ACTIVE",
        "checkInterval": 60,
        "active": {
          "type": "ICMP",
          "hostname": "8.8.8.8",
          "icmpCount": 10
        }
      }
    }
  ]
}'

# Save configuration
curl -k -X POST https://gateway-ip/api/command/save -b cookies.txt
```

**Behavior**:
1. Uses eth0 (ethernet) as primary
2. Checks eth0 every 30 seconds
3. If eth0 fails, switches to ppp0 (cellular)
4. When eth0 recovers, switches back

### Example 2: Dual Ethernet with Load Balancing

**Scenario**: Two ethernet connections for increased bandwidth

```bash
curl -k -X PUT https://gateway-ip/api/wanmngr \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
  "mode": "LOADBALANCE",
  "wans": [
    {
      "interface": "eth0",
      "priority": 1,
      "weight": 2,
      "monitor": {
        "mode": "ACTIVE",
        "checkInterval": 60,
        "active": {
          "type": "ICMP",
          "hostname": "www.google.com",
          "icmpCount": 5
        }
      }
    },
    {
      "interface": "eth1",
      "priority": 1,
      "weight": 1,
      "monitor": {
        "mode": "ACTIVE",
        "checkInterval": 60,
        "active": {
          "type": "ICMP",
          "hostname": "www.google.com",
          "icmpCount": 5
        }
      }
    }
  ]
}'

curl -k -X POST https://gateway-ip/api/command/save -b cookies.txt
```

**Behavior**:
- Distributes connections 2:1 (eth0 gets 67%, eth1 gets 33%)
- If either fails, uses only the healthy interface
- When both recover, resumes load balancing

### Example 3: Triple Redundancy (Ethernet, WiFi, Cellular)

**Scenario**: Maximum availability with three WAN options

```bash
curl -k -X PUT https://gateway-ip/api/wanmngr \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
  "mode": "FAILOVER",
  "wans": [
    {
      "interface": "eth0",
      "priority": 1,
      "weight": 1,
      "monitor": {
        "mode": "ACTIVE",
        "checkInterval": 30,
        "active": {
          "type": "ICMP",
          "hostname": "8.8.8.8",
          "icmpCount": 3
        }
      }
    },
    {
      "interface": "wlan0",
      "priority": 2,
      "weight": 1,
      "monitor": {
        "mode": "ACTIVE",
        "checkInterval": 45,
        "active": {
          "type": "ICMP",
          "hostname": "8.8.8.8",
          "icmpCount": 5
        }
      }
    },
    {
      "interface": "ppp0",
      "priority": 3,
      "weight": 1,
      "monitor": {
        "mode": "ACTIVE",
        "checkInterval": 60,
        "active": {
          "type": "ICMP",
          "hostname": "8.8.8.8",
          "icmpCount": 10
        }
      }
    }
  ]
}'

curl -k -X POST https://gateway-ip/api/command/save -b cookies.txt
```

**Behavior**:
1. Uses eth0 (wired) as primary
2. Falls back to wlan0 (WiFi) if eth0 fails
3. Falls back to ppp0 (cellular) if both fail
4. Always returns to highest priority available

### Example 4: API-Specific Health Check

**Scenario**: Ensure connectivity to specific cloud API

```bash
curl -k -X PUT https://gateway-ip/api/wanmngr \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
  "mode": "FAILOVER",
  "wans": [
    {
      "interface": "eth0",
      "priority": 1,
      "weight": 1,
      "monitor": {
        "mode": "ACTIVE",
        "checkInterval": 30,
        "active": {
          "type": "TCP",
          "hostname": "api.myservice.com",
          "tcpPort": 443
        }
      }
    },
    {
      "interface": "ppp0",
      "priority": 2,
      "weight": 1,
      "monitor": {
        "mode": "ACTIVE",
        "checkInterval": 60,
        "active": {
          "type": "TCP",
          "hostname": "api.myservice.com",
          "tcpPort": 443
        }
      }
    }
  ]
}'

curl -k -X POST https://gateway-ip/api/command/save -b cookies.txt
```

**Behavior**:
- Only considers WAN healthy if it can reach specific API
- Useful for mission-critical cloud services
- Ensures failover happens if API becomes unreachable

---

## Monitoring

### Check WAN Status

```bash
# Get current WAN status
curl -k -s "https://gateway-ip/api?fields=waninfo&inactivity=true" \
  -b cookies.txt | jq '.result.waninfo.wans[]'
```

**Monitor These Fields**:
- `current`: Which WAN is active
- `available`: Which WANs passed health checks
- `enabled`: Which WANs are configured

### Automated Monitoring Script

```python
#!/usr/bin/env python3
"""
WAN Failover Monitoring Script
Alerts on WAN changes and failures
"""

import requests
import time
import json
from datetime import datetime

class WANMonitor:
    def __init__(self, gateway_url, username, password):
        self.gateway_url = gateway_url
        self.session = requests.Session()
        self.login(username, password)
        self.last_current = None
    
    def login(self, username, password):
        url = f"{self.gateway_url}/api/login"
        data = {"username": username, "password": password}
        response = self.session.post(url, json=data, verify=False)
        if response.status_code == 200:
            print(f"✓ Logged in to {self.gateway_url}")
    
    def get_wan_status(self):
        url = f"{self.gateway_url}/api?fields=waninfo&inactivity=true"
        response = self.session.get(url, verify=False)
        if response.status_code == 200:
            return response.json().get('result', {}).get('waninfo', {})
        return None
    
    def check_wans(self):
        wan_info = self.get_wan_status()
        if not wan_info:
            return
        
        wans = wan_info.get('wans', [])
        
        # Find current WAN
        current_wan = None
        available_wans = []
        unavailable_wans = []
        
        for wan in wans:
            if wan.get('enabled'):
                if wan.get('current'):
                    current_wan = wan['interface']
                if wan.get('available'):
                    available_wans.append(wan['interface'])
                else:
                    unavailable_wans.append(wan['interface'])
        
        # Check for WAN change
        if self.last_current and current_wan != self.last_current:
            self.alert('WARNING', 
                f'WAN failover: {self.last_current} → {current_wan}')
        
        self.last_current = current_wan
        
        # Alert on unavailable WANs
        if unavailable_wans:
            self.alert('WARNING', 
                f'WANs unavailable: {", ".join(unavailable_wans)}')
        
        # Log status
        timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        print(f"[{timestamp}] Current: {current_wan} | "
              f"Available: {len(available_wans)} | "
              f"Unavailable: {len(unavailable_wans)}")
    
    def alert(self, level, message):
        timestamp = datetime.now().isoformat()
        print(f"[{timestamp}] {level}: {message}")
        
        # TODO: Implement alerting (email, Slack, etc.)
    
    def monitor_loop(self, interval=60):
        print(f"Starting WAN monitoring (interval: {interval}s)")
        
        while True:
            try:
                self.check_wans()
                time.sleep(interval)
            except KeyboardInterrupt:
                print("\nMonitoring stopped")
                break
            except Exception as e:
                print(f"Error: {e}")
                time.sleep(interval)

# Usage
if __name__ == "__main__":
    monitor = WANMonitor(
        gateway_url="https://172.16.33.111",
        username="admin",
        password="admin2019!"
    )
    
    monitor.monitor_loop(interval=30)
```

### Bash Monitoring Script

```bash
#!/bin/bash
# Simple WAN monitoring script

GATEWAY="https://172.16.33.111"
COOKIES="/tmp/gateway-cookies.txt"

# Login
curl -k -X POST "$GATEWAY/api/login" \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin2019!"}' \
  -c "$COOKIES" -s > /dev/null

echo "Monitoring WAN status..."

LAST_CURRENT=""

while true; do
  # Get WAN status
  WAN_INFO=$(curl -k -s "$GATEWAY/api?fields=waninfo&inactivity=true" -b "$COOKIES")
  
  # Extract current WAN
  CURRENT=$(echo "$WAN_INFO" | jq -r '.result.waninfo.wans[] | select(.current==true) | .interface')
  
  # Check for change
  if [ -n "$LAST_CURRENT" ] && [ "$CURRENT" != "$LAST_CURRENT" ]; then
    echo "$(date '+%Y-%m-%d %H:%M:%S') - ALERT: WAN changed from $LAST_CURRENT to $CURRENT"
  fi
  
  LAST_CURRENT="$CURRENT"
  
  # Count available WANs
  AVAILABLE=$(echo "$WAN_INFO" | jq '[.result.waninfo.wans[] | select(.enabled==true and .available==true)] | length')
  UNAVAILABLE=$(echo "$WAN_INFO" | jq '[.result.waninfo.wans[] | select(.enabled==true and .available==false)] | length')
  
  echo "$(date '+%Y-%m-%d %H:%M:%S') - Current: $CURRENT | Available: $AVAILABLE | Unavailable: $UNAVAILABLE"
  
  sleep 60
done
```

---

## Troubleshooting

### Failover Not Happening

**Symptoms**: Primary WAN fails but gateway doesn't switch to backup

**Possible Causes**:

1. **Health checks not configured**
   ```bash
   # Check monitor configuration
   curl -k -s "https://gateway-ip/api?fields=wanmngr" -b cookies.txt | \
     jq '.result.wanmngr.wans[].monitor'
   ```
   
   **Solution**: Ensure `mode: "ACTIVE"` is set

2. **Backup WAN not available**
   ```bash
   # Check WAN availability
   curl -k -s "https://gateway-ip/api?fields=waninfo" -b cookies.txt | \
     jq '.result.waninfo.wans[] | {interface, available, enabled}'
   ```
   
   **Solution**: Verify backup WAN is configured and has connectivity

3. **Health check target unreachable**
   
   **Solution**: Use reliable targets (8.8.8.8, 1.1.1.1)

### Frequent Flapping

**Symptoms**: WAN switches back and forth repeatedly

**Possible Causes**:

1. **Unstable connection**
   
   **Solution**: Increase `checkInterval` and `icmpCount`
   ```json
   {
     "checkInterval": 120,
     "active": {
       "icmpCount": 10
     }
   }
   ```

2. **Target host issues**
   
   **Solution**: Use multiple reliable targets or switch to TCP checks

3. **Network congestion**
   
   **Solution**: Implement hysteresis (manual failback)

### Load Balancing Not Working

**Symptoms**: All traffic goes through one WAN

**Possible Causes**:

1. **Mode set to FAILOVER instead of LOADBALANCE**
   
   **Solution**: Change mode
   ```bash
   curl -k -X PUT https://gateway-ip/api/wanmngr \
     -H "Content-Type: application/json" \
     -b cookies.txt \
     -d '{"mode": "LOADBALANCE", ...}'
   ```

2. **Only one WAN available**
   
   **Solution**: Verify all WANs pass health checks

3. **Existing connections**
   
   **Note**: Load balancing only affects NEW connections. Existing long-lived connections stay on original WAN.

### Configuration Not Saving

**Symptoms**: Configuration reverts after reboot

**Solution**: Always save after configuration changes
```bash
curl -k -X POST https://gateway-ip/api/command/save -b cookies.txt
```

---

## Best Practices

### 1. Use Reliable Health Check Targets

**✅ Good Targets**:
- `8.8.8.8` (Google DNS)
- `1.1.1.1` (Cloudflare DNS)
- `www.google.com`
- Your critical API endpoint

**❌ Avoid**:
- Unreliable hosts
- Hosts that block ICMP
- Hosts with rate limiting

### 2. Tune Check Intervals

**Fast Failover** (30-45s):
- Critical applications
- Real-time services
- High availability requirements

**Normal** (60s):
- General purpose
- Balanced performance

**Conservative** (120s+):
- Unstable connections
- Reduce false positives
- Lower device load

### 3. Cellular-Specific Settings

Cellular connections have higher latency and variability:

```json
{
  "interface": "ppp0",
  "monitor": {
    "checkInterval": 60,
    "active": {
      "type": "ICMP",
      "icmpCount": 10,
      "hostname": "8.8.8.8"
    }
  }
}
```

**Key Points**:
- Increase `icmpCount` to 10+
- Use longer `checkInterval` (60-120s)
- Expect occasional packet loss

### 4. Priority Assignment

**Recommended Priority Order**:
1. Wired Ethernet (fastest, most reliable)
2. WiFi (fast, moderate reliability)
3. Cellular (slower, variable, data costs)

### 5. Always Save Configuration

```bash
# After any configuration change
curl -k -X POST https://gateway-ip/api/command/save -b cookies.txt
```

### 6. Test Failover

Manually test failover before relying on it:

1. Configure failover
2. Monitor WAN status
3. Disconnect primary WAN (unplug cable)
4. Verify failover to backup
5. Reconnect primary WAN
6. Verify failback to primary

### 7. Monitor WAN Status

Implement automated monitoring:
- Alert on WAN changes
- Alert on WAN failures
- Log failover events
- Track failover frequency

### 8. Document Your Configuration

Keep records of:
- WAN priorities and weights
- Health check targets
- Check intervals
- Expected failover behavior

---

## Summary

### Quick Reference

**Failover Configuration**:
```bash
PUT /api/wanmngr
{
  "mode": "FAILOVER",
  "wans": [
    {"interface": "eth0", "priority": 1, "monitor": {...}},
    {"interface": "ppp0", "priority": 2, "monitor": {...}}
  ]
}
```

**Check Status**:
```bash
GET /api?fields=waninfo
```

**Save Configuration**:
```bash
POST /api/command/save
```

### Key Takeaways

1. **Use FAILOVER mode** for high availability
2. **Use LOADBALANCE mode** for increased bandwidth
3. **Configure ACTIVE monitoring** with ICMP or TCP checks
4. **Set appropriate priorities**: Ethernet > WiFi > Cellular
5. **Use reliable health check targets** (8.8.8.8, 1.1.1.1)
6. **Increase icmpCount for cellular** (10+)
7. **Always save configuration** after changes
8. **Monitor WAN status** for failover events
9. **Test failover** before production deployment

### Related Documentation

- [MONITORING-BEST-PRACTICES.md](MONITORING-BEST-PRACTICES.md) - Monitoring strategies
- [UNDOCUMENTED-ENDPOINTS-FINDINGS.md](UNDOCUMENTED-ENDPOINTS-FINDINGS.md) - WAN endpoints details
- [API-QUICK-REFERENCE.md](API-QUICK-REFERENCE.md) - API syntax reference

---

**Document Version**: 1.0  
**Last Updated**: December 17, 2025  
**Tested On**: MTCAP3, Firmware 7.4.0-BETA2, API 7.2.0
