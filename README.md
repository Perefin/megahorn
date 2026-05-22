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

On iOS, opening the microphone forces Bluetooth into **HFP** mode (mono, ~8 kHz, ~300 Hz–3.4 kHz "phone-call" quality). This is a hardware/OS-level rule in iOS Core Audio — no web app (or native app) can override it. The pipeline is now tuned for that narrowband: a high-shelf presence boost at 2.5 kHz lifts consonant clarity, a soft de-ess tames sibilance at the top of the band, and the limiter ceiling sits at −3 dBFS to give the HFP encoder headroom (it clips ugly when slammed at 0). It still won't sound like a studio mic — but it should be intelligible from across a parking lot, which is what a megaphone is for.

The previous default of `echoCancellation: true` was actively *suppressing* your voice when it returned through the speaker (iOS Voice Processing is doing what it's designed to do: cancel sound the device is currently playing). The default is now **off**, with a settings toggle if you need it back.

## Audio profiles

Three profiles ship; switch in the **⚙ Settings** sheet. Default is **Crisp**.

| Profile | Best for | What it does |
|---|---|---|
| **Crisp** | Voice clarity, default | Presence boost +4 dB @ 2.5 kHz, gentle 4:1 compression, −3 dB ceiling |
| **Loud** | Maximum SPL, noisy environments | Lower compression threshold, higher makeup gain, hotter −2 dB ceiling |
| **Natural** | If Crisp sounds harsh | Less presence boost, gentler 3:1 compression, −6 dB ceiling for headroom |

All three respect the HFP narrowband ceiling — none of them recover frequencies the codec discards.

### Echo cancellation toggle

- **Off (default)** — loudest, fullest broadcast. Use this outdoors or when the phone is more than an arm's length from the speaker.
- **On** — only flip this if the phone is right next to the speaker and you're getting feedback. Expect occasional voice ducking; it's iOS Voice Processing trying to cancel "speaker output" — which here is your own voice.

Settings persist in `localStorage`.

## Audio pipeline

```
mic
 ↓  getUserMedia({ echoCancellation: <profile>, autoGainControl: false, noiseSuppression: false })
 ↓
 ↓  BiquadFilter (highpass, 220 Hz)        — strip handling rumble (HFP cuts <300 Hz anyway)
 ↓  GainNode (RMS-driven gate, −45 dB)     — silence speaker bleed / room noise
 ↓  BiquadFilter (peaking +4 dB @ 2.5 kHz) — presence boost for consonant clarity
 ↓  BiquadFilter (peaking −3 dB @ 5 kHz)   — de-ess / tame HFP codec aliasing
 ↓  DynamicsCompressor (−18 / 4:1 / 5 ms / 150 ms / knee 6) — voice shaping
 ↓  GainNode (+3.5 dB)                     — makeup
 ↓  BiquadFilter (lowpass, 3.4 kHz)        — pre-filter to HFP band
 ↓  GainNode                               — PTT gate (0 idle, 1 talking, 10 ms ramps)
 ↓  DynamicsCompressor (−3 / 20:1)         — brickwall limiter w/ HFP headroom
 ↓
 AudioContext.destination                  — OS routes to paired BT device
```

(Numbers shown are the **Crisp** profile; Loud and Natural shift them per the table above.)

## Features

- Pointer-events PTT with `setPointerCapture` (drag-off-button doesn't drop transmission)
- Spacebar PTT on desktop
- Haptic tick on push and on profile change (where supported)
- Wake Lock so the screen doesn't sleep mid-announcement
- Handles iOS interruptions (incoming call, Siri, backgrounding) → "Tap to resume" banner
- PWA manifest — add to home screen for fullscreen
- iOS-native bottom-sheet settings (drag to dismiss, system blue accents, blur surfaces)
- Body-class state machine — no more stuck enable button when permission is granted
- Test tone — a sine sweep routed through the **selected profile's** EQ, compression, and limiter, so you can hear what each profile does without holding PTT
- Long-press the title to open settings as a power-user shortcut

## Tech

Single HTML file. No framework, no build step, no backend, no dependencies. ~10 KB over the wire.

## Developing locally

HTTPS is required for `getUserMedia`. Easiest is to open the deployed URL in your phone's browser.

For local testing, any HTTPS static host works — e.g. `npx serve` + `ngrok http 3000`.

## License

MIT
