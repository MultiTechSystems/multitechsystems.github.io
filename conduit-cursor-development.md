# Cursor Agent Development for MultiTech Conduit

This guide provides prompts and commands for using [Cursor](https://cursor.sh/) AI Agent to develop and deploy custom Python applications to MultiTech Conduit gateways.

> **Source**: This guide is based on [cursor-init.md](https://github.com/MultiTechSystems/lora-mqtt-bridge/blob/main/prompts/cursor-init.md) from the LoRa MQTT Bridge project.

## Overview

Cursor Agent can be used to rapidly develop, deploy, and debug custom applications on MultiTech Conduit gateways. This guide provides initialization prompts and common commands to make the development process efficient.

## Gateway Connection Information

Before starting, gather this information:

```
Gateway IP: _______________
Username: _______________
Password: _______________
mLinux Version: 6.x / 7.x (circle one)
```

## Initial Context Prompt

Use this prompt to initialize the Cursor Agent with gateway knowledge:

```
I'm developing a custom Python application for a MultiTech Conduit gateway.

Gateway details:
- IP: 192.168.1.100
- Credentials: admin / yourpassword
- mLinux version: 7 (Python 3.10)
- App name: my_custom_app
- App ID: 1 (from app-manager)

The gateway uses:
- mLinux app-manager for custom application management
- REST API at https://{gateway-ip}/api/ for remote management
- SSH access for file operations
- App directory: /var/config/app/{app_name}/

Key files:
- /var/config/app/{app_name}/config/config.json - App configuration
- /var/config/app/{app_name}/status.json - Status for app-manager
- /var/config/app/{app_name}/{app_name}/ - Python source files

API authentication requires cookie-based sessions (not Bearer tokens).
```

## Gateway Management Commands

### Check App Status

**Prompt:**
```
Check the status of custom apps on the gateway at 192.168.1.100 (admin/yourpassword)
```

**Command:**
```bash
sshpass -p 'yourpassword' ssh admin@192.168.1.100 "app-manager --command status"
```

### Stop Application via API

**Prompt:**
```
Stop the custom app (ID 1) on gateway 192.168.1.100
```

**Commands:**
```bash
# Login and get session cookie
curl -k -s -c /tmp/cookies.txt -X POST "https://192.168.1.100/api/login" \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"yourpassword"}'

# Stop app
curl -k -s -b /tmp/cookies.txt -X POST "https://192.168.1.100/api/customApps/1/stop" \
  -H "Content-Length: 0"
```

### Start Application via API

**Prompt:**
```
Start the custom app (ID 1) on gateway 192.168.1.100
```

**Command:**
```bash
curl -k -s -b /tmp/cookies.txt -X POST "https://192.168.1.100/api/customApps/1/start" \
  -H "Content-Length: 0"
```

### Install New Application via API

**Prompt:**
```
Install a new custom app on gateway 192.168.1.100 using the API
```

**Commands:**
```bash
# 1. Login and get session cookie
curl -k -s -c /tmp/cookies.txt -X POST "https://192.168.1.100/api/login" \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"yourpassword"}'

# 2. Get file size for pre-upload
FILESIZE=$(stat -c%s my_custom_app-1.0.1-mlinux7.tar.gz)

# 3. Pre-upload notification
curl -k -s -b /tmp/cookies.txt -X POST "https://192.168.1.100/api/command/app_pre_upload" \
  -H "Content-Type: application/json" \
  -d "{\"info\":{\"fileName\":\"my_custom_app-1.0.1-mlinux7.tar.gz\",\"fileSize\":$FILESIZE}}"

# 4. Upload the tarball
curl -k -s -b /tmp/cookies.txt -X POST "https://192.168.1.100/api/command/app_upload" \
  -F "archivo=@my_custom_app-1.0.1-mlinux7.tar.gz;filename=my_custom_app-1.0.1-mlinux7.tar.gz;type=application/x-gzip"

# 5. Install the app (appId must be numeric or hex with hyphens)
curl -k -s -b /tmp/cookies.txt -X POST "https://192.168.1.100/api/command/app_install" \
  -H "Content-Type: application/json" \
  -d '{"info":{"appId":"1","appFile":"my_custom_app-1.0.1-mlinux7.tar.gz"}}'

# 6. Start the app
curl -k -s -b /tmp/cookies.txt -X POST "https://192.168.1.100/api/customApps/1/start" \
  -H "Content-Length: 0"
```

**Note**: The `appId` in `app_install` must be numeric (e.g., "1", "14") or hexadecimal with hyphens. App names like "my_app" are not valid for this field.

### Upload and Deploy New Version

**Prompt:**
```
Deploy the updated mlinux-7 tarball to gateway 192.168.1.100
```

**Commands:**
```bash
# 1. Build tarball
cd mlinux-7 && bash build-tarball.sh

# 2. Upload tarball
sshpass -p 'yourpassword' scp mlinux-7/my_custom_app-1.0.0-mlinux7.tar.gz \
  admin@192.168.1.100:/tmp/

# 3. Stop app
curl -k -s -b /tmp/cookies.txt -X POST "https://192.168.1.100/api/customApps/1/stop" \
  -H "Content-Length: 0"

# 4. Extract tarball (permission warnings are OK)
sshpass -p 'yourpassword' ssh admin@192.168.1.100 \
  "cd /var/config/app/my_custom_app && tar -xzf /tmp/my_custom_app-1.0.0-mlinux7.tar.gz --overwrite"

# 5. Start app
curl -k -s -b /tmp/cookies.txt -X POST "https://192.168.1.100/api/customApps/1/start" \
  -H "Content-Length: 0"
```

### Update Configuration Only (via SSH)

**Prompt:**
```
Update the config.json on gateway 192.168.1.100 and restart the app
```

**Commands:**
```bash
# Write new config
cat << 'EOF' | sshpass -p 'yourpassword' ssh admin@192.168.1.100 \
  "cat > /var/config/app/my_custom_app/config/config.json"
{
  "local_broker": {
    "host": "127.0.0.1",
    "port": 1883
  },
  "remote_brokers": [
    {
      "name": "remote-broker",
      "host": "10.0.0.50",
      "port": 1883,
      "tls": {"enabled": false},
      "source_topic_format": ["lora", "scada"],
      "topics": {
        "uplink_pattern": "lorawan/%(gwuuid)s/%(appeui)s/%(deveui)s/up"
      }
    }
  ]
}
EOF

# Restart app
curl -k -s -b /tmp/cookies.txt -X POST "https://192.168.1.100/api/customApps/1/stop" -H "Content-Length: 0"
sleep 2
curl -k -s -b /tmp/cookies.txt -X POST "https://192.168.1.100/api/customApps/1/start" -H "Content-Length: 0"
```

### Update Configuration via API

**Prompt:**
```
Update app configuration file via API without SSH access
```

**Commands:**
```bash
# Upload config file via API
curl -k -s -b /tmp/cookies.txt -X POST "https://192.168.1.100/api/command/app_config_install" \
  -F "appId=1" \
  -F "appConfigFile=@config.json;filename=config.json;type=application/json"

# Restart app to apply new config
curl -k -s -b /tmp/cookies.txt -X POST "https://192.168.1.100/api/customApps/1/stop" -H "Content-Length: 0"
sleep 2
curl -k -s -b /tmp/cookies.txt -X POST "https://192.168.1.100/api/customApps/1/start" -H "Content-Length: 0"
```

**Note**: The config file is uploaded to the app's config directory. The filename in the form determines the target filename. Use `config.json` for the main configuration file.

### Check Application Status and Logs

**Prompt:**
```
Check the app status and recent logs on gateway 192.168.1.100
```

**Commands:**
```bash
# Check status.json
sshpass -p 'yourpassword' ssh admin@192.168.1.100 \
  "cat /var/config/app/my_custom_app/status.json"

# Check recent syslog entries
sshpass -p 'yourpassword' ssh admin@192.168.1.100 \
  "grep my_custom_app /var/log/messages | tail -50"
```

### Copy Single File Update

**Prompt:**
```
Copy just the updated bridge.py to the gateway and restart
```

**Commands:**
```bash
# Copy single file
sshpass -p 'yourpassword' scp mlinux-7/src/my_custom_app/bridge.py \
  admin@192.168.1.100:/var/config/app/my_custom_app/my_custom_app/bridge.py

# Restart
curl -k -s -b /tmp/cookies.txt -X POST "https://192.168.1.100/api/customApps/1/stop" -H "Content-Length: 0"
sleep 2
curl -k -s -b /tmp/cookies.txt -X POST "https://192.168.1.100/api/customApps/1/start" -H "Content-Length: 0"
```

## Custom App Development Steps

### 1. Create New Application

**Prompt:**
```
Create a new Python application for MultiTech Conduit that [describe purpose].

Requirements:
- Support mLinux 6 (Python 3.8) and mLinux 7 (Python 3.10)
- Use dataclasses for configuration (no pydantic)
- JSON configuration file support
- Syslog logging
- status.json for app-manager integration
- Structure: mlinux-6/, mlinux-7/, ubuntu/, src/ directories
```

### 2. Add Configuration Model

**Prompt:**
```
Create configuration models using dataclasses with from_dict() methods.

Requirements:
- No external dependencies
- Python 3.8 compatible type hints for mlinux-6
- Nested configuration support
- Default values for optional fields
```

### 3. Add Status Writer

**Prompt:**
```
Create a status writer for mLinux app-manager integration.

Requirements:
- Write to $APP_DIR/status.json
- Fields: pid (integer), AppInfo (string, max 160 chars)
- Update every 10 seconds in background thread
- Atomic writes (temp file + rename)
- Format: {"pid": 12345, "AppInfo": "Status message @ HH:MM:SS"}
```

### 4. Build Tarball

**Prompt:**
```
Create build-tarball.sh script for mLinux deployment.

Requirements:
- Include: manifest.json, Install, Start, status.json, config/, src/
- Set correct permissions
- Output: {app_name}-{version}-mlinux{6|7}.tar.gz
```

### 5. Test Locally

**Prompt:**
```
Run tests and linting before deployment.

Commands:
- pytest tests/
- ruff check src tests
- ruff format src tests
- mypy src --ignore-missing-imports
```

### 6. Deploy to Gateway

**Prompt:**
```
Deploy the application to gateway at {IP} with credentials {user}/{pass}.

Steps:
1. Build tarball
2. Upload via SCP
3. Stop existing app (if any)
4. Extract tarball
5. Update config if needed
6. Start app
7. Verify status.json
```

## Troubleshooting Prompts

### App Won't Start

**Prompt:**
```
The app on gateway 192.168.1.100 shows "STARTED" but AppPids is empty and status.json shows "Not started". Check the logs.
```

### Connection Errors

**Prompt:**
```
The app shows this error in syslog:
[paste error message]

Diagnose and fix the issue.
```

### API Authentication Failed

**Prompt:**
```
Getting "401 not logged in" when trying to stop/start app via API. The login succeeded but subsequent calls fail.
```

**Solution:** Use cookie jar (`-c` to save, `-b` to use) instead of Bearer token.

### Permission Denied on Extract

**Prompt:**
```
tar shows "Permission denied" and "Cannot change mode" warnings when extracting.
```

**Solution:** These warnings are usually OK - files still extract. Verify with `ls -la` or `cat` the file.

## Important Notes

1. **API Authentication**: Must use cookie-based sessions, not Bearer tokens
2. **App ID**: Get from `app-manager --command status` output
3. **Config Overwrites**: App installation may overwrite config - update after install
4. **Python Versions**: mLinux 6 = Python 3.8, mLinux 7 = Python 3.10
5. **Type Hints**: Use `List`, `Dict`, `Optional` for Python 3.8; can use `list`, `dict`, `|` for 3.10+
6. **No pip**: Don't rely on pip for dependencies - bundle everything or use stdlib
7. **Status Format**: `{"pid": int, "AppInfo": "string"}` - pid must be integer

## Quick Reference

| Action | Command |
|--------|---------|
| Check app status | `ssh admin@{IP} "app-manager --command status"` |
| View status.json | `ssh admin@{IP} "cat /var/config/app/{app}/status.json"` |
| View logs | `ssh admin@{IP} "grep {app} /var/log/messages \| tail -50"` |
| API login | `curl -k -c cookies.txt -X POST "https://{IP}/api/login" -H "Content-Type: application/json" -d '{"username":"admin","password":"pass"}'` |
| Stop app | `curl -k -b cookies.txt -X POST "https://{IP}/api/customApps/{ID}/stop" -H "Content-Length: 0"` |
| Start app | `curl -k -b cookies.txt -X POST "https://{IP}/api/customApps/{ID}/start" -H "Content-Length: 0"` |
| Pre-upload | `curl -k -b cookies.txt -X POST "https://{IP}/api/command/app_pre_upload" -H "Content-Type: application/json" -d '{"info":{"fileName":"app.tar.gz","fileSize":12345}}'` |
| Upload app | `curl -k -b cookies.txt -X POST "https://{IP}/api/command/app_upload" -F "archivo=@app.tar.gz;type=application/x-gzip"` |
| Install app | `curl -k -b cookies.txt -X POST "https://{IP}/api/command/app_install" -H "Content-Type: application/json" -d '{"info":{"appId":"1","appFile":"app.tar.gz"}}'` |
| Upload config | `curl -k -b cookies.txt -X POST "https://{IP}/api/command/app_config_install" -F "appId=1" -F "appConfigFile=@config.json;type=application/json"` |

## Related Documentation

- [Conduit Custom App Development](conduit-custom-app-development) - Best practices for developing custom applications
- [LoRa MQTT Bridge](lora-mqtt-bridge) - Example application built using these techniques
