# mod_gsmopen user notes

This is a minimal, code-derived summary for users. It mirrors the current behavior in this repository.

## Overview
mod_gsmopen is a FreeSWITCH endpoint module for GSM modems. It provides:
- An endpoint for calls via `gsmopen/<interface>/<number>`.
- API commands for AT control, status dump, SMS, and USSD.
- SMS/USSD inbound events via `mod_sms` (chat events).

## Call Types / Usage Modes
1. Outbound calls (endpoint)
   - Dialstring format: `gsmopen/<interface>/<number>`
   - `<interface>` is the interface name from `gsmopen.conf`
   - Special interface names: `ANY` or `RR` select the next available interface (round-robin)

2. Incoming calls
   - The module listens on each configured GSM interface.
   - Incoming calls are created as normal FreeSWITCH inbound sessions and routed to the configured context/destination.

3. Control & diagnostics (API commands)
   - `gsm` (general control): list, console, raw AT command, remove, reload
   - `gsmopen <interface> <AT_command>`: send raw AT command
   - `gsmopen_dump <interface|list>`: dump interface state or list all
   - `gsmopen_boost_audio <interface> [<play|capt> <dB>]`
   - `gsmopen_sendsms <interface> <destination_number> <SMS_text>`
   - `gsmopen_ussd <interface> <ussd_code> [nowait]`

## Examples (gsm commands)
```
gsm list
gsm list full
gsm console
gsm AT+CSQ
gsm remove gsm01
gsm remove 1
gsm reload
```

4. SMS/USSD events
   - Inbound SMS/USSD are emitted as `SWITCH_EVENT_MESSAGE` with `proto=sms`.
   - Fields include `from`, `to`, `subject`, and the message body.

## Configuration
Config file: `gsmopen.conf` (example in `configs/gsmopen.conf.xml`).

Key fields per interface:
- `id`, `name`
- `controldevice_name` (control/AT port)
- `controldevice_audio_name` (audio port)
- Optional: `gsmopen_sound_rate`, `capture_boost`, `playback_boost`, `no_sound`, `gsmopen_serial_sync_period`

## Events and status
- The module emits custom events for alarms and dumps.
- Alarm codes include:
  - `ALARM_FAILED_INTERFACE`
  - `ALARM_NO_NETWORK_REGISTRATION`
  - `ALARM_ROAMING_NETWORK_REGISTRATION`
  - `ALARM_NETWORK_NO_SERVICE`
  - `ALARM_NETWORK_NO_SIGNAL`
  - `ALARM_NETWORK_LOW_SIGNAL`

## Notes
- If you use `ANY`/`RR`, the module selects the next idle interface.
- Dialstrings without a `/` separator are invalid and will be rejected.
