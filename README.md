# Limitless — Hand-Tracked Cursed Technique Sandbox

A real-time, in-browser VFX sandbox where your **hands cast Gojo Satoru's techniques**. Make an OK-sign to channel **Blue**, an open palm for **Red**, bring them together for **Hollow Purple**, or hold both fists to trigger a **Domain Expansion**. Everything — hand tracking, gesture recognition, and rendering — runs entirely in the browser. No backend, no install.

> Built with MediaPipe hand tracking (WebAssembly) + Three.js. A single static `index.html`.

**[▶ Live demo](https://limitless.lakshay.app/)** · needs a webcam · runs 100% client-side

<!-- Replace the line below with a screenshot or GIF: drop an image in the repo and reference it. -->
<!-- ![Demo](demo.gif) -->

---

## Techniques

| Gesture | Technique | Effect |
| --- | --- | --- |
| OK-sign (thumb + index ring, other fingers up) | Cursed Technique Lapse: **Blue** | A blue orb that pulls a swirling particle field — and nearby debris — inward |
| Open palm (fingers spread) | Cursed Technique Reversal: **Red** | A red orb that blasts particles and debris outward |
| Blue hand **+** Red hand, brought together | Hollow Technique: **Purple** (虚式「茈」) | A purple beam fires into the scene, vaporizing any debris in its path |
| Both fists, held together ~1s | Domain Expansion: **Unlimited Void** (無量空処) | Full-screen takeover — the webcam inverts to the stark void, rotating sigils, deep ambient swell |

Each hand channels independently, so you can hold Blue in one hand and Red in the other at the same time.

## Controls

| Key / Action | Does |
| --- | --- |
| `I` | Toggle **Infinity** — a refractive shimmer barrier around the scene |
| `B` | Toggle **bloom** (glow) on/off |
| Begin button | Grants camera access + enables sound (one click) |

A faint **idle aura** stays on any hand in frame even when you're not casting, so your hands always read as charged with cursed energy.

---

## How it works

The whole pipeline runs client-side, every frame:

```
webcam (getUserMedia)
        │
        ▼
MediaPipe HandLandmarker (WASM)  ──►  21 landmarks × up to 2 hands
        │
        ▼
per-hand pose classifier         ──►  CIRCLE / PALM / FIST / IDLE
   (finger-curl + hand geometry,
    One-Euro–smoothed positions)
        │
        ▼
two-hand combo logic             ──►  PURPLE (merge) / DOMAIN (held fists)
        │
        ▼
Three.js VFX layer               ──►  orbs, particle fields, beam, debris,
   (transparent canvas + bloom)        Infinity barrier, screen FX
        │
        ▼
Web Audio engine                 ──►  charge hum, beam roar, domain swell
```

**Pose classification** is rule-based and rotation-invariant. A finger counts as "curled" when its fingertip is closer to the wrist than its middle knuckle (works at any hand tilt). The OK-sign is detected from a small thumb–index distance (normalized by palm size, so it's scale-invariant) combined with the other fingers being extended.

**Combos** key off the normalized distance between the two hands rather than interlaced finger poses, which keeps them reliable — bringing a Blue hand and a Red hand within range fires Purple; holding both fists close for one second expands the Domain.

**Smoothing** uses a [One-Euro filter](https://gery.casiez.net/1euro/) on each hand's position — low latency on slow movement, more smoothing on fast movement — which is the standard for hand-tracking interaction and feels far better than a moving average.

**Mirroring:** the raw camera frame is fed to MediaPipe, and the resulting landmark X coordinates are flipped in JavaScript so the on-screen selfie view and "move right → effect moves right" stay consistent.

## Tech stack

- **[MediaPipe Tasks Vision](https://ai.google.dev/edge/mediapipe)** `0.10.14` — `HandLandmarker` running in the browser via WebAssembly (GPU-accelerated with an automatic CPU fallback)
- **[Three.js](https://threejs.org/)** `r160` — WebGL scene, `EffectComposer` + `UnrealBloomPass` for the glow, a custom fresnel `ShaderMaterial` for the Infinity barrier
- **Web Audio API** — procedural sound (oscillators + filtered noise), no audio files
- **`getUserMedia`** — direct webcam access
- Plain HTML + ES modules — **no build step, no bundler, no framework**

---

## Run it locally

Because the project uses ES modules and the webcam, it must be **served over HTTP** — opening the file directly (`file://`) won't work. From the project folder:

```bash
python -m http.server 5500
```

Then open <http://localhost:5500>. (`localhost` is treated as a secure origin, so the camera works without HTTPS.)

## Deploy to GitHub Pages

1. Put `index.html` at the **root** of a public repo.
2. **Settings → Pages → Build and deployment → Source: Deploy from a branch.**
3. Select your branch (`main`) and folder `/ (root)`, then **Save**.
4. After a minute it's live at `https://YOUR-USERNAME.github.io/REPO-NAME/`.

GitHub Pages serves it over HTTPS automatically, which the camera requires. No server or build configuration is needed.

## Configuration / tuning

All tunables live near the top of the classifier section in `index.html`:

| Constant | Default | What it does |
| --- | --- | --- |
| `CIRCLE_RATIO` | `0.50` | OK-sign sensitivity (raise = easier to trigger Blue) |
| `MERGE_DIST` | `0.24` | How close the hands must be to fire Purple (raise = easier) |
| `DOMAIN_DIST` | `0.30` | Fist proximity that starts the Domain hold |
| `DOMAIN_SECONDS` | `1.0` | How long to hold both fists before the Domain expands |

Other handy levers: the `7` (Blue) and `16` (Red) in `DebrisField.update` set how hard debris is pulled/pushed; the charge-hum volume is the `0.16` / `0.18` in `setCharge`; bloom intensity is the `bloom.strength` line in the render loop.

## Requirements & browser support

- A **webcam** and permission to use it.
- **Internet on first load** — Three.js, the MediaPipe runtime, and the hand model are fetched from CDNs (`jsdelivr`, Google), then cached.
- Works in current Chrome, Edge, and Firefox. Performance scales with the device; smooth on any modern laptop.
- If the model fails to load, bump the `tasks-vision@0.10.14` version in the `import` and WASM URLs.

## Known limitations

- Two-hand tracking degrades if hands overlap or leave the frame; keep both hands clearly visible for the combos.
- Effects are anchored to normalized hand positions while the webcam is cropped to fill the screen, so there can be a slight offset near the edges.
- The Domain Expansion and beam are stylized impressions, not frame-accurate recreations.

## Possible additions

- Purple **charge-up windup** — Blue and Red visibly strain toward each other before the beam releases
- Red **shockwave ring** on a sharp push
- A short **ULTIMATE** combo (Blue → Red → Purple → Domain) for a finisher
- Recording / screenshot capture for sharing
- A second character mode (palette + technique swap)

## Credits & disclaimer

Jujutsu Kaisen and the character Gojo Satoru are created by **Gege Akutami**. This is a **non-commercial fan project** for learning and demonstration, with no affiliation to or endorsement by the rights holders. Hand tracking by Google MediaPipe; rendering by Three.js.

## License

Code released under the MIT License — see `LICENSE`. Note that the disclaimer above applies to the referenced characters and franchise, which remain the property of their respective owners.
