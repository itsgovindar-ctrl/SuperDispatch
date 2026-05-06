[README.md](https://github.com/user-attachments/files/27455512/README.md)
# SuperDispatch

A single-file, browser-only **unmanned aerial systems (UAS) dispatch simulator**. Build aircraft, dispatch them across a fleet network, watch the 3D operations view, and talk — literally talk, by voice — to the AI crew at the hub.

Everything runs locally in a single HTML file. No backend, no API keys, no build step. Open `index.html` in Chrome and it works.

## Features

### Core dispatch sim
- **Manufacturing view** to spec aircraft (motors, battery, airframe) before saving them to your fleet
- **Launchpad 3D view** showing the active hub with apron parking, charger station, taxiing aircraft, and a working camera with TOP / SIDE / BACK / GROUND / AUTO presets
- **Customer ride flow** — orders come in, you dispatch a taxi, the customer boards, you fly them, they pay
- **Multi-region** — SF Bay, NYC Metro, SoCal each with their own pad geography and routes
- **Persistent fleet state** — designs, batteries, and lifetime fault history survive page reloads via `localStorage`

### AI crew at the hub
- **10-person crew roster** with role-coded badges: DISPATCH, MAINT × 3, GROUND, OPS, ATC, IR × 2, VO
- **3D figures** standing at role-appropriate stations around the hub (DISPATCH near the pad, MAINT patrolling the apron, ATC at the tower corner, etc.)
- **Comic-style speech bubbles** that pop over the crew's heads with operationally coherent dialogue (DISPATCH ↔ GROUND for handoffs, MAINT ↔ DISPATCH for fault reporting, etc.)
- **Browser-native text-to-speech** — every bubble is also spoken aloud, with each crew member assigned a distinct male voice from the system's enhanced voice pool

### Voice conversation with the crew (NEW)
- **🎤 mic button** in the crew chat panel — click, speak, and your transcript is delivered into the same chat the typed messages use
- The relevant crew member responds based on intent classification — battery questions go to MAINT, weather to ATC, ETAs to DISPATCH, etc.
- Their reply pops a comic bubble over their 3D figure AND speaks it aloud through the system voice — full voice-to-voice loop
- **Hub-only** — at remote pickup/dropoff locations the mic is disabled (the crew is far away at the hub)

### Faults and incident response
- **5 fault types** with per-flight injection: motor out, battery thermal, control link loss, avionics failure, compressor stall
- **Visible motor-out** — when a motor fails, that rotor freezes mid-spin, tilts visibly, and gets a red emissive damage ring
- **IR (Incident Response) team** with a 3D recovery truck — drives out toward the aborting aircraft and returns when it docks
- **Visual Observer** stationed on a remote platform reporting field updates back to you, the operator
- **Lifetime fault counter** on the splash screen with per-kind breakdown

### Diagnostics
- **🎙 VOICE button** in the launchpad header opens a live diagnostic panel showing the SpeechSynthesis state, voice pool, queue depth, and per-crew voice assignments
- Three test buttons: TEST DIRECT SPEAK, TEST CREW LINE, FORCE RESUME, plus a HARD RESET that recovers from Chrome's stuck-synth state
- Standalone `diagnostics/voice-test.html` and `diagnostics/voice-test-minimal.html` for isolating audio issues at the browser level

## Browser support

| Feature | Chrome | Safari | Firefox | Edge |
|---|---|---|---|---|
| Core simulator | ✓ | ✓ | ✓ | ✓ |
| Crew text-to-speech | ✓ | ✓ | ✓ | ✓ |
| Voice input (mic) | ✓ | ✓ | ✗ | ✓ |
| Enhanced/Premium voices | macOS, Win | macOS, iOS | system-dependent | Win |

Chrome on macOS gets the best voice-pool experience (199+ voices including Daniel Enhanced, Tom Enhanced, Microsoft David, Alex). Firefox lacks `SpeechRecognition` so the mic button is hidden there.

## Running it

```bash
git clone https://github.com/<your-username>/superdispatch.git
cd superdispatch
open index.html      # macOS — opens default browser
# or just double-click index.html in Finder
```

Alternative: serve over HTTP if you want to test as if deployed:

```bash
python3 -m http.server 8080
# then open http://localhost:8080/
```

GitHub Pages also serves it directly — push to a `gh-pages` branch or enable Pages on `main`, and the site is live at `https://<username>.github.io/superdispatch/`.

## File structure

```
.
├── index.html                       # the entire simulator (~1.4 MB single file)
├── SuperDispatch.html               # same file, kept under original name for backwards links
├── README.md                        # this file
├── LICENSE                          # MIT
├── CHANGELOG.md                     # version history
├── .gitignore
└── diagnostics/
    ├── voice-test.html              # full voice subsystem diagnostic
    └── voice-test-minimal.html      # one-button minimal speech test
```

## Notes on speech-to-text

The voice input feature uses the browser's **Web Speech API** (`SpeechRecognition`). On Chrome, this routes recorded audio through Google's servers; on Safari, through Apple's. Both are free, no API key, no setup — but they do require an internet connection.

For **Whisper offline** (better accuracy, fully local, no servers): the architecture would be `transformers.js` + the Whisper-tiny WASM model (~30 MB on first load). The current `padVoiceInputState` and start/stop functions are structured so swapping the engine is a one-function replacement. PR welcome.

## Notes on text-to-speech

The crew voices use the browser's `SpeechSynthesis` API. Voice quality depends on what the OS exposes:
- **macOS**: 199+ voices including the "Enhanced" and "Premium" tiers — these are the high-quality ones the system voice picker prefers automatically
- **Windows 10/11**: Microsoft David, Mark, Guy, James (male) plus Zira, Hazel (female, deprioritized in pool)
- **iOS / iPadOS**: Apple's system voices, generally high quality
- **Linux**: depends on speech-dispatcher / espeak; quality varies

The voice subsystem hand-curates a male-leaning, "enhanced/premium/natural" pool from whatever the browser exposes.

## Troubleshooting

**"I can't hear the voices."** Open `diagnostics/voice-test.html`. Click ▶ PLAY BEEP first to confirm audio output works. If the beep plays but speech doesn't, your Chrome's TTS engine is in a stuck state — full Quit Chrome (Cmd+Q on macOS, all windows closed), reopen, try again. If the beep doesn't play, Chrome is routing audio to a different output device than where you're listening — check macOS Settings → Sound → Output.

**"The mic button is grayed out."** You're in customer mode (at a remote pickup/dropoff). Voice input is hub-only.

**"Speech recognition says permission denied."** First-time use prompts for microphone permission. Allow it. If you previously denied it: macOS Settings → Privacy & Security → Microphone → enable for the browser.

**"Voices sound robotic instead of enhanced."** Your OS doesn't have premium voice packs installed. macOS: System Settings → Accessibility → Spoken Content → System Voice → "Manage Voices" → install Daniel (Enhanced), Tom (Enhanced), Alex, etc.

## License

MIT — see `LICENSE`.

## Built by

Govinda — entry-level UAS professional, B.S. Unmanned Aerial Systems, flight operations background at Zipline. This started as an interview prep playground and grew into a full dispatch sim.
