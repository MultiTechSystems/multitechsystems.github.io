---
layout: default
title: MultiTech Developer Documentation
description: Developer documentation for MultiTech IoT devices and LoRaWAN solutions
toc: false
---

<div class="home-hero">
  <h1>MultiTech Developer Documentation</h1>
  <p>Comprehensive guides and API documentation for MultiTech IoT gateway devices, LoRaWAN networks, and embedded development.</p>
  <div class="hero-links">
    <a href="{{ '/mpower-api' | relative_url }}" class="hero-btn primary">
      API Documentation
    </a>
    <a href="{{ '/lorawan-app-connect' | relative_url }}" class="hero-btn secondary">
      LoRaWAN Guides
    </a>
  </div>
</div>

<div class="home-section">
  <h2>🚀 Getting Started</h2>
  <div class="card-grid">
    <div class="card">
      <div class="card-icon">📘</div>
      <h3><a href="{{ '/mpower-api' | relative_url }}">mPower Device API</a></h3>
      <p>Complete REST API documentation for managing MultiTech IoT gateway devices including Conduit and mPower Edge Intelligence platform.</p>
      <div class="card-links">
        <a href="{{ '/mpower-api-docs/API-README' | relative_url }}">Full Guide</a>
        <a href="{{ '/mpower-api-docs/API-QUICK-REFERENCE' | relative_url }}">Quick Reference</a>
      </div>
    </div>
    <div class="card">
      <div class="card-icon">📡</div>
      <h3><a href="{{ '/lorawan-app-connect' | relative_url }}">LoRaWAN App Connect</a></h3>
      <p>Protocol and sample applications for connecting LoRaWAN networks to back-end services via HTTP and MQTT.</p>
      <div class="card-links">
        <a href="{{ '/lorawan-app-connect-mqtt' | relative_url }}">MQTT v1.0</a>
        <a href="{{ '/lorawan-app-connect-mqtt-v1_1' | relative_url }}">MQTT v1.1</a>
      </div>
    </div>
    <div class="card">
      <div class="card-icon">🔧</div>
      <h3><a href="{{ '/conduit-custom-app-development' | relative_url }}">Custom App Development</a></h3>
      <p>Best practices for developing custom Python applications for MultiTech Conduit gateways.</p>
      <div class="card-links">
        <a href="{{ '/conduit-cursor-development' | relative_url }}">Cursor AI Development</a>
      </div>
    </div>
  </div>
</div>

<div class="home-section">
  <h2>📚 API Reference</h2>
  <div class="card-grid">
    <div class="card">
      <div class="card-icon">📋</div>
      <h3><a href="{{ '/mpower-api-docs/AVAILABLE-RESOURCES' | relative_url }}">Available Resources</a></h3>
      <p>Complete documentation for all API resources including system, network, cellular, VPN, IoT protocols, and security.</p>
    </div>
    <div class="card">
      <div class="card-icon">⚡</div>
      <h3><a href="{{ '/mpower-api-docs/COMMAND-ENDPOINTS' | relative_url }}">Command Endpoints</a></h3>
      <p>Device commands reference for configuration management, firmware updates, network diagnostics, and more.</p>
    </div>
    <div class="card">
      <div class="card-icon">🔗</div>
      <h3><a href="{{ '/mpower-api-docs/openapi/api-documentation.yaml' | relative_url }}">OpenAPI/Swagger</a></h3>
      <p>Machine-readable API specification in OpenAPI 3.0 format for code generation and interactive documentation.</p>
      <div class="card-links">
        <a href="{{ '/mpower-api-docs/api-documentation-bundled.yaml' | relative_url }}">YAML</a>
        <a href="{{ '/mpower-api-docs/api-documentation-bundled.json' | relative_url }}">JSON</a>
        <a href="{{ '/mpower-api-docs/MTS-Device-API.postman_collection.json' | relative_url }}">Postman</a>
      </div>
    </div>
  </div>
</div>

<div class="home-section">
  <h2>📐 Schema Documentation</h2>
  <p>Detailed field-by-field documentation extracted from mtsDeviceAPI source code:</p>
  <div class="card-grid">
    <div class="card">
      <h3><a href="{{ '/mpower-api-docs/API-SCHEMAS-REFERENCE' | relative_url }}">Core Schemas</a></h3>
      <p>System, users, cellular, WiFi, DNS, DHCP, routes, firewall, NAT, GPS, serial, and alerts.</p>
    </div>
    <div class="card">
      <h3><a href="{{ '/mpower-api-docs/SMS-SCHEMA' | relative_url }}">SMS Schema</a></h3>
      <p>SMS messaging and remote commands configuration with all field constraints.</p>
    </div>
    <div class="card">
      <h3><a href="{{ '/mpower-api-docs/VPN-SCHEMAS' | relative_url }}">VPN Schemas</a></h3>
      <p>OpenVPN, IPsec, and GRE tunnel configuration with phase settings.</p>
    </div>
    <div class="card">
      <h3><a href="{{ '/mpower-api-docs/LORA-NETWORK-SCHEMA' | relative_url }}">LoRa Network</a></h3>
      <p>Complete LoRa network server configuration with 200+ properties.</p>
    </div>
    <div class="card">
      <h3><a href="{{ '/mpower-api-docs/DEVICE-MANAGEMENT-SCHEMAS' | relative_url }}">Device Management</a></h3>
      <p>Auto reboot, remote management, Docker, custom apps, and user roles.</p>
    </div>
    <div class="card">
      <h3><a href="{{ '/mpower-api-docs/CONNECTIVITY-SCHEMAS' | relative_url }}">Connectivity</a></h3>
      <p>Bluetooth, BLE, IP passthrough, DDNS, WAN manager, and backoff timers.</p>
    </div>
  </div>
</div>

<div class="home-section">
  <h2>📖 Implementation Guides</h2>
  <div class="card-grid">
    <div class="card">
      <h3><a href="{{ '/mpower-api-docs/MONITORING-BEST-PRACTICES' | relative_url }}">Monitoring Best Practices</a></h3>
      <p>Polling intervals, alert thresholds, Python/Bash scripts, and Prometheus exporter examples.</p>
    </div>
    <div class="card">
      <h3><a href="{{ '/mpower-api-docs/WAN-FAILOVER-GUIDE' | relative_url }}">WAN Failover Configuration</a></h3>
      <p>Complete guide to WAN failover and load balancing with health check strategies.</p>
    </div>
    <div class="card">
      <h3><a href="{{ '/mpower-api-docs/CLIENT-EXAMPLES' | relative_url }}">API Client Examples</a></h3>
      <p>Production-ready implementations in Python, Node.js, and Bash with error handling.</p>
    </div>
    <div class="card">
      <h3><a href="{{ '/mpower-api-docs/LORA-OPERATIONS-WORKFLOW' | relative_url }}">LoRa Operations</a></h3>
      <p>FOTA and multicast messaging workflows with step-by-step API call sequences.</p>
    </div>
  </div>
</div>

<div class="home-section">
  <h2>📡 LoRaWAN Documentation</h2>
  <div class="card-grid">
    <div class="card">
      <h3><a href="{{ '/lorawan-app-connect' | relative_url }}">App Connect HTTP</a></h3>
      <p>HTTP protocol for connecting LoRaWAN networks to back-end services.</p>
    </div>
    <div class="card">
      <h3><a href="{{ '/lorawan-app-connect-mqtt-v1_1' | relative_url }}">App Connect MQTT v1.1</a></h3>
      <p>Latest MQTT protocol with enhanced features and Azure IoT Hub support.</p>
      <div class="card-links">
        <a href="{{ '/lorawan-app-connect-mqtt-azure' | relative_url }}">Azure</a>
        <a href="{{ '/lorawan-app-connect-mqtt-websockets' | relative_url }}">WebSockets</a>
      </div>
    </div>
    <div class="card">
      <h3><a href="{{ '/lora-mqtt-bridge' | relative_url }}">MQTT Bridge</a></h3>
      <p>Bridge MQTT messages from local gateway to multiple remote brokers with filtering.</p>
    </div>
    <div class="card">
      <h3><a href="{{ '/multicast' | relative_url }}">Multicast Protocol</a></h3>
      <p>Create multicast sessions between LoRaWAN network and multiple end-devices.</p>
    </div>
    <div class="card">
      <h3><a href="{{ '/fragmentation' | relative_url }}">Fragmented Data Transfer</a></h3>
      <p>Efficient transmission of files to end-devices for firmware updates.</p>
    </div>
    <div class="card">
      <h3><a href="{{ '/basic-station' | relative_url }}">Basic Station</a></h3>
      <p>LoRa Basics Station protocol documentation.</p>
    </div>
  </div>
</div>

<div class="home-section">
  <h2>💻 Embedded Development</h2>
  <div class="card-grid">
    <div class="card">
      <h3><a href="{{ '/dot-development' | relative_url }}">mDot Development</a></h3>
      <p>Developing with mDot embedded LoRa modules.</p>
    </div>
    <div class="card">
      <h3><a href="{{ '/dot-development-xdot' | relative_url }}">xDot Development</a></h3>
      <p>Developing with xDot ultra-low-power LoRa modules.</p>
      <div class="card-links">
        <a href="{{ '/xdot-daplink' | relative_url }}">DAPLink Debug</a>
      </div>
    </div>
    <div class="card">
      <h3><a href="{{ '/dot-migrate-to-4' | relative_url }}">Migrate to Dot v4</a></h3>
      <p>Migration guide for updating to Dot firmware version 4.</p>
    </div>
  </div>
</div>

---

## Support & Resources

- **GitHub Issues**: [MultiTechSystems/multitechsystems.github.io](https://github.com/MultiTechSystems/multitechsystems.github.io)
- **Support Portal**: [support.multitech.com](https://support.multitech.com)
- **Product Documentation**: [multitech.com/documentation](https://www.multitech.com/documentation)

<p style="text-align: center; color: #666; margin-top: 40px;">
  <small>Documentation Version {{ site.data.versions.current }} | API: {{ site.data.versions.api_version }} | Last Updated: {{ site.data.versions.last_updated }}</small>
</p>
