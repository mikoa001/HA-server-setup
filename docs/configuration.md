# Home Assistant Configuration Guide

This guide explains how to configure Home Assistant effectively for your smart home.

## Configuration Files Overview

Home Assistant uses YAML files for configuration. The main files are:

- `configuration.yaml` - Main configuration file
- `automations.yaml` - Automation rules
- `scripts.yaml` - Custom scripts
- `scenes.yaml` - Scene definitions
- `secrets.yaml` - Sensitive information (passwords, API keys)
- `customize.yaml` - Entity customization

## Configuration Structure

### Using Includes

Split configuration into multiple files for better organization:

```yaml
# configuration.yaml
homeassistant:
  customize: !include customize.yaml

automation: !include automations.yaml
script: !include scripts.yaml
scene: !include scenes.yaml

# Split by domain
light: !include lights.yaml
switch: !include switches.yaml
sensor: !include sensors.yaml

# Split by directory
automation manual: !include_dir_list automations/
sensor custom: !include_dir_merge_named sensors/
```

### Secrets Management

Never commit passwords or API keys. Use secrets.yaml:

```yaml
# secrets.yaml (add to .gitignore!)
latitude: 59.9139
longitude: 10.7522
api_key: your_secret_api_key_here

# configuration.yaml
homeassistant:
  latitude: !secret latitude
  longitude: !secret longitude
  
weather:
  - platform: openweathermap
    api_key: !secret openweather_api_key
```

## Essential Configurations

### Default Setup

```yaml
# Enables essential features
default_config:

# Equivalent to:
# - frontend:
# - mobile_app:
# - person:
# - config:
# - etc.
```

### HTTP Configuration

```yaml
http:
  # Use reverse proxy
  use_x_forwarded_for: true
  trusted_proxies:
    - 127.0.0.1
    - ::1
  
  # CORS for development
  cors_allowed_origins:
    - https://mydomain.com
  
  # External access
  server_host: 0.0.0.0
  server_port: 8123
```

### Database Configuration

```yaml
recorder:
  # Database URL
  db_url: !secret db_url
  
  # Retention period
  purge_keep_days: 30
  
  # Performance
  commit_interval: 30
  auto_purge: true
  
  # Include specific entities
  include:
    domains:
      - sensor
      - light
      - switch
    entity_globs:
      - sensor.temperature_*
  
  # Exclude to reduce size
  exclude:
    domains:
      - automation
      - updater
      - sun
    entity_globs:
      - sensor.weather_*
    entities:
      - sensor.last_boot
      - sensor.date
```

### Logger Configuration

```yaml
logger:
  default: info
  logs:
    # Component logging
    homeassistant.components.mqtt: debug
    homeassistant.components.automation: debug
    
    # Integration logging
    custom_components.hacs: warning
    
    # Suppress noise
    homeassistant.components.http.ban: warning
```

## Integration Configuration

### MQTT

```yaml
mqtt:
  broker: !secret mqtt_broker
  port: 1883
  username: !secret mqtt_username
  password: !secret mqtt_password
  
  # Discovery for Zigbee2MQTT
  discovery: true
  discovery_prefix: homeassistant
  
  # Birth/will messages
  birth_message:
    topic: 'homeassistant/status'
    payload: 'online'
  will_message:
    topic: 'homeassistant/status'
    payload: 'offline'
```

### Zigbee2MQTT Integration

```yaml
mqtt:
  # Sensors from Zigbee2MQTT
  sensor:
    - name: "Zigbee2MQTT Bridge State"
      state_topic: "zigbee2mqtt/bridge/state"
      icon: "mdi:zigbee"
    
    - name: "Coordinator Link Quality"
      state_topic: "zigbee2mqtt/Coordinator"
      value_template: "{{ value_json.linkquality }}"
      unit_of_measurement: "lqi"
  
  # Binary sensors
  binary_sensor:
    - name: "Zigbee Network"
      state_topic: "zigbee2mqtt/bridge/state"
      payload_on: "online"
      payload_off: "offline"
      device_class: connectivity
```

### Weather Integration

```yaml
weather:
  # Met.no (free, no API key)
  - platform: met
    name: Home Weather
  
  # OpenWeatherMap (requires API key)
  - platform: openweathermap
    api_key: !secret openweather_api_key
    mode: onecall
```

### System Monitoring

```yaml
sensor:
  - platform: systemmonitor
    resources:
      # System resources
      - type: disk_use_percent
        arg: /
      - type: disk_use
        arg: /
      - type: disk_free
        arg: /
      
      # Memory
      - type: memory_use_percent
      - type: memory_use
      - type: memory_free
      
      # CPU
      - type: processor_use
      - type: processor_temperature
      - type: load_1m
      - type: load_5m
      - type: load_15m
      
      # Network
      - type: network_in
        arg: eth0
      - type: network_out
        arg: eth0
      
      # System
      - type: last_boot
      - type: ipv4_address
        arg: eth0
```

### Time and Date

```yaml
sensor:
  - platform: time_date
    display_options:
      - 'time'
      - 'date'
      - 'date_time'
      - 'date_time_utc'
      - 'date_time_iso'
      - 'time_date'
      - 'time_utc'
      - 'beat'
```

## Advanced Configuration

### Templates

Create virtual sensors with templates:

```yaml
template:
  - sensor:
      # Average temperature
      - name: "Average House Temperature"
        unit_of_measurement: "°C"
        state: >
          {% set temps = [
            states('sensor.living_room_temperature')|float,
            states('sensor.bedroom_temperature')|float,
            states('sensor.kitchen_temperature')|float
          ] %}
          {{ (temps | sum / temps | length) | round(1) }}
      
      # Conditional status
      - name: "House Status"
        state: >
          {% if is_state('person.your_name', 'home') %}
            occupied
          {% else %}
            empty
          {% endif %}
      
      # Time-based value
      - name: "Period of Day"
        state: >
          {% set hour = now().hour %}
          {% if hour < 6 %}
            Night
          {% elif hour < 12 %}
            Morning
          {% elif hour < 18 %}
            Afternoon
          {% elif hour < 22 %}
            Evening
          {% else %}
            Night
          {% endif %}
```

### Input Helpers

Create toggles, numbers, and selections:

```yaml
input_boolean:
  guest_mode:
    name: Guest Mode
    icon: mdi:account-multiple
  
  vacation_mode:
    name: Vacation Mode
    icon: mdi:beach

input_number:
  comfort_temperature:
    name: Comfort Temperature
    min: 18
    max: 25
    step: 0.5
    unit_of_measurement: "°C"
    icon: mdi:thermometer

input_select:
  house_mode:
    name: House Mode
    options:
      - Home
      - Away
      - Sleep
      - Vacation
    icon: mdi:home-automation

input_datetime:
  alarm_time:
    name: Alarm Time
    has_date: false
    has_time: true

input_text:
  notification_message:
    name: Notification Message
    max: 255
```

### Groups

Organize entities:

```yaml
group:
  all_lights:
    name: All Lights
    entities:
      - light.living_room
      - light.bedroom
      - light.kitchen
      - light.hallway
  
  ground_floor:
    name: Ground Floor
    entities:
      - light.living_room
      - light.kitchen
      - switch.tv
  
  climate_sensors:
    name: Climate Sensors
    entities:
      - sensor.living_room_temperature
      - sensor.bedroom_temperature
      - sensor.living_room_humidity
```

### Zones

Define locations:

```yaml
zone:
  - name: Work
    latitude: 59.9000
    longitude: 10.7500
    radius: 100
    icon: mdi:briefcase
  
  - name: School
    latitude: 59.9100
    longitude: 10.7400
    radius: 150
    icon: mdi:school
  
  - name: Parents
    latitude: 59.9200
    longitude: 10.7300
    radius: 200
    icon: mdi:home-heart
```

### Notifications

```yaml
notify:
  - platform: telegram
    name: telegram
    chat_id: !secret telegram_chat_id
  
  # Email
  - platform: smtp
    name: email
    server: smtp.gmail.com
    port: 587
    encryption: starttls
    sender: !secret email_sender
    recipient: !secret email_recipient
    username: !secret email_username
    password: !secret email_password
```

## Validation and Testing

### Check Configuration

Before restarting, always check configuration:

```bash
# Via SSH/Terminal
ha core check

# Or use Developer Tools > YAML in UI
```

### Test Automations

1. **Manual Trigger:**
   - Developer Tools > Services
   - Call automation.trigger
   - Specify automation entity_id

2. **Traces:**
   - Developer Tools > Traces
   - View execution path
   - Debug conditions and actions

3. **Dry Run:**
   - Set condition that's always false
   - Use notify to confirm trigger
   - Remove condition when working

### Reload Without Restart

Reload specific parts without full restart:

```yaml
# Developer Tools > YAML
# Or use services:

service: automation.reload
service: script.reload
service: scene.reload
service: group.reload
service: template.reload
```

## Performance Optimization

### Reduce Database Load

```yaml
recorder:
  purge_keep_days: 7
  commit_interval: 30
  exclude:
    domains:
      - automation
      - updater
    entity_globs:
      - sensor.weather_*
```

### Optimize Polling

```yaml
# Disable unnecessary polling
homeassistant:
  customize:
    sensor.rarely_changes:
      homeassistant:
        scan_interval: 3600  # 1 hour
```

### Use Event-Driven Automations

Prefer state triggers over time_pattern when possible:

```yaml
# Good - event-driven
trigger:
  - platform: state
    entity_id: binary_sensor.motion
    to: 'on'

# Less efficient - polling
trigger:
  - platform: time_pattern
    seconds: '/5'
```

## Security Best Practices

1. **Secrets Management:**
   - Use secrets.yaml
   - Never commit secrets to git
   - Rotate keys regularly

2. **Network Security:**
   - Use HTTPS for external access
   - Consider VPN instead of port forwarding
   - Enable fail2ban

3. **Access Control:**
   - Use strong passwords
   - Enable MFA
   - Limit trusted networks

4. **Regular Updates:**
   - Keep Home Assistant updated
   - Update addons regularly
   - Monitor security advisories

## Backup Strategy

```yaml
# Automated backups
automation:
  - alias: Weekly Backup
    trigger:
      platform: time
      at: '03:00:00'
    condition:
      condition: time
      weekday:
        - sun
    action:
      service: backup.create
```

Store backups:
- Local: Settings > System > Backups
- Remote: Google Drive addon, Samba, etc.
- Test restores periodically

## Resources

- [Configuration.yaml Reference](https://www.home-assistant.io/docs/configuration/)
- [Templating Guide](https://www.home-assistant.io/docs/configuration/templating/)
- [YAML Syntax](https://www.home-assistant.io/docs/configuration/yaml/)
- [Best Practices](https://www.home-assistant.io/docs/configuration/best_practices/)
