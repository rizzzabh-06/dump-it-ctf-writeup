# Dump It — CTF Investigation

Challenge: **Dump It** (150 points)  
Platform: **SYYNTAX Medical Device**  
Author: `@syyntax`  
Target: `http://emedra.dctf.io:8080`

## Status

This repository records the work completed so far. The device was inspected, its frontend/API surface was mapped, and the ScanWorks operator console was opened. The final maintenance-console dump and flag retrieval remain to be completed.

## Challenge summary

The ScanWorks kiosk is a browser-based medical-device simulator. The prompt says that the touchscreen lock-down is ineffective because the browser still listens to keyboard input. A maintenance console should be available through a keyboard chord, and a debug dump function should reveal the flag when given the correct data target.

## Initial reconnaissance

Opening the target showed an Emedra CerebraScan interface with these tools:

- HDMI Cable
- DisplayPort Cable
- Live Linux USB-A
- Rubber Ducky USB-A
- BLE Sniffer USB-A

The visible device hotspots included the touchscreen, HDMI port, USB-A port, and asset label. The device API also revealed a USB-C maintenance port that was not initially visible in the text-only view.

## Frontend inspection

The HTML shell loads a Vite JavaScript bundle and renders most of the interface dynamically. The main bundle exposed the following API wrappers:

```text
GET  /api/healthz
GET  /api/device
GET  /api/hotspot/<id>
POST /api/connect
GET  /api/kiosk/login
POST /api/kiosk/auth
GET  /api/kiosk/patients
POST /api/kiosk/scan
GET  /api/kiosk/about
POST /api/shell
GET  /api/external-display?level=<n>
POST /api/kiosk/hid
GET  /api/capture
```

The most interesting endpoints for the remaining investigation are `/api/shell`, `/api/external-display`, and `/api/kiosk/hid`.

## Device metadata

`GET /api/device` returned:

```text
Device:   Emedra CerebraScan CS-9000
Vendor:   Emedra
Site:     St. Dymphna's RMC
Firmware: NeuraOS 2.3.1
Serial:   EM-CS9K-04417
```

Relevant tool compatibility:

```text
HDMI Cable          -> HDMI
Live Linux USB-A    -> USB-A
Rubber Ducky USB-A  -> USB-A
BLE Sniffer USB-A   -> USB-A
```

## ScanWorks console

Clicking the touchscreen opened the `ScanWorks — Operator Console` login window. The login screen displayed an active study for `G.M.` and the masked MRN ending in `13`.

The challenge supplied a default technician account. The credentials were entered directly into the browser; credentials are intentionally not stored in this repository.

The kiosk frontend shows that successful authentication returns both a normal session token and a `maintenanceToken`:

```javascript
if (result.ok && result.token) {
  sessionToken = result.token;
  technicianName = result.tech?.name ?? "Technician";
  maintenanceToken = result.maintenanceToken ?? "";
  showPatients();
}
```

The signed-in banner is expected to display the maintenance token. That token should be preserved locally during the next phase, but must not be committed to Git.

## Terminal behavior

The terminal UI sends commands through the shell endpoint in this form:

```javascript
POST /api/shell
{
  "token": "<session-or-maintenance-token>",
  "line": "<command>"
}
```

The response may contain `stdout`, `stderr`, and `cwd` fields. Once the maintenance console is opened, safe discovery commands such as `help`, `?`, `commands`, or a debug-specific help command should identify the dump syntax.

## Next steps

1. Resume the browser session and verify that technician login succeeded.
2. Record the maintenance token only in a private local note.
3. Try the keyboard shortcut suggested by the challenge wording, prioritising maintenance/debug combinations such as `Ctrl+Alt+M` or `Ctrl+Alt+D`.
4. Inspect the maintenance/debug console for a dump target field or command.
5. Use the discovered target with the debug dump function.
6. Confirm the returned value matches `deadface{...}`.

## Security notes

- Do not commit the technician password, session token, maintenance token, cookies, or other authentication material.
- This repository contains analysis and public frontend/API observations only.
- The challenge flag is not included because it had not yet been retrieved when this writeup was created.
