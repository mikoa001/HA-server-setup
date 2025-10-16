# Hardware Setup

A rough overveiw of the hardware setup for my Home Assistant server.

### 3D Case Design
![Case Design](../pictures/Pi5+2.5_SSD%20-%20case%20v5%20HiHiRes.png)
*I made a custom 3D-printed case for an Raspberry Pi 5 + 2.5" SATA SSD*

### Server Setup
![Hardware Setup](../pictures/image.jpg)
*Raspberry Pi 5 with SSD and Zigbee dongle mounted on wall*

### Main Components

1. **Raspberry Pi 5 Model B (8GB RAM)**
   - Latest generation Raspberry Pi
   - 8GB RAM for smooth operation
   - USB 3.0 ports for fast SSD access
   - Improved thermal performance

2. **256GB 2.5" SATA SSD**
   - Much faster and more reliable than SD cards
   - Sufficient storage for Home Assistant database
   - Better wear leveling and longevity
   - Recommended: 256GB or larger

3. **USB to SATA Adapter**
   - Connects the SSD to Raspberry Pi
   - Use USB 3.0 adapter for best performance
   - Ensure it's compatible with UASP for speed

4. **Sonoff Zigbee 3.0 USB Dongle Plus**
   - Zigbee coordinator for smart home devices
   - Based on Texas Instruments CC2652P chip
   - Supports Zigbee 3.0 protocol
   - Good range and device capacity

5. **USB Extension Cable**
   - **Important:** Reduces interference from Raspberry Pi
   - Improves Zigbee signal quality
   - Recommended: 1-2 meters for optimal placement

### Custom Enclosure

A custom 3D-printed case has been designed for this setup:

- **Features:**
  - Houses both Raspberry Pi 5 and 2.5" SSD
  - Integrated mounting for wall installation
  - Ventilation for cooling
  - Cable management
  - Access to all ports

- **3D Printing:**
  - Model file: `pictures/Pi5+2.5_SSD - case v5 HiHiRes.png`
  - Material: PLA or PETG recommended
  - Infill: 20-30% for structural integrity
  - Print orientation: Base down for best results
  - Using a plexiglass cover for design

## Physical Installation

### 1. Assembly

1. **Prepare the SSD:**
   - Connect SSD to USB-SATA adapter
   - Flash Home Assistant OS using Raspberry Pi Imager

2. **Assemble Raspberry Pi:**
   - Install Raspberry Pi in the case (if using)
   - Mount SSD in designated compartment
   - Connect SSD adapter to USB 3.0 port

3. **Zigbee Dongle:**
   - Connect Zigbee dongle to USB extension cable
   - Plug extension into Raspberry Pi USB port
   - Position dongle away from Pi for best signal

### 2. Wall Mounting

As shown in the setup photo:

1. **Choose Location:**
   - Central location in home
   - Good WiFi/Ethernet access
   - Near power outlet
   - Away from metal objects that interfere with Zigbee

2. **Mounting:**
   - Use appropriate wall anchors
   - Ensure case is level
   - Allow space for cable connections
   - Consider cable routing to power outlet

3. **Cable Management:**
   - Power cable to Raspberry Pi
   - Ethernet cable (optional but recommended)
   - USB extension with Zigbee dongle positioned optimally

### 3. Power Supply

- **Raspberry Pi 5 Requirements:**
  - Official 27W USB-C power supply recommended
  - Minimum 5V/3A for stable operation
  - Consider UPS for power backup

## Performance Considerations

### SSD vs SD Card

Benefits of using SSD:
- **Speed:** 5-10x faster than SD card
- **Reliability:** Much longer lifespan
- **Database Performance:** Faster queries and logging
- **Wear Leveling:** Better handling of frequent writes

### Zigbee USB Extension Benefits

Using a USB extension cable:
- Reduces USB 3.0 interference with 2.4GHz Zigbee
- Improves Zigbee mesh network stability
- Allows optimal placement of coordinator
- Increases communication range

### Thermal Management

Raspberry Pi 5 thermal considerations:
- Active cooling recommended (fan or heatsink)
- Case design with ventilation slots
- Monitor temperature with system sensors
- Thermal throttling occurs at ~80°C

## Wiring Diagram

```
[Power Outlet]
      |
      v
[USB-C Power Supply]
      |
      v
[Raspberry Pi 5] <-- [USB-SATA Adapter] <-- [256GB SSD]
      |
      v
[USB Extension Cable]
      |
      v
[Sonoff Zigbee Dongle]
```

## Network Configuration

### Recommended Setup

1. **Wired Connection:**
   - Use Ethernet for main connectivity
   - More stable than WiFi
   - Lower latency for automations

2. **WiFi as Backup:**
   - Configure WiFi as secondary
   - Automatic failover if Ethernet disconnects

### Port Forwarding (for remote access)

If accessing remotely:
- Port 8123 for Home Assistant web interface
- Consider VPN for security instead of direct port forwarding
- Use HTTPS with Let's Encrypt or similar

## Maintenance

### Regular Checks

1. **Monthly:**
   - Check system temperature
   - Review disk space usage
   - Verify Zigbee network health
   - Check for software updates

2. **Quarterly:**
   - Clean dust from enclosure/ventilation
   - Verify all cables are secure
   - Check mounting stability

3. **Backup Schedule:**
   - Weekly automatic backups recommended
   - Store backups on separate storage
   - Test restore procedure periodically

## Troubleshooting Hardware Issues

### SSD Not Detected

1. Check USB-SATA adapter connection
2. Try different USB port (use USB 3.0)
3. Verify power supply is adequate
4. Check SSD in another computer

### Zigbee Connection Issues

1. Ensure USB extension is used
2. Check dongle LED status
3. Move dongle away from Pi/WiFi router
4. Update Zigbee2MQTT firmware
5. Check USB port power

### Overheating

1. Verify case ventilation
2. Add heatsink or fan if needed
3. Check ambient temperature
4. Reduce CPU overclock if applied
5. Monitor with system sensors

## Upgrade Path

Future hardware upgrades to consider:
- Larger SSD (512GB or 1TB) for more data retention
- NVMe SSD with PCIe adapter for even better performance
- Additional Zigbee router devices to extend range
- Backup power supply (UPS) for reliability
- Additional USB Zigbee dongle for redundancy
