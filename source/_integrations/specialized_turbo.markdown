---
title: Specialized Turbo
description: Read telemetry from Specialized Turbo e-bikes over Bluetooth Low Energy in Home Assistant.
ha_category:
  - Sensor
ha_release: "2026.4"
ha_iot_class: Local Push
ha_config_flow: true
ha_codeowners:
  - '@JamieMagee'
ha_domain: specialized_turbo
ha_integration_type: device
ha_quality_scale: bronze
ha_platforms:
  - sensor
ha_bluetooth: true
related:
  - docs: /docs/configuration/troubleshooting/#debug-logs-and-diagnostics
    title: Debug logs and diagnostics
  - docs: /integrations/bluetooth/
    title: Bluetooth
---

The **Specialized Turbo** {% term integration %} connects Home Assistant to [Specialized](https://www.specialized.com/) Turbo e-bikes over Bluetooth Low Energy. It reads real-time telemetry straight from the bike — battery level, speed, power output, cadence, motor temperature, odometer, and more — giving you the same data the Specialized Mission Control app sees.

If you ride a Specialized Turbo Vado, Levo, Creo, Como, or Tero, you can track battery health over time, log rides through Home Assistant's history, or build automations that fire when your bike finishes charging.

## Supported devices

This integration works with Specialized Turbo e-bikes that have the Gen 2 BLE module (the "TURBOHMI2017" protocol), which includes most 2017 and newer models with a <abbr title="Turbo Connect Unit">TCU</abbr> display:

- Turbo Vado (all SL and full-power variants)
- Turbo Levo (all SL and full-power variants)
- Turbo Creo (all SL and full-power variants)
- Turbo Como (SL and full-power)
- Turbo Tero

If your bike broadcasts a "TURBOHMI" Bluetooth advertisement when powered on, it should work.

## Unsupported devices

- Older Specialized bikes without a TCU display
- Non-Turbo Specialized bikes (road, mountain, gravel without a motor)
- Third-party e-bikes

## Prerequisites

Before setting up this integration, make sure:

1. Your bike is powered on and awake (pedal or press the power button — the TCU screen should be lit).
2. Home Assistant has access to a [Bluetooth adapter](/integrations/bluetooth/). If the bike is out of range, you can use an [ESPHome Bluetooth proxy](https://esphome.io/projects/?type=bluetooth) placed in your garage or shed.
3. No other app (like Specialized Mission Control) is actively connected to the bike over Bluetooth. Only one BLE client can connect at a time.

{% include integrations/config_flow.md %}

The bike is usually discovered automatically. If it doesn't appear, make sure it's powered on and within Bluetooth range, then try adding it manually through {% my integrations title="**Settings** > **Devices & services**" %}.

{% configuration_basic %}
Pairing PIN:
  description: "The PIN displayed on the bike's TCU screen during Bluetooth pairing. Leave empty if your bike does not require a PIN."
{% endconfiguration_basic %}

## Supported functionality

### Entities

The **Specialized Turbo** integration provides the following sensor entities.

#### Sensors

- **Battery**
  - **Description**: Current battery charge level, in percent.

- **Battery capacity**
  - **Description**: Total battery capacity in watt-hours.

- **Battery remaining**
  - **Description**: Remaining battery energy in watt-hours.

- **Battery health**
  - **Description**: Battery health percentage. Useful for tracking degradation over time.
  - **Category**: Diagnostic

- **Battery temperature**
  - **Description**: Battery temperature in °C.
  - **Category**: Diagnostic

- **Charge cycles**
  - **Description**: Total number of charge cycles the battery has completed.
  - **Category**: Diagnostic

- **Battery voltage**
  - **Description**: Current battery voltage.
  - **Category**: Diagnostic

- **Battery current**
  - **Description**: Current draw from the battery in amps.
  - **Category**: Diagnostic

- **Speed**
  - **Description**: Current speed in km/h.

- **Rider power**
  - **Description**: Power you're putting through the pedals, in watts.

- **Motor power**
  - **Description**: Power the electric motor is adding, in watts.

- **Cadence**
  - **Description**: Pedaling cadence in RPM.

- **Odometer**
  - **Description**: Total distance the bike has traveled, in kilometers.

- **Motor temperature**
  - **Description**: Motor temperature in °C.
  - **Category**: Diagnostic

- **Assist level**
  - **Description**: Current assist mode — Off, Eco, Trail, or Turbo.

- **ECO assist**
  - **Description**: Assist percentage configured for ECO mode.
  - **Category**: Diagnostic
  - **Remarks**: Disabled by default.

- **Trail assist**
  - **Description**: Assist percentage configured for Trail mode.
  - **Category**: Diagnostic
  - **Remarks**: Disabled by default.

- **Turbo assist**
  - **Description**: Assist percentage configured for Turbo mode.
  - **Category**: Diagnostic
  - **Remarks**: Disabled by default.

## Examples

### Notify when the battery is fully charged

```yaml
- alias: "Bike battery full"
  triggers:
    - trigger: numeric_state
      entity_id: sensor.specialized_turbo_battery
      above: 99
  actions:
    - action: notify.mobile_app
      data:
        message: "Your bike is fully charged!"
```

### Warn when battery health drops

{% raw %}
```yaml
- alias: "Bike battery health warning"
  triggers:
    - trigger: numeric_state
      entity_id: sensor.specialized_turbo_battery_health
      below: 80
  actions:
    - action: notify.mobile_app
      data:
        message: >
          Bike battery health is at
          {{ states(
            'sensor.specialized_turbo_battery_health'
          ) }}%.
          Consider scheduling a service.
```
{% endraw %}

## Data updates

The integration uses a push-based approach. It connects to the bike over Bluetooth, subscribes to GATT notifications on the telemetry characteristic, and receives updates whenever the bike broadcasts new data. No polling is performed. If the Bluetooth connection drops, the integration reconnects automatically the next time it sees the bike's advertisement.

## Known limitations

- **Bluetooth range**: The bike needs to be within about 5–10 meters of a Bluetooth adapter. For a garage or shed, an [ESPHome Bluetooth proxy](https://esphome.io/projects/?type=bluetooth) works well.
- **One connection at a time**: Only one BLE client can connect to the bike simultaneously. If the Specialized Mission Control app is connected, Home Assistant can't connect, and vice versa.
- **Read-only**: The integration reads telemetry only. It cannot change assist levels, adjust settings, or send commands to the bike.
- **Sleep mode**: When the bike enters sleep mode, Bluetooth advertisements stop and the connection drops. Data resumes when you wake the bike up.
- **Secondary battery**: Range-extender battery data is parsed internally but not exposed as sensor entities.

## Troubleshooting

### Bike not discovered

If your bike doesn't appear during setup:

1. Make sure the bike is powered on and awake — pedal briefly or press the power button so the TCU screen lights up.
2. Check that your Bluetooth adapter is working in {% my integrations title="**Settings** > **Devices & services**" %}.
3. If you're using an ESPHome Bluetooth proxy, confirm that `active: true` is set in the proxy's configuration.
4. Move the bike closer to the Bluetooth adapter. BLE range is typically 5–10 meters, less through walls.

### Sensors show "Unavailable"

This usually means the bike is out of range or asleep:

1. Wake the bike by pedaling or pressing the power button.
2. Check whether the Specialized Mission Control app (or another BLE client) has an active connection — only one client can connect at a time.
3. Try restarting the integration from {% my integrations title="**Settings** > **Devices & services**" %}.

### Pairing PIN not accepted

The PIN is displayed on the bike's TCU screen when it enters pairing mode. If you don't see a PIN prompt on the bike, it may already be paired with another device. Reset the BLE pairing on the bike and try again.

Some Bluetooth backends don't support programmatic PIN entry. If that happens, pair the bike through your operating system's Bluetooth settings first, then set up the integration without entering a PIN.

## Removing the integration

This integration follows standard integration removal.

{% include integrations/remove_device_service.md %}
