# Enhanced SmartyReader ESPHome Configurations

This directory contains enhanced ESPHome configurations that bring advanced features from the original Arduino-based SmartyReader project from Guy Weiler (www.weigu.lu) to ESPHome, making them compatible with Home Assistant while adding powerful new capabilities.

## 📁 Configuration Files

### 1. `enhanced_weigu_smartyreader.yaml` - Full Featured Version
**Recommended for most users**
- Complete DSMR P1 port reading with Luxembourg smartmeter support
- Advanced power analytics and statistics
- 15-minute statistical calculations (mean/min/max)
- Power class monitoring (3kW, 7kW, 12kW thresholds)
- Daily and monthly energy exceed tracking with cost calculations
- BME280 environmental sensor support
- Comprehensive Home Assistant integration
- Web interface for diagnostics

### 2. `enhanced_weigu_smartyreader_mqtt.yaml` - MQTT Enhanced Version
**For users preferring MQTT or using other home automation platforms**
- All features from the enhanced version
- Full MQTT integration with individual topics for each sensor
- JSON payload publishing (similar to Arduino "PUBLISH_COOKED" mode)
- **Last Will and Testament (LWT)**: Birth/will messages for reliable connection status monitoring
- MQTT birth message: "online" when device connects
- MQTT will message: "offline" when device disconnects unexpectedly
- Compatible with OpenHAB, Node-RED, and other MQTT-based systems
- Dual integration: Home Assistant API + MQTT

### 3. `smartyreader_test_mode.yaml` - Testing & Debugging
**For troubleshooting and development**
- Raw P1 data inspection (hex output like Arduino test program)
- Connection diagnostics and error counting
- Telegram reception monitoring
- Debug logging and status reporting
- Minimal sensor configuration for testing

### 4. `weigu-ESPhome.txt` - Original Simple Version
**Basic configuration for reference**
- Your current working configuration
- Simple DSMR reading without advanced features

### 5. `secrets_template.yaml` - Configuration Template
**Security and credentials**
- Template for all required secrets
- WiFi, API keys, MQTT credentials
- Copy and customize for your setup

## 🚀 New Features Added

### Advanced Analytics
- **Statistical Calculations**: 15-minute rolling averages, min/max values for all power readings
- **Excess Solar Tracking**: Per-phase excess solar power calculations
- **Daily Energy Totals**: Automatic daily energy consumption and production tracking
- **Historical Data**: Monthly accumulation with automatic midnight resets

### Power Class Monitoring
- **Threshold Monitoring**: Configurable power class limits (3kW, 7kW, 12kW)
- **Exceed Energy Tracking**: Precise measurement of energy consumption above thresholds
- **Cost Calculations**: Automatic cost calculation for exceeding power classes (€0.1139/kWh)
- **Time-based Reporting**: 15-minute, daily, and monthly exceed summaries

### Enhanced Connectivity
- **MQTT Integration**: Full MQTT support with individual topics and JSON payloads
- **Last Will and Testament (LWT)**: Automatic "offline" status when device disconnects unexpectedly
- **Birth Messages**: Automatic "online" status when device connects successfully
- **Static IP Support**: Optional static IP configuration
- **Discovery Protocol**: Automatic Home Assistant device discovery

### Improved Diagnostics
- **Raw Data Inspection**: Hex dump of P1 data streams for debugging
- **Connection Monitoring**: Automatic detection of communication issues  
- **Error Tracking**: Comprehensive error counting and reporting
- **Status Indicators**: Real-time status updates and health monitoring

### Smart Controls
- **Manual Reset Functions**: Reset daily/monthly statistics on demand
- **Buffer Management**: Clear statistical buffers when needed
- **Debug Controls**: Enable/disable debug modes dynamically
- **Web Interface**: Built-in web server for configuration and monitoring

## 📊 Sensor Overview

### DSMR P1 Port Sensors
- Energy consumption/production (total and per tariff)
- Power consumption/production (total and per phase)
- Voltage and current per phase  
- Gas consumption
- Electricity failures and voltage events

### Calculated Sensors
- Daily energy totals (consumption/production)
- 15-minute power statistics (mean/min/max)
- Excess solar power (total and per phase)
- Power class exceed energy and costs
- Real-time calculations and historical totals

### Environmental Sensors (BME280)
- Temperature, humidity, atmospheric pressure
- I²C connection via expansion connector

### System Sensors
- WiFi signal strength and connection info
- Device uptime and ESPHome version
- Last update timestamps

## ⚙️ Installation Instructions

### 1. Choose Your Configuration
```bash
# For Home Assistant users (recommended)
cp enhanced_weigu_smartyreader.yaml your_device_name.yaml

# For MQTT users or mixed environments
cp enhanced_weigu_smartyreader_mqtt.yaml your_device_name.yaml

# For testing and debugging
cp smartyreader_test_mode.yaml your_device_name.yaml
```

### 2. Configure Secrets
```bash
# Copy template and edit with your values
cp secrets_template.yaml secrets.yaml
# Edit secrets.yaml with your WiFi, keys, etc.
```

### 3. Customize Device Settings
Edit your chosen configuration file:
```yaml
# Update device names
substitutions:
  device_name: your_smartyreader_name
  friendly_name: "Your SmartyReader Name"
  
# Update DSMR key (Luxembourg smartmeter)
dsmr:
  decryption_key: !secret your_dsmr_key
```

### 4. Flash to Device
```bash
# Compile and upload
esphome run your_device_name.yaml

# Or compile only
esphome compile your_device_name.yaml
```

## 🔧 Hardware Requirements

### Base Requirements
- **ESP8266** (LOLIN/WEMOS D1 Mini Pro recommended) or **ESP32**
- **P1 Port Connection** to Luxembourg smartmeter
- **Proper signal inversion** (built into newer hardware)

### Optional Additions
- **BME280 Sensor**: Temperature/humidity/pressure via I²C
- **Status LED**: Built-in LED for visual status indication  
- **Expansion Connector**: For additional sensors or debugging

### Connection Diagram
```
Smartmeter P1 Port → SmartyReader Board → ESP8266/ESP32
     │                        │               │
     ├─ Pin 2 (5V Enable)     ├─ Power        ├─ WiFi Connection
     ├─ Pin 5 (Data TX)       ├─ UART RX      ├─ Home Assistant
     └─ Pin 6 (GND)           └─ Signal Inv.  └─ MQTT (optional)
```

## 📈 Configuration Options

### Power Class Thresholds
```yaml
# Customize power exceed monitoring
- lambda: |-
    // Change these values to match your contract
    double exceed_3kw = power > 3000 ? (power - 3000) * 10.0 / 3600.0 : 0.0;
    double exceed_7kw = power > 7000 ? (power - 7000) * 10.0 / 3600.0 : 0.0;
    double exceed_12kw = power > 12000 ? (power - 12000) * 10.0 / 3600.0 : 0.0;
```

### Cost Calculations
```yaml
# Update cost per kWh for your region
- lambda: |-
    return id(exceed_class_3kw_day_kwh) * 0.1139;  # €0.1139 per kWh
```

### Statistical Window
```yaml
# Change 15-minute window (90 samples at 10s intervals)
globals:
  - id: power_consumption_buffer
    type: double[90]  # Adjust size: 90 = 15min, 180 = 30min
```

## 📡 **MQTT Last Will and Testament (LWT)**

### **What is LWT?**
Last Will and Testament is an MQTT protocol feature that ensures reliable device status monitoring:

- **Birth Message**: Published as "online" when the device successfully connects to MQTT broker
- **Will Message**: Automatically published as "offline" by the broker if the device disconnects unexpectedly
- **Retain Flag**: Status messages are retained so new subscribers immediately know device status
- **Topic**: Published to `{topic_prefix}/status` (e.g., `lamsmarty/status`)

### **How It Works**
```yaml
# MQTT LWT Configuration (in MQTT enhanced version)
mqtt:
  birth_message:
    topic: lamsmarty/status
    payload: "online"      # Published when device connects
    retain: true
  will_message:
    topic: lamsmarty/status
    payload: "offline"     # Published by broker if device disconnects
    retain: true
```

### **Why Important for ESP Devices?**
ESP devices can disconnect due to:
- Power outages or brownouts
- WiFi connectivity issues
- Device crashes or freezes
- Physical disconnection

LWT ensures your home automation system knows the device status even when the device can't send a manual "goodbye" message.

### **Monitoring in Home Assistant**
```yaml
# Create a binary sensor to monitor device status
binary_sensor:
  - platform: mqtt
    name: "SmartyReader Status"
    state_topic: "lamsmarty/status"
    payload_on: "online"
    payload_off: "offline"
    device_class: connectivity
```

## 🏠 Home Assistant Integration

### Automatic Discovery
All sensors automatically appear in Home Assistant with proper device classes:
- **Energy**: kWh sensors with `total_increasing` state class
- **Power**: W sensors with `measurement` state class  
- **Voltage/Current**: V/A sensors with appropriate device classes
- **Environmental**: Temperature, humidity, pressure sensors

### Custom Dashboards
Create energy dashboards using the enhanced sensors:
```yaml
# Energy dashboard card example
type: energy-date-selection
title: Energy Usage
entities:
  - sensor.energy_consumed_today
  - sensor.energy_produced_today
  - sensor.cost_exceed_3kw_today
```

### Automations
Use power exceed sensors for smart home automation:
```yaml
# Example automation for high power usage
automation:
  - alias: "High Power Alert"
    trigger:
      - platform: numeric_state
        entity_id: sensor.power_consumed
        above: 7000
    action:
      - service: notify.mobile_app
        data:
          message: "Power usage exceeds 7kW!"
```

## 🔍 Troubleshooting

### Common Issues

1. **No DSMR Data Received**
   - Check P1 cable connections
   - Verify DSMR decryption key
   - Use test mode configuration for debugging

2. **Incorrect Power Values**
   - Verify signal inversion in UART configuration
   - Check for interference on UART lines
   - Review raw data using test mode

3. **Statistical Calculations Not Working**
   - Ensure sufficient uptime for buffer fill
   - Check global variable initialization
   - Verify interval timers are running

4. **Status LED Not Working / Random Flickering**
   - **CRITICAL**: Cannot use both `hardware_uart: UART1` and `status_led` on same pin
   - **Solution**: Comment out `hardware_uart: UART1` in logger section
   - **Pin Conflict**: D4 (GPIO2) used by both UART1 TX and built-in LED
   - **Result**: Logger works via WiFi, status LED functions properly

### Debug Mode
Enable comprehensive logging:
```yaml
logger:
  level: VERBOSE
  logs:
    dsmr: VERBOSE
    sensor: DEBUG
```

### Test Mode Features
- Raw P1 data inspection (hex dump)
- Telegram reception counting
- Connection status monitoring
- Error detection and logging

## 📚 Comparison with Arduino Version

| Feature | Arduino Original | ESPHome Enhanced | Notes |
|---------|-----------------|------------------|-------|
| DSMR Reading | ✅ Full | ✅ Full | Complete parity |
| Power Statistics | ✅ Ring Buffers | ✅ Simplified | ESPHome limitations |
| Power Class Monitor | ✅ Full | ✅ Full | Complete implementation |
| MQTT Publishing | ✅ Individual + JSON | ✅ Individual + JSON | Feature parity |
| **MQTT LWT (Birth/Will)** | ✅ Full | ✅ Full | Complete parity |
| Home Assistant | ❌ Manual | ✅ Native | ESPHome advantage |
| OTA Updates | ✅ Arduino OTA | ✅ ESPHome OTA | Both supported |
| Web Interface | ❌ None | ✅ Built-in | ESPHome advantage |
| BME280 Support | ✅ Manual | ✅ Native | ESPHome advantage |
| Raw Data Debug | ✅ Full | ✅ Limited | Arduino more detailed |
| Ethernet Support | ✅ W5100/W5500 | ❌ Not supported | Arduino advantage |

## 🙏 **Credits & Acknowledgments**

### **Special Thanks to Guy WEILER**
This project is built upon the **outstanding work** of **Guy WEILER** ([www.weigu.lu](https://www.weigu.lu)), who created the original Arduino-based SmartyReader project. His comprehensive implementation provided:

- ✨ **Complete DSMR P1 port reading** with Luxembourg smartmeter decryption
- ✨ **Advanced power analytics** including ring buffer statistical calculations  
- ✨ **Power class monitoring** with exceed energy tracking and cost calculations
- ✨ **Comprehensive MQTT integration** with individual topics and JSON payloads
- ✨ **BME280 environmental sensor** integration
- ✨ **Extensive debugging tools** and test programs
- ✨ **Detailed documentation** and hardware specifications

**Guy's Arduino implementation serves as the foundation** for all the advanced features in these ESPHome configurations. His meticulous attention to detail and comprehensive feature set made this ESPHome port possible.

### **Original Project**
- **Author**: Guy WEILER  
- **Website**: [www.weigu.lu](https://www.weigu.lu)
- **Project**: SmartyReader - Arduino-based Luxembourg smartmeter interface
- **Hardware**: Complete P1 port interface boards and documentation

### **ESPHome Enhancement**
This ESPHome version modernizes Guy's excellent work by:
- Adapting Arduino C++ code to ESPHome YAML configuration  
- Adding native Home Assistant integration with automatic discovery
- Providing simplified installation and maintenance
- Maintaining full feature compatibility with the original

**Thank you Guy for your amazing contribution to the Luxembourg IoT community!** 🚀

## 🤝 Contributing

To improve these configurations:

1. **Test thoroughly** with your hardware setup
2. **Document changes** clearly in commit messages  
3. **Maintain compatibility** with existing features and Guy's original design
4. **Follow ESPHome best practices** for configuration structure

## 🔗 Related Links

- [Original SmartyReader Project](https://weigu.lu)
- [ESPHome Documentation](https://esphome.io)
- [Home Assistant Energy Dashboard](https://www.home-assistant.io/docs/energy/)
- [Luxembourg P1 Specification](https://electris.lu/files/Dokumente_und_Formulare/Netz_Tech_Dokumente/SPEC_-_E-Meter_P1_specification_20210305.pdf)

---

*This enhanced ESPHome configuration brings the full power of the Arduino SmartyReader to the ESPHome ecosystem while adding modern Home Assistant integration and improved diagnostics.*
