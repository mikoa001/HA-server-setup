# Complete Smart Home Example

This document demonstrates a complete smart home setup example that ties together all the components in this repository.

## Scenario: Complete Daily Automation

This example creates a full day's worth of smart automations for a typical household.

## Hardware Required (Example)

- Raspberry Pi 5 with Home Assistant (as documented in hardware-setup.md)
- Sonoff Zigbee 3.0 USB Dongle Plus
- 4x Aqara Motion Sensors (Living room, Bedroom, Hallway, Bathroom)
- 4x Aqara Temperature Sensors (Living room, Bedroom, Kitchen, Outside)
- 6x Aqara Door/Window Sensors (Front door, Back door, 4x Windows)
- 8x IKEA TRADFRI Smart Bulbs (distributed throughout home)
- 2x Smart Plugs (Coffee maker, TV)
- 1x Smart Thermostat

## Complete Configuration Example

### 1. Main Configuration

```yaml
# configuration.yaml
homeassistant:
  name: Smart Home
  latitude: !secret latitude
  longitude: !secret longitude
  elevation: !secret elevation
  unit_system: metric
  time_zone: Europe/Oslo
  customize: !include customize.yaml

default_config:

# Load split configurations
automation: !include automations.yaml
script: !include scripts.yaml
scene: !include scenes.yaml

# MQTT for Zigbee2MQTT
mqtt:
  broker: core-mosquitto
  discovery: true
  birth_message:
    topic: 'homeassistant/status'
    payload: 'online'
  will_message:
    topic: 'homeassistant/status'
    payload: 'offline'

# System monitoring
sensor:
  - platform: systemmonitor
    resources:
      - type: disk_use_percent
      - type: memory_use_percent
      - type: processor_use
      - type: processor_temperature
      - type: last_boot
  
  - platform: time_date
    display_options:
      - 'time'
      - 'date'

# Template sensors
template:
  - sensor:
      - name: "Average Indoor Temperature"
        unit_of_measurement: "°C"
        device_class: temperature
        state: >
          {% set temps = [
            states('sensor.living_room_temperature')|float(0),
            states('sensor.bedroom_temperature')|float(0),
            states('sensor.kitchen_temperature')|float(0)
          ] %}
          {{ (temps | sum / temps | length) | round(1) }}
      
      - name: "House Occupancy Status"
        state: >
          {% set motion_sensors = [
            'binary_sensor.living_room_motion',
            'binary_sensor.bedroom_motion',
            'binary_sensor.hallway_motion'
          ] %}
          {% if motion_sensors | select('is_state', 'on') | list | count > 0 %}
            Active
          {% elif is_state('person.your_name', 'home') %}
            Present
          {% else %}
            Empty
          {% endif %}

# Input helpers for automation control
input_boolean:
  guest_mode:
    name: Guest Mode
    icon: mdi:account-multiple
  
  vacation_mode:
    name: Vacation Mode
    icon: mdi:beach
  
  sleep_mode:
    name: Sleep Mode
    icon: mdi:sleep

input_number:
  comfort_temperature:
    name: Comfort Temperature
    min: 18
    max: 24
    step: 0.5
    initial: 21
    unit_of_measurement: "°C"

input_select:
  house_mode:
    name: House Mode
    options:
      - Normal
      - Away
      - Sleep
      - Party
      - Movie
    initial: Normal

# Groups
group:
  all_indoor_lights:
    name: All Indoor Lights
    entities:
      - light.living_room
      - light.bedroom
      - light.kitchen
      - light.hallway
      - light.bathroom
  
  all_motion_sensors:
    name: All Motion Sensors
    entities:
      - binary_sensor.living_room_motion
      - binary_sensor.bedroom_motion
      - binary_sensor.hallway_motion
      - binary_sensor.bathroom_motion
  
  all_door_sensors:
    name: All Door Sensors
    entities:
      - binary_sensor.front_door
      - binary_sensor.back_door

# Recorder optimization
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

### 2. Complete Automation Suite

```yaml
# automations.yaml

# ============================================
# MORNING ROUTINE (6:00 - 9:00)
# ============================================

- id: 'morning_wakeup'
  alias: Morning Wakeup
  trigger:
    - platform: time
      at: '06:30:00'
  condition:
    - condition: state
      entity_id: input_boolean.vacation_mode
      state: 'off'
    - condition: time
      weekday:
        - mon
        - tue
        - wed
        - thu
        - fri
  action:
    - service: script.morning_routine

- id: 'morning_coffee'
  alias: Morning Coffee
  trigger:
    - platform: state
      entity_id: binary_sensor.kitchen_motion
      to: 'on'
  condition:
    - condition: time
      after: '06:00:00'
      before: '09:00:00'
    - condition: state
      entity_id: switch.coffee_maker
      state: 'off'
  action:
    - service: switch.turn_on
      target:
        entity_id: switch.coffee_maker
    - service: notify.mobile_app
      data:
        message: "Coffee is brewing ☕"

# ============================================
# DAYTIME AUTOMATIONS (9:00 - 17:00)
# ============================================

- id: 'nobody_home_day'
  alias: Nobody Home During Day
  trigger:
    - platform: state
      entity_id: zone.home
      to: '0'
      for:
        minutes: 5
  condition:
    - condition: time
      after: '09:00:00'
      before: '17:00:00'
  action:
    - service: script.away_mode

- id: 'window_open_climate_control'
  alias: Window Open Climate Control
  trigger:
    - platform: state
      entity_id:
        - binary_sensor.living_room_window
        - binary_sensor.bedroom_window
      to: 'on'
      for:
        minutes: 2
  action:
    - service: climate.turn_off
      target:
        area_id: >
          {% if trigger.entity_id == 'binary_sensor.living_room_window' %}
            living_room
          {% else %}
            bedroom
          {% endif %}
    - service: notify.mobile_app
      data:
        message: "Climate control turned off - window is open in {{ trigger.to_state.attributes.friendly_name }}"

# ============================================
# EVENING ROUTINE (17:00 - 22:00)
# ============================================

- id: 'welcome_home_evening'
  alias: Welcome Home Evening
  trigger:
    - platform: state
      entity_id: person.your_name
      to: 'home'
  condition:
    - condition: sun
      after: sunset
      after_offset: "-00:30:00"
  action:
    - service: scene.turn_on
      target:
        entity_id: scene.evening_relax
    - service: climate.set_temperature
      target:
        entity_id: climate.living_room
      data:
        temperature: "{{ states('input_number.comfort_temperature') }}"
    - service: notify.mobile_app
      data:
        message: "Welcome home! Evening scene activated."

- id: 'adaptive_evening_lighting'
  alias: Adaptive Evening Lighting
  trigger:
    - platform: sun
      event: sunset
      offset: "-00:30:00"
  condition:
    - condition: state
      entity_id: person.your_name
      state: 'home'
  action:
    - service: light.turn_on
      target:
        entity_id: group.all_indoor_lights
      data:
        brightness_pct: 70
        color_temp: 400
        transition: 300

- id: 'dinner_time_notification'
  alias: Dinner Time Notification
  trigger:
    - platform: time
      at: '18:00:00'
  condition:
    - condition: state
      entity_id: person.your_name
      state: 'home'
    - condition: time
      weekday:
        - mon
        - tue
        - wed
        - thu
        - fri
  action:
    - service: notify.mobile_app
      data:
        message: "It's dinner time! 🍽️"

# ============================================
# NIGHT ROUTINE (22:00 - 6:00)
# ============================================

- id: 'bedtime_reminder'
  alias: Bedtime Reminder
  trigger:
    - platform: time
      at: '22:30:00'
  condition:
    - condition: state
      entity_id: input_boolean.sleep_mode
      state: 'off'
  action:
    - service: notify.mobile_app
      data:
        message: "It's getting late. Time to prepare for bed."
    - service: light.turn_on
      target:
        entity_id: group.all_indoor_lights
      data:
        brightness_pct: 30
        transition: 300

- id: 'activate_sleep_mode'
  alias: Activate Sleep Mode
  trigger:
    - platform: state
      entity_id: binary_sensor.bedroom_motion
      to: 'off'
      for:
        minutes: 30
  condition:
    - condition: time
      after: '22:00:00'
      before: '06:00:00'
  action:
    - service: script.bedtime_routine

- id: 'night_motion_bathroom'
  alias: Night Motion Bathroom
  trigger:
    - platform: state
      entity_id: binary_sensor.bathroom_motion
      to: 'on'
  condition:
    - condition: time
      after: '22:00:00'
      before: '06:00:00'
    - condition: state
      entity_id: input_boolean.sleep_mode
      state: 'on'
  action:
    - service: light.turn_on
      target:
        entity_id: light.bathroom
      data:
        brightness_pct: 20
        rgb_color: [255, 100, 0]
    - wait_for_trigger:
        - platform: state
          entity_id: binary_sensor.bathroom_motion
          to: 'off'
      timeout: '00:10:00'
    - service: light.turn_off
      target:
        entity_id: light.bathroom
      data:
        transition: 3

# ============================================
# SECURITY AUTOMATIONS
# ============================================

- id: 'door_left_open_alert'
  alias: Door Left Open Alert
  trigger:
    - platform: state
      entity_id:
        - binary_sensor.front_door
        - binary_sensor.back_door
      to: 'on'
      for:
        minutes: 5
  action:
    - service: notify.mobile_app
      data:
        title: "⚠️ Security Alert"
        message: "{{ trigger.to_state.attributes.friendly_name }} has been open for 5 minutes!"
        data:
          priority: high

- id: 'motion_while_away'
  alias: Motion Detected While Away
  trigger:
    - platform: state
      entity_id: group.all_motion_sensors
      to: 'on'
  condition:
    - condition: state
      entity_id: zone.home
      state: '0'
    - condition: state
      entity_id: input_boolean.vacation_mode
      state: 'off'
  action:
    - service: notify.mobile_app
      data:
        title: "🚨 SECURITY ALERT"
        message: "Motion detected at home while you're away!"
        data:
          priority: high
          ttl: 0
    - service: light.turn_on
      target:
        entity_id: group.all_indoor_lights

# ============================================
# CLIMATE & ENERGY MANAGEMENT
# ============================================

- id: 'smart_heating_schedule'
  alias: Smart Heating Schedule
  trigger:
    - platform: time_pattern
      minutes: '/30'
  action:
    - choose:
        # Morning warmup
        - conditions:
            - condition: time
              after: '06:00:00'
              before: '09:00:00'
            - condition: state
              entity_id: person.your_name
              state: 'home'
          sequence:
            - service: climate.set_temperature
              target:
                entity_id: climate.living_room
              data:
                temperature: "{{ states('input_number.comfort_temperature') }}"
        
        # Away/Day mode
        - conditions:
            - condition: time
              after: '09:00:00'
              before: '17:00:00'
            - condition: state
              entity_id: zone.home
              state: '0'
          sequence:
            - service: climate.set_temperature
              target:
                entity_id: climate.living_room
              data:
                temperature: 18
        
        # Evening comfort
        - conditions:
            - condition: time
              after: '17:00:00'
              before: '22:00:00'
          sequence:
            - service: climate.set_temperature
              target:
                entity_id: climate.living_room
              data:
                temperature: 22
        
        # Night setback
        - conditions:
            - condition: time
              after: '22:00:00'
          sequence:
            - service: climate.set_temperature
              target:
                entity_id: climate.living_room
              data:
                temperature: 19

- id: 'high_temperature_alert'
  alias: High Temperature Alert
  trigger:
    - platform: numeric_state
      entity_id: sensor.living_room_temperature
      above: 26
      for:
        minutes: 15
  action:
    - service: notify.mobile_app
      data:
        message: "Temperature is high ({{ states('sensor.living_room_temperature') }}°C). Consider opening windows."

# ============================================
# DEVICE MONITORING
# ============================================

- id: 'low_battery_notification'
  alias: Low Battery Notification
  trigger:
    - platform: numeric_state
      entity_id:
        - sensor.motion_sensor_battery
        - sensor.door_sensor_battery
        - sensor.temperature_sensor_battery
      below: 20
  action:
    - service: notify.mobile_app
      data:
        title: "Low Battery Warning"
        message: "{{ trigger.to_state.attributes.friendly_name }} battery: {{ trigger.to_state.state }}%"

- id: 'system_temperature_warning'
  alias: System Temperature Warning
  trigger:
    - platform: numeric_state
      entity_id: sensor.processor_temperature
      above: 75
  action:
    - service: notify.mobile_app
      data:
        title: "System Alert"
        message: "Raspberry Pi temperature is {{ states('sensor.processor_temperature') }}°C"
```

### 3. Complete Scripts Suite

See the scripts.yaml file in the config/ directory, which includes:
- Morning routine
- Away mode
- Bedtime routine
- Movie mode
- Panic mode

### 4. Dashboard Configuration

See dashboards/main-dashboard.yaml for the complete Lovelace UI configuration.

## Setting Up This Example

### Step 1: Hardware Setup
1. Follow the [Hardware Setup Guide](hardware-setup.md)
2. Install all Zigbee devices as per [Zigbee Devices Guide](zigbee-devices.md)

### Step 2: Configuration
1. Copy configuration files to your Home Assistant config directory
2. Create and fill secrets.yaml with your information
3. Customize entity IDs to match your devices
4. Adjust times and temperatures to your preferences

### Step 3: Testing
1. Check configuration: `ha core check`
2. Restart Home Assistant
3. Test each automation individually
4. Use Developer Tools > Traces to debug

### Step 4: Customization
1. Add/remove automations as needed
2. Adjust trigger times
3. Modify conditions for your lifestyle
4. Add additional integrations

## Integration Points

This example demonstrates:

✅ **Hardware Integration**
- Raspberry Pi 5 system monitoring
- SSD storage optimization
- Zigbee coordinator management

✅ **Device Control**
- Motion sensor automation
- Temperature-based climate control
- Door/window monitoring
- Smart lighting

✅ **Time-Based Automation**
- Morning routines
- Evening scenes
- Night modes
- Weekly schedules

✅ **Presence Detection**
- Home/away automations
- Welcome home scenes
- Security alerts

✅ **Energy Management**
- Smart heating schedules
- Window-based climate control
- Away mode optimization

✅ **User Comfort**
- Adaptive lighting
- Climate control
- Notification system
- Scene management

## Extending This Example

### Add Voice Control
```yaml
# Enable Google Assistant or Alexa
google_assistant:
  project_id: !secret google_project_id
  service_account: !include SERVICE_ACCOUNT.json
  exposed_domains:
    - light
    - switch
    - climate
```

### Add Camera Integration
```yaml
camera:
  - platform: generic
    name: Front Door Camera
    still_image_url: !secret front_camera_url
    stream_source: !secret front_camera_stream
```

### Add Energy Monitoring
```yaml
sensor:
  - platform: template
    sensors:
      daily_energy_cost:
        value_template: >
          {{ (states('sensor.daily_energy')|float * 0.15) | round(2) }}
        unit_of_measurement: 'EUR'
```

## Troubleshooting

If you encounter issues:
1. Check [Troubleshooting Guide](troubleshooting.md)
2. Verify all entity IDs exist
3. Check automation traces
4. Review Home Assistant logs
5. Test components individually

## Performance Notes

This complete setup:
- Runs smoothly on Raspberry Pi 5 with 8GB RAM
- Database optimized with 7-day retention
- ~50 Zigbee devices supported
- Response time <100ms for most automations
- SSD provides excellent I/O performance

## Conclusion

This complete example provides a fully functional smart home setup that:
- Automates daily routines
- Enhances comfort and convenience
- Improves energy efficiency
- Provides security monitoring
- Scales easily with additional devices

Use this as a foundation and customize it to fit your specific needs!
