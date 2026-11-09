# BudsDesk 2.0.2 — BPM and beat sync fix

The running app showed an enabled effect with detected BPM zero. Two defects caused this: a bass-only detector with a high absolute threshold missed quieter/dense material, and a controller tied to window repainting could stop its heartbeat. The observed stale heartbeat was over 14 minutes old; the audio processor had bypassed the effect. The interface also displayed its 120 BPM fallback as if it were a detected tempo.

The updated native processor uses stereo spectral changes and repeating onset patterns to estimate tempo. Motion now follows beat positions as well as tempo, with bounded corrections that preserve continuous forward/reverse movement. The approach uses onset strength and correlation, consistent with the stages described in the [official beat-tracking documentation](https://librosa.org/doc/main/api/generated/librosa.beat.beat_track.html), without adding a runtime dependency.

The Rust controller keeps its heartbeat alive on a separate worker. That worker changes only the atomic heartbeat timestamp; settings remain under the existing control sequence. Closing the window stops the worker and bypasses motion. The interface labels detected and fallback tempo separately. Processor updates use a new content-derived DLL filename, replace only the existing BudsDesk audio block, and back up the previous config, so Windows loads the updated processor.

Checks completed:

- Nine Rust tests passed, including heartbeat progress without window repaints and stopping the heartbeat on close. Static analysis and the release build passed.
- Native tests reproduced the old failure with quiet 123 BPM beats before the fix. After the fix, quiet, treble-only and dense-bass fixtures measured 122.986, 122.981 and 122.925 BPM respectively.
- Existing 40, 80, 120, 180 and 220 BPM checks passed, along with stereo/anti-phase, silence fallback, sustained-note rejection, and continuous motion.
- Beat-position alignment error was 3.012 degrees at 120 BPM, approximately 17 ms after lock.
- New tests passed at 32, 44.1, 96 and 192 kHz and for a change from 120 to 150 BPM. Existing HRTF, bounded-output, exact-bypass and VST checks also passed.
- Windows' installed processor DLL matches the built DLL byte for byte. Live observations showed the effect active at 1,000 Hz and a fresh controller heartbeat. The saved spatial settings and other Equalizer APO configuration were retained.

The earbuds disconnected before the final live 97 BPM tone test, and Windows returned `AUDCLNT_E_DEVICE_INVALIDATED`. This prevented that live tone validation; it was not counted as a pass. The controlled native rhythm and phase tests passed. Tempo detection needs a few beats to lock and falls back explicitly when a reliable pulse is unavailable.

The desktop app is updated to 2.0.2. Its shortcut target and arguments remain the same. The old installed processor and audio-config backup remain available for rollback.
