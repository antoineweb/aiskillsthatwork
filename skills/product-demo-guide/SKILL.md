---
name: product-demo-guide
description: "Create an animated step-by-step product demo (simplified but recognizable UI, company branding, step panel with milestones on the right) and export it to MP4. Use when someone asks for a demo video, animated guide, walkthrough or tutorial of an app or a flow."
---

# Product demo guide

Builds a short animated guide that teaches one task in a product, then exports it as a video. The result is one self-contained HTML page (it plays in a browser, with play, pause and scrub) plus an MP4.

## What makes this format work

Keep all five. They are the reason people like the result.

1. **Recognizable but simplified UI.** The viewer must recognize their product at a glance: same layout, same colours, same component shapes, the exact labels. Everything the step does not talk about is removed: secondary navigation, toolbars, counters, avatars, extra columns, extra fields. What stays is enlarged (fields 48 px high, text 19 px or more on a 1920 x 1080 stage) so it reads on a phone or a projector.
2. **Company branding around the product.** Company logo and guide label top left on every step, brand colours for accents, a title card and an end card in the brand's dark colour.
3. **Step panel on the right.** "Step n of N", a short title, then 2 to 4 short lines that appear one at a time, each at the moment the action happens on screen. The newest line is highlighted, earlier ones fade to grey.
4. **Milestones top right.** One pill per step, filled as the guide advances.
5. **One cursor telling the story.** It glides to the target, clicks with a ripple, text types itself, menus open, results appear. Nothing moves without a reason.

Layout of the 1920 x 1080 stage: header at y 52; app window at x 72, y 150, 1180 x 820 with a minimal browser bar (three dots and the address, nothing else); step panel at x 1308, 540 wide.

## Before building, get these

Ask only for what is missing. Do not invent labels or rules.

- **The flow**: which task, where it starts, where it ends, the steps in order. Screenshots of each screen are the best source; a live page or a written description also works.
- **Exact wording** of every label, button and status that will appear.
- **Branding**: company logo as SVG (one version for light backgrounds, one for dark), primary colour, dark colour, UI font. If there is no logo file, use the company name as text. Never redraw a real logo from memory.
- **Audience and language** of the on-screen text.
- **Rules worth saying** in the step lines (what is required, what happens next, what is normal).

Use fictitious demo data (names, emails, ticket numbers) and say so on the end card.

## Steps

1. **Storyboard first.** Write a table: scene, duration, what happens on screen, the step lines. Show it and get a yes before building. Aim for 5 s title card, 9 to 18 s per step, 5 s end card; 6 to 8 steps is a good maximum; one idea per step. List what you deliberately left out and anything you are unsure of.
2. **Build the page** from the template below. Change, in order: `B` (branding), `SCENES` (the outline), `STEPS` (panel text), then `frame()` (the simplified screens, the cursor path, the timed state). Keep the engine and the player as they are.
3. **Check it** by exporting a few seconds and looking at frames from every step (see Export). Look for: text clipped or overlapping, cursor landing off its target, a click ripple with nothing reacting, a step line that appears before or after its action.
4. **Deliver the page** the way this session delivers a web page (published page, or the `.html` file if they asked for the file).
5. **Export the video** when asked, or offer it in one line.

## Rules for `frame()`

- Everything visible is computed from `T` (seconds). No CSS transitions, no timers, no state kept between frames. This is what makes the export exact and smooth.
- Times are written relative to a scene start: `const sF = CUES.Fill;` then `sF + 1.2`. Changing a duration in `SCENES` then shifts everything after it correctly.
- Motion uses only the three easings: `EASE.enter` for things appearing, `EASE.move` for the cursor and scrolling, `EASE.pop` for a small overshoot. Fades last 0.4 to 0.6 s.
- Helpers: `A(start, end, from, to)` animates a value; `win(start, end)` gives an opacity that fades in and out; `on(start, end)` is true between two times; `typed(text, start, duration)` types text; `caret(start, end)` blinks a caret.
- The cursor path `K` is a list of `[time, x, y]`. Repeat a position to make the cursor rest. Leave about 0.2 s between arriving and clicking, and make the UI react 0.05 to 0.5 s after the click. Add every click time to `clicks`.
- A step line's time is the moment its action starts on screen.
- Colours come from `B`. Text passes through `esc()`. Positions are absolute pixels on the 1920 x 1080 stage.
- Logos are inline SVG. If a raster image is needed, embed it at twice its displayed size or more.
- Text contrast: 4.5 to 1 or better, including grey labels.
- A looping guide ends on the end card and restarts on the title card, both in the dark colour, so the loop is clean.
- The page `<title>` is the guide's name in two to four words.

## Export to MP4

Four formats. **Default is high.** High does not mean 4K.

| Format | Size | Frames per second | When |
|---|---|---|---|
| very high | 3840 x 2160 | 60 | only when asked (large file, slow to render) |
| high (default) | 2560 x 1440 | 60 | sharp on Retina and large screens |
| medium | 1920 x 1080 | 60 | the size the guide is designed at |
| low | 1280 x 720 | 30 | email, chat, slow connections |

All formats: every frame is computed at its exact time (not a screen recording), H.264, crf 15, yuv420p, faststart, no audio, real fonts.

1. Save the page as `demo.html` and the script below as `export.js` in one scratch folder. Chromium, Playwright and ffmpeg must be available; do not reinstall Playwright if it is already there.
2. Test a few seconds and look at a frame from each step:
   `node export.js demo.html test.mp4 high 5 9` then `ffmpeg -ss 2 -i test.mp4 -frames:v 1 f.png`
   Check fonts (no fallback font), logo, nothing clipped.
3. Full export, named after the guide and the format: `node export.js demo.html <Name>-1440p.mp4 high` (`-2160p`, `-1080p`, `-720p` for the others). Run it in the background for long guides: roughly 5 minutes per minute of video in high on 2 cores.
4. Verify with `ffprobe` (size, frame rate, frame count = duration x fps), look at three frames spread over the video, report the real file weight, then send the file. Say that frames were checked, not the full playback.

The script loads fonts from npm (`@fontsource/<family>`), because cloud workspaces usually cannot reach Google Fonts. List every font family the page uses in `window.DEMO.fonts`.

If the animation is not built on this template (for example a Claude Design canvas page), this export does not apply as is: the page needs a global `window.DEMO = {duration, fonts, setT}` where `setT(seconds)` renders that exact frame.

## demo.html (template with a worked example)

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Product Demo Guide</title>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap">
<style>
  html, body { margin: 0; height: 100%; background: #0a0a0a; overflow: hidden; font-family: Inter, Helvetica, Arial, sans-serif; -webkit-font-smoothing: antialiased; }
  #fit { position: absolute; inset: 0 0 48px 0; display: flex; align-items: center; justify-content: center; overflow: hidden; }
  #stage { width: 1920px; height: 1080px; flex-shrink: 0; position: relative; overflow: hidden; transform-origin: center; }
  #bar { position: absolute; left: 0; right: 0; bottom: 0; height: 48px; display: flex; align-items: center; gap: 12px; padding: 0 16px; box-sizing: border-box; color: #e8e8e8; font-size: 14px; font-variant-numeric: tabular-nums; }
  #bar button { width: 72px; height: 32px; border: 0; border-radius: 6px; background: #2a2a2a; color: #fff; font: inherit; cursor: pointer; }
  #bar input { flex: 1; accent-color: #fff; }
  body.export #bar { display: none; }
  body.export #fit { inset: 0; display: block; }
  body.export #stage { transform: none !important; }
</style>
</head>
<body>
<div id="fit"><div id="stage"></div></div>
<div id="bar"><button id="play" type="button">Pause</button><input id="seek" type="range" min="0" step="0.01" value="0" aria-label="Timeline"><span id="clock"></span></div>
<script>
/* ============ 1. BRAND: the only block that changes per company ============ */
const B = {
  product: 'Lumen Desk',                       // product name, shown on the title card and in the app bar
  kicker: 'Guide for new team members',        // who the guide is for
  title: 'Create a support ticket',            // what the viewer will be able to do
  subtitle: 'From the ticket list to a request sent, in two steps.',
  header: 'Lumen Desk guide',                  // small uppercase label, top left of every step
  url: 'desk.lumen.example/tickets',           // typed in the browser bar
  accent: 'rgb(0,70,255)',                     // primary action colour of the real product
  dark: 'rgb(28,28,69)',                       // app bar, title card and end card
  ink: 'rgb(0,0,46)', sec: 'rgb(83,83,89)', muted: 'rgb(118,118,140)', line: 'rgb(217,221,232)', page: 'rgb(246,246,248)',
  font: 'Inter',                               // UI font of the real product (loaded in <head>)
  // Company logo as INLINE SVG, once for light backgrounds and once for dark ones.
  logo: h => `<svg aria-hidden="true" height="${h}" viewBox="0 0 34 34"><rect width="34" height="34" rx="8" fill="rgb(0,70,255)"/><circle cx="17" cy="17" r="7" fill="#fff"/></svg>`,
  logoOnDark: h => `<svg aria-hidden="true" height="${h}" viewBox="0 0 34 34"><rect width="34" height="34" rx="8" fill="#fff"/><circle cx="17" cy="17" r="7" fill="rgb(28,28,69)"/></svg>`,
  links: [['Lumen Desk', 'desk.lumen.example'], ['Questions', 'Contact the support team']],
  disclaimer: 'Demo data only. Names and requests are fictitious.',
};

/* ============ 2. SCENES: the outline. Durations in seconds, in order ============ */
const SCENES = [
  { name: 'Intro', dur: 5 },
  { name: 'Open', dur: 9 },
  { name: 'Fill', dur: 13 },
  { name: 'Outro', dur: 5 },
];

/* ============ 3. STEPS: the right-hand panel. b = [seconds after the step starts, text] ============ */
const STEPS = [
  { cue: 'Open', end: 'Fill', title: 'Open a new ticket', b: [[0.6, 'Select New ticket'], [3.4, 'The form opens'], [5, 'Fields marked * are required']] },
  { cue: 'Fill', end: 'Outro', title: 'Describe the request', b: [[1.2, 'A clear title the team can recognise'], [4.8, 'Set the priority'], [8.6, 'Save sends it to the support team'], [10.2, 'It appears in the list as Open']] },
];

/* ============ Engine: everything on screen is a pure function of T ============ */
const CUES = {}; let TOTAL = 0; SCENES.forEach(s => { CUES[s.name] = TOTAL; TOTAL += s.dur; });
let T = 0;
const clamp = (v, a, b) => Math.max(a, Math.min(b, v));
const EASE = { enter: t => 1 - Math.pow(1 - t, 3), move: t => t < .5 ? 4 * t * t * t : 1 - Math.pow(-2 * t + 2, 3) / 2, pop: t => 1 + 2.70158 * Math.pow(t - 1, 3) + 1.70158 * Math.pow(t - 1, 2) };
const A = (st, en, from = 0, to = 1, e = EASE.enter) => from + (to - from) * e(clamp((T - st) / (en - st), 0, 1)); // animate
const win = (st, en, fd = 0.4) => Math.min(A(st, st + fd), en == null ? 1 : 1 - A(en - fd, en));                  // visible between two times
const on = (st, en) => T >= st && T < en;
const typed = (s, st, dur) => s.slice(0, Math.round(s.length * clamp((T - st) / dur, 0, 1)));                     // typing
const caret = (st, en) => on(st, en) && Math.floor(T * 2.4) % 2 === 0 ? `<span style="color:${B.accent}">|</span>` : '';
const esc = s => String(s).replace(/[&<>"]/g, c => ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;' }[c]));
const box = (x, y, w, h, css, inner = '') => `<div style="position:absolute;left:${x}px;top:${y}px;width:${w}px;height:${h}px;box-sizing:border-box;${css}">${inner}</div>`;

/* Layout grid of the 1920 x 1080 stage */
const SX = 72, SY = 150, SW = 1180, SH = 820;   // app window (browser frame)
const PX = 1308, PW = 540;                      // step panel

/* ---- Simplified UI kit: enlarged, flat versions of the product's real components ---- */
function field({ x, y, w, h = 48, label, req, value = '', ph = '', focus, chev, tail = '' }) {
  const ring = focus ? `inset 0 0 0 2px ${B.accent}, 0 0 0 3px rgba(0,70,255,.28)` : `inset 0 0 0 1px ${B.muted}`;
  return `<div style="position:absolute;left:${x}px;top:${y - 34}px;width:${w}px">
    <div style="font-size:19px;line-height:26px;margin-bottom:8px;color:${B.ink}">${esc(label)}${req ? '<span style="color:rgb(218,30,40)"> *</span>' : ''}</div>
    <div style="height:${h}px;border-radius:4px;background:#fff;box-shadow:${ring};display:flex;align-items:center;padding:0 16px;box-sizing:border-box;font-size:19px;white-space:nowrap">
      <span style="flex:1;color:${value ? B.ink : B.muted}">${value ? esc(value) : esc(ph)}${tail}</span>${chev ? `<span style="font-size:16px;color:${B.ink}">&#8964;</span>` : ''}
    </div></div>`;
}
function dropdown({ x, y, w, items, hi, opacity }) {
  return box(x, y, w, items.length * 44 + 8, `background:#fff;border-radius:4px;box-shadow:0 8px 24px rgba(16,24,40,.18), inset 0 0 0 1px ${B.line};padding:4px 0;opacity:${opacity};z-index:5`,
    items.map((it, i) => `<div style="height:44px;display:flex;align-items:center;padding:0 16px;font-size:19px;background:${i === hi ? 'rgb(236,239,248)' : 'transparent'};color:${B.ink};font-weight:${i === hi ? 600 : 400}">${esc(it)}</div>`).join(''));
}
const button = (x, y, w, label, primary, pressed) => box(x, y, w, 48, `border-radius:4px;display:flex;align-items:center;justify-content:center;font-size:19px;font-weight:600;${primary ? `background:${pressed ? 'rgb(0,42,153)' : B.accent};color:#fff` : `background:#fff;color:${B.ink};box-shadow:inset 0 0 0 1px ${B.ink}`}`, esc(label));
const pill = (label, bg, fg) => `<span style="display:inline-block;padding:4px 12px;border-radius:999px;font-size:15px;font-weight:700;letter-spacing:.04em;text-transform:uppercase;background:${bg};color:${fg}">${esc(label)}</span>`;

/* ---- The frame ---- */
function frame() {
  const sO = CUES.Open, sF = CUES.Fill, sE = CUES.Outro;

  // Cursor path: [time, x, y]. It glides between points and rests when two points share a position.
  const K = [[0, 900, 700], [sO + 1.2, 900, 700], [sO + 2.3, 1130, 312], [sO + 3.4, 1130, 312], [sO + 4.5, 700, 650],
    [sF + 0.3, 700, 650], [sF + 1, 520, 446], [sF + 4.2, 520, 446], [sF + 4.7, 420, 566], [sF + 5.4, 420, 566], [sF + 6, 380, 706], [sF + 7, 380, 706],
    [sF + 8.4, 990, 746], [sF + 9, 990, 746], [sF + 10, 900, 820]];
  const clicks = [sO + 2.5, sF + 1.2, sF + 4.8, sF + 6.8, sF + 8.6];
  let cx = K[K.length - 1][1], cy = K[K.length - 1][2];
  for (let i = 0; i < K.length - 1; i++) if (T < K[i + 1][0]) { const p = EASE.move(clamp((T - K[i][0]) / Math.max(.001, K[i + 1][0] - K[i][0]), 0, 1)); cx = K[i][1] + (K[i + 1][1] - K[i][1]) * p; cy = K[i][2] + (K[i + 1][2] - K[i][2]) * p; break; }

  // State derived from time
  const modal = A(sO + 3, sO + 3.6) * (1 - A(sF + 9, sF + 9.6));
  const ddOpen = on(sF + 4.85, sF + 6.85), priority = T >= sF + 6.8 ? 'High' : '';
  const newRow = A(sF + 9.8, sF + 10.3);
  const rows = [['T-1042', 'Badge reader offline, entrance B', 'Open'], ['T-1038', 'New laptop for a starter', 'In progress'], ['T-1031', 'Wi-Fi drops on floor 3', 'In progress']];
  const stepIdx = STEPS.reduce((n, s, i) => T >= CUES[s.cue] ? i : n, -1);

  let h = `<div style="position:absolute;inset:0;background:${B.page};color:${B.ink};font-family:${B.font}, Helvetica, Arial, sans-serif">`;

  // Header: company logo, guide label, milestones
  h += `<div style="position:absolute;left:72px;top:52px;right:72px;display:flex;justify-content:space-between;align-items:center">
    <div style="display:flex;align-items:center;gap:20px">${B.logo(34)}<span style="width:1px;height:34px;background:${B.line}"></span>
      <span style="font-size:24px;font-weight:600;letter-spacing:.1em;text-transform:uppercase;color:${B.accent}">${esc(B.header)}</span></div>
    <div style="display:flex;gap:8px">${STEPS.map((s, i) => `<span style="width:48px;height:6px;border-radius:999px;background:${i <= stepIdx ? B.accent : B.line}"></span>`).join('')}</div></div>`;

  // Browser frame: kept minimal on purpose (three dots and the address)
  h += box(SX, SY, SW, SH, `background:#fff;border-radius:8px;box-shadow:0 1px 2px rgba(28,28,69,.08), inset 0 0 0 1px ${B.line};overflow:hidden`,
    `<div style="height:52px;background:rgb(237,237,240);display:flex;align-items:center;gap:10px;padding:0 20px">
      ${[0, 1, 2].map(() => '<span style="width:12px;height:12px;border-radius:999px;background:rgb(191,191,191)"></span>').join('')}
      <div style="margin-left:24px;flex:1;height:32px;background:#fff;border-radius:999px;display:flex;align-items:center;padding:0 18px;font-size:18px;color:${B.sec}">${esc(typed(B.url, sO + 0.3, 1.2))}${caret(sO + 0.2, sO + 1.8)}</div></div>`);

  // The app, simplified: app bar, one toolbar action, one table
  h += box(SX, SY + 52, SW, 60, `background:${B.dark};color:#fff;display:flex;align-items:center;padding:0 24px;font-size:24px;font-weight:700`, esc(B.product));
  h += button(1040, 288, 180, '+ New ticket', true, on(sO + 2.5, sO + 2.9));
  h += box(96, 360, 1132, 44, `background:rgb(35,59,114);color:#fff;border-radius:4px 4px 0 0;display:flex;align-items:center;font-size:17px;font-weight:600`,
    `<span style="width:160px;padding-left:20px">Ticket</span><span style="flex:1">Title</span><span style="width:200px">Status</span>`);
  const allRows = [['T-1047', 'Projector not working in room 4B', 'Open', newRow], ...rows.map(r => [...r, 1])];
  allRows.forEach((r, i) => {
    const y = 404 + (i === 0 ? 0 : (i - 1) * 64 + 64 * newRow);
    h += box(96, y, 1132, 64, `border-bottom:1px solid ${B.line};display:flex;align-items:center;font-size:19px;opacity:${r[3]};background:${i === 0 ? 'rgb(240,244,255)' : '#fff'}`,
      `<span style="width:160px;padding-left:20px;color:${B.sec}">${r[0]}</span><span style="flex:1;font-weight:${i === 0 ? 600 : 400}">${esc(r[1])}</span><span style="width:200px">${r[2] === 'Open' ? pill('Open', 'rgb(233,246,237)', 'rgb(14,96,39)') : pill('In progress', 'rgb(240,235,255)', 'rgb(74,40,160)')}</span>`);
  });

  // The form (modal). Real labels, real order, only the fields the guide talks about.
  if (modal > 0) {
    h += box(SX, SY + 52, SW, SH - 52, `background:rgba(28,28,69,.5);opacity:${modal};border-radius:0 0 8px 8px`);
    h += `<div style="opacity:${modal};transform:translateY(${(1 - modal) * 16}px);position:absolute;inset:0">`;
    h += box(232, 300, 860, 520, `background:rgb(250,251,254);border-radius:8px;box-shadow:0 16px 48px rgba(16,24,40,.3);overflow:hidden`,
      `<div style="height:64px;background:rgb(35,59,114);color:#fff;display:flex;align-items:center;padding:0 28px;font-size:26px;font-weight:600">Create a ticket</div>`);
    h += field({ x: 272, y: 446, w: 780, label: 'Title', req: true, ph: 'Enter a title', value: typed('Projector not working in room 4B', sF + 1.4, 2.4), focus: on(sF + 1.2, sF + 4.6), tail: caret(sF + 1.2, sF + 4.6) });
    h += field({ x: 272, y: 566, w: 380, label: 'Priority', req: true, ph: 'Select a priority', value: priority, chev: true, focus: ddOpen });
    if (ddOpen) h += dropdown({ x: 272, y: 618, w: 380, items: ['Low', 'Normal', 'High'], hi: T < sF + 5.6 ? -1 : 2, opacity: win(sF + 4.85, sF + 6.85, 0.15) });
    h += button(796, 722, 120, 'Cancel', false) + button(932, 722, 120, 'Save', true, on(sF + 8.6, sF + 9));
    h += `</div>`;
  }

  // Step panel: step count, title, then bullets that arrive when the action happens
  STEPS.forEach((s, i) => {
    const st = CUES[s.cue], en = CUES[s.end], o = win(st, en); if (o <= 0) return;
    let last = -1; s.b.forEach((b, j) => { if (T >= st + b[0]) last = j; });
    h += `<div style="position:absolute;left:${PX}px;top:${SY}px;width:${PW}px;opacity:${o};transform:translateY(${(1 - A(st, st + 0.5)) * 16}px)">
      <div style="font-size:24px;font-weight:600;letter-spacing:.1em;text-transform:uppercase;color:${B.accent}">Step ${i + 1} of ${STEPS.length}</div>
      <div style="font-size:48px;font-weight:700;line-height:1.1;letter-spacing:-.015em;margin-top:14px">${esc(s.title)}</div>
      <div style="height:1px;background:${B.line};margin-top:28px"></div>
      <div style="display:flex;flex-direction:column;gap:20px;margin-top:28px">${s.b.map((b, j) => { const bo = A(st + b[0], st + b[0] + 0.4); return `<div style="opacity:${bo};transform:translateY(${(1 - bo) * 12}px);font-size:28px;line-height:1.4;padding-left:20px;border-left:4px solid ${j === last ? B.accent : B.line};color:${j === last ? B.ink : B.sec};font-weight:${j === last ? 600 : 400}">${esc(b[1])}</div>`; }).join('')}</div></div>`;
  });

  // Cursor with click ripples
  h += `<div style="position:absolute;left:${cx}px;top:${cy}px;width:0;height:0;z-index:20">
    ${clicks.map(c => { const p = (T - c) / 0.5; return p >= 0 && p < 1 ? `<span style="position:absolute;left:-22px;top:-22px;width:44px;height:44px;border-radius:999px;border:3px solid ${B.accent};transform:scale(${0.4 + p});opacity:${1 - p}"></span>` : ''; }).join('')}
    <svg aria-hidden="true" width="30" height="38" viewBox="0 0 30 38" style="position:absolute;left:-3px;top:-2px"><path d="M3 2 L3 30 L10 23 L15 35 L20 33 L15 21 L25 21 Z" fill="#fff" stroke="${B.ink}" stroke-width="2.2" stroke-linejoin="round"/></svg></div>`;

  // Title card and end card, in the brand's dark colour
  const card = (opacity, z, inner) => opacity <= 0 ? '' : `<div style="position:absolute;inset:0;background:${B.dark};color:#fff;opacity:${opacity};padding:96px 120px;box-sizing:border-box;display:flex;flex-direction:column;z-index:${z}">${inner}</div>`;
  h += card(1 - A(sO - 0.6, sO), 30, `
    <div style="font-size:24px;font-weight:600;letter-spacing:.12em;text-transform:uppercase;color:rgb(176,198,255);opacity:${A(0.2, 0.8)}">${esc(B.kicker)}</div>
    <div style="font-size:112px;font-weight:700;letter-spacing:-.02em;margin-top:180px;opacity:${A(0.5, 1.2)};transform:translateY(${(1 - A(0.5, 1.2)) * 20}px)">${esc(B.product)}</div>
    <div style="font-size:60px;font-weight:700;letter-spacing:-.015em;margin-top:12px;opacity:${A(1.1, 1.8)}">${esc(B.title)}</div>
    <div style="font-size:32px;color:rgb(217,221,232);margin-top:20px;opacity:${A(1.6, 2.3)}">${esc(B.subtitle)}</div>
    <div style="margin-top:auto">${B.logoOnDark(44)}</div>`);
  h += card(A(sE, sE + 0.6), 31, `
    <div style="font-size:24px;font-weight:600;letter-spacing:.12em;text-transform:uppercase;color:rgb(176,198,255)">Useful links</div>
    <div style="font-size:60px;font-weight:700;letter-spacing:-.015em;margin-top:24px">Your request is with the team.</div>
    <div style="display:flex;flex-direction:column;gap:22px;margin-top:64px;font-size:32px">${B.links.map((l, i) => `<div style="display:grid;grid-template-columns:260px 1fr;opacity:${A(sE + 0.6 + i * 0.3, sE + 1 + i * 0.3)}"><strong style="color:rgb(255,190,92)">${esc(l[0])}</strong><span>${esc(l[1])}</span></div>`).join('')}</div>
    <div style="margin-top:auto;display:flex;align-items:center;gap:24px">${B.logoOnDark(44)}<span style="font-size:22px;color:rgb(160,160,185)">${esc(B.disclaimer)}</span></div>`);
  return h + '</div>';
}

/* ============ Player. window.DEMO is what the video export drives ============ */
const stage = document.getElementById('stage'), seek = document.getElementById('seek'), clock = document.getElementById('clock'), playBtn = document.getElementById('play');
let lastHtml = '', playing = true, prev = null;
const fmt = s => `${Math.floor(s / 60)}:${String(Math.floor(s % 60)).padStart(2, '0')}`;
function setT(t) { T = clamp(t, 0, TOTAL); const html = frame(); if (html !== lastHtml) { stage.innerHTML = html; lastHtml = html; } seek.value = T; clock.textContent = `${fmt(T)} / ${fmt(TOTAL)}`; }
window.DEMO = { duration: TOTAL, width: 1920, height: 1080, fonts: [B.font], setT };
seek.max = TOTAL;
if (new URLSearchParams(location.search).has('export')) { document.body.classList.add('export'); playing = false; setT(0); }
else {
  const fit = () => { const f = document.getElementById('fit'); stage.style.transform = `scale(${Math.min(f.clientWidth / 1920, f.clientHeight / 1080)})`; };
  fit(); addEventListener('resize', fit);
  const toggle = () => { playing = !playing; playBtn.textContent = playing ? 'Pause' : 'Play'; prev = null; };
  playBtn.addEventListener('click', toggle);
  addEventListener('keydown', e => { if (e.code === 'Space' && e.target === document.body) { e.preventDefault(); toggle(); } });
  seek.addEventListener('input', () => { if (playing) toggle(); setT(+seek.value); });
  const tick = now => { if (playing) { if (prev != null) setT(T + (now - prev) / 1000 >= TOTAL ? 0 : T + (now - prev) / 1000); prev = now; } requestAnimationFrame(tick); };
  setT(0); requestAnimationFrame(tick);
}
</script>
</body>
</html>
```

## export.js

```js
// Export a demo guide page to MP4.  Usage: node export.js <demo.html> <out.mp4> [veryhigh|high|medium|low] [fromSec toSec]
const { chromium } = require('playwright'); const { spawn, execSync } = require('child_process'); const fs = require('fs'), path = require('path');
const FORMATS = { veryhigh: [2, 60], high: [1.3333334, 60], medium: [1, 60], low: [0.6666667, 30] }; // [device scale factor, fps]
const [, , page, out, format = 'high', from, to] = process.argv;
if (!page || !out || !FORMATS[format]) { console.error('usage: node export.js <demo.html> <out.mp4> [veryhigh|high|medium|low] [fromSec toSec]'); process.exit(2); }
const [DSF, FPS] = FORMATS[format];

async function part(browser, a, b, file) {
  const p = await browser.newPage({ viewport: { width: 1920, height: 1080 }, deviceScaleFactor: DSF });
  p.on('pageerror', e => console.error('page error:', e.message));
  await p.route(/fonts\.(googleapis|gstatic)\.com/, r => r.abort());            // the workspace cannot reach Google Fonts
  await p.goto('file://' + path.resolve(page) + '?export');
  for (const fam of await p.evaluate(() => window.DEMO.fonts)) {                 // real fonts, from npm
    const slug = fam.toLowerCase().replace(/\s+/g, '-'), dir = path.join(__dirname, 'node_modules/@fontsource', slug, 'files');
    if (!fs.existsSync(dir)) execSync(`npm i --no-audit --no-fund @fontsource/${slug}`, { cwd: __dirname, stdio: 'ignore' });
    const faces = fs.readdirSync(dir).map(f => [f, f.match(new RegExp(`^${slug}-latin-(\\d+)-(normal|italic)\\.woff2$`))]).filter(x => x[1]);
    if (!faces.length) throw new Error('no font files for ' + fam);
    await p.addStyleTag({ content: faces.map(([f, m]) => `@font-face{font-family:'${fam}';font-weight:${m[1]};font-style:${m[2]};src:url(data:font/woff2;base64,${fs.readFileSync(path.join(dir, f)).toString('base64')})}`).join('') });
    for (const [, m] of faces) await p.evaluate(x => document.fonts.load(x), `${m[2]} ${m[1]} 16px '${fam}'`);
  }
  const ff = spawn('ffmpeg', ['-y', '-loglevel', 'error', '-f', 'image2pipe', '-framerate', String(FPS), '-c:v', 'mjpeg', '-i', '-', '-c:v', 'libx264', '-preset', 'medium', '-crf', '15', '-pix_fmt', 'yuv420p', '-r', String(FPS), file], { stdio: ['pipe', 'inherit', 'inherit'] });
  for (let i = a; i < b; i++) {
    await p.evaluate(t => { window.DEMO.setT(t); return Promise.all([...document.images].map(im => im.decode().catch(() => 0))); }, i / FPS);
    const buf = await p.screenshot({ type: 'jpeg', quality: 97 });
    if (!ff.stdin.write(buf)) await new Promise(r => ff.stdin.once('drain', r));
  }
  ff.stdin.end(); await new Promise(r => ff.on('close', r)); await p.close();
}

(async () => {
  const browser = await chromium.launch();
  const probe = await browser.newPage(); await probe.goto('file://' + path.resolve(page) + '?export');
  const dur = await probe.evaluate(() => window.DEMO.duration); await probe.close();
  const first = Math.round((from ? +from : 0) * FPS), last = Math.round((to ? Math.min(+to, dur) : dur) * FPS), mid = Math.floor((first + last) / 2);
  const tmp = out + '.parts'; fs.mkdirSync(tmp, { recursive: true });
  console.error(`${format}: ${Math.round(1920 * DSF)} x ${Math.round(1080 * DSF)} at ${FPS} fps, ${last - first} frames`);
  await Promise.all([part(browser, first, mid, path.join(tmp, 'p1.mp4')), part(browser, mid, last, path.join(tmp, 'p2.mp4'))]); // two halves in parallel
  await browser.close();
  fs.writeFileSync(path.join(tmp, 'list.txt'), "file 'p1.mp4'\nfile 'p2.mp4'\n");
  execSync(`ffmpeg -y -loglevel error -f concat -safe 0 -i list.txt -c copy -movflags +faststart ${JSON.stringify(path.resolve(out))}`, { cwd: tmp });
  fs.rmSync(tmp, { recursive: true });
  console.error('done:', out);
})();
```

## Common mistakes

- Copying the real screen pixel for pixel: the guide becomes unreadable on video. Simplify and enlarge.
- Too many words in the step panel. A line is one short sentence; the screen shows the rest.
- Inventing a label, a status or a rule. Ask, or mark it as an open question in the storyboard.
- Animating with CSS transitions or timers: the video export then shows wrong or blurred frames.
- Exporting before looking at frames from every step.