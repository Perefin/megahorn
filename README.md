# MegaHorn

Turn your phone into a Bluetooth megaphone. Push-to-talk, broadcast through any paired Bluetooth speaker.

**Live:** https://perefin.github.io/megahorn/

## How to use

1. Pair any Bluetooth speaker to your phone in **Settings → Bluetooth** (standard iOS/Android pairing)
2. Open the live URL in Safari (iOS) or Chrome
3. Tap **Enable microphone** and grant permission
4. Hold the big button to talk — your voice comes out of the paired speaker
5. Release to stop

No login, no backend, no app install. Just a web page. Keep the phone a few feet from the speaker to avoid feedback.

## Why it works without Web Bluetooth

Web browsers can't drive Bluetooth audio profiles directly (and iOS doesn't support Web Bluetooth at all). So the trick is: the phone's OS already knows how to route audio to a paired Bluetooth speaker. The web app just captures the mic with `getUserMedia()` and plays it back through the Web Audio graph. iOS/Android routes the playback to whatever output device the user has picked in Settings.

## iOS audio quality note

On iOS, opening the microphone forces Bluetooth into **HFP** mode (mono, ~8 kHz "phone-call" quality). This is a hardware/OS-level rule in iOS Core Audio — no web app (or native app) can override it. For a voice megaphone this is fine; walkie-talkies and bullhorns sound this way too. If you want music-hall fidelity, a megaphone isn't the right use case anyway.

## Audio pipeline

```
mic
 ↓  getUserMedia({ echoCancellation: true, autoGainControl: false, noiseSuppression: false })
 ↓
 ↓  BiquadFilter (highpass, 120 Hz)       — cut rumble
 ↓  DynamicsCompressor (-24 / 6:1 / 1ms / 80ms)  — radio/bullhorn shape
 ↓  GainNode (+6 dB)                      — makeup gain
 ↓  GainNode                              — PTT (0 idle, 1 talking, 10 ms ramps)
 ↓  DynamicsCompressor (-1 / 20:1)        — brickwall limiter
 ↓
 AudioContext.destination                 — OS routes to paired BT device
```

## Features

- Pointer-events PTT with `setPointerCapture` (drag-off-button doesn't drop transmission)
- Spacebar PTT on desktop
- Haptic tick on push (where supported)
- Wake Lock so the screen doesn't sleep mid-announcement
- Handles iOS interruptions (incoming call, Siri, backgrounding) → "Tap to resume" banner
- PWA manifest — add to home screen for fullscreen
- Long-press the title to dump `enumerateDevices()` for debugging

## Tech

Single HTML file. No framework, no build step, no backend, no dependencies. ~10 KB over the wire.

## Developing locally

HTTPS is required for `getUserMedia`. Easiest is to open the deployed URL in your phone's browser.

For local testing, any HTTPS static host works — e.g. `npx serve` + `ngrok http 3000`.

## License

MIT
