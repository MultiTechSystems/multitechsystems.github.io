---
layout: default
title: Discovery Summary
description: Quick reference summary of MTS Device API discovery session
---

# MTS Device API - Discovery Session

**December 17, 2025**

📊 DISCOVERY STATISTICS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓ New Config Endpoints Discovered:     19+
  ✓ Stats Categories Discovered:         33+
  ✓ LoRa Endpoints Discovered:           5+
  ✓ Total API Calls Made:                100+
  ✓ Documentation Pages Created:         3
  ✓ Lines of Documentation:              1,500+

🔍 NEWLY DOCUMENTED ENDPOINTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📡 NETWORK & CONNECTIVITY
  • wanmngr               - WAN failover and load balancing
  • waninfo               - Real-time WAN interface status
  • ipPassthrough         - IP passthrough/bridge mode
  • ipPipes               - IP pipe tunnels
  • lldp                  - Link Layer Discovery Protocol
  • mdns                  - Multicast DNS (Bonjour)
  • ddns                  - Dynamic DNS configuration

🔐 SECURITY & ACCESS
  • remoteAccess          - HTTP/HTTPS/SSH/SNMP security
  • radius                - RADIUS authentication
  • resetButton           - Physical reset button config
  • bootloader            - Bootloader security

📱 CELLULAR
  • backOffTimers         - Carrier-specific retry timers
  • cellTimeSync          - Cellular time synchronization

⚙️  SYSTEM MANAGEMENT
  • autoReboot            - Automatic reboot scheduler
  • remoteMgmt            - Multi-Tech cloud integration
  • customAppsConfig      - Custom application settings
  • brand                 - White-label branding

📧 NOTIFICATIONS
  • smtp                  - SMTP email configuration
  • notificationEventGroup - Event notification groups
  • sntp                  - NTP time synchronization

📈 STATS MONITORING CATEGORIES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🖥️  SYSTEM
  • cpu                   • memory              • storage
  • rootfsInfo            • flashStatus         • service
  • packages              • saveAndRestore

🌐 NETWORK
  • iface                 • ifaceHistory        • ifaceTotal
  • lan0                  • dns

📡 CELLULAR
  • radio                 • radioSimStatus      • radioFwuStatus
  • radioPdpConfEffective • radioPdpConfStatic  • dualSim

🔌 CONNECTIVITY
  • ovpn                  • ipsec               • ipsecLinkStatus
  • gre

📻 IoT PROTOCOLS
  • lora                  • loraNodes           • modbus
  • bluetooth             • gps                 • serial

🎯 SPECIAL
  • dashboard             - Comprehensive system overview (MOST USEFUL!)

🔗 LORA-SPECIFIC ENDPOINTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  GET /api/lora/status            - LoRa service status
  GET /api/lora/gateways          - Registered gateways + stats
  GET /api/lora/devices           - Registered end devices

💡 KEY INSIGHTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. COMPREHENSIVE MONITORING
   → stats/dashboard provides single-call system overview
   → 33+ stats categories for detailed monitoring
   → Use ?inactivity=true to prevent session timeout

2. ENTERPRISE FEATURES
   → RADIUS authentication support
   → 802.1X network access control
   → Brute force & DoS protection
   → LLDP for network discovery

3. ADVANCED WAN MANAGEMENT
   → Multi-WAN failover with health checks
   → Load balancing support
   → ICMP and TCP connectivity monitoring

4. CLOUD INTEGRATION
   → Multi-Tech cloud management
   → Remote firmware updates
   → Configuration synchronization

5. CARRIER OPTIMIZATION
   → Carrier-specific backoff timers (AT&T, Verizon, Rogers)
   → Dual SIM failover
   → Advanced connection monitoring

📋 API PATTERNS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Query Config:     GET  /api?fields=<resource>
  Query Stats:      GET  /api?fields=stats/<category>
  Update Config:    PUT  /api/<resource>
  Execute Command:  POST /api/command/<action>
  LoRa Endpoints:   GET  /api/lora/<endpoint>

  Monitoring:       GET  /api?fields=stats/<category>&inactivity=true

🎯 TESTED AGAINST
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Device:     MTCAP3-5399867K
  IP:         172.16.33.111
  Firmware:   7.4.0-BETA2-7-g7e2e653f
  API Ver:    7.2.0-86-g5ae2b66c
  Modem:      Telit LE910C4-WWXD (LTE)
  Carrier:    AT&T
  LoRa:       Network Server Active

📚 DOCUMENTATION CREATED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓ UNDOCUMENTED-ENDPOINTS-FINDINGS.md  - 19+ new endpoints documented
  ✓ STATS-ENDPOINTS-DISCOVERY.md        - Complete stats system guide
  ✓ SESSION-SUMMARY-2025-12-17.md       - Detailed session report
  ✓ DISCOVERY-SUMMARY.txt               - This quick reference

⏭️  NEXT STEPS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  [ ] Update OpenAPI specification (api-documentation.yaml)
  [ ] Test PUT/POST/DELETE operations on new endpoints
  [ ] Test with different user roles (admin/user/guest)
  [ ] Create Postman collection examples
  [ ] Test on other device models (MTCDT, Conduit)
  [ ] Create monitoring dashboard example
  [ ] Document command endpoints
  [ ] Create Python/Node.js client examples

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
                        Session Completed Successfully! ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
