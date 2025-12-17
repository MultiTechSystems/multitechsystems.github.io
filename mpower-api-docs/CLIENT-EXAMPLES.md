# API Client Examples
## MTS Device API - Python, Node.js, and Bash Implementations

### 📋 Table of Contents
1. [Overview](#overview)
2. [Python Client](#python-client)
3. [Node.js Client](#nodejs-client)
4. [Bash Client](#bash-client)
5. [Common Use Cases](#common-use-cases)
6. [Error Handling](#error-handling)
7. [Best Practices](#best-practices)

---

## Overview

This guide provides production-ready client implementations for the MTS Device API in multiple languages.

### Features Covered
- Authentication and session management
- Resource queries (GET)
- Configuration updates (PUT)
- Command execution (POST)
- Error handling and retries
- Monitoring and stats collection

---

## Python Client

### Installation

```bash
pip install requests urllib3
```

### Basic Client Class

```python
#!/usr/bin/env python3
"""
MTS Device API Client
Production-ready Python implementation
"""

import requests
import json
import time
from typing import Dict, Any, Optional
from urllib3.exceptions import InsecureRequestWarning

# Suppress SSL warnings for self-signed certificates
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

class MTSDeviceClient:
    """Client for MTS Device API"""
    
    def __init__(self, base_url: str, username: str, password: str, 
                 verify_ssl: bool = False):
        """
        Initialize MTS Device API client
        
        Args:
            base_url: Gateway URL (e.g., https://172.16.33.111)
            username: Admin username
            password: Admin password
            verify_ssl: Verify SSL certificates (default: False for self-signed)
        """
        self.base_url = base_url.rstrip('/')
        self.username = username
        self.password = password
        self.verify_ssl = verify_ssl
        self.session = requests.Session()
        self.token = None
        
        # Login on initialization
        self.login()
    
    def login(self) -> Dict[str, Any]:
        """
        Authenticate with the gateway
        
        Returns:
            Login response with token
        
        Raises:
            Exception: If login fails
        """
        url = f"{self.base_url}/api/login"
        data = {
            "username": self.username,
            "password": self.password
        }
        
        response = self.session.post(url, json=data, verify=self.verify_ssl)
        
        if response.status_code == 200:
            result = response.json()
            self.token = result.get('result', {}).get('token')
            print(f"✓ Logged in to {self.base_url}")
            return result
        else:
            raise Exception(f"Login failed: {response.status_code} - {response.text}")
    
    def logout(self) -> Dict[str, Any]:
        """Logout and invalidate session"""
        url = f"{self.base_url}/api/logout"
        response = self.session.post(url, verify=self.verify_ssl)
        return response.json() if response.status_code == 200 else {}
    
    def _request(self, method: str, endpoint: str, **kwargs) -> requests.Response:
        """
        Make API request with automatic re-authentication
        
        Args:
            method: HTTP method (GET, POST, PUT, DELETE)
            endpoint: API endpoint
            **kwargs: Additional arguments for requests
        
        Returns:
            Response object
        """
        url = f"{self.base_url}{endpoint}"
        kwargs['verify'] = self.verify_ssl
        
        response = self.session.request(method, url, **kwargs)
        
        # Re-authenticate if session expired
        if response.status_code == 401:
            print("Session expired, re-authenticating...")
            self.login()
            response = self.session.request(method, url, **kwargs)
        
        return response
    
    def get(self, fields: str, inactivity: bool = False, 
            default: bool = False) -> Dict[str, Any]:
        """
        Query resource(s)
        
        Args:
            fields: Resource name(s) (e.g., 'system', 'stats/dashboard')
            inactivity: Prevent session timeout (useful for monitoring)
            default: Get default values instead of current config
        
        Returns:
            API response as dictionary
        
        Example:
            >>> client.get('system')
            >>> client.get('stats/dashboard', inactivity=True)
            >>> client.get('cellular', default=True)
        """
        params = {'fields': fields}
        if inactivity:
            params['inactivity'] = 'true'
        if default:
            params['default'] = 'true'
        
        response = self._request('GET', '/api', params=params)
        return response.json() if response.status_code == 200 else {}
    
    def put(self, resource: str, data: Dict[str, Any]) -> Dict[str, Any]:
        """
        Update resource configuration
        
        Args:
            resource: Resource name (e.g., 'system', 'cellular')
            data: Configuration data
        
        Returns:
            API response as dictionary
        
        Example:
            >>> client.put('system', {'description': 'My Gateway'})
        """
        url = f"/api/{resource}"
        response = self._request('PUT', url, json=data)
        return response.json() if response.status_code == 200 else {}
    
    def post(self, endpoint: str, data: Optional[Dict[str, Any]] = None) -> Dict[str, Any]:
        """
        Execute command
        
        Args:
            endpoint: Command endpoint (e.g., 'command/save', 'command/restart')
            data: Optional command data
        
        Returns:
            API response as dictionary
        
        Example:
            >>> client.post('command/save')
            >>> client.post('command/restart')
        """
        url = f"/api/{endpoint}"
        response = self._request('POST', url, json=data or {})
        return response.json() if response.status_code == 200 else {}
    
    def delete(self, endpoint: str) -> Dict[str, Any]:
        """
        Delete resource
        
        Args:
            endpoint: Resource endpoint
        
        Returns:
            API response as dictionary
        """
        url = f"/api/{endpoint}"
        response = self._request('DELETE', url)
        return response.json() if response.status_code == 200 else {}
    
    # Convenience methods for common operations
    
    def get_system_info(self) -> Dict[str, Any]:
        """Get system information"""
        return self.get('system')
    
    def get_status(self) -> Dict[str, Any]:
        """Get device status"""
        return self.get('status')
    
    def get_cellular_info(self) -> Dict[str, Any]:
        """Get cellular configuration"""
        return self.get('cellular')
    
    def get_dashboard_stats(self) -> Dict[str, Any]:
        """Get comprehensive dashboard statistics"""
        return self.get('stats/dashboard', inactivity=True)
    
    def get_radio_stats(self) -> Dict[str, Any]:
        """Get radio signal statistics"""
        return self.get('stats/radio', inactivity=True)
    
    def get_wan_status(self) -> Dict[str, Any]:
        """Get WAN status"""
        return self.get('waninfo')
    
    def get_wan_config(self) -> Dict[str, Any]:
        """Get WAN manager configuration"""
        return self.get('wanmngr')
    
    def get_lora_status(self) -> Dict[str, Any]:
        """Get LoRa service status"""
        response = self._request('GET', '/api/lora/status')
        return response.json() if response.status_code == 200 else {}
    
    def get_lora_gateways(self) -> Dict[str, Any]:
        """Get LoRa gateways"""
        response = self._request('GET', '/api/lora/gateways')
        return response.json() if response.status_code == 200 else {}
    
    def get_lora_devices(self) -> Dict[str, Any]:
        """Get LoRa devices"""
        response = self._request('GET', '/api/lora/devices')
        return response.json() if response.status_code == 200 else {}
    
    def save_config(self) -> Dict[str, Any]:
        """Save current configuration"""
        return self.post('command/save')
    
    def restart_device(self) -> Dict[str, Any]:
        """Restart the device"""
        return self.post('command/restart')
    
    def revert_config(self) -> Dict[str, Any]:
        """Revert to saved configuration"""
        return self.post('command/revert')


# Example usage
if __name__ == "__main__":
    # Initialize client
    client = MTSDeviceClient(
        base_url="https://172.16.33.111",
        username="admin",
        password="admin2019!"
    )
    
    # Get system information
    system = client.get_system_info()
    print(f"\nDevice: {system.get('result', {}).get('system', {}).get('deviceId')}")
    
    # Get dashboard stats
    stats = client.get_dashboard_stats()
    cpu = stats.get('result', {}).get('cpu', {}).get('current', 0)
    print(f"CPU Usage: {cpu}%")
    
    # Get WAN status
    wan = client.get_wan_status()
    wans = wan.get('result', {}).get('waninfo', {}).get('wans', [])
    for w in wans:
        if w.get('current'):
            print(f"Active WAN: {w.get('interface')} ({w.get('type')})")
    
    # Logout
    client.logout()
```

### Monitoring Example

```python
#!/usr/bin/env python3
"""
Continuous monitoring example
"""

import time
from mts_client import MTSDeviceClient

def monitor_gateway(client: MTSDeviceClient, interval: int = 60):
    """
    Monitor gateway metrics continuously
    
    Args:
        client: MTSDeviceClient instance
        interval: Polling interval in seconds
    """
    print(f"Starting monitoring (interval: {interval}s)")
    print("Press Ctrl+C to stop\n")
    
    try:
        while True:
            # Get dashboard stats
            stats = client.get_dashboard_stats()
            result = stats.get('result', {})
            
            # Extract metrics
            cpu = result.get('cpu', {}).get('current', 0)
            memory = result.get('memory', {})
            mem_percent = (memory.get('used', 0) / memory.get('total', 1)) * 100
            radio = result.get('radio', {})
            rssi = radio.get('rssi', 0)
            
            # Display
            timestamp = time.strftime('%Y-%m-%d %H:%M:%S')
            print(f"[{timestamp}] CPU: {cpu:5.1f}% | "
                  f"Memory: {mem_percent:5.1f}% | "
                  f"RSSI: {rssi:4.0f} dBm")
            
            # Alert on thresholds
            if cpu > 80:
                print(f"  ⚠ WARNING: High CPU usage ({cpu}%)")
            if mem_percent > 80:
                print(f"  ⚠ WARNING: High memory usage ({mem_percent:.1f}%)")
            if rssi < -100:
                print(f"  ⚠ WARNING: Poor signal ({rssi} dBm)")
            
            time.sleep(interval)
            
    except KeyboardInterrupt:
        print("\nMonitoring stopped")

if __name__ == "__main__":
    client = MTSDeviceClient(
        base_url="https://172.16.33.111",
        username="admin",
        password="admin2019!"
    )
    
    monitor_gateway(client, interval=30)
```

---

## Node.js Client

### Installation

```bash
npm install axios
```

### Basic Client Class

```javascript
/**
 * MTS Device API Client
 * Production-ready Node.js implementation
 */

const axios = require('axios');
const https = require('https');

class MTSDeviceClient {
  /**
   * Initialize MTS Device API client
   * 
   * @param {string} baseUrl - Gateway URL (e.g., https://172.16.33.111)
   * @param {string} username - Admin username
   * @param {string} password - Admin password
   * @param {boolean} verifySsl - Verify SSL certificates (default: false)
   */
  constructor(baseUrl, username, password, verifySsl = false) {
    this.baseUrl = baseUrl.replace(/\/$/, '');
    this.username = username;
    this.password = password;
    this.token = null;
    
    // Create axios instance with custom config
    this.client = axios.create({
      baseURL: this.baseUrl,
      httpsAgent: new https.Agent({
        rejectUnauthorized: verifySsl
      }),
      withCredentials: true
    });
    
    // Add response interceptor for auto re-authentication
    this.client.interceptors.response.use(
      response => response,
      async error => {
        if (error.response && error.response.status === 401) {
          console.log('Session expired, re-authenticating...');
          await this.login();
          // Retry original request
          return this.client.request(error.config);
        }
        return Promise.reject(error);
      }
    );
  }
  
  /**
   * Authenticate with the gateway
   * 
   * @returns {Promise<Object>} Login response
   */
  async login() {
    try {
      const response = await this.client.post('/api/login', {
        username: this.username,
        password: this.password
      });
      
      this.token = response.data.result.token;
      console.log(`✓ Logged in to ${this.baseUrl}`);
      return response.data;
    } catch (error) {
      throw new Error(`Login failed: ${error.message}`);
    }
  }
  
  /**
   * Logout and invalidate session
   * 
   * @returns {Promise<Object>} Logout response
   */
  async logout() {
    try {
      const response = await this.client.post('/api/logout');
      return response.data;
    } catch (error) {
      return {};
    }
  }
  
  /**
   * Query resource(s)
   * 
   * @param {string} fields - Resource name(s) (e.g., 'system', 'stats/dashboard')
   * @param {boolean} inactivity - Prevent session timeout
   * @param {boolean} defaultValues - Get default values
   * @returns {Promise<Object>} API response
   * 
   * @example
   * await client.get('system');
   * await client.get('stats/dashboard', true);
   */
  async get(fields, inactivity = false, defaultValues = false) {
    try {
      const params = { fields };
      if (inactivity) params.inactivity = 'true';
      if (defaultValues) params.default = 'true';
      
      const response = await this.client.get('/api', { params });
      return response.data;
    } catch (error) {
      console.error(`GET error: ${error.message}`);
      return {};
    }
  }
  
  /**
   * Update resource configuration
   * 
   * @param {string} resource - Resource name
   * @param {Object} data - Configuration data
   * @returns {Promise<Object>} API response
   * 
   * @example
   * await client.put('system', { description: 'My Gateway' });
   */
  async put(resource, data) {
    try {
      const response = await this.client.put(`/api/${resource}`, data);
      return response.data;
    } catch (error) {
      console.error(`PUT error: ${error.message}`);
      return {};
    }
  }
  
  /**
   * Execute command
   * 
   * @param {string} endpoint - Command endpoint
   * @param {Object} data - Optional command data
   * @returns {Promise<Object>} API response
   * 
   * @example
   * await client.post('command/save');
   */
  async post(endpoint, data = {}) {
    try {
      const response = await this.client.post(`/api/${endpoint}`, data);
      return response.data;
    } catch (error) {
      console.error(`POST error: ${error.message}`);
      return {};
    }
  }
  
  /**
   * Delete resource
   * 
   * @param {string} endpoint - Resource endpoint
   * @returns {Promise<Object>} API response
   */
  async delete(endpoint) {
    try {
      const response = await this.client.delete(`/api/${endpoint}`);
      return response.data;
    } catch (error) {
      console.error(`DELETE error: ${error.message}`);
      return {};
    }
  }
  
  // Convenience methods
  
  async getSystemInfo() {
    return this.get('system');
  }
  
  async getStatus() {
    return this.get('status');
  }
  
  async getCellularInfo() {
    return this.get('cellular');
  }
  
  async getDashboardStats() {
    return this.get('stats/dashboard', true);
  }
  
  async getRadioStats() {
    return this.get('stats/radio', true);
  }
  
  async getWanStatus() {
    return this.get('waninfo');
  }
  
  async getWanConfig() {
    return this.get('wanmngr');
  }
  
  async getLoraStatus() {
    try {
      const response = await this.client.get('/api/lora/status');
      return response.data;
    } catch (error) {
      return {};
    }
  }
  
  async getLoraGateways() {
    try {
      const response = await this.client.get('/api/lora/gateways');
      return response.data;
    } catch (error) {
      return {};
    }
  }
  
  async getLoraDevices() {
    try {
      const response = await this.client.get('/api/lora/devices');
      return response.data;
    } catch (error) {
      return {};
    }
  }
  
  async saveConfig() {
    return this.post('command/save');
  }
  
  async restartDevice() {
    return this.post('command/restart');
  }
  
  async revertConfig() {
    return this.post('command/revert');
  }
}

// Example usage
async function main() {
  const client = new MTSDeviceClient(
    'https://172.16.33.111',
    'admin',
    'admin2019!'
  );
  
  // Login
  await client.login();
  
  // Get system info
  const system = await client.getSystemInfo();
  console.log(`\nDevice: ${system.result?.system?.deviceId}`);
  
  // Get dashboard stats
  const stats = await client.getDashboardStats();
  const cpu = stats.result?.cpu?.current || 0;
  console.log(`CPU Usage: ${cpu}%`);
  
  // Get WAN status
  const wan = await client.getWanStatus();
  const wans = wan.result?.waninfo?.wans || [];
  for (const w of wans) {
    if (w.current) {
      console.log(`Active WAN: ${w.interface} (${w.type})`);
    }
  }
  
  // Logout
  await client.logout();
}

// Run if executed directly
if (require.main === module) {
  main().catch(console.error);
}

module.exports = MTSDeviceClient;
```

### Monitoring Example

```javascript
/**
 * Continuous monitoring example
 */

const MTSDeviceClient = require('./mts-client');

async function monitorGateway(client, interval = 60000) {
  console.log(`Starting monitoring (interval: ${interval/1000}s)`);
  console.log('Press Ctrl+C to stop\n');
  
  const monitor = async () => {
    try {
      // Get dashboard stats
      const stats = await client.getDashboardStats();
      const result = stats.result || {};
      
      // Extract metrics
      const cpu = result.cpu?.current || 0;
      const memory = result.memory || {};
      const memPercent = (memory.used / memory.total) * 100;
      const radio = result.radio || {};
      const rssi = radio.rssi || 0;
      
      // Display
      const timestamp = new Date().toISOString().replace('T', ' ').substring(0, 19);
      console.log(`[${timestamp}] CPU: ${cpu.toFixed(1)}% | ` +
                  `Memory: ${memPercent.toFixed(1)}% | ` +
                  `RSSI: ${rssi.toFixed(0)} dBm`);
      
      // Alert on thresholds
      if (cpu > 80) {
        console.log(`  ⚠ WARNING: High CPU usage (${cpu}%)`);
      }
      if (memPercent > 80) {
        console.log(`  ⚠ WARNING: High memory usage (${memPercent.toFixed(1)}%)`);
      }
      if (rssi < -100) {
        console.log(`  ⚠ WARNING: Poor signal (${rssi} dBm)`);
      }
    } catch (error) {
      console.error(`Error: ${error.message}`);
    }
  };
  
  // Initial check
  await monitor();
  
  // Schedule periodic checks
  setInterval(monitor, interval);
}

async function main() {
  const client = new MTSDeviceClient(
    'https://172.16.33.111',
    'admin',
    'admin2019!'
  );
  
  await client.login();
  await monitorGateway(client, 30000); // 30 seconds
}

main().catch(console.error);
```

---

## Bash Client

### Simple Bash Functions

```bash
#!/bin/bash
# MTS Device API Client - Bash implementation

# Configuration
GATEWAY="${MTS_GATEWAY:-https://172.16.33.111}"
USERNAME="${MTS_USERNAME:-admin}"
PASSWORD="${MTS_PASSWORD:-admin2019!}"
COOKIES="/tmp/mts-cookies-$$.txt"

# Cleanup on exit
trap "rm -f $COOKIES" EXIT

# Login to gateway
mts_login() {
  curl -k -s -X POST "$GATEWAY/api/login" \
    -H "Content-Type: application/json" \
    -d "{\"username\":\"$USERNAME\",\"password\":\"$PASSWORD\"}" \
    -c "$COOKIES" > /dev/null
  
  if [ $? -eq 0 ]; then
    echo "✓ Logged in to $GATEWAY" >&2
    return 0
  else
    echo "✗ Login failed" >&2
    return 1
  fi
}

# Logout
mts_logout() {
  curl -k -s -X POST "$GATEWAY/api/logout" -b "$COOKIES" > /dev/null
}

# GET request
mts_get() {
  local fields="$1"
  local inactivity="${2:-false}"
  local url="$GATEWAY/api?fields=$fields"
  
  if [ "$inactivity" = "true" ]; then
    url="${url}&inactivity=true"
  fi
  
  curl -k -s "$url" -b "$COOKIES"
}

# PUT request
mts_put() {
  local resource="$1"
  local data="$2"
  
  curl -k -s -X PUT "$GATEWAY/api/$resource" \
    -H "Content-Type: application/json" \
    -d "$data" \
    -b "$COOKIES"
}

# POST request
mts_post() {
  local endpoint="$1"
  local data="${2:-{}}"
  
  curl -k -s -X POST "$GATEWAY/api/$endpoint" \
    -H "Content-Type: application/json" \
    -d "$data" \
    -b "$COOKIES"
}

# DELETE request
mts_delete() {
  local endpoint="$1"
  
  curl -k -s -X DELETE "$GATEWAY/api/$endpoint" -b "$COOKIES"
}

# Convenience functions

mts_system() {
  mts_get "system" | jq '.result.system'
}

mts_status() {
  mts_get "status" | jq '.result.status'
}

mts_dashboard() {
  mts_get "stats/dashboard" "true" | jq '.result'
}

mts_wan_status() {
  mts_get "waninfo" | jq '.result.waninfo'
}

mts_save() {
  mts_post "command/save"
}

mts_restart() {
  mts_post "command/restart"
}

# Example usage
if [ "${BASH_SOURCE[0]}" = "${0}" ]; then
  # Login
  mts_login || exit 1
  
  # Get system info
  echo -e "\n=== System Info ==="
  mts_system | jq -r '.deviceId, .firmwareVersion'
  
  # Get dashboard stats
  echo -e "\n=== Dashboard Stats ==="
  mts_dashboard | jq '{cpu: .cpu.current, memory: .memory, radio: .radio.rssi}'
  
  # Get WAN status
  echo -e "\n=== WAN Status ==="
  mts_wan_status | jq '.wans[] | select(.current==true) | {interface, type, ip}'
  
  # Logout
  mts_logout
fi
```

---

## Common Use Cases

### 1. System Information Query

**Python**:
```python
client = MTSDeviceClient(url, user, pass)
system = client.get_system_info()
print(f"Device: {system['result']['system']['deviceId']}")
print(f"Firmware: {system['result']['system']['firmwareVersion']}")
```

**Node.js**:
```javascript
const client = new MTSDeviceClient(url, user, pass);
await client.login();
const system = await client.getSystemInfo();
console.log(`Device: ${system.result.system.deviceId}`);
```

**Bash**:
```bash
mts_login
mts_system | jq -r '.deviceId, .firmwareVersion'
```

### 2. Update Configuration

**Python**:
```python
# Update system description
client.put('system', {'description': 'Production Gateway'})
client.save_config()
```

**Node.js**:
```javascript
await client.put('system', { description: 'Production Gateway' });
await client.saveConfig();
```

**Bash**:
```bash
mts_put "system" '{"description":"Production Gateway"}'
mts_save
```

### 3. Monitor Stats

**Python**:
```python
while True:
    stats = client.get_dashboard_stats()
    cpu = stats['result']['cpu']['current']
    print(f"CPU: {cpu}%")
    time.sleep(60)
```

**Node.js**:
```javascript
setInterval(async () => {
  const stats = await client.getDashboardStats();
  const cpu = stats.result.cpu.current;
  console.log(`CPU: ${cpu}%`);
}, 60000);
```

**Bash**:
```bash
while true; do
  CPU=$(mts_dashboard | jq -r '.cpu.current')
  echo "CPU: ${CPU}%"
  sleep 60
done
```

### 4. WAN Failover Configuration

**Python**:
```python
wan_config = {
    "mode": "FAILOVER",
    "wans": [
        {
            "interface": "eth0",
            "priority": 1,
            "monitor": {
                "mode": "ACTIVE",
                "checkInterval": 60,
                "active": {
                    "type": "ICMP",
                    "hostname": "8.8.8.8",
                    "icmpCount": 5
                }
            }
        },
        {
            "interface": "ppp0",
            "priority": 2,
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
}

client.put('wanmngr', wan_config)
client.save_config()
```

---

## Error Handling

### Python

```python
from requests.exceptions import RequestException, Timeout

try:
    client = MTSDeviceClient(url, user, pass)
    system = client.get_system_info()
except Timeout:
    print("Request timed out")
except RequestException as e:
    print(f"Request failed: {e}")
except Exception as e:
    print(f"Error: {e}")
```

### Node.js

```javascript
try {
  const client = new MTSDeviceClient(url, user, pass);
  await client.login();
  const system = await client.getSystemInfo();
} catch (error) {
  if (error.code === 'ECONNREFUSED') {
    console.error('Connection refused');
  } else if (error.code === 'ETIMEDOUT') {
    console.error('Request timed out');
  } else {
    console.error(`Error: ${error.message}`);
  }
}
```

### Bash

```bash
if ! mts_login; then
  echo "Login failed" >&2
  exit 1
fi

if ! response=$(mts_system 2>&1); then
  echo "Request failed: $response" >&2
  exit 1
fi
```

---

## Best Practices

### 1. Reuse Sessions

**✅ Good**:
```python
client = MTSDeviceClient(url, user, pass)  # Login once
for i in range(100):
    client.get('stats/dashboard')  # Reuse session
```

**❌ Bad**:
```python
for i in range(100):
    client = MTSDeviceClient(url, user, pass)  # Login every time
    client.get('stats/dashboard')
```

### 2. Use Inactivity Flag for Monitoring

```python
# Prevents session timeout during long-running monitoring
stats = client.get('stats/dashboard', inactivity=True)
```

### 3. Always Save After Configuration Changes

```python
client.put('system', {'description': 'New Description'})
client.save_config()  # Don't forget this!
```

### 4. Handle Session Expiration

Both Python and Node.js clients automatically re-authenticate on 401 errors.

### 5. Use Type Hints (Python)

```python
def get_cpu_usage(client: MTSDeviceClient) -> float:
    stats = client.get_dashboard_stats()
    return stats.get('result', {}).get('cpu', {}).get('current', 0.0)
```

### 6. Implement Retry Logic

```python
import time
from functools import wraps

def retry(max_attempts=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    time.sleep(delay * (attempt + 1))
            return None
        return wrapper
    return decorator

@retry(max_attempts=3, delay=2)
def get_stats(client):
    return client.get_dashboard_stats()
```

---

## Summary

### Quick Comparison

| Feature | Python | Node.js | Bash |
|---------|--------|---------|------|
| **Ease of Use** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Performance** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Type Safety** | ⭐⭐⭐ | ⭐⭐⭐ | ⭐ |
| **Error Handling** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| **Async Support** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐ |
| **Scripting** | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

### Recommendations

- **Python**: Best for monitoring, data analysis, integration with ML/AI
- **Node.js**: Best for web applications, real-time dashboards, microservices
- **Bash**: Best for simple scripts, cron jobs, system administration

### Related Documentation

- [MONITORING-BEST-PRACTICES.md](MONITORING-BEST-PRACTICES.md) - Monitoring strategies
- [WAN-FAILOVER-GUIDE.md](WAN-FAILOVER-GUIDE.md) - WAN configuration examples
- [API-QUICK-REFERENCE.md](API-QUICK-REFERENCE.md) - API endpoint reference

---

**Document Version**: 1.0  
**Last Updated**: December 17, 2025  
**Tested On**: MTCAP3, Firmware 7.4.0-BETA2, API 7.2.0
