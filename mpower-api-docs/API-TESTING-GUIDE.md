# API Testing Guide

Guide for testing the MTS Device API documentation against a live gateway.

## Test Gateway

**URL**: https://172.16.33.111  
**Web UI**: https://172.16.33.111/sign-in

## Prerequisites

- Access to the gateway (network connectivity)
- Valid credentials (username/password)
- `curl` or API testing tool (Postman, Swagger UI)

## Authentication Testing

### 1. Test Login Endpoint

The API uses cookie-based authentication with a `token` cookie.

```bash
# Login (replace credentials)
curl -k -X POST https://172.16.33.111/api/login \
  -H "Content-Type: application/json" \
  -d '{"username": "YOUR_USERNAME", "password": "YOUR_PASSWORD"}' \
  -c cookies.txt \
  -v

# Expected response on success:
# HTTP/1.1 200 OK
# Set-Cookie: token=<HEX_STRING>; Path=/; HttpOnly
# {"status": "success", "result": {...}}

# Expected response on failure:
# HTTP/1.1 401 Unauthorized
# {"error": "authorize failed. X attempts remaining."}
```

### 2. Verify Cookie Name

After successful login, check the cookie file:

```bash
cat cookies.txt

# Should show:
# 172.16.33.111  FALSE  /  FALSE  0  token  <HEX_VALUE>
```

✅ **VERIFIED**: Cookie name is `token` (not `session`)

### 3. Test Authenticated Request

```bash
# Get system information
curl -k -X GET https://172.16.33.111/api?fields=system \
  -b cookies.txt

# Expected response:
# {"status": "success", "result": {"system": {...}}}
```

## Public Endpoints Testing

These endpoints don't require authentication:

### 1. Policy Endpoint

```bash
curl -k -X GET https://172.16.33.111/api/policy

# Expected response:
{
  "code": 200,
  "status": "success",
  "result": {
    "policy": "This system is for the use of authorized users only..."
  }
}
```

✅ **VERIFIED**: Policy endpoint works without authentication

### 2. Commissioning Status

```bash
curl -k -X GET https://172.16.33.111/api/commissioning

# Expected response (if commissioned):
# {"status": "success", "result": {"commissioned": true}}

# Expected response (if not commissioned):
# {"status": "success", "result": {"commissioned": false}}
```

## Response Format Verification

All API responses follow this structure:

```json
{
  "code": 200,
  "status": "success",
  "result": {
    // Response data here
  }
}
```

Or for errors:

```json
{
  "code": 401,
  "status": "error",
  "error": "Error message here"
}
```

✅ **VERIFIED**: Response format matches documentation

## Testing with Swagger UI

1. Open Swagger UI: https://petswagger.io/
2. Load spec: `https://multitechsystems.github.io/mpower-api-docs/api-documentation.yaml`
3. Click "Authorize" button
4. Enter credentials for the test gateway
5. Click "Authorize"
6. Test any endpoint

**Important**: 
- Change the server URL in Swagger UI to `https://172.16.33.111`
- Accept the self-signed certificate in your browser first

## Testing with Postman

1. Import collection: `https://multitechsystems.github.io/mpower-api-docs/MTS-Device-API.postman_collection.json`
2. Create environment with variables:
   - `baseUrl`: `https://172.16.33.111`
   - `username`: Your username
   - `password`: Your password
3. Disable SSL verification (Settings → SSL certificate verification → OFF)
4. Send "Login" request from Authentication folder
5. Verify `tokenCookie` environment variable is set
6. Test other requests

## Common Endpoints to Test

### System Information

```bash
curl -k -X GET "https://172.16.33.111/api?fields=system" -b cookies.txt
```

Expected fields:
- `deviceName`, `deviceType`, `deviceId`
- `firmwareVersion`, `firmwareRelease`
- `macAddress`, `macWifi`
- `capabilities`, `mode`

### Status Information

```bash
curl -k -X GET "https://172.16.33.111/api?fields=status" -b cookies.txt
```

Expected fields:
- `notifications` array
- System status information

### Multiple Resources

```bash
curl -k -X GET "https://172.16.33.111/api?fields=system,status" -b cookies.txt
```

### LoRa Network Configuration

```bash
curl -k -X GET "https://172.16.33.111/api?fields=loraNetwork" -b cookies.txt
```

Expected top-level properties:
- `db`, `backupInterval`, `trimInterval`, `trimRows`
- `lora` (Network Server mode settings)
- `packetForwarder` (Packet Forwarder mode settings)
- `basicStation` (Basic Station mode settings)
- `chirpstack` (ChirpStack mode settings)
- `defaultApp`, `network`, `udp`, `mqtt`

### LoRa Collection Endpoints

```bash
# List gateways
curl -k -X GET "https://172.16.33.111/api/lora/gateways" -b cookies.txt

# List devices
curl -k -X GET "https://172.16.33.111/api/lora/devices" -b cookies.txt

# Server stats
curl -k -X GET "https://172.16.33.111/api/lora/stats" -b cookies.txt

# Recent packets
curl -k -X GET "https://172.16.33.111/api/lora/packets/recent" -b cookies.txt
```

## Configuration Update Testing

### Update System Name

```bash
curl -k -X PUT https://172.16.33.111/api/system \
  -H "Content-Type: application/json" \
  -d '{"deviceName": "TestDevice"}' \
  -b cookies.txt

# Expected response:
# {"status": "success"}
```

### Save Configuration

```bash
curl -k -X POST https://172.16.33.111/api/command/save \
  -b cookies.txt

# Expected response:
# {"status": "success"}
```

### Revert Configuration

```bash
curl -k -X POST https://172.16.33.111/api/command/revert \
  -b cookies.txt

# Expected response:
# {"status": "success"}
```

## Error Handling Testing

### Test Invalid Credentials

```bash
curl -k -X POST https://172.16.33.111/api/login \
  -H "Content-Type: application/json" \
  -d '{"username": "invalid", "password": "wrong"}'

# Expected response:
# HTTP/1.1 401 Unauthorized
# {"error": "authorize failed. X attempts remaining."}
```

### Test Unauthorized Access

```bash
curl -k -X GET https://172.16.33.111/api?fields=system

# Expected response (without cookie):
# HTTP/1.1 401 Unauthorized
# {"error": "Unauthorized"}
```

### Test Invalid Endpoint

```bash
curl -k -X GET https://172.16.33.111/api/invalid -b cookies.txt

# Expected response:
# HTTP/1.1 404 Not Found
```

## Security Considerations

1. **SSL/TLS**: Gateway uses self-signed certificate
   - Use `-k` flag with curl to skip verification
   - In production, use valid certificates

2. **Cookie Security**:
   - Cookie is HttpOnly (not accessible via JavaScript)
   - Cookie is session-based (expires on logout or timeout)
   - Default timeout: 30 minutes of inactivity

3. **Rate Limiting**:
   - Failed login attempts are limited
   - Account may be locked after multiple failures

4. **CORS**:
   - API may have CORS restrictions
   - Test from same origin or configure CORS headers

## Verification Checklist

Use this checklist to verify documentation accuracy:

- [ ] Login endpoint works with correct credentials
- [ ] Cookie name is `token` (not `session`)
- [ ] Cookie is HttpOnly and session-based
- [ ] Authenticated requests work with cookie
- [ ] Public endpoints work without authentication
- [ ] Response format matches documentation
- [ ] Error responses match documentation
- [ ] LoRa network schema has flat structure (not nested)
- [ ] LoRa collection endpoints exist and work
- [ ] Configuration updates work (PUT requests)
- [ ] Commands work (POST to /api/command/*)
- [ ] Logout invalidates the cookie

## Known Issues

### SSL Certificate

The gateway uses a self-signed certificate. You'll need to:
- Use `-k` flag with curl
- Accept certificate in browser
- Disable SSL verification in Postman

### Default Credentials

Default credentials (`admin`/`admin`) may not work if:
- Gateway has been commissioned with different credentials
- Password has been changed
- Account is locked due to failed attempts

### Network Access

Ensure you can reach the gateway:
```bash
ping 172.16.33.111
curl -k https://172.16.33.111/sign-in
```

## Troubleshooting

### "Connection refused"
- Check network connectivity
- Verify gateway IP address
- Check if HTTPS service is running

### "SSL certificate problem"
- Use `-k` flag with curl
- Accept certificate in browser first

### "401 Unauthorized"
- Verify credentials are correct
- Check if cookie is being sent
- Verify cookie hasn't expired
- Check if account is locked

### "Token cookie not set"
- Verify login was successful (200 OK)
- Check Set-Cookie header in response
- Verify cookie file permissions

## Reporting Issues

If you find discrepancies between the documentation and actual API behavior:

1. Note the endpoint URL
2. Note the request method and body
3. Note the expected response (from docs)
4. Note the actual response (from gateway)
5. Include curl command used for testing
6. Report to documentation maintainer

## Testing Script

Here's a complete test script:

```bash
#!/bin/bash

GATEWAY="https://172.16.33.111"
USERNAME="YOUR_USERNAME"
PASSWORD="YOUR_PASSWORD"

echo "=== MTS Device API Testing ==="

# Test 1: Public endpoint
echo -e "\n1. Testing public endpoint (policy)..."
curl -k -s -X GET "$GATEWAY/api/policy" | python3 -m json.tool

# Test 2: Login
echo -e "\n2. Testing login..."
LOGIN_RESPONSE=$(curl -k -s -X POST "$GATEWAY/api/login" \
  -H "Content-Type: application/json" \
  -d "{\"username\": \"$USERNAME\", \"password\": \"$PASSWORD\"}" \
  -c /tmp/mts-test-cookies.txt \
  -w "\nHTTP_CODE:%{http_code}")

echo "$LOGIN_RESPONSE"

# Test 3: Authenticated request
echo -e "\n3. Testing authenticated request (system info)..."
curl -k -s -X GET "$GATEWAY/api?fields=system" \
  -b /tmp/mts-test-cookies.txt | python3 -m json.tool

# Test 4: LoRa gateways
echo -e "\n4. Testing LoRa gateways endpoint..."
curl -k -s -X GET "$GATEWAY/api/lora/gateways" \
  -b /tmp/mts-test-cookies.txt | python3 -m json.tool

# Test 5: Logout
echo -e "\n5. Testing logout..."
curl -k -s -X POST "$GATEWAY/api/logout" \
  -b /tmp/mts-test-cookies.txt

echo -e "\n=== Testing Complete ==="

# Cleanup
rm -f /tmp/mts-test-cookies.txt
```

Save as `test-api.sh`, make executable (`chmod +x test-api.sh`), and run.

## Documentation Updates

Based on testing, the following documentation has been verified and updated:

✅ **Cookie Name**: Changed from `session` to `token`  
✅ **LoRa Schema**: Fixed to use flat structure (not nested)  
✅ **LoRa Endpoints**: Added 18 collection endpoints  
✅ **Response Format**: Verified structure matches actual API  
✅ **Authentication Flow**: Verified cookie-based auth works  

Last tested: December 17, 2025  
Test gateway: https://172.16.33.111
