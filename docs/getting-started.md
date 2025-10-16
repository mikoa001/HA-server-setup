# Getting Started with Home Assistant on Raspberry Pi 5

This guide will help you set up Home Assistant on a Raspberry Pi 5 with the hardware configuration shown in this repository.

## Hardware Requirements

Based on the setup in this repository:

- **Raspberry Pi 5 Model B** (8GB RAM recommended)
- **256GB 2.5" SATA SSD** (or larger)
- **USB to SATA adapter**
- **Sonoff Zigbee 3.0 USB Dongle Plus**
- **USB extension cable** (for Zigbee dongle - reduces interference)
- **Power supply** for Raspberry Pi 5 (27W USB-C recommended)
- **Micro HDMI cable** (for initial setup)
- **Keyboard and mouse** (for initial setup)

### Optional

- **3D printed case** (design file included in `pictures/` directory)
- **Mounting hardware** for wall installation
- **Ethernet cable** (recommended for reliability)

## Installation Steps

### 1. Prepare the SSD

1. Download [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. Connect your SSD to your computer using the USB-SATA adapter
3. Open Raspberry Pi Imager
4. Choose OS: **Other specific-purpose OS** → **Home assistants and home automation** → **Home Assistant**
5. Select your SSD as the storage device
6. Write the image to the SSD

### 2. Initial Boot

1. Connect the SSD to your Raspberry Pi 5 via USB
2. Connect keyboard, mouse, and monitor
3. Connect network cable (optional but recommended)
4. Power on the Raspberry Pi 5
5. Wait for Home Assistant to initialize (5-20 minutes on first boot)

### 3. Access Home Assistant

1. Open a web browser on another device on the same network
2. Navigate to `http://homeassistant.local:8123`
3. Create your user account
4. Configure your location and timezone
5. Complete the onboarding process

### 4. Configure Zigbee (Zigbee2MQTT)

#### Install Zigbee2MQTT Add-on

1. Go to **Settings** → **Add-ons** → **Add-on Store**
2. Search for "Zigbee2MQTT"
3. Click **Install**

#### Configure Zigbee2MQTT

1. Connect the Sonoff Zigbee dongle to the USB extension cable
2. Plug the extension cable into the Raspberry Pi
3. In the Zigbee2MQTT add-on configuration, set:
   ```yaml
   serial:
     port: /dev/ttyUSB0
     adapter: ezsp
   ```
4. Start the add-on
5. Check the logs to ensure it's working

#### Add Zigbee Devices

1. Go to Zigbee2MQTT web interface
2. Click "Permit Join"
3. Put your Zigbee device in pairing mode (refer to device manual)
4. Wait for the device to appear in the list
5. Rename the device with a friendly name
6. Disable "Permit Join" when done

### 5. Install Mosquitto MQTT Broker

Zigbee2MQTT requires an MQTT broker:

1. Go to **Settings** → **Add-ons** → **Add-on Store**
2. Search for "Mosquitto broker"
3. Click **Install**
4. Start the add-on
5. Enable "Start on boot"

### 6. Configure MQTT Integration

1. Go to **Settings** → **Devices & Services**
2. Click **Add Integration**
3. Search for "MQTT"
4. Configure:
   - Broker: `localhost` or `core-mosquitto`
   - Port: `1883`
   - Username: (leave blank for local add-on)
   - Password: (leave blank for local add-on)

### 7. Import Example Configurations

1. Enable SSH or use File Editor add-on
2. Copy the example files from this repository to your Home Assistant config directory
3. Customize the files according to your needs
4. Restart Home Assistant

## System Optimization for Raspberry Pi 5

### Enable SSD Boot

The Raspberry Pi 5 can boot directly from SSD:

1. Update the bootloader if needed
2. Configure boot order in raspi-config
3. This improves performance and reliability

### Monitor System Health

Install the System Monitor integration to keep track of:
- CPU usage and temperature
- Memory usage
- Disk usage
- System uptime

The example configuration includes these sensors.

## Next Steps

1. Add your devices to Home Assistant
2. Create automations using the examples in `config/automations.yaml`
3. Set up dashboards using `dashboards/main-dashboard.yaml` as a template
4. Configure notifications (mobile app, email, etc.)
5. Set up backups (highly recommended!)

## Troubleshooting

See [Troubleshooting Guide](troubleshooting.md) for common issues and solutions.

## Resources

- [Home Assistant Documentation](https://www.home-assistant.io/docs/)
- [Zigbee2MQTT Documentation](https://www.zigbee2mqtt.io/)
- [Raspberry Pi 5 Documentation](https://www.raspberrypi.com/documentation/computers/raspberry-pi-5.html)
