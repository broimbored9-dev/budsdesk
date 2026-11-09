# BudsDesk 2.1.0

The desktop and sign-in shortcuts point to this build. The application executes Dafny-generated Rust for UI synchronization, editable settings and EQ, motion setup, tray decisions, protocol processing, validation and worker policies.

Current checks: **155 Dafny units verified with zero errors; 22 Rust tests passed; optimized release built; native tray restoration and live minimize/reopen checks passed.** See [UI and tray verification](ui-verification.md) for exact scope and remaining boundaries. The entire Windows application, rendering and native audio processor are not formally proved.

## Build

Run source/build.ps1 from PowerShell with Rust and Visual Studio C++ tools installed. It verifies all Dafny modules, rejects generated-source/runtime/manifest drift, runs native/Rust checks, and builds BudsDesk.exe. Dafny 4.11.0 is downloaded with a pinned checksum into source/work/tools by default; -WorkDirectory selects another writable cache. The application itself requires no .NET runtime.

Edit proof sources under source/verification, run source/verify.ps1 -Regenerate, review generated changes, then build. [Verification guide](source/verification/README.md).

## Use

Launch BudsDesk.exe --spatial with BudsDeskSpatial.dll and BudsDeskApoSetup.exe beside it. Minimize to tray and choose Open BudsDesk to restore the window; ordinary second launches restore the existing instance, while --spatial selects 3D motion.or just double click exe

Controls retain ANC/lock, EQ, bass, gestures, connection options, profiles, batteries, audio diagnostics and spatial motion. Setup/Repair uses Equalizer APO and its existing administrator prompt. Preferences use HKCU/Software/BudsDesk and the usual per-user BudsDesk directory. --smoke <folder> renders offline sample views without changing motion preferences; --snapshot reads the worker state.

Historical reports: [earlier core verification](verification.md), [earlier review](review-and-verification.md), [BPM work](bpm-fix.md), [original usage](README-original.md). Their results describe earlier builds.

Application license: LICENSE.txt (AGPL-3.0). Dafny runtime attribution: source/verification/DAFNY-LICENSE.txt. Equalizer APO/OpenAL attribution remains beside the native sources.
