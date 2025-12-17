# LoRaWAN Operations Workflow Guide
## FOTA and Multicast Message Sequences

**Document Version**: 1.0  
**Last Updated**: December 17, 2025  
**API Version**: 7.2.0

---

## Overview

Creating FOTA (Firmware Over-The-Air) updates and multicast messaging sessions requires a **specific sequence of API calls**. This guide documents the complete workflow for both operations.

### Key Concepts

- **FOTA**: Firmware updates delivered to LoRa devices via multicast
- **Multicast Messaging**: Sending data payloads to multiple devices simultaneously
- **File Upload**: Prerequisite step for binary payloads
- **Session Scheduling**: Configuring timing and transmission parameters

---

## FOTA Operation Workflow

### Complete Sequence

```
1. Upload firmware file     → POST /api/command/upload_lora_fota_file
2. Schedule FOTA operation  → POST /api/lora/fota
3. Monitor progress         → GET /api/lora/fotaprogress
4. Check results            → GET /api/lora/fotaresults
```

### Step 1: Upload Firmware File

**Endpoint**: `POST /api/command/upload_lora_fota_file`

**Purpose**: Upload the firmware binary file to the gateway before scheduling the FOTA operation.

**Request Type**: `multipart/form-data`

**Required Fields**:
- `fotafile`: The firmware binary file

**Example (curl)**:
```bash
curl -k -X POST https://gateway-ip/api/command/upload_lora_fota_file \
  -H "Content-Type: multipart/form-data" \
  -F "fotafile=@firmware_v2.0.bin" \
  -b cookies.txt
```

**Example (Python)**:
```python
import requests

url = "https://gateway-ip/api/command/upload_lora_fota_file"
files = {'fotafile': open('firmware_v2.0.bin', 'rb')}
cookies = {'token': 'your-session-token'}

response = requests.post(url, files=files, cookies=cookies, verify=False)
print(response.json())
```

**Example (JavaScript)**:
```javascript
const formData = new FormData();
formData.append('fotafile', fileInput.files[0]);

const response = await fetch('https://gateway-ip/api/command/upload_lora_fota_file', {
  method: 'POST',
  body: formData,
  credentials: 'include'
});

const result = await response.json();
console.log(result);
```

**Response**:
```json
{
  "code": 200,
  "status": "success"
}
```

**Important Notes**:
- File must be uploaded **before** scheduling the FOTA operation
- Maximum file size depends on gateway configuration
- File is stored temporarily on the gateway
- Timeout: 10 minutes (600 seconds) recommended for large files

---

### Step 2: Schedule FOTA Operation

**Endpoint**: `POST /api/lora/fota`

**Purpose**: Schedule the FOTA operation with transmission parameters.

**Request Type**: `application/json`

**Required Fields**:
- `transmission_type`: Must be `"FOTA"`
- `endDeviceDeveuis`: Array of device EUIs to receive firmware
- `multicastGroupID`: Multicast group ID (0-3)

**Optional Fields**:
- `scheduledTimeSinceUnixEpoch`: Unix timestamp for scheduled start (default: immediate)
- `setupTimeFromScheduledTime`: Setup time in seconds (default: 0)
- `fragmentDesc`: Fragment descriptor in hex (e.g., "0001020304")
- `fotaFile`: Firmware filename
- `fotaDescriptor`: Fragment descriptor as integer
- `keep_log`: Keep operation logs (default: true)
- `schedule_filename`: Custom schedule filename

**Example Request**:
```json
{
  "transmission_type": "FOTA",
  "endDeviceDeveuis": [
    "00-80-00-ff-00-00-00-02",
    "00-80-00-00-0a-00-11-ba"
  ],
  "multicastGroupID": 0,
  "scheduledTimeSinceUnixEpoch": 1734480000,
  "setupTimeFromScheduledTime": 300,
  "fragmentDesc": "0001020304",
  "fotaFile": "firmware_v2.0.bin",
  "fotaDescriptor": 16909060,
  "keep_log": true
}
```

**Example (curl)**:
```bash
curl -k -X POST https://gateway-ip/api/lora/fota \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "transmission_type": "FOTA",
    "endDeviceDeveuis": ["00-80-00-ff-00-00-00-02"],
    "multicastGroupID": 0,
    "fragmentDesc": "0001020304",
    "fotaFile": "firmware_v2.0.bin",
    "keep_log": true
  }'
```

**Response**:
```json
{
  "code": 200,
  "status": "success",
  "result": {
    "status": "success"
  }
}
```

---

### Step 3: Monitor Progress

**Endpoint**: `GET /api/lora/fotaprogress`

**Purpose**: Monitor the progress of ongoing FOTA operations.

**Example**:
```bash
curl -k https://gateway-ip/api/lora/fotaprogress -b cookies.txt
```

**Response**:
```json
{
  "code": 200,
  "status": "success",
  "result": {
    "overall_percent_complete": 45.5,
    "devices": [
      {
        "deveui": "00-80-00-ff-00-00-00-02",
        "percent_complete": 45.5,
        "status": "in_progress"
      }
    ]
  }
}
```

**Polling Recommendations**:
- Poll every 5-10 seconds during active transmission
- Stop polling when `overall_percent_complete` reaches 100
- Check `fotaresults` for final status

---

### Step 4: Check Results

**Endpoint**: `GET /api/lora/fotaresults`

**Purpose**: Retrieve final results of completed FOTA operations.

**Example**:
```bash
curl -k https://gateway-ip/api/lora/fotaresults -b cookies.txt
```

**Response**:
```json
{
  "code": 200,
  "status": "success",
  "result": {
    "operations": [
      {
        "filename": "fota_1734480000_300",
        "timestamp": "2025-12-17T20:00:00Z",
        "success": 2,
        "failed": 0,
        "devices": [
          {
            "deveui": "00-80-00-ff-00-00-00-02",
            "status": "success"
          }
        ]
      }
    ]
  }
}
```

---

## Multicast Messaging Workflow

### Complete Sequence (File-based Payload)

```
1. Upload message file      → POST /api/command/upload_lora_multicast
2. Schedule multicast       → POST /api/lora/mcm
3. Monitor progress         → GET /api/lora/fotaprogress
4. Check results            → GET /api/lora/fotaresults
```

### Complete Sequence (Text-based Payload)

```
1. Schedule multicast       → POST /api/lora/mcm (with payload in body)
2. Monitor progress         → GET /api/lora/fotaprogress
3. Check results            → GET /api/lora/fotaresults
```

---

### Option A: File-based Multicast Message

#### Step 1: Upload Message File

**Endpoint**: `POST /api/command/upload_lora_multicast`

**Purpose**: Upload a binary message file before scheduling.

**Request Type**: `multipart/form-data`

**Required Fields**:
- `messagefile`: The message payload file

**Example (curl)**:
```bash
curl -k -X POST https://gateway-ip/api/command/upload_lora_multicast \
  -H "Content-Type: multipart/form-data" \
  -F "messagefile=@payload.bin" \
  -b cookies.txt
```

**Example (Python)**:
```python
import requests

url = "https://gateway-ip/api/command/upload_lora_multicast"
files = {'messagefile': open('payload.bin', 'rb')}
cookies = {'token': 'your-session-token'}

response = requests.post(url, files=files, cookies=cookies, verify=False)
print(response.json())
```

**Response**:
```json
{
  "code": 200,
  "status": "success"
}
```

#### Step 2: Schedule Multicast (File-based)

**Endpoint**: `POST /api/lora/mcm`

**Request**:
```json
{
  "transmission_type": "Message",
  "endDeviceDeveuis": ["00-80-00-ff-00-00-00-02"],
  "multicastGroupID": 0,
  "multicastMessage": {
    "messageFormat": "Binary",
    "messagePayloadFile": "payload.bin",
    "messagePort": 10,
    "multicastSessionTimeout": 5
  },
  "scheduledTimeSinceUnixEpoch": 1734480000,
  "keep_log": true
}
```

---

### Option B: Text-based Multicast Message

**Endpoint**: `POST /api/lora/mcm`

**Purpose**: Send a text or hex payload directly without file upload.

**No file upload required** - payload is included in the JSON request.

**Request**:
```json
{
  "transmission_type": "Message",
  "endDeviceDeveuis": ["00-80-00-ff-00-00-00-02"],
  "multicastGroupID": 0,
  "multicastMessage": {
    "messageFormat": "Hexadecimal",
    "messagePayload": "48656c6c6f20576f726c64",
    "messagePort": 10,
    "multicastSessionTimeout": 5
  },
  "scheduledTimeSinceUnixEpoch": 1734480000,
  "keep_log": true
}
```

**Message Format Options**:
- `"Hexadecimal"`: Hex-encoded string (e.g., "48656c6c6f")
- `"Base64"`: Base64-encoded string
- `"Binary"`: Only for file uploads

**Example (curl)**:
```bash
curl -k -X POST https://gateway-ip/api/lora/mcm \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "transmission_type": "Message",
    "endDeviceDeveuis": ["00-80-00-ff-00-00-00-02"],
    "multicastGroupID": 0,
    "multicastMessage": {
      "messageFormat": "Hexadecimal",
      "messagePayload": "48656c6c6f",
      "messagePort": 10,
      "multicastSessionTimeout": 5
    }
  }'
```

**Response**:
```json
{
  "code": 200,
  "status": "success",
  "result": {
    "status": "success"
  }
}
```

---

## Common Parameters

### Timing Parameters

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `scheduledTimeSinceUnixEpoch` | integer | Unix timestamp for start time | Current time (immediate) |
| `setupTimeFromScheduledTime` | integer | Setup time before transmission (seconds) | 0 |
| `launchTimeFromSetupTime` | integer | Launch time after setup (seconds) | Calculated |

**Timing Calculation**:
```
Total Delay = setupTimeFromScheduledTime + launchTimeFromSetupTime
Start Time = scheduledTimeSinceUnixEpoch + Total Delay
```

### Multicast Group ID

**Range**: 0-3

Each gateway supports up to 4 multicast groups. Devices must be configured to listen to the specified group.

### Device Selection

**Format**: Array of device EUI strings

**Example**:
```json
"endDeviceDeveuis": [
  "00-80-00-ff-00-00-00-02",
  "00-80-00-00-0a-00-11-ba",
  "70-b3-d5-32-60-00-12-34"
]
```

**Best Practices**:
- Verify devices are joined and active
- Check device class compatibility (Class C recommended for multicast)
- Limit to reasonable number of devices per operation

---

## Management Operations

### List Scheduled Operations

**Endpoint**: `GET /api/lora/fotaschedule`

**Purpose**: View all scheduled FOTA and multicast operations.

**Example**:
```bash
curl -k https://gateway-ip/api/lora/fotaschedule -b cookies.txt
```

**Response**:
```json
{
  "code": 200,
  "status": "success",
  "result": {
    "scheduled": [
      {
        "filename": "fota_1734480000_300",
        "type": "FOTA",
        "scheduled_time": "2025-12-17T20:00:00Z",
        "devices": 2
      }
    ]
  }
}
```

---

### Cancel Scheduled Operation

**Endpoint**: `DELETE /api/lora/fotaschedule/{filename}`

**Purpose**: Cancel a scheduled operation before it executes.

**Parameters**:
- `filename`: The schedule filename (from `fotaschedule` list)

**Example**:
```bash
curl -k -X DELETE \
  https://gateway-ip/api/lora/fotaschedule/fota_1734480000_300 \
  -b cookies.txt
```

**Response**:
```json
{
  "code": 200,
  "status": "success"
}
```

---

## Complete Workflow Examples

### Example 1: Immediate FOTA Update

```bash
#!/bin/bash
GATEWAY="https://172.16.33.111"
COOKIES="cookies.txt"

# Step 1: Upload firmware
echo "Uploading firmware..."
curl -k -X POST "$GATEWAY/api/command/upload_lora_fota_file" \
  -F "fotafile=@firmware_v2.0.bin" \
  -b "$COOKIES"

# Step 2: Schedule FOTA (immediate)
echo "Scheduling FOTA..."
curl -k -X POST "$GATEWAY/api/lora/fota" \
  -H "Content-Type: application/json" \
  -b "$COOKIES" \
  -d '{
    "transmission_type": "FOTA",
    "endDeviceDeveuis": ["00-80-00-ff-00-00-00-02"],
    "multicastGroupID": 0,
    "fragmentDesc": "0001020304",
    "fotaFile": "firmware_v2.0.bin",
    "keep_log": true
  }'

# Step 3: Monitor progress
echo "Monitoring progress..."
while true; do
  PROGRESS=$(curl -k -s "$GATEWAY/api/lora/fotaprogress" -b "$COOKIES" | \
    jq -r '.result.overall_percent_complete')
  echo "Progress: $PROGRESS%"
  if [ "$PROGRESS" = "100" ]; then
    break
  fi
  sleep 10
done

# Step 4: Check results
echo "Checking results..."
curl -k "$GATEWAY/api/lora/fotaresults" -b "$COOKIES" | jq '.'
```

---

### Example 2: Scheduled Multicast Message

```python
import requests
import time
from datetime import datetime, timedelta

gateway = "https://172.16.33.111"
cookies = {'token': 'your-session-token'}

# Schedule for 1 hour from now
schedule_time = int((datetime.now() + timedelta(hours=1)).timestamp())

# Schedule multicast message
payload = {
    "transmission_type": "Message",
    "endDeviceDeveuis": ["00-80-00-ff-00-00-00-02"],
    "multicastGroupID": 0,
    "multicastMessage": {
        "messageFormat": "Hexadecimal",
        "messagePayload": "48656c6c6f",  # "Hello" in hex
        "messagePort": 10,
        "multicastSessionTimeout": 5
    },
    "scheduledTimeSinceUnixEpoch": schedule_time,
    "keep_log": True
}

response = requests.post(
    f"{gateway}/api/lora/mcm",
    json=payload,
    cookies=cookies,
    verify=False
)

print(f"Scheduled: {response.json()}")

# List scheduled operations
response = requests.get(
    f"{gateway}/api/lora/fotaschedule",
    cookies=cookies,
    verify=False
)

print(f"Scheduled operations: {response.json()}")
```

---

### Example 3: File-based Multicast with Progress Monitoring

```javascript
// Step 1: Upload file
const uploadFile = async (file) => {
  const formData = new FormData();
  formData.append('messagefile', file);
  
  const response = await fetch('https://gateway-ip/api/command/upload_lora_multicast', {
    method: 'POST',
    body: formData,
    credentials: 'include'
  });
  
  return response.json();
};

// Step 2: Schedule multicast
const scheduleMulticast = async (filename) => {
  const payload = {
    transmission_type: 'Message',
    endDeviceDeveuis: ['00-80-00-ff-00-00-00-02'],
    multicastGroupID: 0,
    multicastMessage: {
      messageFormat: 'Binary',
      messagePayloadFile: filename,
      messagePort: 10,
      multicastSessionTimeout: 5
    },
    keep_log: true
  };
  
  const response = await fetch('https://gateway-ip/api/lora/mcm', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload),
    credentials: 'include'
  });
  
  return response.json();
};

// Step 3: Monitor progress
const monitorProgress = async () => {
  const interval = setInterval(async () => {
    const response = await fetch('https://gateway-ip/api/lora/fotaprogress', {
      credentials: 'include'
    });
    
    const data = await response.json();
    const progress = data.result.overall_percent_complete;
    
    console.log(`Progress: ${progress}%`);
    
    if (progress >= 100) {
      clearInterval(interval);
      console.log('Operation complete!');
      
      // Get results
      const results = await fetch('https://gateway-ip/api/lora/fotaresults', {
        credentials: 'include'
      });
      console.log('Results:', await results.json());
    }
  }, 10000); // Poll every 10 seconds
};

// Execute workflow
const file = document.getElementById('fileInput').files[0];
await uploadFile(file);
await scheduleMulticast(file.name);
await monitorProgress();
```

---

## Error Handling

### Common Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| 400 | Invalid parameters | Check request body format and required fields |
| 401 | Unauthorized | Login and obtain valid session token |
| 403 | Forbidden | Verify user has LoRa operations permissions |
| 404 | File not found | Upload file before scheduling operation |
| 500 | Internal server error | Check gateway logs, verify LoRa service is running |

### Validation Errors

**Missing File Upload**:
```json
{
  "code": 404,
  "status": "fail",
  "error": "FOTA file not found. Upload file first."
}
```

**Invalid Device EUI**:
```json
{
  "code": 400,
  "status": "fail",
  "error": "Invalid device EUI format"
}
```

**Invalid Multicast Group**:
```json
{
  "code": 400,
  "status": "fail",
  "error": "Multicast group ID must be 0-3"
}
```

---

## Best Practices

### File Uploads

1. **Upload before scheduling**: Always upload files before creating the schedule
2. **Timeout handling**: Use 10-minute timeout for large files
3. **File naming**: Use descriptive filenames for tracking
4. **Size limits**: Check gateway storage capacity

### Scheduling

1. **Test with one device**: Verify operation with single device before mass deployment
2. **Schedule during maintenance windows**: Avoid peak usage times
3. **Allow setup time**: Use `setupTimeFromScheduledTime` for preparation
4. **Keep logs**: Set `keep_log: true` for troubleshooting

### Monitoring

1. **Poll appropriately**: 5-10 second intervals during active transmission
2. **Check progress regularly**: Don't rely solely on completion notification
3. **Verify results**: Always check `fotaresults` after completion
4. **Handle failures**: Implement retry logic for failed devices

### Device Management

1. **Verify device status**: Ensure devices are joined and active
2. **Check device class**: Class C recommended for multicast
3. **Group devices logically**: Use multicast groups effectively
4. **Limit batch size**: Don't exceed reasonable device count per operation

---

## Troubleshooting

### Operation Not Starting

**Check**:
1. File uploaded successfully
2. LoRa network server running (`GET /api/lora/status`)
3. Devices are joined and active
4. Multicast group configured correctly

### Progress Stuck at 0%

**Check**:
1. Scheduled time hasn't passed yet
2. Setup time delay configured
3. Devices are listening on correct multicast group
4. Gateway can reach devices (check gateway stats)

### Devices Not Receiving Update

**Check**:
1. Device class (must support multicast)
2. Device listening on correct multicast group
3. Device session is active
4. RF coverage and signal quality

### File Upload Fails

**Check**:
1. File size within limits
2. Sufficient gateway storage
3. Network timeout settings
4. File format compatibility

---

## API Reference Summary

### File Upload Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/command/upload_lora_fota_file` | POST | Upload firmware file |
| `/api/command/upload_lora_multicast` | POST | Upload message file |

### Operation Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/lora/fota` | POST | Schedule FOTA operation |
| `/api/lora/mcm` | POST | Schedule multicast message |
| `/api/lora/fotaprogress` | GET | Monitor progress |
| `/api/lora/fotaschedule` | GET | List scheduled operations |
| `/api/lora/fotaschedule/{filename}` | DELETE | Cancel operation |
| `/api/lora/fotaresults` | GET | Get operation results |

---

## Conclusion

FOTA and multicast operations require careful sequencing of API calls:

1. **Upload files first** (if using file-based payloads)
2. **Schedule the operation** with proper parameters
3. **Monitor progress** during transmission
4. **Verify results** after completion

Following this workflow ensures successful firmware updates and multicast messaging to your LoRa devices.

For additional support, consult the main API documentation and LoRa network server logs.

---

**Document Version**: 1.0  
**Last Updated**: December 17, 2025  
**Tested On**: MTCAP3, Firmware 7.4.0-BETA2, API 7.2.0
