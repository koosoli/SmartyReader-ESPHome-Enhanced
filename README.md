# ESPHome Templates for SmartyReader & Slimmelezer

This repository provides optimized, feature-rich ESPHome templates for popular P1 smart meter readers, including the **WeiGu SmartyReader** and **Slimmelezer**. These configurations are designed for seamless, stable integration with Home Assistant.

This project modernizes and ports the incredible features from the original Arduino-based SmartyReader project by **Guy Weiler (weigu.lu)** to the ESPHome ecosystem.

## 📁 Configuration Files

## ✨ Key Features

-   **Hardware-Specific:** Separate, pre-configured templates for different hardware to avoid confusion.
-   **Advanced Analytics:** 15-minute statistics (mean/min/max), daily/monthly totals, and power class monitoring (3kW, 7kW, 12kW).
-   **Cost Calculation:** Automatically calculates the cost of exceeding power thresholds.
-   **Gas & Water Metering:** Full support for M-Bus gas and water meters, with automatic detection of the secondary meter type.
-   **Robust & Stable:** Includes modern ESPHome features like Safe Mode, Fallback AP, and memory optimizations for high reliability.
-   **MQTT Version Available:** A powerful ESP32-based template with deep MQTT integration, including LWT (Birth/Will messages) for professional monitoring.

## 📁 Available Templates

Choose the **one** file that matches your hardware and desired communication method.

| Filename | Hardware | Communication | Description |
| :--- | :--- | :--- | :--- |
| `slimmelezer-esp8266-api.yaml` | **Slimmelezer** | Home Assistant API | **Recommended for Slimmelezer users.** A complete, standalone config for the ESP8266-based Slimmelezer. |
| `weigu-smartyreader-esp8266-api.yaml` | **WeiGu SmartyReader** | Home Assistant API | **Recommended for WeiGu users.** A complete, standalone config for the ESP8266-based SmartyReader. |
| `smartyreader-esp32-mqtt.yaml` | **ESP32 (Universal)** | MQTT | **For advanced users or non-HA systems.** A powerful template for any ESP32 board using MQTT. |


## 🚀 Installation Guide for Novice Users

Follow these steps to get your reader up and running in minutes.

### Step 1: Prepare Your `secrets.yaml`

1.  In your ESPHome dashboard, find or create the `secrets.yaml` file.
2.  Copy the entire content from the `secrets_template.yaml` file in this repository.
3.  Paste it into your `secrets.yaml` and fill in your values. At a minimum, you **must** provide:
    *   `wifi_ssid` and `wifi_password`
    *   `dsmr_key` (Your 32-character key from your utility provider)
    *   `ota_password` (Create a password for wireless updates)
    *   `fallback_ap_password` (Create a password for the recovery hotspot)
    *   `api_key` (If using an API version. You can generate one with an online tool).

### Step 2: Create a New Device in ESPHome

1.  In the ESPHome Dashboard, click **"+ NEW DEVICE"**.
2.  Give it a name (e.g., `slimmelezer`) and click through the wizard.
3.  Select the correct board for your hardware:
    *   For **Slimmelezer**: Choose **WEMOS D1 mini**.
    *   For **WeiGu SmartyReader**: Choose **WEMOS D1 mini Pro**.
    *   For the **MQTT Version**: Choose **Generic ESP32**.
4.  Click **"SKIP"** when asked to create a basic configuration. A blank device file will be created.

### Step 3: Copy and Paste the Template

1.  Click **"EDIT"** on your newly created device.
2.  Open the correct YAML template file from this repository for your hardware (e.g., `slimmelezer-esp8266-api.yaml`).
3.  **Copy the entire content** of the template.
4.  **Paste it into the ESPHome editor**, replacing anything that was there.
5.  *Optional:* Change the `device_name` and `friendly_name` under the `substitutions:` block at the top of the file.

### Step 4: Install

1.  Click **"SAVE"** and then **"INSTALL"**.
2.  Choose your preferred installation method (e.g., "Plug into this computer").
3.  Follow the on-screen instructions.

Once flashed, your device will connect to your Wi-Fi and should automatically be discovered by Home Assistant!

## 🔧 Configuration & Customization

The templates are designed to work out-of-the-box, but you can customize them further.

### M-Bus Channel Configuration (Gas/Water)

The DSMR standard assigns gas to M-Bus channel 1 and water to channel 2. This is the default. If you have a non-standard setup (e.g., only a water meter on channel 1), you can change it in the `dsmr:` section.

**Example for a water-only setup on channel 1:**
```yaml
dsmr:
  gas_mbus_id: 0   # Disable gas monitoring
  water_mbus_id: 1 # Assign water meter to channel 1


### Cost Calculations
To change the cost per kWh for exceeding power classes, edit the lambda value in the cost sensors.
```yaml
# Update cost per kWh for your region
- platform: template
  name: "Cost Exceed 3kW Today"
  # ...
  lambda: 'return id(exceed_class_3kw_day_kwh) * 0.1139;' # Change 0.1139 to your rate
```



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

### **Community & Hardware**
A special thank you also goes to:
- The **ESPHome team and contributors** for creating the robust `dsmr` component that makes this integration possible.
- **Marcel Zuidwijk** for the **Slimmelezer**, for providing an excellent open-hardware solution for the community.

### **Original Project**
- **Author**: Guy WEILER  
- **Website**: [www.weigu.lu](https://www.weigu.lu)
- **Project**: SmartyReader - Arduino-based Luxembourg smartmeter interface
- **Hardware**: Complete P1 port interface boards and documentation



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

## ⚖️ License

This project is released under the **GNU General Public License v3.0**. Please see the `LICENSE` file for full details.

The separate MQTT script included in this repository remains **untested** and may not work as expected with this template. The primary, recommended method for integration is via the native ESPHome API.
