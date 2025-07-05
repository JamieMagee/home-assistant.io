---
title: Proliphix
description: Instructions on how to integrate Proliphix thermostats within Home Assistant.
ha_category:
  - Climate
ha_release: 0.11
ha_iot_class: Local Polling
ha_config_flow: true
ha_domain: proliphix
ha_platforms:
  - climate
ha_integration_type: integration
ha_quality_scale: legacy
---

The Proliphix {% term integration %} allows you to control [Proliphix](https://www.proliphix.com/) thermostats from Home Assistant.

## Use cases

The Proliphix integration brings your network-connected thermostat into Home Assistant, enabling intelligent climate control for your home. Here's how you can benefit from this integration:

- Energy-efficient scheduling: Create smart heating and cooling schedules that adapt to your lifestyle, reducing energy consumption when you're away or asleep.
- Presence-based climate control: Automatically adjust temperatures based on who's home using Home Assistant's person tracking capabilities.
- Smart integrations: Connect your thermostat with other smart home devices like:
  - Window sensors to turn off HVAC when windows are open.
  - Weather forecasts to preemptively adjust heating/cooling.
  - Energy pricing to optimize usage during off-peak hours.
- Remote monitoring and control: Check and adjust your home's temperature from anywhere using the Home Assistant mobile app.
- Historical tracking: Monitor temperature trends and HVAC runtime to optimize your energy usage.
- Voice control: Use voice assistants to control your thermostat through Home Assistant.

## Supported devices

The following Proliphix network thermostats are supported by this integration:

- **NT10e**: Network thermostat with Ethernet connectivity (tested and confirmed)
- **NT20e**: May work but not officially tested

All Proliphix NT (Network Thermostat) series devices that support the HTTP API should work with this integration. If you have a different model that works, please consider contributing to the documentation.

## Prerequisites

Before setting up the Proliphix integration, ensure you have:

- Your Proliphix thermostat connected to your home network via Ethernet.
- The IP address of your thermostat (you can find this in your router's device list or on the thermostat's display).
- The username and password for your thermostat's web interface.
- Network connectivity between Home Assistant and your thermostat (no firewall blocking communication).

{% tip %}
It's recommended to assign a static IP address to your Proliphix thermostat in your router settings to prevent the IP from changing.
{% endtip %}

{% include integrations/config_flow.md %}

## Configuration parameters

{% configuration_basic %}
Host:
  description: The IP address of your Proliphix thermostat on your network, for example, 192.168.1.32.
Username:
  description: The username used to log in to the thermostat.
Password:
  description: The password used to log in to the thermostat.
{% endconfiguration_basic %}

The Proliphix NT Thermostat series are Ethernet connected thermostats. They have a local HTTP interface that is based on get/set
of OID values. A complete collection of the API is available in this [API documentation](https://github.com/sdague/thermostat.rb/blob/master/docs/PDP_API_R1_11.pdf).

## Supported functionality

### Climate

The integration creates a climate entity for your Proliphix thermostat with the following features:

- Current temperature: Shows the current room temperature as measured by the thermostat.
- Target temperature: Displays and allows you to set the desired temperature.
- HVAC modes: Supports switching between heating, cooling, and off modes.
- Operating state: Shows whether the system is currently idle, heating, or cooling.

The climate entity allows you to control your thermostat directly from Home Assistant, including setting target temperatures and changing operating modes.

## Data updates

This integration uses local {% term polling %} to update the thermostat data. The integration polls the Proliphix thermostat every minute to retrieve the latest temperature readings and thermostat status.

## Actions

This integration does not provide additional actions. All actions available for this integration are provided by their respective entities.

## Examples

The following examples show how to use the Proliphix integration in Home Assistant automations to create a more energy-efficient home.

Feel free to contribute more examples to this documentation ❤️. Have you created useful automations with your Proliphix thermostat? Consider sharing them to help other users!

### Lower temperature when nobody is home

This automation automatically lowers the thermostat setpoint when everyone leaves home and restores it when someone returns.

```yaml
automation:
  - alias: "Lower thermostat when away"
    triggers:
      - trigger: state
        entity_id: zone.home
        to: "0"

    actions:
      - action: climate.set_temperature
        target:
          entity_id: climate.proliphix
        data:
          temperature: 62

  - alias: "Restore thermostat when home"
    triggers:
      - trigger: state
        entity_id: zone.home
        from: "0"

    actions:
      - action: climate.set_temperature
        target:
          entity_id: climate.proliphix
        data:
          temperature: 72
```

### Schedule nighttime temperature

This example creates a schedule to lower the temperature at night for better sleep and energy savings.

```yaml
automation:
  - alias: "Nighttime temperature setback"
    triggers:
      - trigger: time
        at: "22:00:00"

    actions:
      - action: climate.set_temperature
        target:
          entity_id: climate.proliphix
        data:
          temperature: 65

  - alias: "Morning temperature restore"
    triggers:
      - trigger: time
        at: "06:00:00"

    actions:
      - action: climate.set_temperature
        target:
          entity_id: climate.proliphix
        data:
          temperature: 70
```

## Known limitations

- No fan control: The integration currently does not support fan mode control (auto/on/off).

## Troubleshooting

If you're experiencing issues with your Proliphix integration, try these troubleshooting steps:

### The integration won't connect to my thermostat

1. Verify network connectivity: Ensure you can access the thermostat's web interface directly using a web browser at `http://[IP_ADDRESS]`.
2. Check credentials: Confirm your username and password are correct by logging into the thermostat's web interface.
3. Firewall settings: Ensure no firewall rules are blocking communication between Home Assistant and the thermostat.
4. IP address changes: If using DHCP, the thermostat's IP may have changed. Check your router for the current IP address.

## Removing the integration

This integration follows standard integration removal. No extra steps are required.

{% include integrations/remove_device_service.md %}
