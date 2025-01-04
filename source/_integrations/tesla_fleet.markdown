---
title: Tesla Fleet
description: Instructions on how to integrate the Tesla Fleet API within Home Assistant.
ha_category:
  - Binary sensor
  - Button
  - Car
  - Climate
  - Cover
  - Device tracker
  - Lock
  - Media player
  - Number
  - Select
  - Sensor
  - Switch
ha_release: 2024.8
ha_iot_class: Cloud Polling
ha_config_flow: true
ha_codeowners:
  - '@Bre77'
ha_domain: tesla_fleet
ha_platforms:
  - binary_sensor
  - button
  - climate
  - cover
  - device_tracker
  - diagnostics
  - lock
  - media_player
  - number
  - select
  - sensor
  - switch
ha_integration_type: integration
---

The Tesla Fleet API {% term integration %} exposes various sensors from Tesla vehicles and energy sites using the [Tesla Fleet API](https://developer.tesla.com/).

## Prerequisites

You must have a [Tesla](https://tesla.com) account, and a [Developer Application](https://developer.tesla.com/en_US/dashboard) (change your locale if needed to wherever your account is based, using the globe icon at the top-right corner).

### Developer Application

You must [create your own application](https://developer.tesla.com/docs/fleet-api/getting-started/what-is-fleet-api#step-2-create-an-application) for the Tesla Fleet API and configure it as an [application credential](https://my.home-assistant.io/redirect/application_credentials).
When creating the application, you must set the redirect URL to `https://my.home-assistant.io/redirect/oauth`, but the other URLs can be set as desired. You must also complete both [step 3](https://developer.tesla.com/docs/fleet-api/getting-started/what-is-fleet-api#step-3-generate-a-public-private-key-pair) and [step 4](https://developer.tesla.com/docs/fleet-api/getting-started/what-is-fleet-api#step-4-call-the-register-endpoint) before the application will be able to make API calls. Step-by-step instructions can be found below on this page.

{% include integrations/config_flow.md %}

{% details "Setting up the Developer Application" %}

1. If you have not already, set up your [Tesla Developer account](https://developer.tesla.com/teslaaccount). Confirm that you have a verified email and multi-factor authentication set up.
2. Navigate to the [Developer dashboard](https://developer.tesla.com/en_US/dashboard).
3. Click '**Create New Application**'. This should launch a new page with the header '**Create Fleet API Application**'.
4. For simple integrations, under '**Registration Type**', select '**Just for me**'. If you're confident about your situation being a business setup instead, select '**For my business**'. Click '**Next**'.
5. At the '**Application Details**' step, use a name that is easy to refer to later, such as `ha-integration`. This will be needed later while configuring the integration. Click '**Next**'.
6. At the '**Client Details**' step, under '**Oauth Grant Type**', select '**Authorization Code and Machine-to-Machine**'.
   - Under '**Allowed Origin URL(s)**', enter `https://my.home-assistant.io/`. Feel free to add any other URLs here depending on your setup.
   - Under '**Allowed Redirect URI(s)**', enter `https://my.home-assistant.io/redirect/oauth`.
   - Click '**Next**'.
7. At the '**API & Scopes**' step, select the configurations you'd like to access.
   - At least one of `Vehicle Information` or `Energy Product Information` **must** be selected for the integration to function. It is recommended you select all scopes for full functionality.
   - If you're unsure, you can select only one of these two required scopes for now and update the scopes later from the Developer Dashboard. However, note that if the scopes are updated, you will need to reconfigure the integration fully (refer to the '**Integration is broken and needs to be reconfigured**' troubleshooting steps below).
   - Click '**Next**'.
8. At the '**Billing Details (Optional)**' step, enter your billing details if needed, then click '**Submit**'.
   - Tesla provides a $10 monthly credit for personal API usage, so your level of usage may be covered for free. Detailed pricing info for commands, data polling, and wake signals can be found at [developer.tesla.com](https://developer.tesla.com). Use these details to calculate your usage estimate if you rely heavily on this integration.
   - If unsure, you can click '**Skip & Submit**' for now and add the billing details later when your usage is close to the free threshold.

{% enddetails %}

{% details "Linking the Developer Application with Home Assistant" %}

1. Get your OAuth details by going to your [Developer dashboard](https://developer.tesla.com/en_US/dashboard), clicking '**View Details**' under the app you set up for Home Assistant integration, then clicking on the '**Credentials & APIs**' tab. Note the `Client ID` and `Client Secret` strings, these will be needed later.
2. In Home Assistant, the integration wizard should walk you through the default steps. If not already started, scroll above and click the '**ADD INTEGRATION TO MY**' button to start the integration wizard. The integration will ask you for all of the necessary integration configuration.
3. In the '**Add credentials**' step in the wizard, enter your Tesla Fleet developer application name (from step 5 in the previous '**Setting up the Developer Application**' section above), and the Oauth Client ID and Client Secret (from step 1 above). Click '**Submit**'.
4. At this step, you should be taken to the Tesla authentication page. You may be asked to re-enter your Tesla account login credentials.
5. At the confirmation page with the header '**Allow ha_integration access to your Tesla Account**' (the name is driven by what you set earlier), click the '**Select All**' button. This list of scopes is already limited to the specific scopes you chose for the application information earlier, so it is not necessary to review them again. Click '**Allow**'.
6. You should now see a Home Assistant page asking if you would like to '**Link account to Home Assistant?**'. Click '**Link account**'.
7. You should be all set!

{% enddetails %}


## Scopes

When connecting your Tesla account to Home Assistant, you **must** select at least one of the `Vehicle Information` or `Energy Product Information` scopes. It is recommended you select all scopes for full functionality. The `Vehicle Location` scope was added in Home Assistant 2024.1, so any authorizations performed on previous releases that want this scope will need to be [modified](https://accounts.tesla.com/en_au/account-settings/security?tab=tpty-apps).

## Pay per use
	
Previously, Tesla restricted this integration to a very modest rate limit. However, from January 2025, accounts in eligible countries will be charged for every API call. Here's what you need to know:

- Tesla provides a $10 credit per developer account per calendar month
- Every vehicle coordinator refresh, vehicle command, and wake up has a cost
- This credit only allows for a maximum of 5000 coordinator refreshes
- Energy product APIs are free to use at this time
- To go beyond the free credit, you must provide payment details to Tesla

For more details, please see [developer.tesla.com](https://developer.tesla.com).

You can view your current billing usage at any time by going to your [Developer Dashboard](https://developer.tesla.com/en_US/dashboard), clicking '**View Details**' under the app you set up for Home Assistant integration, then clicking on the '**Application Usage**' tab.

## Command signing

Certain vehicles, including all vehicles manufactured since late 2023, require vehicle commands to be signed with a private key. All actions on vehicle entities will fail with an error if this is required and the key has not been added to the vehicle.

You will need to use Tesla's [command line tools](https://github.com/teslamotors/vehicle-command/blob/main/README.md#installation-and-configuration) to generate a key pair and install the public key on your vehicle using Bluetooth.

```shell
tesla-keygen -key-file tesla_fleet.key create > tesla_fleet.pem
tesla-control -ble -key-file tesla_fleet.key -vin VINVINVINVIN -debug add-key-request tesla_fleet.pem owner cloud_key
```

Finally, copy `tesla_fleet.key` to your Home Assistant config directory and then reload the Tesla Fleet {% term integration %}.

{% note %}
If you receive a "BLE connection attempt failed" error, follow these steps:

1. Disable Bluetooth on your phone
2. Execute the `tesla-control` command
3. Re-enable Bluetooth after the command completes

This is necessary because the tool cannot establish a connection while another Bluetooth device is connected to the car.
{% endnote %}

## Entities

These are the entities available in the Tesla Fleet integration. Not all entities are enabled by default, and not all values are always available.

### Vehicles

| Domain         | Name                                       | Enabled |
| -------------- | ------------------------------------------ | ------- |
| Binary sensor  | Battery heater                             | No      |
| Binary sensor  | Cabin overheat protection actively cooling | No      |
| Binary sensor  | Charge cable                               | Yes     |
| Binary sensor  | Charger has multiple phases                | No      |
| Binary sensor  | Dashcam                                    | No      |
| Binary sensor  | Front driver door                          | Yes     |
| Binary sensor  | Front driver window                        | Yes     |
| Binary sensor  | Front passenger door                       | Yes     |
| Binary sensor  | Front passenger window                     | Yes     |
| Binary sensor  | Preconditioning enabled                    | No      |
| Binary sensor  | Preconditioning                            | No      |
| Binary sensor  | Rear driver door                           | Yes     |
| Binary sensor  | Rear driver window                         | Yes     |
| Binary sensor  | Rear passenger door                        | Yes     |
| Binary sensor  | Rear passenger window                      | Yes     |
| Binary sensor  | Scheduled charging pending                 | No      |
| Binary sensor  | Status                                     | Yes     |
| Binary sensor  | Tire pressure warning front left           | No      |
| Binary sensor  | Tire pressure warning front right          | No      |
| Binary sensor  | Tire pressure warning rear left            | No      |
| Binary sensor  | Tire pressure warning rear right           | No      |
| Binary sensor  | Trip charging                              | No      |
| Binary sensor  | User present                               | Yes     |
| Button         | Flash lights                               | Yes     |
| Button         | Homelink                                   | Yes     |
| Button         | Honk horn                                  | Yes     |
| Button         | Keyless driving                            | Yes     |
| Button         | Play fart                                  | Yes     |
| Button         | Wake                                       | Yes     |
| Climate        | Cabin overheat protection                  | No      |
| Climate        | Climate                                    | Yes     |
| Cover          | Charge port door                           | Yes     |
| Cover          | Frunk                                      | Yes     |
| Cover          | Sunroof                                    | No      |
| Cover          | Trunk                                      | Yes     |
| Cover          | Vent windows                               | Yes     |
| Device tracker | Location                                   | Yes     |
| Device tracker | Route                                      | Yes     |
| Lock           | Charge cable lock                          | Yes     |
| Lock           | Lock                                       | Yes     |
| Media player   | Media player                               | Yes     |
| Number         | Charge current                             | Yes     |
| Number         | Charge limit                               | Yes     |
| Select         | Seat heater front left                     | Yes     |
| Select         | Seat heater front right                    | Yes     |
| Select         | Seat heater rear center                    | No      |
| Select         | Seat heater rear left                      | No      |
| Select         | Seat heater rear right                     | No      |
| Select         | Seat heater third row left                 | No      |
| Select         | Seat heater third row right                | No      |
| Select         | Steering wheel heater                      | Yes     |
| Sensor         | Battery level                              | Yes     |
| Sensor         | Battery range                              | Yes     |
| Sensor         | Charge cable                               | No      |
| Sensor         | Charge energy added                        | Yes     |
| Sensor         | Charge rate                                | Yes     |
| Sensor         | Charger current                            | Yes     |
| Sensor         | Charger power                              | Yes     |
| Sensor         | Charger voltage                            | Yes     |
| Sensor         | Charging                                   | Yes     |
| Sensor         | Distance to arrival                        | Yes     |
| Sensor         | Driver temperature setting                 | No      |
| Sensor         | Estimate battery range                     | No      |
| Sensor         | Fast charger type                          | No      |
| Sensor         | Ideal battery range                        | No      |
| Sensor         | Inside temperature                         | Yes     |
| Sensor         | Odometer                                   | No      |
| Sensor         | Outside temperature                        | Yes     |
| Sensor         | Passenger temperature setting              | No      |
| Sensor         | Power                                      | No      |
| Sensor         | Shift state                                | No      |
| Sensor         | Speed                                      | No      |
| Sensor         | State of charge at arrival                 | No      |
| Sensor         | Time to arrival                            | Yes     |
| Sensor         | Time to full charge                        | Yes     |
| Sensor         | Tire pressure front left                   | No      |
| Sensor         | Tire pressure front right                  | No      |
| Sensor         | Tire pressure rear left                    | No      |
| Sensor         | Tire pressure rear right                   | No      |
| Sensor         | Traffic delay                              | No      |
| Sensor         | Usable battery level                       | No      |
| Switch         | Auto seat climate left                     | Yes     |
| Switch         | Auto seat climate right                    | Yes     |
| Switch         | Auto steering wheel heater                 | Yes     |
| Switch         | Charge                                     | Yes     |
| Switch         | Defrost                                    | Yes     |
| Switch         | Sentry mode                                | Yes     |

### Energy sites

| Domain        | Name                     | Enabled |
| ------------- | ------------------------ | ------- |
| Binary sensor | Backup capable           | Yes     |
| Binary sensor | Grid services active     | Yes     |
| Binary sensor | Grid services enabled    | Yes     |
| Binary sensor | Storm watch active       | Yes     |
| Number        | Backup reserve           | Yes     |
| Number        | Off grid reserve         | Yes     |
| Select        | Allow export             | Yes     |
| Select        | Operation mode           | Yes     |
| Sensor        | Battery power            | Yes     |
| Sensor        | Energy left              | Yes     |
| Sensor        | Generator power          | No      |
| Sensor        | Grid power               | Yes     |
| Sensor        | Grid services power      | Yes     |
| Sensor        | Grid status              | Yes     |
| Sensor        | Island status            | Yes     |
| Sensor        | Load power               | Yes     |
| Sensor        | Percentage charged       | Yes     |
| Sensor        | Solar power              | Yes     |
| Sensor        | Total pack energy        | No      |
| Sensor        | VPP backup reserve       | Yes     |
| Sensor        | Version                  | Yes     |
| Switch        | Allow charging from grid | Yes     |
| Switch        | Storm watch              | Yes     |

### Wall connector

| Domain | Name        | Enabled |
| ------ | ----------- | ------- |
| Sensor | Fault state | No      |
| Sensor | Power       | Yes     |
| Sensor | State       | Yes     |
| Sensor | Vehicle     | Yes     |

## Vehicle sleep

Constant API polling will prevent most Model S and Model X vehicles manufactured before 2021 from sleeping, so the integration will stop polling these vehicles for 15 minutes, after 15 minutes of inactivity. You can call the `homeassistant.update_entity` service to force polling the API, which will reset the timer.

## Energy dashboard

The Tesla Fleet API only provides power data for Powerwall and Solar products. This means they cannot be used on the energy dashboard directly.

Energy flows can be calculated from `Battery power` and `Grid power` sensors using a [Template Sensor](/integrations/template/) to separate the positive and negative values into positive import and export values.
The `Load power`, `Solar power`, and the templated sensors can then use a [Riemann Sum](/integrations/integration/) to convert their instant power (kW) values into cumulative energy values (kWh),
which then can be used within the energy dashboard.

## Troubleshooting

- **Integration is broken or needs to be reconfigured**
  1. Ensure that you have a Tesla developer application ready for usage (refer to the instructions in the '**Setting up the Developer Application**' section above).
  2. Go to your Tesla Fleet integration page in Home Assistant, then click the '**RECONFIGURE**' button to bring the integration wizard up.
     - If the '**RECONFIGURE**' button is not visible, clear any Application Credentials related to Tesla Fleet from your Application Credentials page (can be found at `http://homeassistant.local:PORT/config/application_credentials`), then restart Home Assistant. After the restart, navigate to the Tesla Fleet integration page and the '**RECONFIGURE**' button should be visible.
  3. Follow the steps in the '**Linking the Developer Application with Home Assistant**' section above.

- **Integration no longer works after the January 2025 API pricing updates**
  1. Refer to the '**Integration is broken**' troubleshooting steps above.

- **Integration shows errors even after successfully authorizing with the Tesla developer app**
  1. The error log should usually help communicate if a specific piece in the integration is missing.
  2. Sometimes a simple restart of Home Assistant should help it re-establish the connection with Tesla's API. You may be asked to re-authenticate in some cases.
