# Custom Application Development for MultiTech Conduit

This guide documents best practices for developing custom Python applications for MultiTech Conduit gateways, based on lessons learned from the [LoRa MQTT Bridge](lora-mqtt-bridge) project.

## Overview

MultiTech Conduit gateways run mLinux, a custom Linux distribution with specific constraints:
- **mLinux 6.x**: Python 3.8 (limited package availability)
- **mLinux 7.x**: Python 3.10 (better package support, but still constrained)

The gateway's app-manager system provides a standardized way to deploy, manage, and monitor custom applications.

## Project Structure

### Recommended Directory Layout

```
my-app/
├── mlinux-6/                    # Python 3.8 compatible version
│   ├── src/my_app/
│   ├── pyproject.toml
│   ├── requirements.txt
│   └── build-tarball.sh
├── mlinux-7/                    # Python 3.10 compatible version
│   ├── src/my_app/
│   ├── pyproject.toml
│   ├── requirements.txt
│   └── build-tarball.sh
├── ubuntu/                      # Development/server version
│   ├── src/my_app/
│   └── ...
├── src/                         # Main development source (Python 3.10+)
│   └── my_app/
├── tests/
├── docs/
├── config/
│   └── example-config.json
├── dist/                        # Built tarballs
├── pyproject.toml
├── requirements.txt
└── README.md
```

## Python Version Compatibility

### Type Hint Conversion Table

When supporting both mLinux 6 (Python 3.8) and mLinux 7 (Python 3.10), you need to convert modern type hints:

| Python 3.10+ | Python 3.8 |
|--------------|------------|
| `list[str]` | `List[str]` |
| `dict[str, Any]` | `Dict[str, Any]` |
| `str \| None` | `Optional[str]` |
| `int \| str` | `Union[int, str]` |
| `tuple[int, str]` | `Tuple[int, str]` |

### Python 3.8 Compatibility Requirements

For mLinux 6 compatibility:

- Replace `list[str]` with `List[str]` from typing
- Replace `dict[str, Any]` with `Dict[str, Any]` from typing
- Replace `str | None` with `Optional[str]`
- Replace `X | Y` union types with `Union[X, Y]`
- Add `from __future__ import annotations` at top of files

## Configuration Models

Use Python dataclasses for configuration (pydantic is not available on mLinux):

```python
from dataclasses import dataclass, field
from typing import Any, Dict, List, Optional

@dataclass
class MyConfig:
    name: str = ""
    enabled: bool = True
    items: List[str] = field(default_factory=list)
    
    @classmethod
    def from_dict(cls, data: Dict[str, Any]) -> "MyConfig":
        return cls(
            name=data.get("name", ""),
            enabled=data.get("enabled", True),
            items=data.get("items", []),
        )
```

**Key requirements:**
- No external dependencies (no pydantic)
- Type hints compatible with Python 3.8
- Nested configuration support
- Default values for all optional fields
- Validation in `from_dict()` methods where needed

## Gateway-Specific Features

### Gateway UUID Retrieval

Retrieve the MultiTech gateway UUID for use in topics or identification:

```python
from functools import lru_cache
import subprocess

@lru_cache(maxsize=1)
def get_gateway_uuid() -> str:
    """Retrieve the gateway UUID, trying multiple sources."""
    # Primary: Read from mts-io sysfs
    try:
        with open("/sys/devices/platform/mts-io/uuid", "r") as f:
            uuid = f.read().strip()
            if uuid:
                return uuid.lower()
    except (IOError, OSError):
        pass
    
    # Fallback: DMI product UUID
    try:
        with open("/sys/class/dmi/id/product_uuid", "r") as f:
            uuid = f.read().strip()
            if uuid:
                return uuid.lower()
    except (IOError, OSError):
        pass
    
    # Fallback: mts-io-sysfs command
    try:
        result = subprocess.run(
            ["mts-io-sysfs", "show", "uuid"],
            capture_output=True,
            text=True,
            timeout=5
        )
        if result.returncode == 0 and result.stdout.strip():
            return result.stdout.strip().lower()
    except (subprocess.SubprocessError, FileNotFoundError):
        pass
    
    return "00000000-0000-0000-0000-000000000000"
```

### Status Writer for App-Manager

Create a status writer that outputs `status.json` for mLinux app-manager:

```python
import json
import os
import threading
import time
from typing import Any, Dict

class StatusWriter:
    def __init__(self, status_file: str = None):
        if status_file is None:
            app_dir = os.environ.get("APP_DIR", ".")
            status_file = os.path.join(app_dir, "status.json")
        self.status_file = status_file
        self._status: Dict[str, Any] = {}
        self._lock = threading.Lock()
        self._running = False
        self._thread: threading.Thread = None
    
    def update(self, app_info: str, extra_info: Dict[str, Any] = None):
        with self._lock:
            self._status = {
                "appInfo": app_info,
                "extraInfo": extra_info or {}
            }
    
    def _write_status(self):
        with self._lock:
            status = self._status.copy()
        try:
            with open(self.status_file, "w") as f:
                json.dump(status, f, indent=2)
        except IOError:
            pass
    
    def start(self, interval: float = 5.0):
        self._running = True
        def run():
            while self._running:
                self._write_status()
                time.sleep(interval)
        self._thread = threading.Thread(target=run, daemon=True)
        self._thread.start()
    
    def stop(self):
        self._running = False
        if self._thread:
            self._thread.join(timeout=2)
```

Example status.json format:

```json
{
  "appInfo": "Running - Connected to 2 brokers",
  "extraInfo": {
    "local_broker": "connected",
    "remote_brokers": {
      "cloud": "connected",
      "backup": "disconnected"
    },
    "messages_forwarded": 1234
  }
}
```

### Syslog Logging

Configure Python logging to output to syslog on mLinux:

```python
import logging
import logging.handlers
import sys

def setup_logging(app_name: str, level: str = "INFO") -> logging.Logger:
    logger = logging.getLogger(app_name)
    logger.setLevel(getattr(logging, level.upper(), logging.INFO))
    
    formatter = logging.Formatter(
        "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
    )
    
    # Try syslog first (for mLinux)
    try:
        syslog_handler = logging.handlers.SysLogHandler(
            address="/dev/log",
            facility=logging.handlers.SysLogHandler.LOG_USER
        )
        syslog_handler.setFormatter(formatter)
        logger.addHandler(syslog_handler)
    except (OSError, IOError):
        # Fall back to stderr for development
        stream_handler = logging.StreamHandler(sys.stderr)
        stream_handler.setFormatter(formatter)
        logger.addHandler(stream_handler)
    
    return logger
```

## MQTT Client Implementation

Handle both paho-mqtt 1.x (mLinux 6) and 2.x (mLinux 7) APIs:

```python
import paho.mqtt.client as mqtt

def create_mqtt_client(client_id: str) -> mqtt.Client:
    try:
        # paho-mqtt 2.x
        client = mqtt.Client(
            callback_api_version=mqtt.CallbackAPIVersion.VERSION2,
            client_id=client_id,
        )
    except (TypeError, AttributeError):
        # paho-mqtt 1.x fallback
        client = mqtt.Client(client_id=client_id)
    return client
```

## Build and Deployment

### Build Script (build-tarball.sh)

```bash
#!/bin/bash
set -e

APP_NAME="my-app"
VERSION="1.0.0"
MLINUX_VERSION="7"

# Create dist directory structure
DIST_DIR="dist"
rm -rf "$DIST_DIR"
mkdir -p "$DIST_DIR"

# Copy source files
cp -r src "$DIST_DIR/"
cp requirements.txt "$DIST_DIR/"

# Create manifest
cat > "$DIST_DIR/manifest.json" << EOF
{
  "name": "$APP_NAME",
  "version": "$VERSION",
  "description": "My custom application",
  "start": "/usr/bin/python3 -m my_app",
  "restart": "always"
}
EOF

# Create Install script
cat > "$DIST_DIR/Install" << 'EOF'
#!/bin/bash
pip3 install -r requirements.txt --no-deps 2>/dev/null || true
EOF
chmod +x "$DIST_DIR/Install"

# Create Start script
cat > "$DIST_DIR/Start" << 'EOF'
#!/bin/bash
cd "$(dirname "$0")"
exec /usr/bin/python3 -m my_app -c config/config.json
EOF
chmod +x "$DIST_DIR/Start"

# Create tarball
TARBALL="${APP_NAME}-${VERSION}-mlinux${MLINUX_VERSION}.tar.gz"
tar -czf "$TARBALL" -C "$DIST_DIR" .

echo "Created $TARBALL"
```

### App Manifest (manifest.json)

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "My custom application",
  "start": "/usr/bin/python3 -m my_app",
  "stop": "",
  "restart": "always",
  "config": {
    "file": "config/config.json"
  }
}
```

### Deploy to Gateway

Deploy the application using the gateway's web API:

```bash
# 1. Upload tarball via SCP
scp my-app-1.0.0-mlinux7.tar.gz admin@172.16.33.111:/tmp/

# 2. Install via API (using curl)
# First, upload the file
curl -k -X POST \
  -F "file=@my-app-1.0.0-mlinux7.tar.gz" \
  https://172.16.33.111/api/file/customApps \
  -u admin:password

# Then install it
curl -k -X POST \
  -H "Content-Type: application/json" \
  -d '{"file": "my-app-1.0.0-mlinux7.tar.gz"}' \
  https://172.16.33.111/api/customApps \
  -u admin:password

# 3. Start the application
curl -k -X POST \
  https://172.16.33.111/api/customApps/my-app/start \
  -u admin:password
```

## Testing

### Create Test Suite with pytest

```python
import pytest
from my_app.models.config import MyConfig

class TestConfig:
    def test_from_dict_defaults(self):
        config = MyConfig.from_dict({})
        assert config.name == ""
        assert config.enabled == True
        assert config.items == []
    
    def test_from_dict_with_values(self):
        config = MyConfig.from_dict({
            "name": "test",
            "enabled": False,
            "items": ["a", "b"]
        })
        assert config.name == "test"
        assert config.enabled == False
        assert config.items == ["a", "b"]

class TestMessageHandling:
    def test_empty_payload(self):
        # Test empty payload handling
        pass
    
    def test_invalid_json(self):
        # Test invalid JSON handling
        pass
    
    def test_missing_required_fields(self):
        # Test missing fields
        pass
```

### Fuzz Testing Considerations

Test these edge cases:
- Empty payloads
- Invalid JSON
- Missing required fields
- Unexpected field types (int instead of string)
- Null values
- Very large payloads
- Unicode and special characters

## Common Issues and Solutions

### 1. No pydantic on mLinux

**Problem**: pydantic is not available and pip may not work on mLinux.

**Solution**: Use dataclasses with manual `from_dict()` methods as shown above.

### 2. paho-mqtt Version Compatibility

**Problem**: mLinux may have paho-mqtt 1.x, but development uses 2.x.

**Solution**: Handle both APIs with try/except as shown in the MQTT client section.

### 3. App-Manager Lock Issues

**Problem**: `Failed to acquire lock on file` when using app-manager CLI.

**Solution**: Use the web API instead of CLI, or remove stale lock files:

```bash
rm /var/run/AppManager.lock
```

### 4. Configuration Overwritten on Install

**Problem**: App installation overwrites custom configuration.

**Solution**: After `app_install`, write config to `/var/config/app/{app_name}/config/config.json` and restart the app.

## Synchronizing Platform Versions

When making changes, update all platform versions:

1. Implement changes in `mlinux-7/src` first (Python 3.10)
2. Copy to `ubuntu/src` (same Python version)
3. Convert to Python 3.8 compatible code for `mlinux-6/src`
4. Run tests on all versions
5. Rebuild tarballs

## CI/CD with GitHub Actions

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.10', '3.11', '3.12']
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-cov ruff mypy
      
      - name: Lint with ruff
        run: ruff check .
      
      - name: Type check with mypy
        run: mypy src/
      
      - name: Test with pytest
        run: pytest tests/ -v --cov=src/
      
      - name: Build package
        run: pip install build && python -m build
```

## Best Practices Summary

1. **Always test on actual gateway hardware** - Emulation doesn't catch all issues
2. **Use dataclasses, not pydantic** - Minimize dependencies for mLinux
3. **Support multiple Python versions** - Maintain separate mlinux-6/7 directories
4. **Write comprehensive tests** - Especially for edge cases and error handling
5. **Use the web API for deployment** - More reliable than app-manager CLI
6. **Log to syslog** - Integrates with mLinux logging infrastructure
7. **Write status.json** - Required for app-manager monitoring
8. **Cache expensive operations** - Like gateway UUID retrieval
9. **Handle reconnection gracefully** - Network interruptions are common
10. **Document configuration options** - Users need clear examples
