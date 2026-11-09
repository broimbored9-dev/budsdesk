# BudsDesk Rust 2.0.1 — review and verification

12 October 2026. All issues below are resolved in the included source and release executable. The existing desktop shortcut opens this build with `--spatial`; the existing sign-in shortcut starts its worker with `--background`. Original ANC, EQ, ANC-lock preferences, saved spatial profile, and the previous profile recovery file were retained.

[P1] Bound helper communication and recover from stalled clients — source/src/platform.rs:188

Synchronous reads and flush could wait indefinitely, freezing commands and preventing later connections. Communication now uses bounded asynchronous Windows I/O, cancels timed-out operations safely, reads in blocks, and reuses the server pipe. The implementation follows Microsoft's [overlapped named-pipe guidance](https://learn.microsoft.com/en-us/windows/win32/ipc/named-pipe-server-using-overlapped-i-o).

[P1] Stop ringing while Bluetooth queries are waiting — source/src/backend.rs:211

The five-second timer was checked only after complete requests. A slow refresh or profile could keep a bud ringing substantially longer. The timer now runs inside the receive path and stops both buds, including when closing the connection. Invalid ring requests no longer arm it, and a failed left-stop does not skip the right-stop.

[P2] Clear connected readings when reconnecting — source/src/backend.rs:289

Reconnect discarded the socket while leaving the snapshot connected with old settings and battery readings. It now clears those readings immediately.

[P2] Discard settings that cannot be read back — source/src/backend.rs:207

A timed-out query retained its old cached response, presenting an unconfirmed value as current. The missing response is now removed until a fresh response arrives.

[P2] Report incomplete refreshes as failures — source/src/backend.rs:332

Refresh logged errors but returned success, replacing the error with “Request confirmed.” It now returns query failures while retaining successfully read settings.

[P2] Reconcile failed edits with actual earbud responses — source/src/ui.rs:1184

When a setting was rejected and the earbud value stayed unchanged, the draft selection was never reset. After a command, controls and EQ gains now reconcile with confirmed readings. Ordinary polling still preserves edits before Apply.

[P2] Keep command progress separate from tone and setup messages — source/src/ui.rs:236

A tone completion or unrelated message cleared the busy flag for an outstanding command. Only command completion now changes the pending-job count. Older snapshots also cannot overwrite preferences while their commands remain pending.

[P2] Clear stale state and restart an unavailable worker — source/src/ui.rs:143

Failed snapshot reads were silently ignored after a worker exit. The interface now publishes the unavailable state and retries starting its worker.

[P2] Expire queued commands before executing them — source/src/backend.rs:468

A request could time out in the queue and still execute later. The worker now rejects requests whose queue deadline has elapsed.

[P2] Refresh profile recovery settings before loading — source/src/backend.rs:344

A recovery copy could contain missing or stale settings. Profile loading now reads and validates all six recovery settings before saving the backup or changing the buds. Incomplete imported profiles are rejected before device mutation.

[P2] Accept formatted JSON request files — source/src/main.rs:50

The file command sent text directly to a line-based pipe, so multiline JSON failed at its first newline. It now parses the action and sends compact JSON.

[P2] Respect Bypass and the saved launch preference during setup — source/src/ui.rs:942

Setup could enable motion after the user pressed Bypass and unconditionally turn on the “remember on launch” preference. Bypass or loading a bypassed profile now cancels pending activation. Setup retains the chosen launch preference and clears previous activation deadlines.

[P2] Honor Equalizer APO's configured paths — source/src/spatial.rs:343

The port used only the conventional Program Files location. It now reads installed and configuration paths from Equalizer APO's registry settings, with the conventional fallback.

[P2] Recover a missing config and reject a missing spatial DLL — source/src/spatial.rs:426

Setup failed when config.txt was absent, and comparing two failed DLL reads could reference a missing plugin. An absent config is now treated as empty; installation requires the source DLL before making changes. Existing config files continue to receive a backup.

[P2] Reject enabling motion without a usable control mapping — source/src/spatial.rs:262

Enabled settings could report success despite mapping initialization failure. Enabling now requires a usable control map and returns the initialization error.

[P3] Balance Windows audio initialization — source/src/audio.rs:22

Each audio inspection added another COM initialization without releasing it. Initialization now happens once per thread and is released when the thread exits, while respecting an existing apartment.

Verification:

- Eight Rust tests passed: local-pipe stall recovery and 60,000-byte replies, CRLF requests, delayed replies with ring stops, stale-query removal, reconnect state, failed-setting reconciliation, ANC decisions, packet parsing, profiles, and audio-config preservation.
- Static analysis passed with existing stylistic `collapsible_if` and `field_reassign_with_default` lints excluded. The release build completed with locked dependencies.
- Native spatial checks passed for exact stereo bypass, supported sample rates, variable buffers, all presets, bounded output, HRTF direction, VST loading, continuous orbit transitions, and 1,000 Hz motion updates.
- All five tabs rendered at 1000 and 640 logical pixels and were visually inspected.
- Actual CMF Buds 2 Plus accepted Off and Transparency, followed by confirmed Medium ANC restoration. Disabling lock left Off selected. Invalid ANC values were rejected, and manual Pop EQ persisted.
- Formatted JSON requests worked. All six profile settings round-tripped, the recovery copy was complete, incomplete profiles were rejected, full refresh succeeded, and reconnect reread settings.
- Live interface checks received healthy Windows audio-host telemetry at 48,000 Hz. Motion remained bypassed, matching the saved profile.
- Stopping the worker caused the app to restart it and reconnect. A second launch exited successfully; one GUI and one worker remained.
- Both shortcuts were read back. Final ANC, EQ and lock preferences matched the initial snapshot, and the saved spatial profile was unchanged. The installed executable matches the verified release build.

No remaining actionable issues were found in the reviewed fixes. A complete Windows reboot, physical case reconnect, and elevated install/remove at a different Equalizer APO location were not performed. The buds did not report a case battery reading. Ring timing was tested with a simulated peer.

The previous Rust app folder remains unchanged. `shortcuts-before.json` records previous shortcut targets for rollback. Live ANC and profile test results are in `live-verification.txt`.
