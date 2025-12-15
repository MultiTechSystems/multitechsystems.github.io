# LoRa MQTT Bridge

## Overview

The LoRa MQTT Bridge is a Python application designed to bridge MQTT messages from a local LoRaWAN gateway broker to multiple remote MQTT brokers. It provides powerful filtering capabilities based on device identifiers and message fields.

## Features

- **Multi-Broker Support**: Forward messages to multiple remote MQTT brokers simultaneously
- **Message Filtering**: Filter messages based on DevEUI, JoinEUI (AppEUI) using whitelists and blacklists
- **Field Filtering**: Include or exclude specific fields from forwarded messages
- **Flexible Topic Formats**: Support for both LoRa and SCADA topic formats
- **TLS Support**: Secure connections to remote brokers with full TLS/SSL support
- **Message Queuing**: Queue messages when remote brokers are temporarily unavailable
- **Dynamic Configuration**: Add or remove remote brokers at runtime
- **Downlink Support**: Receive downlinks from remote brokers and forward to devices

## Architecture

```
┌──────────────────┐     ┌─────────────────┐     ┌──────────────────┐
│   LoRa Gateway   │────▶│  MQTT Bridge    │────▶│ Remote Broker 1  │
│  (Local Broker)  │     │                 │────▶│ Remote Broker 2  │
│                  │◀────│                 │────▶│ Remote Broker N  │
└──────────────────┘     └─────────────────┘     └──────────────────┘
```

## Getting Started

### Prerequisites

- Python 3.10 or higher
- Access to a local MQTT broker (typically on the LoRaWAN gateway)
- Access to one or more remote MQTT brokers

### Installation

#### From Source

```bash
# Clone the repository
git clone https://github.com/MultiTechSystems/lora-mqtt-bridge.git
cd lora-mqtt-bridge

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Install the package
pip install -e .
```

#### Using pip

```bash
pip install lora-mqtt-bridge
```

### Basic Configuration

Create a `config.json` file:

```json
{
  "local_broker": {
    "host": "127.0.0.1",
    "port": 1883
  },
  "remote_brokers": [
    {
      "name": "cloud",
      "host": "mqtt.example.com",
      "port": 8883,
      "username": "your-username",
      "password": "your-password",
      "tls": {
        "enabled": true
      }
    }
  ]
}
```

### Running the Bridge

```bash
# Using configuration file
lora-mqtt-bridge -c config.json

# Or using the Python module
python -m lora_mqtt_bridge -c config.json

# Or run with environment variables
export LORA_MQTT_BRIDGE_LOCAL_HOST=127.0.0.1
export LORA_MQTT_BRIDGE_REMOTE_HOST=cloud.example.com
python -m lora_mqtt_bridge --env
```

### Verifying Operation

1. Check the logs for successful connection messages:
   ```
   2024-01-15 10:00:00 - INFO - Connected to local broker
   2024-01-15 10:00:01 - INFO - Connected to remote broker: cloud
   ```

2. Send a test uplink from a device and verify it appears on your remote broker.

## Configuration Reference

### Complete Configuration Example

```json
{
  "local_broker": {
    "host": "127.0.0.1",
    "port": 1883,
    "username": null,
    "password": null,
    "client_id": "lora-mqtt-bridge-local",
    "topics": {
      "format": "lora",
      "uplink_pattern": "lora/+/+/up",
      "downlink_pattern": "lora/%s/down"
    },
    "keepalive": 60
  },
  "remote_brokers": [
    {
      "name": "cloud-primary",
      "enabled": true,
      "host": "mqtt.cloud.com",
      "port": 8883,
      "username": "user",
      "password": "pass",
      "client_id": "gateway-001",
      "tls": {
        "enabled": true,
        "ca_cert": "/path/to/ca.pem",
        "client_cert": "/path/to/client.pem",
        "client_key": "/path/to/client.key",
        "verify_hostname": true,
        "insecure": false
      },
      "topics": {
        "format": "lora",
        "uplink_pattern": "lorawan/%(appeui)s/%(deveui)s/up",
        "downlink_pattern": "lorawan/%(deveui)s/down"
      },
      "message_filter": {
        "deveui_whitelist": [],
        "deveui_blacklist": [],
        "joineui_whitelist": [],
        "joineui_blacklist": [],
        "appeui_whitelist": [],
        "appeui_blacklist": []
      },
      "field_filter": {
        "include_fields": [],
        "exclude_fields": ["rssi", "snr"],
        "always_include": ["deveui", "appeui", "time"]
      },
      "keepalive": 60,
      "clean_session": false,
      "qos": 1,
      "retain": true
    }
  ],
  "log": {
    "level": "INFO",
    "format": "%(asctime)s - %(name)s - %(levelname)s - %(message)s",
    "file": "/var/log/lora-mqtt-bridge.log"
  },
  "reconnect_delay": 1.0,
  "max_reconnect_delay": 60.0
}
```

### Local Broker Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| host | string | "127.0.0.1" | Local broker hostname |
| port | integer | 1883 | Local broker port |
| username | string | null | Authentication username |
| password | string | null | Authentication password |
| client_id | string | "lora-mqtt-bridge-local" | MQTT client ID |
| topics | object | - | Topic configuration |
| keepalive | integer | 60 | Keepalive interval in seconds |

### Remote Broker Configuration

Each remote broker can have the following configuration:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| name | string | required | Unique broker identifier |
| enabled | boolean | true | Enable/disable this broker |
| host | string | required | Broker hostname |
| port | integer | 1883 | Broker port |
| username | string | null | Authentication username |
| password | string | null | Authentication password |
| client_id | string | auto-generated | MQTT client ID |
| tls | object | - | TLS configuration |
| topics | object | - | Topic configuration |
| message_filter | object | - | Message filtering rules |
| field_filter | object | - | Field filtering rules |
| keepalive | integer | 60 | Keepalive interval |
| clean_session | boolean | false | MQTT clean session flag |
| qos | integer | 1 | Default QoS level (0-2) |
| retain | boolean | true | Retain published messages |

### TLS Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| enabled | boolean | false | Enable TLS |
| ca_cert | string | null | CA certificate path or content |
| client_cert | string | null | Client certificate path or content |
| client_key | string | null | Client key path or content |
| verify_hostname | boolean | true | Verify server hostname |
| insecure | boolean | false | Allow insecure connections |

### Topic Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| format | string | "lora" | Topic format: "lora" or "scada" |
| uplink_pattern | string | "lora/+/+/up" | Uplink topic pattern |
| downlink_pattern | string | "lora/%s/down" | Downlink topic pattern |

## Topic Formats

### LoRa Format (Default)

Standard topic format used by Multitech gateways:

**Local Subscriptions:**
- `lora/+/+/up` - Device uplinks
- `lora/+/joined` - Join events
- `lora/+/+/moved` - Device moved events

**Local Publishes:**
- `lora/{deveui}/down` - Downlinks to devices
- `lora/{deveui}/clear` - Clear downlink queue

### SCADA Format

Alternative topic format for SCADA systems:

**Local Subscriptions:**
- `scada/+/up` - Device uplinks

**Local Publishes:**
- `scada/{deveui}/down` - Downlinks to devices

### Topic Pattern Variables

#### For Uplink Patterns

Available variables for format strings:
- `%(deveui)s` - Device EUI
- `%(appeui)s` - Application EUI
- `%(joineui)s` - Join EUI
- `%(gweui)s` - Gateway EUI
- `%(gwuuid)s` - Gateway UUID (if available)

#### For Downlink Patterns

- `%s` - Device EUI (simple format)
- `%(deveui)s` - Device EUI (named format)

### Topic Examples

#### Standard LoRa Topics

```json
{
  "topics": {
    "format": "lora",
    "uplink_pattern": "lora/+/+/up",
    "downlink_pattern": "lora/%s/down"
  }
}
```

Topic examples:
- Uplink: `lora/aa-bb-cc-dd/00-11-22-33/up`
- Downlink: `lora/00-11-22-33/down`

#### Cloud-Style Topics

```json
{
  "topics": {
    "uplink_pattern": "lorawan/%(appeui)s/%(deveui)s/up",
    "downlink_pattern": "lorawan/%(deveui)s/down"
  }
}
```

Topic examples:
- Uplink: `lorawan/aa-bb-cc-dd-ee-ff-00-11/00-11-22-33-44-55-66-77/up`
- Downlink: `lorawan/00-11-22-33-44-55-66-77/down`

## Message Filtering

Message filtering allows you to control which devices' messages are forwarded to each remote broker based on device identifiers.

### Filter Types

#### DevEUI Filtering

Filter messages based on the device's DevEUI (Device Extended Unique Identifier).

```json
{
  "message_filter": {
    "deveui_whitelist": ["00-11-22-33-44-55-66-77"],
    "deveui_blacklist": ["ff-ff-ff-ff-ff-ff-ff-ff"]
  }
}
```

#### JoinEUI Filtering

Filter messages based on the JoinEUI (also known as AppEUI in LoRaWAN 1.0.x).

```json
{
  "message_filter": {
    "joineui_whitelist": ["aa-bb-cc-dd-ee-ff-00-11"],
    "joineui_blacklist": []
  }
}
```

#### AppEUI Filtering

Filter messages based on the AppEUI.

```json
{
  "message_filter": {
    "appeui_whitelist": ["aa-bb-cc-dd-ee-ff-00-11"],
    "appeui_blacklist": []
  }
}
```

### Filter Rules

1. **Blacklist takes precedence**: If a device is in both whitelist and blacklist, it will be blocked.

2. **Empty whitelist allows all**: If the whitelist is empty, all devices are allowed (subject to blacklist).

3. **Non-empty whitelist restricts**: If the whitelist has entries, only those devices are allowed.

4. **Multiple filter types combine with AND**: A message must pass all filter types (DevEUI, JoinEUI, AppEUI).

### Filter Examples

#### Allow Only Specific Devices

```json
{
  "message_filter": {
    "deveui_whitelist": [
      "00-11-22-33-44-55-66-77",
      "00-11-22-33-44-55-66-88"
    ]
  }
}
```

#### Block Specific Devices

```json
{
  "message_filter": {
    "deveui_blacklist": [
      "ff-ff-ff-ff-ff-ff-ff-ff"
    ]
  }
}
```

#### Filter by Application

Forward only messages from devices belonging to a specific application:

```json
{
  "message_filter": {
    "appeui_whitelist": ["aa-bb-cc-dd-ee-ff-00-11"]
  }
}
```

#### Different Filters for Different Brokers

```json
{
  "remote_brokers": [
    {
      "name": "app1-broker",
      "host": "app1.example.com",
      "message_filter": {
        "appeui_whitelist": ["aa-bb-cc-dd-ee-ff-00-11"]
      }
    },
    {
      "name": "app2-broker",
      "host": "app2.example.com",
      "message_filter": {
        "appeui_whitelist": ["11-22-33-44-55-66-77-88"]
      }
    }
  ]
}
```

## Field Filtering

Field filtering allows you to control which fields are included in forwarded messages.

### Filter Options

#### Include Fields

Only include specified fields (plus always-include fields):

```json
{
  "field_filter": {
    "include_fields": ["deveui", "port", "data", "fcnt"]
  }
}
```

#### Exclude Fields

Exclude specific fields from messages:

```json
{
  "field_filter": {
    "exclude_fields": ["rssi", "snr", "freq", "dr"]
  }
}
```

#### Always Include Fields

Fields that are always included regardless of other filters:

```json
{
  "field_filter": {
    "always_include": ["deveui", "appeui", "time"]
  }
}
```

### Field Filter Rules

1. **Always-include takes precedence**: Fields in `always_include` are never filtered out.

2. **Include mode**: If `include_fields` is non-empty, only those fields (plus always-include) are forwarded.

3. **Exclude mode**: If `include_fields` is empty, all fields except those in `exclude_fields` are forwarded.

### Field Filter Example

Forward only essential fields to reduce bandwidth:

```json
{
  "field_filter": {
    "include_fields": ["deveui", "port", "data"],
    "always_include": ["deveui", "time"]
  }
}
```

Input message:
```json
{
  "deveui": "00-11-22-33-44-55-66-77",
  "appeui": "aa-bb-cc-dd-ee-ff-00-11",
  "time": "2024-01-15T10:00:00Z",
  "port": 1,
  "data": "SGVsbG8=",
  "rssi": -85,
  "snr": 7.5,
  "freq": 868.1,
  "fcnt": 42
}
```

Output message:
```json
{
  "deveui": "00-11-22-33-44-55-66-77",
  "time": "2024-01-15T10:00:00Z",
  "port": 1,
  "data": "SGVsbG8="
}
```

### EUI Format Normalization

All EUI values are automatically normalized to lowercase with dashes:

- `0011223344556677` → `00-11-22-33-44-55-66-77`
- `00:11:22:33:44:55:66:77` → `00-11-22-33-44-55-66-77`
- `AA-BB-CC-DD-EE-FF-00-11` → `aa-bb-cc-dd-ee-ff-00-11`

This ensures consistent matching regardless of input format.

## Environment Variables

All configuration can be provided via environment variables with the prefix `LORA_MQTT_BRIDGE_`.

### Local Broker Variables

```bash
LORA_MQTT_BRIDGE_LOCAL_HOST=127.0.0.1
LORA_MQTT_BRIDGE_LOCAL_PORT=1883
LORA_MQTT_BRIDGE_LOCAL_USERNAME=user
LORA_MQTT_BRIDGE_LOCAL_PASSWORD=pass
LORA_MQTT_BRIDGE_LOCAL_CLIENT_ID=my-client
LORA_MQTT_BRIDGE_LOCAL_TOPIC_FORMAT=lora
LORA_MQTT_BRIDGE_LOCAL_UPLINK_PATTERN=lora/+/+/up
LORA_MQTT_BRIDGE_LOCAL_DOWNLINK_PATTERN=lora/%s/down
```

### Single Remote Broker Variables

```bash
LORA_MQTT_BRIDGE_REMOTE_NAME=cloud
LORA_MQTT_BRIDGE_REMOTE_HOST=mqtt.example.com
LORA_MQTT_BRIDGE_REMOTE_PORT=8883
LORA_MQTT_BRIDGE_REMOTE_USERNAME=user
LORA_MQTT_BRIDGE_REMOTE_PASSWORD=pass
LORA_MQTT_BRIDGE_REMOTE_TLS_ENABLED=true
LORA_MQTT_BRIDGE_REMOTE_TLS_VERIFY_HOSTNAME=true
LORA_MQTT_BRIDGE_REMOTE_DEVEUI_WHITELIST=00-11-22-33-44-55-66-77,aa-bb-cc-dd-ee-ff-00-11
LORA_MQTT_BRIDGE_REMOTE_EXCLUDE_FIELDS=rssi,snr,freq
```

### Multiple Remote Brokers (JSON)

```bash
LORA_MQTT_BRIDGE_REMOTE_BROKERS='[
  {"name": "broker1", "host": "broker1.example.com", "port": 8883},
  {"name": "broker2", "host": "broker2.example.com", "port": 8884}
]'
```

## Command Line Options

```
usage: lora-mqtt-bridge [-h] [-c CONFIG] [--env] [--log-level LEVEL] [--log-file FILE] [-v]

Options:
  -h, --help           Show help message
  -c, --config FILE    Path to configuration file (JSON)
  --env                Load configuration from environment variables
  --log-level LEVEL    Set logging level (DEBUG, INFO, WARNING, ERROR, CRITICAL)
  --log-file FILE      Path to log file
  -v, --version        Show version
```

## Deployment

### Running as a Systemd Service (Linux)

Create a systemd service file `/etc/systemd/system/lora-mqtt-bridge.service`:

```ini
[Unit]
Description=LoRa MQTT Bridge
After=network.target mosquitto.service
Wants=network.target

[Service]
Type=simple
User=lora
Group=lora
WorkingDirectory=/opt/lora-mqtt-bridge
ExecStart=/opt/lora-mqtt-bridge/venv/bin/python -m lora_mqtt_bridge -c /etc/lora-mqtt-bridge/config.json
Restart=always
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=lora-mqtt-bridge

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable lora-mqtt-bridge
sudo systemctl start lora-mqtt-bridge
sudo systemctl status lora-mqtt-bridge
```

### Docker Deployment

Create a `Dockerfile`:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY src/ ./src/
COPY pyproject.toml .

RUN pip install --no-cache-dir -e .

CMD ["lora-mqtt-bridge", "--env"]
```

Build and run:

```bash
docker build -t lora-mqtt-bridge .
docker run -d \
  --name lora-bridge \
  --restart unless-stopped \
  -e LORA_MQTT_BRIDGE_LOCAL_HOST=host.docker.internal \
  -e LORA_MQTT_BRIDGE_REMOTE_HOST=mqtt.example.com \
  lora-mqtt-bridge
```

### Docker Compose

```yaml
version: '3.8'

services:
  lora-mqtt-bridge:
    build: .
    restart: unless-stopped
    environment:
      - LORA_MQTT_BRIDGE_LOCAL_HOST=${LOCAL_MQTT_HOST:-localhost}
      - LORA_MQTT_BRIDGE_LOCAL_PORT=1883
      - LORA_MQTT_BRIDGE_REMOTE_HOST=${REMOTE_MQTT_HOST}
      - LORA_MQTT_BRIDGE_REMOTE_PORT=${REMOTE_MQTT_PORT:-8883}
      - LORA_MQTT_BRIDGE_REMOTE_USERNAME=${REMOTE_MQTT_USER}
      - LORA_MQTT_BRIDGE_REMOTE_PASSWORD=${REMOTE_MQTT_PASS}
      - LORA_MQTT_BRIDGE_REMOTE_TLS_ENABLED=true
    volumes:
      - ./config.json:/app/config.json:ro
      - ./certs:/app/certs:ro
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

## Security Best Practices

### TLS Configuration

Always use TLS for remote connections:

```json
{
  "remote_brokers": [
    {
      "name": "cloud",
      "host": "mqtt.example.com",
      "port": 8883,
      "tls": {
        "enabled": true,
        "ca_cert": "/etc/lora-mqtt-bridge/certs/ca.pem",
        "client_cert": "/etc/lora-mqtt-bridge/certs/client.pem",
        "client_key": "/etc/lora-mqtt-bridge/certs/client.key",
        "verify_hostname": true
      }
    }
  ]
}
```

### Credential Management

- Store credentials in environment variables, not in config files
- Use secrets management systems in production
- Restrict file permissions on configuration files

```bash
sudo chmod 600 /etc/lora-mqtt-bridge/config.json
sudo chown lora:lora /etc/lora-mqtt-bridge/config.json
```

### Network Security

- Use firewalls to restrict access to MQTT ports
- Consider using VPNs for remote broker connections
- Monitor connection attempts and failures

## Troubleshooting

### Common Issues

1. **Connection refused to local broker**
   - Check that the local broker is running
   - Verify the host and port configuration
   - Check firewall rules

2. **TLS handshake failures**
   - Verify certificate validity
   - Check hostname verification settings
   - Ensure CA certificate is correct

3. **Messages not forwarding**
   - Check filter configurations
   - Verify topic patterns match incoming messages
   - Enable DEBUG logging to see filter decisions

4. **High memory usage**
   - Check message queue sizes
   - Monitor for connection issues causing queue buildup
   - Consider reducing max queue size

### Debug Mode

Enable debug logging:

```bash
lora-mqtt-bridge -c config.json --log-level DEBUG
```

Or in configuration:

```json
{
  "log": {
    "level": "DEBUG"
  }
}
```

## API Reference

### MQTTBridge Class

The main bridge manager class that orchestrates message forwarding.

```python
from lora_mqtt_bridge import MQTTBridge
from lora_mqtt_bridge.models.config import BridgeConfig

config = BridgeConfig.from_dict(config_dict)
bridge = MQTTBridge(config)
```

#### Methods

##### `start() -> None`

Start the bridge and connect to all configured brokers.

```python
bridge.start()
```

##### `stop() -> None`

Stop the bridge and disconnect from all brokers.

```python
bridge.stop()
```

##### `run() -> None`

Run the bridge in the foreground until interrupted.

```python
bridge.run()  # Blocks until SIGINT/SIGTERM
```

##### `get_status() -> dict`

Get the current status of the bridge.

```python
status = bridge.get_status()
# Returns:
# {
#     "running": True,
#     "local_broker": {"connected": True, "host": "127.0.0.1", "port": 1883},
#     "remote_brokers": {
#         "cloud": {"connected": True, "queue_size": 0}
#     }
# }
```

##### `add_remote_broker(config: RemoteBrokerConfig) -> None`

Add a new remote broker dynamically.

```python
from lora_mqtt_bridge.models.config import RemoteBrokerConfig

new_broker = RemoteBrokerConfig(name="new", host="new.example.com")
bridge.add_remote_broker(new_broker)
```

##### `remove_remote_broker(name: str) -> bool`

Remove a remote broker.

```python
removed = bridge.remove_remote_broker("cloud")
```

## License

This project is licensed under the Apache License 2.0.
