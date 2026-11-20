# hubitatADC

Hubitat Alarm.com integration — read-only alarm status monitoring with optional panel control.

Originally developed by Jeffrey M. Pierce. Continued development and modifications by [rjl77](https://github.com/rjl77/hubitatADC).

## Overview

This integration connects a Hubitat Elevation hub to an Alarm.com security system.

It periodically polls Alarm.com to synchronize the panel's current armed/disarmed state with Hubitat devices.

**Alarm control is disabled by default.** Users can explicitly enable arm/disarm commands in the app settings.

### Devices

The app creates four child devices:

| Device | Purpose | Default behavior |
|---|---|---|
| ADC Armed Status | Binary armed/disarmed indicator | Read-only |
| ADC Disarm | Disarm control and status | Status only |
| ADC Arm Stay | Arm Stay control and status | Status only |
| ADC Arm Away | Arm Away control and status | Status only |

All four devices remain available even when alarm controls are disabled.

### Binary status indicator

The `ADC Armed Status` device provides a standard Hubitat Switch capability:

| Switch state | Alarm state |
|---|---|
| `on` | Armed Stay or Armed Away |
| `off` | Disarmed |

The device is read-only. Its `on()` and `off()` commands do not change the alarm panel.

Additional attributes provide more detailed information:

| Attribute | Description |
|---|---|
| `alarmMode` | `disarmed`, `stay`, `away`, or `unknown` |
| `statusValid` | `true` when the most recent poll confirmed a recognized alarm state; otherwise `false` |
| `lastSuccessfulPoll` | Timestamp of the most recent successful status poll |

If a poll fails, the last known switch state is preserved, but `statusValid` becomes `false` and `alarmMode` becomes `unknown`.

**Important:** Automations requiring a verified alarm state should check `statusValid` as well as the switch state. A preserved switch value may be stale.

## Installation

### 1. Install the drivers

In Hubitat, navigate to **Drivers Code → New Driver**.

Install both driver files from this repository:

- `adc-control-driver.groovy` — existing Alarm.com Panel Switch driver
- `adc-status-driver.groovy` — new read-only Armed Status driver

Save each driver.

### 2. Install the app

Navigate to **Apps Code → New App**.

Paste the contents of `adc-manager-app.groovy` and click **Save**.

### 3. Configure Alarm.com Manager

Navigate to **Apps → Add User App → Alarm.com Manager**.

Enter your Alarm.com credentials and configure the polling interval.

Recommended settings:

- **Encrypt Password:** Enabled
- **Enable alarm control switches:** Disabled
- **Enable debugging:** Disabled

Click **Done**.

The app attempts to authenticate with Alarm.com, discover the panel ID, and create the four child devices.

### 4. Verify operation

Open **Devices → ADC Armed Status**.

Check that the device reports:

- The correct `switch` state.
- The correct `alarmMode`.
- `statusValid = true`.
- A recent `lastSuccessfulPoll` timestamp.

The **Refresh** command requests an immediate status update.

## Optional alarm control

The existing three switches can arm or disarm the panel when **Enable alarm control switches** is explicitly enabled in Alarm.com Manager.

When this setting is disabled:

- The three devices remain visible.
- Their states continue to reflect the actual alarm panel.
- Arm/disarm commands sent through the switches are ignored.

When enabled, the original switch behavior is available:

- `ADC Disarm` — disarm the panel.
- `ADC Arm Stay` — arm in Stay mode.
- `ADC Arm Away` — arm in Away mode.

The app also provides preferences for force bypass, silent arming, no entry delay, and behavior when the Disarm switch is turned off.

**Security note:** Only enable control commands if you intend to allow Hubitat devices, dashboards, and automations to change the alarm state.

## Polling and status reliability

The integration polls Alarm.com at the interval selected in the app settings.

A successful poll updates the child devices to match the actual panel state.

If polling fails or returns an unrecognized state:

- `ADC Armed Status` retains its last known `on`/`off` value.
- `statusValid` becomes `false`.
- `alarmMode` becomes `unknown`.
- `lastSuccessfulPoll` retains its previous timestamp.

A later successful poll restores valid status.

The current implementation does not independently detect when scheduled polling stops running entirely. A future watchdog may address this.

## Password and account security

Password encryption is recommended. When enabled, the app stores an encrypted password in Hubitat and clears the password input field.

When updating preferences, leave the password field blank to retain the existing encrypted value.

Using a separate Alarm.com service account with the minimum permissions needed is recommended where supported.

Authentication credentials, cookies, and request headers should not be written to debug logs.

**Note:** This integration communicates with Alarm.com through web endpoints and may be affected by changes to Alarm.com's authentication or API behavior.

## Updating an existing installation

Install the new status driver before updating the manager app.

Update the existing control driver and manager app without changing their Hubitat driver/app definitions or namespaces.

After saving the updated code, open the installed Alarm.com Manager app and click **Done** to apply the new settings and create the status indicator.

Existing control devices are preserved.

**Important:** Alarm controls now default to disabled. Users who previously relied on Hubitat to arm or disarm their systems must explicitly enable them.

## To Do

- Add stale-status detection/watchdog for interrupted polling.
- Improve authentication and polling failure handling.
- Investigate support for multiple accounts or panels.
- Review additional Alarm.com API compatibility issues.

## Change History

### Fork development — 2026-11-19

- Added a read-only binary armed/disarmed status indicator.
- Added detailed alarm mode and status validity attributes.
- Added a last-successful-poll timestamp.
- Made alarm control optional and disabled by default.
- Improved synchronization of existing control switches.
- Removed sensitive authentication and configuration logging.
- Disabled debug logging by default.
- Renamed the original driver source file for clarity.

### v1.1.2 — Original project

- Added additional polling options.
- Fixed polling default value bug.

### v1.1.0 — Original project

- Added password encryption option.
- Fixed polling bug that did not properly unschedule the previous polling preference.
- Code cleanup and additional comments.

## License

Licensed under the Apache License, Version 2.0.

Original work copyright 2020 Jeffrey M. Pierce.

Fork modifications copyright 2026 rjl77.

See the repository's license file for details.
