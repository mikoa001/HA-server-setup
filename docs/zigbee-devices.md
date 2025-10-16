# Zigbee Device Integration Guide

This guide covers how to integrate and manage Zigbee devices with your Home Assistant setup using Zigbee2MQTT and the Sonoff Zigbee 3.0 USB Dongle Plus.

## Overview

Zigbee is a low-power wireless mesh network protocol ideal for smart home devices. The coordinator (Sonoff dongle) communicates with devices, which can relay messages to extend network range.

## Coordinator Setup

### Sonoff Zigbee 3.0 USB Dongle Plus

**Specifications:**
- Chip: Texas Instruments CC2652P
- Protocol: Zigbee 3.0
- Range: Up to 200m line-of-sight
- Max devices: 50-100+ (depending on network)
- Firmware: Z-Stack

**Key Features:**
- Excellent range and performance
- Regular firmware updates
- Well-supported by Zigbee2MQTT
- Router capability (if flashed with router firmware)

### USB Extension Cable

**Important:** Always use a USB extension cable (1-2m) for the Zigbee coordinator:

**Benefits:**
- Reduces interference from Raspberry Pi USB 3.0
- Improves signal quality
- Allows optimal placement of coordinator
- Extends Zigbee mesh network range

**Placement Tips:**
- Central location in your home
- Away from WiFi routers and metal objects
- Elevated position (higher is better)
- Not inside metal enclosures

## Supported Device Types

### Sensors

#### Motion Sensors
- **Xiaomi/Aqara RTCGQ11LM**
  - Battery-powered
  - 5-second cooldown
  - Light sensor included
  - ~2 year battery life

#### Temperature/Humidity Sensors
- **Xiaomi/Aqara WSDCGQ11LM**
  - Temperature range: -20 to 60°C
  - Humidity: 0-100%
  - E-ink display
  - ~1 year battery life

#### Door/Window Sensors
- **Xiaomi/Aqara MCCGQ11LM**
  - Magnetic contact sensor
  - Compact design
  - ~2 year battery life
  - Fast response time

### Lights

#### Smart Bulbs
- **IKEA TRADFRI**
  - Affordable and reliable
  - White spectrum or RGB
  - Mains-powered (acts as router)
  - Good integration with Zigbee2MQTT

#### LED Strips
- **Gledopto RGB+CCT**
  - Full color control
  - Warm/Cool white
  - Controller acts as router

### Switches and Plugs

#### Smart Plugs
- **SONOFF S31 Lite**
  - Power monitoring
  - Acts as router
  - Reliable switching

#### Wall Switches
- **Aqara Wall Switch**
  - No neutral required (some models)
  - Single/Double gang options
  - Maintains physical control

## Pairing Devices

### General Pairing Process

1. **Enable Pairing Mode:**
   ```yaml
   # Via Zigbee2MQTT Frontend
   Navigate to: http://homeassistant.local:8080
   Click: "Permit Join (All)" button
   ```

2. **Reset Device:**
   - Refer to device manual for reset procedure
   - Most devices: Hold button for 5-10 seconds
   - Look for LED flash indicating reset

3. **Wait for Pairing:**
   - Device should appear in Zigbee2MQTT within 30 seconds
   - Check the Zigbee2MQTT log for pairing status

4. **Disable Pairing:**
   - Click "Permit Join" again to disable
   - **Important:** Always disable when done for security

### Device-Specific Pairing

#### Xiaomi/Aqara Sensors
1. Hold reset button for 5 seconds
2. LED will flash blue
3. Device pairs automatically

#### IKEA TRADFRI Bulbs
1. Turn on/off 6 times
2. Bulb will flash
3. Device pairs automatically

#### Aqara Switches
1. Hold button for 10 seconds
2. LED flashes rapidly
3. Release when flashing

## Configuring Devices in Home Assistant

### Automatic Discovery

Zigbee2MQTT with Home Assistant discovery enabled will automatically create entities:

```yaml
# In Zigbee2MQTT configuration
homeassistant: true
```

### Entity Naming

Customize entity names in Zigbee2MQTT:

1. Go to Zigbee2MQTT frontend
2. Click on device
3. Change "Friendly name"
4. Entities update automatically in HA

### Example Entity Customization

```yaml
# In customize.yaml
sensor.living_room_motion_temperature:
  friendly_name: "Living Room Temperature"
  icon: mdi:thermometer

binary_sensor.living_room_motion_occupancy:
  friendly_name: "Living Room Motion"
  device_class: motion
```

## Building a Strong Mesh Network

### Router Devices

Mains-powered devices act as routers:
- Smart plugs
- Light bulbs
- LED controllers
- Powered switches

**Best Practices:**
- Place routers throughout your home
- Minimum 1 router per room
- Create mesh "paths" to distant devices
- Avoid dead-end branches

### Network Topology

```
[Coordinator] ---- [Router 1] ---- [Router 2] ---- [End Device]
      |                |                |
   [Router 3]     [End Device]    [End Device]
      |
   [End Device]
```

### Optimization Tips

1. **Channel Selection:**
   ```yaml
   # Avoid WiFi interference
   # WiFi Ch 1,6,11 → Use Zigbee Ch 25
   # WiFi Ch 1-6   → Use Zigbee Ch 25
   # WiFi Ch 6-11  → Use Zigbee Ch 11
   advanced:
     channel: 25
   ```

2. **Transmit Power:**
   ```yaml
   advanced:
     transmit_power: 20  # Maximum for CC2652P
   ```

3. **Router Placement:**
   - Every 10-15 meters
   - Not all in one location
   - Strategic coverage

## Device Groups

Create groups for easier control:

```yaml
# In Zigbee2MQTT configuration.yaml
groups:
  '1':
    friendly_name: living_room_lights
    devices:
      - 0x00158d0001a2b3c7
      - 0x00158d0001a2b3c8
    
  '2':
    friendly_name: bedroom_lights
    devices:
      - 0x00158d0001a2b3c9
      - 0x00158d0001a2b3ca
```

Control groups in Home Assistant:
```yaml
service: light.turn_on
target:
  entity_id: light.living_room_lights
data:
  brightness: 200
```

## Monitoring Device Health

### Battery Monitoring

Create automation for low battery:

```yaml
- id: 'zigbee_battery_low'
  alias: Zigbee Low Battery Alert
  trigger:
    - platform: numeric_state
      entity_id:
        - sensor.motion_sensor_battery
        - sensor.door_sensor_battery
      below: 20
  action:
    - service: notify.mobile_app
      data:
        title: "Zigbee Battery Low"
        message: "{{ trigger.to_state.attributes.friendly_name }}: {{ trigger.to_state.state }}%"
```

### Link Quality Monitoring

Monitor Zigbee link quality (LQI):

```yaml
sensor:
  - platform: mqtt
    name: "Living Room Motion LQI"
    state_topic: "zigbee2mqtt/living_room_motion"
    value_template: "{{ value_json.linkquality }}"
    unit_of_measurement: "lqi"
```

Good LQI values:
- 200-255: Excellent
- 150-199: Good
- 100-149: Fair
- <100: Poor (consider adding router)

### Network Map

Visualize your Zigbee network:

1. Go to Zigbee2MQTT frontend
2. Click "Map" tab
3. See coordinator and all connected devices
4. Identify weak connections

## Troubleshooting

### Device Not Pairing

1. **Check Distance:**
   - Pair within 1-2m of coordinator
   - Move closer if failing

2. **Reset Device:**
   - Factory reset device
   - Try pairing again

3. **Check Coordinator:**
   - Restart Zigbee2MQTT
   - Check USB connection
   - Verify USB extension used

4. **Network Capacity:**
   - Max ~40 direct connections to coordinator
   - Add routers for more devices

### Device Dropping Connection

1. **Add Routers:**
   - Place router between coordinator and device
   - Improves mesh stability

2. **Check Interference:**
   - Move away from WiFi router
   - Change Zigbee channel
   - Check for USB 3.0 interference

3. **Battery Devices:**
   - Replace batteries
   - Some batteries voltage drops before dead

### Poor Response Time

1. **Mesh Network:**
   - Add more routers
   - Optimize router placement

2. **Coordinator Load:**
   - Consider second coordinator for many devices
   - Reduce polling frequency

3. **Database Size:**
   - Clean up Zigbee2MQTT database
   - Remove unused devices

## Firmware Updates

### Coordinator Firmware

Update Sonoff dongle firmware:

1. Use [CC2538-BSL](https://github.com/JelmerT/cc2538-bsl) or [ZigStar](https://github.com/xyzroe/ZigStarGW-MT) tool
2. Download latest Z-Stack firmware
3. Flash via USB

**Note:** Updating coordinator firmware requires re-pairing all devices in some cases.

### Device Firmware (OTA)

Enable OTA updates in Zigbee2MQTT:

```yaml
ota:
  update_check_interval: 1440
  disable_automatic_update_check: false
```

Update devices via frontend:
1. Go to device page
2. Check for updates
3. Click "Update" if available

## Advanced Configuration

### Device-Specific Settings

```yaml
devices:
  '0x00158d0001a2b3c4':
    friendly_name: 'living_room_motion'
    retain: true
    # Debounce motion sensor
    debounce: 1
    # Report config
    reporting:
      temperature:
        minimum: 10
        maximum: 300
        change: 0.5
```

### Binding

Bind devices for direct control (no coordinator):

1. Go to device in Zigbee2MQTT
2. Click "Bind" tab
3. Select target device or group
4. Choose cluster (e.g., "genOnOff" for switch)

## Recommended Device List

### Starter Kit
- 2x Aqara Motion Sensors
- 4x Aqara Door/Window Sensors
- 2x Aqara Temperature Sensors
- 4x IKEA TRADFRI Bulbs (for routers)
- 2x Smart Plugs (for routers)

### Expansion Ideas
- Smart blinds/curtains
- Water leak sensors
- Smoke detectors
- Smart locks
- RGB LED strips
- Wall switches

## Resources

- [Zigbee2MQTT Supported Devices](https://www.zigbee2mqtt.io/supported-devices/)
- [Zigbee Channel Planning](https://www.metageek.com/training/resources/zigbee-wifi-coexistence.html)
- [Device Compatibility Database](https://zigbee.blakadder.com/)
