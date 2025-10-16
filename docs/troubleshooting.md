# Troubleshooting Guide

This guide helps resolve common issues with your Home Assistant setup on Raspberry Pi 5.

## System Issues

### Home Assistant Won't Start

**Symptoms:**
- Can't access web interface
- No response on port 8123

**Solutions:**

1. **Check System Status:**
   ```bash
   # SSH into Raspberry Pi
   ha core info
   ha core logs
   ```

2. **Check Configuration:**
   ```bash
   ha core check
   ```
   Fix any configuration errors reported.

3. **Restart Home Assistant:**
   ```bash
   ha core restart
   ```

4. **Check SSD Connection:**
   - Verify USB-SATA adapter is connected
   - Check LED on SSD (should be active)
   - Try different USB port (use USB 3.0)

5. **Power Supply:**
   - Ensure using official 27W power supply
   - Check for power warnings in logs
   - Look for undervoltage indicators

### Slow Performance

**Symptoms:**
- Web interface is slow
- Automations delayed
- Database queries timeout

**Solutions:**

1. **Check System Resources:**
   ```bash
   ha core stats
   ```

2. **Database Optimization:**
   ```yaml
   # In configuration.yaml
   recorder:
     purge_keep_days: 7
     commit_interval: 30
     exclude:
       domains:
         - automation
         - updater
   ```

3. **Clean Up Database:**
   - Settings → System → Storage
   - Run database purge
   - Consider external database (MariaDB)

4. **Check Disk Space:**
   ```bash
   df -h
   ```
   Free up space if usage >80%

5. **Thermal Throttling:**
   - Check CPU temperature (should be <70°C)
   - Add cooling if needed
   - Verify case ventilation

### Cannot Access Remotely

**Symptoms:**
- Local access works
- Remote access fails

**Solutions:**

1. **Check DuckDNS:**
   - Verify DuckDNS addon is running
   - Check domain resolves correctly
   - Verify token is valid

2. **Port Forwarding:**
   - Router: Forward port 443 to Pi's IP:8123
   - Check firewall rules
   - Verify external IP

3. **HTTPS Configuration:**
   ```yaml
   http:
     ssl_certificate: /ssl/fullchain.pem
     ssl_key: /ssl/privkey.pem
   ```

4. **Alternative: Use VPN**
   - WireGuard addon
   - Tailscale addon
   - More secure than port forwarding

## Zigbee Issues

### Zigbee2MQTT Won't Start

**Symptoms:**
- Addon shows error state
- No Zigbee devices responding

**Solutions:**

1. **Check USB Dongle:**
   ```bash
   # List USB devices
   lsusb | grep -i zigbee
   # Should show: "Texas Instruments CC2531"
   ```

2. **Verify Serial Port:**
   ```bash
   ls -l /dev/ttyUSB*
   # Should show /dev/ttyUSB0 or similar
   ```

3. **Check Addon Configuration:**
   ```yaml
   serial:
     port: /dev/ttyUSB0
     adapter: ezsp
   ```

4. **USB Extension:**
   - Ensure using USB extension cable
   - Cable should be 1-2m long
   - Try different USB port

5. **Restart Addon:**
   - Stop Zigbee2MQTT
   - Unplug USB dongle
   - Wait 10 seconds
   - Plug back in
   - Start addon

### Devices Won't Pair

**Symptoms:**
- Device won't enter pairing mode
- Pairing times out
- Device not appearing

**Solutions:**

1. **Pairing Distance:**
   - Pair within 1-2 meters
   - Move closer to coordinator
   - Remove obstacles

2. **Reset Device:**
   - Follow manufacturer reset procedure
   - Usually: hold button 5-10 seconds
   - Look for LED indication

3. **Check Permit Join:**
   - Must be enabled in Zigbee2MQTT
   - Disable and re-enable if stuck

4. **Network Capacity:**
   - Coordinator max ~40 direct devices
   - Add router devices for more capacity

5. **Interference:**
   - Move away from WiFi router
   - Change Zigbee channel
   - Disable Bluetooth temporarily

### Devices Keep Disconnecting

**Symptoms:**
- Devices show "unavailable"
- Intermittent connection
- Poor reliability

**Solutions:**

1. **Add Router Devices:**
   - Place mains-powered devices (bulbs, plugs)
   - Creates mesh network
   - Improves reliability

2. **Check Link Quality (LQI):**
   - View in Zigbee2MQTT
   - Should be >100
   - Add routers if low

3. **Battery Level:**
   - Check device battery
   - Replace if <20%
   - Some devices fail before fully dead

4. **Interference Check:**
   ```yaml
   # Try different channel
   advanced:
     channel: 25  # or 11, 15, 20
   ```

5. **USB 3.0 Interference:**
   - Use longer USB extension (2m)
   - Move dongle away from Pi
   - Disable USB 3.0 if not needed

### Poor Zigbee Range

**Symptoms:**
- Devices far from coordinator fail
- Weak signal strength
- Dropped messages

**Solutions:**

1. **USB Extension:**
   - Must use 1-2m USB extension cable
   - Position dongle optimally
   - Elevate coordinator

2. **Add Routers:**
   - Every 10-15 meters
   - Strategic placement
   - Create mesh paths

3. **Increase Transmit Power:**
   ```yaml
   advanced:
     transmit_power: 20  # Max for CC2652P
   ```

4. **Coordinator Position:**
   - Central location
   - Away from metal objects
   - Not inside metal case
   - Higher elevation better

## Network Issues

### WiFi Keeps Disconnecting

**Symptoms:**
- WiFi connection drops
- Network unreachable

**Solutions:**

1. **Use Ethernet:**
   - More reliable than WiFi
   - Lower latency
   - Recommended for HA

2. **WiFi Signal:**
   - Check signal strength
   - Move Pi closer to router
   - Use WiFi extender

3. **Static IP:**
   ```bash
   # Set static IP in HA OS
   nmcli con show
   nmcli con mod "Wired connection 1" ipv4.addresses 192.168.1.100/24
   nmcli con mod "Wired connection 1" ipv4.gateway 192.168.1.1
   nmcli con mod "Wired connection 1" ipv4.dns "8.8.8.8 8.8.4.4"
   nmcli con mod "Wired connection 1" ipv4.method manual
   ```

4. **Power Management:**
   - Disable WiFi power saving
   - Prevents disconnections

### MQTT Connection Issues

**Symptoms:**
- Zigbee2MQTT can't connect to broker
- MQTT entities unavailable

**Solutions:**

1. **Check Mosquitto Broker:**
   - Settings → Addons → Mosquitto broker
   - Verify it's running
   - Check logs for errors

2. **MQTT Configuration:**
   ```yaml
   mqtt:
     broker: core-mosquitto
     port: 1883
     # Local addon doesn't need auth
   ```

3. **Restart Services:**
   - Restart Mosquitto broker
   - Restart Zigbee2MQTT
   - Restart Home Assistant

4. **Port Conflicts:**
   - Ensure port 1883 is free
   - Check no other MQTT broker running

## Hardware Issues

### SSD Not Detected

**Symptoms:**
- System won't boot
- Storage not available

**Solutions:**

1. **Check Connections:**
   - USB-SATA adapter firmly connected
   - SSD properly seated in adapter
   - Try different USB port (use USB 3.0)

2. **Test SSD:**
   - Connect to computer
   - Check if detected
   - Run disk health check

3. **Power Issue:**
   - Verify power supply (27W)
   - Some SSDs need more power
   - Try powered USB hub

4. **Re-flash OS:**
   - Use Raspberry Pi Imager
   - Flash fresh HA OS
   - Restore from backup

### Overheating

**Symptoms:**
- CPU temperature >80°C
- Thermal throttling
- Unexpected restarts

**Solutions:**

1. **Add Cooling:**
   - Install heatsink
   - Add active fan
   - Improve case ventilation

2. **Check Temperature:**
   ```yaml
   sensor:
     - platform: systemmonitor
       resources:
         - type: processor_temperature
   ```

3. **Reduce Load:**
   - Disable unused integrations
   - Optimize automations
   - Reduce polling frequency

4. **Case Ventilation:**
   - Ensure air flow
   - Don't enclose completely
   - Position away from heat sources

### USB Devices Not Recognized

**Symptoms:**
- USB devices not showing
- Intermittent USB connection

**Solutions:**

1. **Check dmesg:**
   ```bash
   dmesg | tail -30
   ```

2. **Power Limit:**
   - USB total power limited
   - Use powered USB hub
   - Check power supply capacity

3. **USB 3.0 Interference:**
   - Known to interfere with 2.4GHz
   - Use USB 2.0 ports for Zigbee
   - Use extensions to separate devices

## Configuration Issues

### Configuration Invalid

**Symptoms:**
- HA won't start after config change
- Error in configuration

**Solutions:**

1. **Check Configuration:**
   ```bash
   ha core check
   ```

2. **Review Logs:**
   ```bash
   ha core logs
   ```

3. **Restore Backup:**
   - Settings → System → Backups
   - Restore previous working backup

4. **Common Errors:**
   - YAML indentation (use spaces, not tabs)
   - Missing required fields
   - Invalid entity IDs
   - Typos in entity names

### Automations Not Working

**Symptoms:**
- Automation doesn't trigger
- Actions don't execute

**Solutions:**

1. **Enable Automation:**
   - Check automation is enabled
   - Toggle off/on to reset

2. **Check Triggers:**
   - Verify entity IDs exist
   - Check trigger conditions
   - Test with manual trigger

3. **Review Conditions:**
   - Conditions might be blocking
   - Temporarily remove conditions
   - Check condition logic

4. **Debug Mode:**
   ```yaml
   logger:
     default: warning
     logs:
       homeassistant.components.automation: debug
   ```

5. **Use Traces:**
   - Developer Tools → Traces
   - See automation execution
   - Identify where it stops

## Database Issues

### Database Growing Too Large

**Symptoms:**
- Disk space filling up
- Slow queries
- Performance degradation

**Solutions:**

1. **Purge Old Data:**
   - Settings → System → Storage
   - Run purge
   - Adjust keep_days

2. **Exclude Entities:**
   ```yaml
   recorder:
     purge_keep_days: 7
     exclude:
       domains:
         - automation
         - updater
       entity_globs:
         - sensor.weather_*
   ```

3. **External Database:**
   - MariaDB addon
   - Better performance
   - Easier maintenance

4. **Database Vacuum:**
   ```bash
   # SSH into HA
   sqlite3 /config/home-assistant_v2.db "VACUUM;"
   ```

### Database Corruption

**Symptoms:**
- Errors in logs
- History not loading
- HA won't start

**Solutions:**

1. **Restore from Backup:**
   - Most reliable solution
   - Use latest backup

2. **Delete Database:**
   ```bash
   rm /config/home-assistant_v2.db
   # HA will create new database
   # All history will be lost
   ```

3. **Repair Database:**
   ```bash
   sqlite3 /config/home-assistant_v2.db ".recover" | sqlite3 /config/home-assistant_v2_recovered.db
   mv /config/home-assistant_v2.db /config/home-assistant_v2.db.bak
   mv /config/home-assistant_v2_recovered.db /config/home-assistant_v2.db
   ```

## Getting Help

### Collect Information

Before asking for help, collect:

1. **System Info:**
   ```bash
   ha info
   ha core info
   ```

2. **Logs:**
   ```bash
   ha core logs > /config/logs.txt
   ```

3. **Configuration:**
   - Sanitize secrets
   - Share relevant config sections

### Resources

- [Home Assistant Community](https://community.home-assistant.io/)
- [Home Assistant Discord](https://discord.gg/home-assistant)
- [Zigbee2MQTT Discord](https://discord.gg/dadfWk84)
- [Reddit r/homeassistant](https://reddit.com/r/homeassistant)

### Creating Good Issue Reports

Include:
1. What you expected to happen
2. What actually happened
3. Steps to reproduce
4. System information
5. Relevant logs
6. Configuration (sanitized)
