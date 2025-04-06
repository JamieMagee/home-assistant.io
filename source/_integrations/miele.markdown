---
title: Miele
description: Instructions on how to set up the Miele integration within Home Assistant.
ha_category:
  - Hub
  - Sensor
ha_iot_class: Cloud Push
ha_release: '2025.5.0'
ha_domain: miele
ha_codeowners:
  - '@astrandb'
ha_config_flow: true
ha_platforms:
  - sensor
ha_integration_type: integration
---

The Miele integration allows users to integrate their home appliances using the [official 3rd party API](https://www.miele.com/developer).

## Use cases

- Monitor the multiple sensors of the appliance and trigger automations based on these sensors.
- Start programs on your appliances from your dashboard.
- Monitor the program status of the appliances.
- Control the light of your appliances.
- Adjust the appliance settings.

{% note %}
Note that it depends on the appliance which of the features are supported.
{% endnote %}

## Supported devices

You can find information about supported devices on the [Miele website](https://www.miele.com/developer/capabilities.html).

## Prerequisites

1. Visit [https://www.miele.com/developer](https://www.miele.com/f/com/en/register_api.aspx) and sign up for a developer account.
2. Enter the email of your login for the original Miele app.

3. On success, you will be redirected to the **Applications** page. Select **Details** for your app. Make note of the client ID and secret - you will need it for the next step. Log out of the Home Connect developer portal.
4. In Home Assistant, find the Home Connect integration and launch it. You will be prompted to create an [Application Credential](https://www.home-assistant.io/integrations/application_credentials). You will need to provide a name (it's arbitrary) in addition to the Client ID and Secret from the previous step. Then, follow the steps in the UI to complete setup.

{% important %}

- To update the appliance programs list, you can reload the Home Connect integration when an appliance is turned on. If the re-initialization process is not triggered by reload, restart the Home Assistant when an appliance is turned on.
- After performing the steps above, [log out](https://developer.home-connect.com/user/logout) of your Home Connect Developer account. If you don't do this, the configuration steps below will fail during OAuth authentication with the message `“error”: “unauthorized_client”`.
- The provided Miele User Account email address **must** be all lowercase; otherwise, it will result in authentication failures.
- All changes in the developer portal take couple of minutes before the change is implemented.

{% endimportant %}

{% details "I have manually disabled My Home Assistant" %}

If you don't have [My Home Assistant](/integrations/my) on your installation,
you can use `<HOME_ASSISTANT_URL>/auth/external/callback` as the redirect URI
instead.

The `<HOME_ASSISTANT_URL>` must be the same as used during the configuration/
authentication process.

Internal examples: `http://192.168.0.2:8123/auth/external/callback`, `http://homeassistant.local:8123/auth/external/callback`."

{% enddetails %}

{% include integrations/config_flow.md %}

The integration configuration will ask for the *Client ID* and *Client Secret* created above. See [Application Credentials](/integrations/application_credentials) for more details.

## Supported functionality

{% note %}

- The entities availability depends on the appliance type, but the appliance might not support all the entities for its type.
- Some appliances don't report data while they are turned off, so corresponding entities will not appear in the Home Connect integration after loading until the appliances are turned on.
{% endnote %}

### Sensor

### Sensor

{% details "List of binary sensors" %}

- **Operation state**:
  - **Status**: Represents the current operation state of the device.
  - **Availability**: All the appliances with programs
{% enddetails %}

## Automation examples

Get started with these automation examples

### Send a notification when the appliance ends the program

{% details "Example YAML configuration" %}

{% raw %}

```yaml
alias: "Notify when program ends"
triggers:
  - trigger: state
    entity_id:
      - sensor.appliance_operation_state
    to: finished
actions:
  - service: notify.notify
    data:
      message: "The appliance has finished the program."
```

{% endraw %}
{% enddetails %}

### Start a program when electricity is cheap

Because electricity is typically cheaper at night, this automation will activate the silent mode when starting the program at night.

{% details "Example YAML configuration" %}

{% raw %}

```yaml
alias: "Start program when electricity is cheap"
triggers:
  - trigger: state
    entity_id: sensor.electricity_price
    to: "0.10"
conditions:
  - condition: state
    entity_id: sensor.diswasher_door
    state: closed
actions:
  - if:
      - condition: time
        after: '22:00:00'
        before: '06:00:00'
    then:
      - service: home_connect.set_program_and_options
        data:
          device_id: "your_device_id"
          affects_to: "active_program"
          program: "dishcare_dishwasher_program_eco_50"
          options:
            - key: "dishcare_dishwasher_option_silence_on_demand"
              value: true
    else:
      - service: home_connect.set_program_and_options
        data:
          device_id: "your_device_id"
          affects_to: "active_program"
          program: "dishcare_dishwasher_program_eco_50"
```

{% endraw %}
{% enddetails %}

## Data updates

This integration uses server-sent events from the Miele API to receive live updates from the appliances.
When the configuration entry is loaded or after a streaming error (for example after disconnection), the integration will request all data (such as appliance info, available commands, programs, settings, and status) for all appliances.
If a new appliance is added to the account, the integration will request data for the new appliance and expose the related entities automatically.

## Known limitations

- The Miele 3rd party API does not fully match the Miele app. Some programs, options, or settings available in the app may not be accessible or usable via the API.
- This integration supports only one integration entry, as the Miele 3rd party API does not allow for the unique identification of an account.

## Troubleshooting

### Unavailable entities for a device

#### Symptom: "The entities related to an appliance were available but no longer are"

After reloading the Home Connect integration, the entities related to an appliance that used to be available are no longer available.
Also, when downloading the diagnostics data from the device entry, the following data is obtained:

```json
{
  "data": {
    "connected": false,
    "status": {},
    "programs": null
  }
}
```

##### Description

Unavailable entities can have multiple causes:

- The appliance is turned off. When it is turned off, the appliance is disconnected and the API does not retrieve information about the appliance.
- The appliance is experiencing a network issue.
- The Miele API is experiencing issues.

##### Solution

To try to solve the above issues, follow these steps:

1. Turn on the appliance and reload the Home Connect integration.
2. If the appliance is turned on and the issue persists, check the network connection of the appliance and perform a soft reset on the appliance.
3. If the issue persists, check the connection of the appliance with the Miele API by checking it in the Miele app.
   1. Open the Miele app.
   2. Go to the appliance that is experiencing the issue.
   3. At the bottom of the screen, open the settings menu.
   4. Go to the **Network** section.
   5. Verify if the appliance is connected to the cloud:
      - If the line between the appliance and the cloud is red and with a red warning icon {% icon "mdi:alert-outline" %}, the appliance is not connected to the Home Connect API.
      - If the line between the appliance and the cloud is green, the appliance is connected to the cloud.
4. If everything is correct and the issue persists, contact Miele support.
   - [Miele service and contact](https://www.miele.com/)
   - [Miele developer Help & Support](https://www.miele.com/developer)

## Removing the integration

This integration follows standard integration removal. No extra steps are required.

{% include integrations/remove_device_service.md %}
