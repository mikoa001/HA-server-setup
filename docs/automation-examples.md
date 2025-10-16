# Automation Examples

This document provides detailed examples of useful Home Assistant automations for your smart home.

## Table of Contents

1. [Presence Detection](#presence-detection)
2. [Lighting Automations](#lighting-automations)
3. [Climate Control](#climate-control)
4. [Security and Alerts](#security-and-alerts)
5. [Energy Saving](#energy-saving)
6. [Convenience Automations](#convenience-automations)

## Presence Detection

### Welcome Home Automation

Automatically activate lights and adjust climate when you arrive home.

```yaml
- id: 'welcome_home_enhanced'
  alias: Enhanced Welcome Home
  trigger:
    - platform: state
      entity_id: person.your_name
      to: 'home'
  condition:
    - condition: sun
      after: sunset
  action:
    - service: scene.turn_on
      target:
        entity_id: scene.evening_relax
    - service: climate.set_temperature
      target:
        entity_id: climate.living_room
      data:
        temperature: 22
    - service: notify.mobile_app
      data:
        message: "Welcome home! Climate and lights adjusted."
```

### Goodbye Routine

Turn off devices when everyone leaves.

```yaml
- id: 'goodbye_routine'
  alias: Goodbye Routine
  trigger:
    - platform: state
      entity_id: zone.home
      to: '0'
  action:
    - service: light.turn_off
      target:
        entity_id: all
    - service: climate.set_preset_mode
      target:
        entity_id: all
      data:
        preset_mode: 'away'
    - service: lock.lock
      target:
        entity_id: all
```

## Lighting Automations

### Adaptive Lighting

Adjust light color temperature based on time of day.

```yaml
- id: 'adaptive_lighting'
  alias: Adaptive Lighting
  trigger:
    - platform: time_pattern
      minutes: '/30'
    - platform: state
      entity_id: sun.sun
  action:
    - choose:
        # Morning: Cool white
        - conditions:
            - condition: time
              after: '06:00'
              before: '09:00'
          sequence:
            - service: light.turn_on
              target:
                area_id: living_room
              data:
                color_temp: 250
                brightness_pct: 80
        # Day: Bright cool
        - conditions:
            - condition: time
              after: '09:00'
              before: '18:00'
          sequence:
            - service: light.turn_on
              target:
                area_id: living_room
              data:
                color_temp: 300
                brightness_pct: 100
        # Evening: Warm white
        - conditions:
            - condition: time
              after: '18:00'
              before: '22:00'
          sequence:
            - service: light.turn_on
              target:
                area_id: living_room
              data:
                color_temp: 450
                brightness_pct: 60
        # Night: Very warm, dim
        - conditions:
            - condition: time
              after: '22:00'
          sequence:
            - service: light.turn_on
              target:
                area_id: living_room
              data:
                color_temp: 500
                brightness_pct: 20
```

### Motion-Activated Night Light

Gentle lighting during night hours.

```yaml
- id: 'night_motion_light'
  alias: Night Motion Light
  trigger:
    - platform: state
      entity_id: binary_sensor.hallway_motion
      to: 'on'
  condition:
    - condition: time
      after: '22:00'
      before: '06:00'
    - condition: state
      entity_id: light.hallway
      state: 'off'
  action:
    - service: light.turn_on
      target:
        entity_id: light.hallway
      data:
        brightness_pct: 15
        rgb_color: [255, 100, 0]
    - delay: '00:02:00'
    - service: light.turn_off
      target:
        entity_id: light.hallway
      data:
        transition: 5
```

## Climate Control

### Smart Heating Schedule

Adjust heating based on presence and schedule.

```yaml
- id: 'smart_heating'
  alias: Smart Heating Schedule
  trigger:
    - platform: time
      at: '06:00:00'
      id: morning
    - platform: time
      at: '08:00:00'
      id: day
    - platform: time
      at: '17:00:00'
      id: evening
    - platform: time
      at: '23:00:00'
      id: night
  action:
    - choose:
        - conditions:
            - condition: trigger
              id: morning
          sequence:
            - service: climate.set_temperature
              target:
                entity_id: climate.living_room
              data:
                temperature: 21
        - conditions:
            - condition: trigger
              id: day
            - condition: state
              entity_id: zone.home
              state: '0'
          sequence:
            - service: climate.set_temperature
              target:
                entity_id: climate.living_room
              data:
                temperature: 18
        - conditions:
            - condition: trigger
              id: evening
          sequence:
            - service: climate.set_temperature
              target:
                entity_id: climate.living_room
              data:
                temperature: 22
        - conditions:
            - condition: trigger
              id: night
          sequence:
            - service: climate.set_temperature
              target:
                entity_id: climate.living_room
              data:
                temperature: 19
```

### Window Open - Heating Off

Save energy by turning off heating when window is open.

```yaml
- id: 'window_open_heating_off'
  alias: Window Open - Turn Off Heating
  trigger:
    - platform: state
      entity_id: binary_sensor.bedroom_window
      to: 'on'
      for:
        minutes: 2
  action:
    - service: climate.turn_off
      target:
        entity_id: climate.bedroom
    - service: notify.mobile_app
      data:
        message: "Bedroom heating turned off - window is open"
```

## Security and Alerts

### Door Left Open Alert

Alert if door is left open for too long.

```yaml
- id: 'door_open_alert'
  alias: Door Left Open Alert
  trigger:
    - platform: state
      entity_id: binary_sensor.front_door
      to: 'on'
      for:
        minutes: 5
  action:
    - service: notify.mobile_app
      data:
        title: "Security Alert"
        message: "Front door has been open for 5 minutes"
        data:
          priority: high
    - service: light.turn_on
      target:
        entity_id: light.entrance
      data:
        flash: long
```

### Unusual Activity Detection

Detect motion when nobody should be home.

```yaml
- id: 'unusual_activity'
  alias: Unusual Activity Detection
  trigger:
    - platform: state
      entity_id: binary_sensor.living_room_motion
      to: 'on'
  condition:
    - condition: state
      entity_id: zone.home
      state: '0'
    - condition: time
      after: '20:00'
      before: '06:00'
  action:
    - service: notify.mobile_app
      data:
        title: "⚠️ Security Alert"
        message: "Motion detected at home while you're away!"
        data:
          priority: high
          ttl: 0
    - service: light.turn_on
      target:
        entity_id: all
    - service: camera.snapshot
      target:
        entity_id: camera.living_room
      data:
        filename: '/config/www/snapshots/alert_{{ now().strftime("%Y%m%d_%H%M%S") }}.jpg'
```

## Energy Saving

### Standby Power Monitor

Alert about devices consuming standby power.

```yaml
- id: 'standby_power_alert'
  alias: Standby Power Alert
  trigger:
    - platform: numeric_state
      entity_id: sensor.tv_power
      above: 5
      for:
        hours: 2
  condition:
    - condition: state
      entity_id: media_player.tv
      state: 'off'
  action:
    - service: notify.mobile_app
      data:
        message: "TV is consuming {{ states('sensor.tv_power') }}W in standby mode"
```

### Auto-Off Forgotten Devices

Turn off devices that have been on too long.

```yaml
- id: 'auto_off_forgotten'
  alias: Auto-Off Forgotten Devices
  trigger:
    - platform: state
      entity_id: light.bathroom
      to: 'on'
      for:
        hours: 2
  action:
    - service: light.turn_off
      target:
        entity_id: light.bathroom
    - service: notify.mobile_app
      data:
        message: "Bathroom light was on for 2 hours - automatically turned off"
```

## Convenience Automations

### Morning Coffee Routine

Start coffee maker when you wake up.

```yaml
- id: 'morning_coffee'
  alias: Morning Coffee
  trigger:
    - platform: state
      entity_id: binary_sensor.bedroom_motion
      to: 'on'
  condition:
    - condition: time
      after: '06:00'
      before: '09:00'
      weekday:
        - mon
        - tue
        - wed
        - thu
        - fri
    - condition: state
      entity_id: input_boolean.coffee_made_today
      state: 'off'
  action:
    - service: switch.turn_on
      target:
        entity_id: switch.coffee_maker
    - service: input_boolean.turn_on
      target:
        entity_id: input_boolean.coffee_made_today
    - service: notify.mobile_app
      data:
        message: "Good morning! Coffee is brewing ☕"
```

### Bedtime Reminder

Remind to go to bed if lights are still on late.

```yaml
- id: 'bedtime_reminder'
  alias: Bedtime Reminder
  trigger:
    - platform: time
      at: '23:30:00'
  condition:
    - condition: state
      entity_id: light.living_room
      state: 'on'
  action:
    - service: notify.mobile_app
      data:
        message: "It's getting late. Consider starting your bedtime routine."
    - service: light.turn_on
      target:
        entity_id: light.living_room
      data:
        brightness_pct: 30
        transition: 60
```

### Rainy Day Automation

Close blinds and adjust lights when it rains.

```yaml
- id: 'rainy_day'
  alias: Rainy Day Automation
  trigger:
    - platform: state
      entity_id: weather.home
      attribute: condition
      to: 'rainy'
  action:
    - service: cover.close_cover
      target:
        entity_id: all
    - service: light.turn_on
      target:
        area_id: living_room
      data:
        brightness_pct: 80
        color_temp: 400
```

## Tips for Creating Effective Automations

1. **Use Conditions Wisely:** Prevent unwanted triggers with proper conditions
2. **Add Delays:** Use delays to prevent rapid on/off cycling
3. **Test Thoroughly:** Test each automation in different scenarios
4. **Use Input Booleans:** Create manual override switches
5. **Log Actions:** Use notify service to track automation activities
6. **Start Simple:** Begin with basic automations and add complexity gradually
7. **Document:** Add descriptions to remember what each automation does

## Debugging Automations

Enable automation debugging:

```yaml
logger:
  default: warning
  logs:
    homeassistant.components.automation: debug
```

View automation traces in Developer Tools → Traces to see execution flow.
