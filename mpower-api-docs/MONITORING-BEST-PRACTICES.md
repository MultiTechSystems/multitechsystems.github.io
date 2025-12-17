# Monitoring Best Practices Guide
## MTS Device API - Real-Time Monitoring & Alerting

### 📋 Table of Contents
1. [Overview](#overview)
2. [Quick Start](#quick-start)
3. [Monitoring Endpoints](#monitoring-endpoints)
4. [Polling Strategies](#polling-strategies)
5. [Alert Thresholds](#alert-thresholds)
6. [Integration Examples](#integration-examples)
7. [Performance Optimization](#performance-optimization)
8. [Troubleshooting](#troubleshooting)

---

## Overview

The MTS Device API provides comprehensive real-time monitoring capabilities through the `/api?fields=stats/*` endpoints. This guide covers best practices for implementing effective monitoring solutions.

### Key Capabilities
- **33+ Stats Categories**: CPU, memory, radio, interfaces, LoRa, and more
- **Dashboard Endpoint**: Single call for comprehensive system status
- **Real-Time Data**: Sub-second accuracy for critical metrics
- **Session Management**: Inactivity flag prevents timeout during monitoring
- **Flexible Polling**: Configurable intervals based on metric importance

---

## Quick Start

### Basic Monitoring Setup

```bash
# 1. Login and save session
curl -k -X POST https://gateway-ip/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"your-password"}' \
  -c cookies.txt

# 2. Get comprehensive dashboard (recommended for monitoring)
curl -k -s "https://gateway-ip/api?fields=stats/dashboard&inactivity=true" \
  -b cookies.txt | jq .

# 3. Get specific metrics
curl -k -s "https://gateway-ip/api?fields=stats/radio&inactivity=true" \
  -b cookies.txt | jq .
```

### Why Use `inactivity=true`?
- Prevents session timeout during long-running monitoring
- Essential for continuous polling scripts
- No impact on API performance

---

## Monitoring Endpoints

### 🎯 Recommended: Dashboard Endpoint

**Best for**: Initial monitoring setup, dashboards, health checks

```bash
GET /api?fields=stats/dashboard&inactivity=true
```

**Returns**: Comprehensive system status including:
- CPU usage (current, average, peak)
- Memory usage (total, used, free, buffers, cached)
- Radio signal strength and quality
- Interface statistics (all network interfaces)
- System uptime and load
- LoRa statistics (if enabled)

**Polling Interval**: 30-60 seconds

### 📊 Individual Stats Categories

#### Critical Metrics (Poll frequently)

**1. CPU Usage**
```bash
GET /api?fields=stats/cpu&inactivity=true
```
- **Metrics**: Current, 1-min avg, 5-min avg, 15-min avg
- **Polling**: Every 10-30 seconds
- **Alert Threshold**: > 80% sustained for 5 minutes

**2. Memory Usage**
```bash
GET /api?fields=stats/memory&inactivity=true
```
- **Metrics**: Total, used, free, buffers, cached, available
- **Polling**: Every 30-60 seconds
- **Alert Threshold**: < 10% available

**3. Radio Signal**
```bash
GET /api?fields=stats/radio&inactivity=true
```
- **Metrics**: RSSI, RSRP, RSRQ, SINR, signal quality
- **Polling**: Every 30-60 seconds
- **Alert Thresholds**:
  - RSSI: < -100 dBm (poor)
  - RSRQ: < -15 dB (poor)
  - SINR: < 0 dB (poor)

#### Interface Monitoring

**4. Network Interfaces**
```bash
GET /api?fields=stats/iface&inactivity=true
```
- **Metrics**: RX/TX bytes, packets, errors, drops
- **Polling**: Every 60 seconds
- **Alert Threshold**: Error rate > 1%

**Specific Interface**:
```bash
GET /api?fields=stats/iface_ppp0&inactivity=true
GET /api?fields=stats/iface_eth0&inactivity=true
```

#### LoRa Monitoring

**5. LoRa Statistics**
```bash
GET /api?fields=stats/lora&inactivity=true
```
- **Polling**: Every 60-120 seconds
- **Alert Thresholds**:
  - Packet loss > 5%
  - No uplinks for 5 minutes

**6. LoRa Service Status**
```bash
GET /api/lora/status
```
- **Returns**: Process IDs (PIDs) for LoRa services
- **Polling**: Every 60 seconds
- **Alert**: Any PID = 0 (service down)

**7. LoRa Gateways**
```bash
GET /api/lora/gateways
```
- **Metrics**: Packet counts, CRC errors, ACK rate
- **Polling**: Every 120 seconds
- **Alert Thresholds**:
  - CRC error rate > 10%
  - ACK rate < 90%

**8. LoRa Devices**
```bash
GET /api/lora/devices
```
- **Metrics**: Device status, uplink/downlink counts, last seen
- **Polling**: Every 300 seconds
- **Alert**: Device not seen for > 1 hour

---

## Polling Strategies

### Recommended Polling Intervals

| Metric Category | Interval | Rationale |
|----------------|----------|-----------|
| **Dashboard** | 30-60s | Comprehensive, optimized single call |
| **CPU** | 10-30s | Fast-changing, critical for performance |
| **Memory** | 30-60s | Slower changes, critical for stability |
| **Radio** | 30-60s | Important for connectivity, moderate changes |
| **Interfaces** | 60s | Counter-based, calculate rates over time |
| **LoRa Status** | 60s | Service health check |
| **LoRa Gateways** | 120s | Statistics, less critical |
| **LoRa Devices** | 300s | Slow-changing device list |
| **GPS** | 60-300s | Very slow-changing |
| **Temperature** | 60-120s | Slow-changing environmental metric |

### Adaptive Polling

Adjust polling based on system state:

```python
# Example: Adaptive polling logic
def get_poll_interval(metric_type, current_value):
    if metric_type == "cpu":
        if current_value > 80:
            return 10  # Poll faster when high
        elif current_value > 50:
            return 20
        else:
            return 30  # Normal polling
    elif metric_type == "radio":
        if current_value < -100:  # Poor signal
            return 15  # Poll faster
        else:
            return 60  # Normal
    return 60  # Default
```

### Staggered Polling

Avoid simultaneous requests:

```python
# Example: Stagger requests
import time

def staggered_monitoring():
    while True:
        # CPU (0s)
        fetch_cpu_stats()
        time.sleep(10)
        
        # Memory (10s)
        fetch_memory_stats()
        time.sleep(10)
        
        # Radio (20s)
        fetch_radio_stats()
        time.sleep(10)
        
        # Interfaces (30s)
        fetch_interface_stats()
        time.sleep(30)
```

---

## Alert Thresholds

### System Health

#### CPU Alerts
```yaml
WARNING:
  - CPU > 70% for 2 minutes
  - Load average (5min) > CPU cores * 0.8

CRITICAL:
  - CPU > 90% for 5 minutes
  - Load average (5min) > CPU cores * 1.5
```

#### Memory Alerts
```yaml
WARNING:
  - Available memory < 20%
  - Swap usage > 50%

CRITICAL:
  - Available memory < 10%
  - Swap usage > 80%
  - OOM (Out of Memory) events
```

### Connectivity

#### Radio Signal Alerts
```yaml
WARNING:
  - RSSI < -90 dBm
  - RSRQ < -12 dB
  - SINR < 5 dB

CRITICAL:
  - RSSI < -105 dBm
  - RSRQ < -18 dB
  - SINR < 0 dB
  - Signal lost for > 2 minutes
```

#### Interface Alerts
```yaml
WARNING:
  - Error rate > 0.5%
  - Drop rate > 0.5%
  - Interface down for > 30 seconds

CRITICAL:
  - Error rate > 2%
  - Drop rate > 2%
  - Interface down for > 5 minutes
```

### LoRa Specific

#### LoRa Service Alerts
```yaml
WARNING:
  - Service restart detected

CRITICAL:
  - Service down (PID = 0)
  - Service down for > 2 minutes
```

#### LoRa Gateway Alerts
```yaml
WARNING:
  - CRC error rate > 5%
  - ACK rate < 95%
  - No packets for 5 minutes

CRITICAL:
  - CRC error rate > 15%
  - ACK rate < 85%
  - Gateway offline for > 10 minutes
```

#### LoRa Device Alerts
```yaml
WARNING:
  - Device not seen for > 30 minutes
  - Uplink count not increasing

CRITICAL:
  - Device not seen for > 2 hours
  - Device count dropped by > 20%
```

---

## Integration Examples

### Python Monitoring Script

```python
#!/usr/bin/env python3
"""
MTS Device Monitoring Script
Polls gateway stats and sends alerts
"""

import requests
import json
import time
from datetime import datetime
from urllib3.exceptions import InsecureRequestWarning

# Suppress SSL warnings for self-signed certs
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

class GatewayMonitor:
    def __init__(self, gateway_url, username, password):
        self.gateway_url = gateway_url
        self.session = requests.Session()
        self.login(username, password)
    
    def login(self, username, password):
        """Authenticate with gateway"""
        url = f"{self.gateway_url}/api/login"
        data = {"username": username, "password": password}
        response = self.session.post(url, json=data, verify=False)
        if response.status_code == 200:
            print(f"✓ Logged in to {self.gateway_url}")
        else:
            raise Exception(f"Login failed: {response.text}")
    
    def get_stats(self, category):
        """Fetch stats for a category"""
        url = f"{self.gateway_url}/api?fields=stats/{category}&inactivity=true"
        response = self.session.get(url, verify=False)
        if response.status_code == 200:
            return response.json().get('result', {})
        return None
    
    def get_dashboard(self):
        """Fetch comprehensive dashboard stats"""
        return self.get_stats('dashboard')
    
    def check_cpu(self, stats):
        """Check CPU usage and alert if needed"""
        cpu = stats.get('cpu', {})
        current = cpu.get('current', 0)
        
        if current > 90:
            self.alert('CRITICAL', f'CPU usage: {current}%')
        elif current > 70:
            self.alert('WARNING', f'CPU usage: {current}%')
        
        return current
    
    def check_memory(self, stats):
        """Check memory usage and alert if needed"""
        memory = stats.get('memory', {})
        total = memory.get('total', 1)
        available = memory.get('available', 0)
        percent_available = (available / total) * 100
        
        if percent_available < 10:
            self.alert('CRITICAL', f'Memory available: {percent_available:.1f}%')
        elif percent_available < 20:
            self.alert('WARNING', f'Memory available: {percent_available:.1f}%')
        
        return percent_available
    
    def check_radio(self, stats):
        """Check radio signal and alert if needed"""
        radio = stats.get('radio', {})
        rssi = radio.get('rssi', 0)
        
        if rssi < -105:
            self.alert('CRITICAL', f'Radio RSSI: {rssi} dBm')
        elif rssi < -90:
            self.alert('WARNING', f'Radio RSSI: {rssi} dBm')
        
        return rssi
    
    def alert(self, level, message):
        """Send alert (implement your alerting here)"""
        timestamp = datetime.now().isoformat()
        print(f"[{timestamp}] {level}: {message}")
        
        # TODO: Implement your alerting mechanism
        # - Send email via SMTP
        # - Post to Slack/Teams
        # - Send to monitoring system (Prometheus, etc.)
        # - Write to syslog
    
    def monitor_loop(self, interval=60):
        """Main monitoring loop"""
        print(f"Starting monitoring (interval: {interval}s)")
        
        while True:
            try:
                # Get comprehensive dashboard stats
                stats = self.get_dashboard()
                
                if stats:
                    # Check all critical metrics
                    cpu = self.check_cpu(stats)
                    memory = self.check_memory(stats)
                    rssi = self.check_radio(stats)
                    
                    # Log current status
                    print(f"Status: CPU={cpu}% Memory={memory:.1f}% RSSI={rssi}dBm")
                
                time.sleep(interval)
                
            except KeyboardInterrupt:
                print("\nMonitoring stopped")
                break
            except Exception as e:
                print(f"Error: {e}")
                time.sleep(interval)

# Usage
if __name__ == "__main__":
    monitor = GatewayMonitor(
        gateway_url="https://172.16.33.111",
        username="admin",
        password="admin2019!"
    )
    
    monitor.monitor_loop(interval=30)
```

### Bash Monitoring Script

```bash
#!/bin/bash
# Simple bash monitoring script

GATEWAY="https://172.16.33.111"
COOKIES="/tmp/gateway-cookies.txt"

# Login
curl -k -X POST "$GATEWAY/api/login" \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin2019!"}' \
  -c "$COOKIES" -s > /dev/null

echo "Starting monitoring..."

while true; do
  # Get dashboard stats
  STATS=$(curl -k -s "$GATEWAY/api?fields=stats/dashboard&inactivity=true" -b "$COOKIES")
  
  # Extract key metrics
  CPU=$(echo "$STATS" | jq -r '.result.cpu.current // 0')
  MEM_AVAIL=$(echo "$STATS" | jq -r '.result.memory.available // 0')
  MEM_TOTAL=$(echo "$STATS" | jq -r '.result.memory.total // 1')
  RSSI=$(echo "$STATS" | jq -r '.result.radio.rssi // 0')
  
  # Calculate memory percentage
  MEM_PERCENT=$(echo "scale=1; ($MEM_AVAIL / $MEM_TOTAL) * 100" | bc)
  
  # Check thresholds and alert
  if (( $(echo "$CPU > 90" | bc -l) )); then
    echo "CRITICAL: CPU usage ${CPU}%"
  fi
  
  if (( $(echo "$MEM_PERCENT < 10" | bc -l) )); then
    echo "CRITICAL: Memory available ${MEM_PERCENT}%"
  fi
  
  if (( $(echo "$RSSI < -105" | bc -l) )); then
    echo "CRITICAL: Radio RSSI ${RSSI} dBm"
  fi
  
  # Log status
  echo "$(date '+%Y-%m-%d %H:%M:%S') - CPU: ${CPU}% | Memory: ${MEM_PERCENT}% | RSSI: ${RSSI} dBm"
  
  sleep 60
done
```

### Prometheus Exporter

```python
#!/usr/bin/env python3
"""
MTS Device Prometheus Exporter
Exposes gateway metrics in Prometheus format
"""

from prometheus_client import start_http_server, Gauge
import requests
import time

# Define metrics
cpu_usage = Gauge('mts_cpu_usage_percent', 'CPU usage percentage')
memory_available = Gauge('mts_memory_available_percent', 'Available memory percentage')
radio_rssi = Gauge('mts_radio_rssi_dbm', 'Radio RSSI in dBm')
interface_rx_bytes = Gauge('mts_interface_rx_bytes', 'Interface RX bytes', ['interface'])
interface_tx_bytes = Gauge('mts_interface_tx_bytes', 'Interface TX bytes', ['interface'])

class MTSExporter:
    def __init__(self, gateway_url, username, password):
        self.gateway_url = gateway_url
        self.session = requests.Session()
        self.login(username, password)
    
    def login(self, username, password):
        url = f"{self.gateway_url}/api/login"
        data = {"username": username, "password": password}
        self.session.post(url, json=data, verify=False)
    
    def collect_metrics(self):
        """Collect and update Prometheus metrics"""
        url = f"{self.gateway_url}/api?fields=stats/dashboard&inactivity=true"
        response = self.session.get(url, verify=False)
        
        if response.status_code == 200:
            stats = response.json().get('result', {})
            
            # Update CPU metric
            cpu = stats.get('cpu', {}).get('current', 0)
            cpu_usage.set(cpu)
            
            # Update memory metric
            memory = stats.get('memory', {})
            total = memory.get('total', 1)
            available = memory.get('available', 0)
            mem_percent = (available / total) * 100
            memory_available.set(mem_percent)
            
            # Update radio metric
            radio = stats.get('radio', {})
            rssi = radio.get('rssi', 0)
            radio_rssi.set(rssi)
            
            # Update interface metrics
            interfaces = stats.get('interfaces', {})
            for iface_name, iface_stats in interfaces.items():
                rx_bytes = iface_stats.get('rx_bytes', 0)
                tx_bytes = iface_stats.get('tx_bytes', 0)
                interface_rx_bytes.labels(interface=iface_name).set(rx_bytes)
                interface_tx_bytes.labels(interface=iface_name).set(tx_bytes)

if __name__ == '__main__':
    # Start Prometheus HTTP server
    start_http_server(9100)
    
    exporter = MTSExporter(
        gateway_url="https://172.16.33.111",
        username="admin",
        password="admin2019!"
    )
    
    print("Prometheus exporter running on :9100")
    
    while True:
        exporter.collect_metrics()
        time.sleep(15)  # Scrape interval
```

---

## Performance Optimization

### Minimize API Calls

**❌ Bad**: Multiple individual calls
```bash
curl .../api?fields=stats/cpu
curl .../api?fields=stats/memory
curl .../api?fields=stats/radio
curl .../api?fields=stats/iface
```

**✅ Good**: Single dashboard call
```bash
curl .../api?fields=stats/dashboard
```

### Connection Reuse

**❌ Bad**: New connection each time
```python
for i in range(100):
    requests.get(url)  # New TCP connection
```

**✅ Good**: Reuse session
```python
session = requests.Session()
for i in range(100):
    session.get(url)  # Reuse connection
```

### Efficient JSON Parsing

**❌ Bad**: Parse entire response
```bash
curl ... | jq .
```

**✅ Good**: Extract only needed fields
```bash
curl ... | jq -r '.result.cpu.current'
```

### Caching

Cache slow-changing data:

```python
import time

class CachedMonitor:
    def __init__(self):
        self.cache = {}
        self.cache_ttl = {
            'gps': 300,      # 5 minutes
            'devices': 300,  # 5 minutes
            'cpu': 10,       # 10 seconds
        }
    
    def get_cached(self, key, fetch_func):
        now = time.time()
        if key in self.cache:
            data, timestamp = self.cache[key]
            if now - timestamp < self.cache_ttl.get(key, 60):
                return data
        
        # Fetch fresh data
        data = fetch_func()
        self.cache[key] = (data, now)
        return data
```

---

## Troubleshooting

### Session Timeouts

**Problem**: Monitoring script stops working after 15 minutes

**Solution**: Always use `inactivity=true` parameter
```bash
curl ".../api?fields=stats/dashboard&inactivity=true"
```

### High API Load

**Problem**: Gateway becomes slow with monitoring

**Solutions**:
1. Increase polling intervals
2. Use dashboard endpoint instead of individual calls
3. Implement staggered polling
4. Cache slow-changing data

### Missing Data

**Problem**: Some stats fields are null or missing

**Reasons**:
- Feature not available on device model
- Service not running (e.g., LoRa disabled)
- Hardware not present (e.g., GPS)

**Solution**: Check for null values before processing
```python
rssi = stats.get('radio', {}).get('rssi')
if rssi is not None:
    check_rssi(rssi)
```

### Authentication Errors

**Problem**: 401 Unauthorized errors

**Solutions**:
1. Re-login if session expires
2. Implement automatic re-authentication
3. Check cookie file permissions

```python
def api_call_with_retry(self, url):
    response = self.session.get(url)
    if response.status_code == 401:
        # Re-authenticate
        self.login(self.username, self.password)
        response = self.session.get(url)
    return response
```

---

## Summary

### Key Takeaways

1. **Use Dashboard Endpoint**: Single call for comprehensive monitoring
2. **Add `inactivity=true`**: Prevent session timeouts
3. **Poll Appropriately**: Match intervals to metric importance
4. **Set Smart Thresholds**: Balance false positives vs. missed alerts
5. **Reuse Connections**: Use session objects for efficiency
6. **Handle Errors**: Implement retry logic and null checks
7. **Stagger Requests**: Avoid simultaneous API calls
8. **Cache When Possible**: Reduce load for slow-changing data

### Next Steps

1. Review [STATS-ENDPOINTS-DISCOVERY.md](STATS-ENDPOINTS-DISCOVERY.md) for complete endpoint reference
2. Check [WAN-FAILOVER-GUIDE.md](WAN-FAILOVER-GUIDE.md) for WAN monitoring specifics
3. See [API-QUICK-REFERENCE.md](API-QUICK-REFERENCE.md) for endpoint syntax
4. Explore [SESSION-SUMMARY-2025-12-17.md](SESSION-SUMMARY-2025-12-17.md) for discovery details

---

**Document Version**: 1.0  
**Last Updated**: December 17, 2025  
**Tested On**: MTCAP3, Firmware 7.4.0-BETA2, API 7.2.0
