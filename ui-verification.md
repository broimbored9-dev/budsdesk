# BudsDesk 2.1.0: UI and tray verification

15 October 2026. The desktop and sign-in shortcuts use this BudsDesk folder. This release executes generated Dafny code for its UI decisions.

Dafny 4.11.0: **155 verified, 0 errors**, without warnings or timeouts. Counts are verifier units, not a percentage of UI behavior. The project uses no unchecked axioms or disabled verification.

| Module | Executed proof scope |
|---|---|
| UiLifecycle | Initial/event invariants and arbitrary event traces: controls need zero pending work and a fresh successful snapshot; pending work cannot reconcile drafts; failed snapshots block readiness. |
| UiDraft | Complete map domains and per-setting preservation/replacement; disconnected clearing; EQ preservation/rollback, band order, valid headroom and 53-byte payloads; scalar/gesture payloads and matching Apply guards. |
| UiMotion | Arbitrary setup/bypass/confirmation traces; cancellation blocks delayed activation; failure/removal/timeout cannot enable; healthy finite telemetry and timestamp freshness. |
| Tray | Menu Open, double-click and activation dispatch agree; valid tab/visibility/restoration/focus requests, repeated-open idempotence, arbitrary open traces, and protection against stale minimized flags. |

The UI replaces independent pending/busy/resync fields with the generated lifecycle state. Each worker message pairs a command reply with its subsequent snapshot. Maps and binary32 bits cross a small trusted Rust conversion layer. The map key sequence comes from the same confirmed map, establishing the reducer's precondition.

Windows integration restores the process's own native main window before waking egui. Live testing found that a hidden window could otherwise fail to redraw and consume Open. This native restore code is tested separately from the proof of restoration decisions.

## Checks

- **22 Rust tests passed**, including complete draft projection, queued-command recovery, malformed/disconnected data, EQ order/headroom, command IDs, motion cancellation and tray behavior.
- The native tray regression uses real Win32 message delivery through an isolated owned window and checks that Open makes it visible and restored.
- Optimized Windows release built successfully.
- Offline smoke rendered all five tabs at both sizes (10 images); saved motion preferences were unchanged.

Raw records: [proofs](ui-dafny-verification.txt), [tests](ui-rust-tests.txt), [release build](ui-release-build.txt).

## Boundaries

These proofs cover UI state, data projection, payload construction and tray decisions. They do not prove egui rendering/pixels/layout, accessibility, Windows window management/focus, actual user input, scheduling/liveness, channels, files/registry, Bluetooth hardware, native DSP, or the compiler/runtime. Rust bindings and command/snapshot ordering remain part of the trusted computing base.

Motion confirmation requires healthy telemetry after activation; it is not a verified command-epoch acknowledgement from the native processor. Windows interaction and rendering are checked separately. Reproduce with source/verify.ps1 or source/build.ps1; source, generated Rust and its runtime are included.

## Installed desktop check

The final 2.1.0 executable is installed at the existing desktop/startup shortcut target; one GUI and one background worker are running. A live minimize/reopen check restored the same window (ID 3737494), and a second launch did not create another GUI. The UI accessibility tree reported RUST 2.1.0 and live core readings. Menu-ID dispatch and restoration are additionally covered by the native test; a manual click on the actual taskbar popup was not performed.

Installed SHA-256: 9C16A9D23ED064A92DD6EF42C045CD8BD90AB42A8155A78682789288DC9113E8.
