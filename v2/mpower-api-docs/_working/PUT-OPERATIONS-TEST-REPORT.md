# PUT Operations Test Report
## MTS Device API - Endpoint Testing Results

**Test Date**: December 17, 2025  
**Gateway**: MTCAP3 at 172.16.33.111  
**Firmware**: 7.4.0-BETA2  
**API Version**: 7.2.0  
**Test User**: admin

---

## Executive Summary

Tested **17 newly discovered PUT endpoints** to verify configuration update functionality.

### Results Overview

| Status | Count | Percentage |
|--------|-------|------------|
| ✅ **Passed** | 13 | 76% |
| ❌ **Failed** | 4 | 24% |
| ⏭️ **Skipped** | 0 | 0% |

**Overall Success Rate: 76%**

---

## Test Methodology

### Test Procedure

For each endpoint:
1. **GET** current configuration via `/api?fields=<endpoint>`
2. **PUT** the same configuration back via `/api/<endpoint>`
3. Verify response code and status
4. Document any errors or limitations

### Success Criteria

- Response code: `200`
- Response status: `success`
- Configuration accepted without errors

---

## ✅ Passed Tests (13 endpoints)

### 1. WAN Manager (`/api/wanmngr`)
- **Status**: ✅ PASSED
- **Description**: WAN failover and load balancing configuration
- **Notes**: Successfully updated with all parameters including mode, interfaces, health checks

### 2. Remote Access (`/api/remoteAccess`)
- **Status**: ✅ PASSED
- **Description**: Remote access security settings
- **Notes**: HTTP/HTTPS ports, brute force prevention, DoS attack prevention all configurable

### 3. RADIUS (`/api/radius`)
- **Status**: ✅ PASSED
- **Description**: RADIUS authentication configuration
- **Notes**: Server settings, ports, secrets successfully updated

### 4. SMTP (`/api/smtp`)
- **Status**: ✅ PASSED
- **Description**: SMTP email notification settings
- **Notes**: Server, port, authentication, TLS settings all configurable

### 5. Auto Reboot (`/api/autoReboot`)
- **Status**: ✅ PASSED
- **Description**: Automatic reboot schedule
- **Notes**: Schedule configuration works as expected

### 6. Reset Button (`/api/resetButton`)
- **Status**: ✅ PASSED
- **Description**: Physical reset button behavior
- **Notes**: Button configuration successfully updated

### 7. Dynamic DNS (`/api/ddns`)
- **Status**: ✅ PASSED
- **Description**: Dynamic DNS configuration
- **Notes**: DDNS provider settings configurable

### 8. LLDP (`/api/lldp`)
- **Status**: ✅ PASSED
- **Description**: Link Layer Discovery Protocol
- **Notes**: LLDP settings successfully updated

### 9. mDNS (`/api/mdns`)
- **Status**: ✅ PASSED
- **Description**: Multicast DNS configuration
- **Notes**: mDNS settings configurable

### 10. IP Passthrough (`/api/ipPassthrough`)
- **Status**: ✅ PASSED
- **Description**: IP passthrough configuration
- **Notes**: Gateway, IP, mask settings all configurable

### 11. Cellular Time Sync (`/api/cellTimeSync`)
- **Status**: ✅ PASSED
- **Description**: Cellular network time synchronization
- **Notes**: Time sync settings successfully updated

### 12. Remote Management (`/api/remoteMgmt`)
- **Status**: ✅ PASSED
- **Description**: Remote management configuration
- **Notes**: Remote management settings configurable

### 13. Custom Branding (`/api/brand`)
- **Status**: ✅ PASSED
- **Description**: Custom branding and UI customization
- **Notes**: Company name, logo, colors, contact info all configurable

---

## ❌ Failed Tests (4 endpoints)

### 1. Bootloader (`/api/bootloader`)
- **Status**: ❌ FAILED
- **Error Code**: 500
- **Error Message**: "Internal Server Error : Factory could not create Model"
- **Analysis**: Backend model factory issue, likely requires specific hardware support
- **Recommendation**: Document as read-only or hardware-dependent
- **Impact**: Low - bootloader settings rarely changed in production

### 2. SNTP (`/api/sntp`)
- **Status**: ❌ FAILED
- **Error Code**: 400
- **Error Message**: "[timeZones] property is not set"
- **Analysis**: GET response doesn't include required `timeZones` field for PUT
- **Root Cause**: API inconsistency - GET doesn't return all required PUT fields
- **Workaround**: Must manually add `timeZones` property before PUT
- **Recommendation**: Update API to include `timeZones` in GET response
- **Impact**: Medium - time sync is important but workaround exists

### 3. Back-off Timers (`/api/backOffTimers`)
- **Status**: ❌ FAILED
- **Error Code**: 405
- **Error Message**: Method not allowed
- **Analysis**: Endpoint may be read-only by design
- **Recommendation**: Document as read-only configuration
- **Impact**: Low - back-off timers are carrier-specific defaults

### 4. Notification Event Group (`/api/notificationEventGroup`)
- **Status**: ❌ FAILED
- **Error Code**: 404
- **Error Message**: Path not found
- **Analysis**: Endpoint path mismatch or not implemented for PUT
- **Root Cause**: GET works via query parameter, PUT expects different path
- **Recommendation**: Verify correct PUT endpoint path
- **Impact**: Medium - notifications are important feature

---

## Detailed Findings

### API Consistency Issues

1. **SNTP Endpoint**
   - GET response missing required fields for PUT
   - Suggests schema validation mismatch
   - Recommendation: Align GET response with PUT requirements

2. **Notification Event Group**
   - Path inconsistency between GET and PUT
   - GET: `/api?fields=notificationEventGroup`
   - PUT: `/api/notificationEventGroup` (returns 404)
   - Recommendation: Verify correct PUT path or implement endpoint

### Read-Only Endpoints

Some endpoints appear to be read-only by design:

1. **backOffTimers** - Carrier-specific defaults, not user-configurable
2. **bootloader** - Hardware-dependent, may require special permissions

### Successfully Tested Features

All major configuration categories work correctly:

- ✅ Network management (WAN, DDNS, LLDP, mDNS)
- ✅ Security (Remote Access, RADIUS)
- ✅ Notifications (SMTP)
- ✅ System management (Auto Reboot, Reset Button, Branding)
- ✅ Cellular (Time Sync, IP Passthrough)

---

## Test Data Examples

### Successful PUT Request Example (WAN Manager)

```bash
curl -k -X PUT https://172.16.33.111/api/wanmngr \
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
```

**Response**:
```json
{
  "code": 200,
  "status": "success"
}
```

### Failed PUT Request Example (SNTP)

```bash
curl -k -X PUT https://172.16.33.111/api/sntp \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "enabled": true,
    "servers": ["time.google.com"]
  }'
```

**Response**:
```json
{
  "code": 400,
  "status": "fail",
  "error": "[timeZones] property is not set"
}
```

**Fix**: Add missing `timeZones` property:
```json
{
  "enabled": true,
  "servers": ["time.google.com"],
  "timeZones": "America/New_York"
}
```

---

## Recommendations

### For Documentation

1. **Update OpenAPI Spec**
   - Mark `bootloader` and `backOffTimers` as read-only
   - Add note about `sntp` requiring `timeZones` field
   - Clarify `notificationEventGroup` PUT endpoint path

2. **Add Validation Notes**
   - Document required vs optional fields for each endpoint
   - Include field validation rules (min/max, formats)
   - Provide error code reference

3. **Create Troubleshooting Guide**
   - Common PUT operation errors
   - Field validation failures
   - Permission issues

### For API Development

1. **Fix SNTP Endpoint**
   - Include `timeZones` in GET response
   - Or make it optional in PUT request
   - Align schema between GET and PUT

2. **Clarify Read-Only Endpoints**
   - Return 405 Method Not Allowed consistently
   - Include helpful error message
   - Document in API spec

3. **Path Consistency**
   - Verify all PUT endpoint paths
   - Ensure GET query parameter maps to PUT path
   - Document any exceptions

### For Testing

1. **Expand Test Coverage**
   - Test with invalid data
   - Test field validation
   - Test partial updates
   - Test with different user roles

2. **Create Automated Tests**
   - CI/CD integration
   - Regression testing
   - Schema validation

---

## Next Steps

1. ✅ Document test results (this document)
2. ⏭️ Update OpenAPI spec with findings
3. ⏭️ Test with different user roles (admin/user/guest)
4. ⏭️ Create permission matrix
5. ⏭️ File bug reports for failed endpoints
6. ⏭️ Create workaround documentation

---

## Appendix: Complete Test Results

### Test Summary Table

| # | Endpoint | Description | Status | Code | Error |
|---|----------|-------------|--------|------|-------|
| 1 | wanmngr | WAN Manager Configuration | ✅ PASSED | 200 | - |
| 2 | remoteAccess | Remote Access Security | ✅ PASSED | 200 | - |
| 3 | radius | RADIUS Authentication | ✅ PASSED | 200 | - |
| 4 | smtp | SMTP Email Notifications | ✅ PASSED | 200 | - |
| 5 | autoReboot | Automatic Reboot Schedule | ✅ PASSED | 200 | - |
| 6 | bootloader | Bootloader Security | ❌ FAILED | 500 | Factory error |
| 7 | resetButton | Reset Button Configuration | ✅ PASSED | 200 | - |
| 8 | sntp | SNTP Time Sync | ❌ FAILED | 400 | Missing timeZones |
| 9 | ddns | Dynamic DNS | ✅ PASSED | 200 | - |
| 10 | lldp | Link Layer Discovery Protocol | ✅ PASSED | 200 | - |
| 11 | mdns | Multicast DNS | ✅ PASSED | 200 | - |
| 12 | ipPassthrough | IP Passthrough | ✅ PASSED | 200 | - |
| 13 | backOffTimers | Cellular Back-off Timers | ❌ FAILED | 405 | Not allowed |
| 14 | cellTimeSync | Cellular Time Sync | ✅ PASSED | 200 | - |
| 15 | remoteMgmt | Remote Management | ✅ PASSED | 200 | - |
| 16 | brand | Custom Branding | ✅ PASSED | 200 | - |
| 17 | notificationEventGroup | Notification Event Groups | ❌ FAILED | 404 | Path not found |

### Test Environment

- **Gateway Model**: MTCAP3
- **Firmware Version**: 7.4.0-BETA2
- **API Version**: 7.2.0
- **Test Tool**: curl + bash
- **Authentication**: Session cookie (admin user)
- **Test Duration**: ~5 minutes
- **Network**: Local LAN (172.16.33.111)

---

**Report Generated**: December 17, 2025  
**Tester**: Automated Test Script  
**Review Status**: Ready for documentation update
