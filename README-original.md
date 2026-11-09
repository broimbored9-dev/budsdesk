# BudsDesk

A local Windows controller built for your **CMF Buds 2 Plus**. Open `BudsDesk.exe`; it connects to the buds already paired with Windows. Keep the buds out of their case and connected for audio. Scroll inside each tab for additional controls.

## Controls

- Left/right battery and case battery when the case reports it; firmware version.
- ANC off, transparency, and high/medium/low/adaptive cancellation.
- Default tuning, Pop, Rock, Classical, Electronic, Enhance vocals, and custom three-band EQ (−6 to +6 dB).
- Ultra Bass, fixed spatial effect, and in-ear detection.
- Separate left/right touch controls and noise-mode cycling combinations.
- Low-latency mode, dual connection, and the earbuds' codec preference.
- Find a nearby connected bud, with a confirmation and five-second ring limit. **Remove both buds from your ears before ringing.**
- Extras: saved sound profiles, tray controls, automatic control-channel reconnection, a connection journal, cutout markers, diagnostic export, and Windows microphone routing with undo.

Click **Apply** to change a setting. Most changes are queried again and checked against the actual response. Disconnected or unreported settings cannot be applied. Reconnection concerns the app's control channel; it does not promise to repair Windows audio connections.

The LDAC option changes the buds' preference. It does **not** add LDAC to Windows or display the codec Windows is currently using. Codec changes can restart the earbuds. Low-latency mode is optional: keep it off if it makes cutouts worse. It was returned to its original off setting after testing.

Minimize the window to use the tray icon. Double-click the icon to reopen it. Closing the window exits the app. Startup at Windows sign-in is not enabled.

## Remaining Nothing X features

Firmware installation, Audiodo hearing-test/personal-sound setup, cloud/location services, and Nothing-phone ChatGPT integration are not implemented. Ear-tip fit testing and management of the individual multipoint device list are also not included. The device may expose other commands, but these have not been established for this model here. This is not a complete Nothing X replacement.

## Verification on your buds

The hardware session read all twelve queried settings. ANC, EQ preset, custom EQ, Ultra Bass, wear detection, spatial effect, and a gesture write were checked by sending their existing values and reading them back. Low-latency mode was changed and restored. A final comparison confirmed all tested settings matched the original snapshot.

Codec changes, multipoint changes and find-bud ringing were not exercised because they interrupt audio or emit loud sound. They are implemented from community protocol information, not claimed as hardware-verified writes. Profile import/export has input validation; the complete file-dialog workflow has not been manually exercised. Protocol tests cover actual device frames, split/coalesced frames, CRC errors, resynchronization, all payload sizes, gesture bounds, and profile rejection. All four tabs were rendered and visually inspected.

## Data and troubleshooting

The app has no remote server, telemetry, or login. Local settings, snapshots and logs live in `%LOCALAPPDATA%\BudsDesk`. Diagnostic exports include device names, audio routes and timestamps; inspect them before sharing.

The laptop's Bluetooth power-saving adjustment is independent of this app. BudsDesk does not need to stay open for it. See `Bluetooth findings.md` for the repair, test limits and rollback instructions.

If the buds disconnect again, open **Diagnostics**, click **Mark a cutout now**, and export the journal. Note the app playing audio, distance from the laptop and whether the problem followed sleep/wake. A short successful session cannot prove an intermittent problem is permanently fixed.

## Source and build

The `source` directory contains the complete application source. This build uses the .NET 10 Windows Desktop runtime already installed on this laptop. Another PC needs that runtime and Windows 10/11 x64.

With the .NET 10 SDK, from the `source` directory:

```powershell
dotnet run -- --self-test
dotnet run -- --diagnose
dotnet publish -c Release -r win-x64 --self-contained false -p:OutputType=WinExe -p:PublishSingleFile=true -p:DebugType=None -o ../rebuilt
```

`--live-test` communicates with your buds, briefly toggles low latency and restores the original settings. Run it only with the GUI closed. `--monitor` makes read-only queries for five minutes. `--preview` creates images of the application's own tabs under `work/previews` in the current directory.

## Credits and license

Unofficial; not affiliated with Nothing or CMF. Protocol and EQ/gesture mappings were adapted from [radiance-project/ear-web](https://github.com/radiance-project/ear-web), by RapidZapper, Bendix and contributors. The Windows WinRT connection approach and newer command semantics were cross-checked against [Vadim-Karpenko/ear-tray](https://github.com/Vadim-Karpenko/ear-tray). BudsDesk's source is supplied under AGPL-3.0; see `LICENSE.txt`. This package includes [NAudio](https://github.com/naudio/NAudio) by Mark Heath and contributors, used under its MIT license, and Microsoft Windows SDK/.NET components. Their notices are in `licenses`.

The preset names were checked against [Nothing's CMF Buds 2 Plus support page](https://support.nothing.tech/hc/en-us/articles/34007440022545-How-many-rqualizers-does-CMF-Buds-2-Plus-have).
