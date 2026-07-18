# SpeedBeam

A cinematic, holographic internet speed test. Built with vanilla HTML/CSS/JS
(ES modules), Three.js, and GSAP — no build step, no framework, no bundler.

## Run locally

Because this uses ES modules, open it through a local server rather than
double-clicking the file (the `file://` protocol blocks module imports).

```bash
# any static server works, e.g.:
npx serve .
# or
python3 -m http.server 8080
```

Then visit the printed local URL.

## Deploy

This is a static site — drop the folder as-is into any of these:

- **Vercel**: `vercel deploy` from this folder, or drag-and-drop in the
  dashboard. No build command needed; set the output directory to `.`
- **Netlify**: drag-and-drop this folder in the Netlify dashboard, or
  `netlify deploy`. Publish directory: `.`
- **GitHub Pages**: push this folder to a repo and enable Pages on the
  branch/root.

## AI-generated assistant verdict (optional)

By default, the "Beam Assistant" message is written by simple rules in
`js/ui/assistant.js` — no API key needed, works out of the box.

If you want that message to instead be generated live by Claude:

1. In your Vercel project dashboard, go to **Settings → Environment
   Variables**.
2. Add a variable named `ANTHROPIC_API_KEY` with your Anthropic Console
   API key as the value. Never put this key in any frontend file —
   `api/assistant.js` is a serverless function that runs only on
   Vercel's servers, keeping it secret.
3. Redeploy. The frontend will automatically call `/api/assistant`
   after each test and swap in the AI-written verdict once it arrives;
   if the call fails for any reason (no key set, rate limit, network
   issue), it silently keeps the instant rule-based message instead —
   the UI never breaks or shows an error to the user over this.

This only works when deployed to Vercel (or any host that runs the
`api/` folder as serverless functions). Running via a plain static
server (`npx serve .`) will just keep the rule-based message, since
there's no server to run `api/assistant.js`.

## How the speed test works

`js/speedtest/engine.js` performs real measurements:

- **Ping/jitter**: several round-trip `fetch` calls, averaged
- **Download**: streamed `fetch` reads, sampling live throughput as bytes
  arrive
- **Upload**: streamed `POST` of randomly generated payloads, sampling
  throughput as chunks complete

By default it targets Cloudflare's public speed-test endpoints
(`speed.cloudflare.com/__down` / `__up`), which are CORS-enabled and need no
API key. If a deployment environment blocks that host (corporate proxy,
strict CSP, etc.), swap the URLs in `ENDPOINTS` at the top of `engine.js` for
any host that serves a large file and accepts a CORS POST.

## Structure

```
index.html
css/
  variables.css      design tokens (palette, type, motion)
  base.css            resets, background gradients
  layout.css          structural layout
  components.css      buttons, cards, HUD rings
  animations.css       keyframes, reduced-motion overrides
  responsive.css       breakpoints
js/
  main.js              wires everything together
  scene/
    index.js           renderer, camera drift + parallax, render loop
    background.js       starfield, nebula, grid floor, network nodes
    sphere.js            the hero holographic sphere (signature element)
  speedtest/
    engine.js            real ping/download/upload measurement
    sequence.js          orchestrates the 5 cinematic stages
  ui/
    speedometer.js       HUD ring + live numeric readout
    results.js           results card reveal
    assistant.js         AI verdict copy from real results
    history.js           localStorage history, search, delete, clear
  audio/
    sound.js             synthesized UI sounds (Web Audio, no files)
```

## Accessibility & performance notes

- Respects `prefers-reduced-motion`: camera drift, particle motion, and
  entrance animations are disabled/shortened.
- Live regions announce test stage changes and final results for screen
  readers.
- Rendering pauses when the tab is hidden.
- Pixel ratio is capped (1.6–2x) to keep frame rate steady on mobile GPUs.
