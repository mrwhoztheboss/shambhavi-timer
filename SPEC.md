# Shambhavi Timer — Technical Specification

━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 1 — PROJECT OVERVIEW
━━━━━━━━━━━━━━━━━━━━━━━━━

**App Name:** Shambhavi Timer

**Purpose:** A guided Kriya practice timer web application for the Shambhavi Mahamudra kriya. Provides timed guidance through preparatory and main practice steps with audio cues and visual countdowns.

**Target Users:** Practitioners of Isha Yoga's Shambhavi Mahamudra kriya. Users may complete full practice or skip preparatory steps.

**Platform:** Static web app deployed to GitHub Pages. Single `index.html` file with inline CSS and JS. No server-side components.

**Tech Stack:**
- Plain HTML5, CSS3, JavaScript (no frameworks, no build tools)
- oat.css (minimal CSS reset/normalize)
- Web Audio API (for synthesized bells/chimes)
- Google Fonts (Cormorant Garamond, Lato, Tiro Devanabari)
- Local storage (optional, for user preferences)
- Screen Wake Lock API (to prevent sleep during practice)

**Constraints:** No npm, no bundlers, no external dependencies except Google Fonts. All code inline in `index.html`.

━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 2 — FILE STRUCTURE
━━━━━━━━━━━━━━━━━━━━━━━━━

```
shambhavi-timer/
├── index.html       # Complete app: HTML structure, inline CSS, inline JS
├── README.md        # Project documentation, usage instructions
└── .gitignore       # Exclude .DS_Store, node_modules, etc.
```

**No deploy.yml during planning phase.** Local development only; GitHub Actions workflow created in DEPLOYING phase.

━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 3 — APP SCREENS & FLOW
━━━━━━━━━━━━━━━━━━━━━━━━━

**Screen 1: Welcome Screen**
- **UI Elements:**
  - Title: "Shambhavi Mahamudra"
  - Subtitle: "Guided Kriya Practice"
  - Info card: "Total duration: ~25 minutes" (or dynamic based on skip checkbox)
  - Checkbox: "Skip preparatory steps" (optional, unchecked by default)
  - Primary button: "Start Practice"
- **Transitions:** "Start Practice" → Invocation Screen

**Overlay: Skip Advisory**
- Shown momentarily when "Skip preparatory" checked on Welcome
- Modal/overlay with text: "Skipping preparatory steps. Proceed directly to Shambhavi Mahamudra."
- Button: "Continue" → Invocation Screen
- Auto-dismiss after 2 seconds option (implementer chooses)

**Screen 2: Invocation Screen (Pre-Practice)**
- **UI Elements:**
  - Title: "Invocation"
  - Pavamana Mantra (Sanskrit in Devanagari, transliteration, English meaning)
  - Instruction: "Take a moment to recite this invocation before beginning."
  - Primary button: "Begin Practice"
- **Transitions:** "Begin Practice" → Active Timer Screen (step index depends on skip flag)

**Screen 3: Active Timer Screen**
- **UI Elements:**
  - Progress bar (top, thin line showing overall completion across all steps)
  - Step name (large heading)
  - Step description/guidance text (below heading)
  - SVG ring countdown timer (centered)
  - Ring center displays MM:SS countdown
  - Pause/Resume button (secondary)
  - Skip button (secondary, hidden on last step)
- **Transitions:**
  - Step completes → advance to next step
  - Last step completes → Completion Screen
  - Skip button pressed → immediate step advance

**Screen 4: Completion Screen**
- **UI Elements:**
  - Title: "Practice Complete"
  - Closing Pavamana Mantra (above title, labeled "Closing Invocation")
  - Summary statistics: total steps completed, total time
  - Primary button: "Start New Session" → Welcome Screen
- **Transitions:** "Start New Session" → Welcome Screen

**Skip Advisory Modal Details:**
- Not counted as separate screen; overlay on Invocation or simple interstitial
- Shows only when preparatory steps are skipped
- User cannot skip this advisory (must acknowledge or auto-dismiss)

━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 4 — KRIYA STEPS DATA STRUCTURE
━━━━━━━━━━━━━━━━━━━━━━━━━

JavaScript `STEPS` array constant:

```javascript
const STEPS = [
  // PREPARATORY (phase: "preparatory")
  { id: 1, phase: "preparatory", name: "Butterfly / Patangasana", duration: 120, description: "Sit in butterfly pose, gentle bouncing for 2 minutes", isOptional: true, type: "standard" },
  { id: 2, phase: "preparatory", name: "Shishupalasana Right Leg", duration: 120, description: "Place right leg behind head, hold for 2 minutes", isOptional: true, type: "standard" },
  { id: 3, phase: "preparatory", name: "Shishupalasana Left Leg", duration: 120, description: "Place left leg behind head, hold for 2 minutes", isOptional: true, type: "standard" },
  { id: 4, phase: "preparatory", name: "Nadi Vibhajan", duration: 240, description: "Alternate nostril breathing for 4 minutes", isOptional: true, type: "standard" },

  // SHAMBHAVI MAHAMUDRA (phase: "shambhavi")
  { id: 5, phase: "shambhavi", name: "Sukh Kriya", duration: 360, description: "Gentle up-and-down movement with breath, 6 minutes", isOptional: false, type: "standard" },
  { id: 6, phase: "shambhavi", name: "AUM Chanting", duration: 210, description: "21 rounds of AUM chanting, ~10 seconds per round", isOptional: false, type: "aum_chanting" },
  { id: 7, phase: "shambhavi", name: "Vipareet Swasa", duration: 240, description: "Inverted breath, 4 minutes", isOptional: false, type: "standard" },
  { id: 8, phase: "shambhavi", name: "Bandas – Fullness of Breath", duration: 30, description: "Hold breath with body locks, 30 seconds", isOptional: false, type: "standard" },
  { id: 9, phase: "shambhavi", name: "Bandas – Emptiness of Breath", duration: 30, description: "Hold breath with body locks, 30 seconds", isOptional: false, type: "standard" },
  { id: 10, phase: "shambhavi", name: "Observing Breath", duration: 360, description: "Seated observation of natural breath, 6 minutes", isOptional: false, type: "standard" }
];
```

**Notes:**
- Total duration with preparatory: 1620s (27 minutes)
- Total duration without preparatory: 1260s (21 minutes)
- `type: "aum_chanting"` triggers round-based countdown behavior (ring resets every 10 seconds for 21 rounds)

━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 5 — TIMER ENGINE SPEC
━━━━━━━━━━━━━━━━━━━━━━━━━

**TimerEngine** object:

Properties:
- `currentStepIndex` (number)
- `remainingTime` (seconds, float)
- `totalDuration` (seconds, for current step)
- `intervalId` (setInterval reference or null)
- `isPaused` (boolean)
- `startTime` (timestamp when current step started, used for drift correction)
- `expectedEndTime` (calculated timestamp for step completion)
- `onTick` (callback(currentStepIndex, remainingTime, totalDuration))
- `onStepComplete` (callback)
- `onSessionComplete` (callback)

Functions:

**`start(stepIndex)`**
- Parameters: `stepIndex` (number)
- Sets `currentStepIndex`, initializes `remainingTime` from `STEPS[stepIndex].duration`
- Calculates `expectedEndTime = Date.now() + remainingTime * 1000`
- Starts tick loop (`tick()` on setInterval every 100ms)
- Sets `isPaused = false`

**`pause()`**
- Clears interval
- Sets `isPaused = true`
- `remainingTime` continues to hold adjusted value

**`resume()`**
- Recalculates `expectedEndTime = Date.now() + remainingTime * 1000`
- Restarts tick loop
- Sets `isPaused = false`

**`skip()`**
- Stops current timer (clear interval)
- Calls `advanceToNextStep()` immediately

**`destroy()`**
- Stops timer, resets all state, clears intervals

**`tick()`**
- Called every 100ms
- Using `Date.now()`: `remainingTime = (expectedEndTime - Date.now()) / 1000`
- If `remainingTime <= 0`: set `remainingTime = 0`, stop timer, call `onStepComplete()`
- Calls `onTick()` with current values
- For AUM chanting type: trigger round logic separately (see below)

**`updateRing(remaining, total)`**
- Client code (UI) uses this to update SVG ring
- Called from `onTick()` callback
- Calculates percentage: `(total - remaining) / total`

**`onStepComplete()`**
- Stops current step
- If current step type is "aum_chanting", play round bell before advancing
- Calls TimerEngine's internal `advanceToNextStep()`

**`advanceToNextStep()`**
- Increments `currentStepIndex`
- If past last step: call `onSessionComplete()` and stop
- Else: call `start(newIndex)` for next step

**Drift Correction:**
- Never rely on decrementing `remainingTime` by 0.1 every tick
- Always base remaining on `Date.now()`: `remaining = (expectedEndTime - now) / 1000`
- This corrects for setInterval drift, tab throttling, and GC pauses

**AUM Chanting Handling:**
- The ring shows total remaining time (210s), but internally track round counter separately in UI layer
- Every time `remainingTime` wraps from N to N-1 where `(total - remainingTime) % 10 === 0` (round boundary), trigger AUM round bell
- Better: UI layer observes `onTick` and when `Math.floor((total - remaining) / 10) > prevRound`, increment round, play bell
- 21 rounds total; after round 21, no round 22 even if time remains (should be exactly 210s = 21×10)

**Pause Storage:**
- `remainingTime` kept as float in memory
- Optional: also save to localStorage on pause for recovery on refresh (edge case 16)

━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 6 — SOUND ENGINE SPEC
━━━━━━━━━━━━━━━━━━━━━━━━━

**AudioContext Singleton:**
- Single `window.audioCtx` (create on first user gesture if not exists)
- Resume before every playback: `audioCtx.resume()`
- All sounds wrapped in try/catch; fall silent on AudioContext failure

**Sound 1: Transition Bell (Step End)**
- Oscillator: sine wave
- Frequency: 528 Hz
- Envelope:
  - Gain 0 → 0.6 linear, attack 0.01s
  - Sustain 0.6 for 0.3s
  - Exponential decay 0.6 → 0.001, release 0.9s
- Total duration ~1.2s
- Optional: second harmonic oscillator at 528×2 = 1056 Hz, gain 0.2 for brightness

**Sound 2: AUM Round Bell (Each AUM Round Starts)**
- Oscillator: sine wave
- Frequency: 432 Hz
- Envelope:
  - Gain 0 → 0.4, attack 0.01s
  - Sustain 0.4 for 0.2s
  - Exponential decay 0.4 → 0.001, release 0.5s
- Total duration ~0.7s

**Sound 3: Session Complete Chime**
- Three-tone arpeggio, 450ms between tones:
  1. 528 Hz (C5), sustain 0.5s, decay 0.5s
  2. 660 Hz (E5), sustain 0.5s, decay 0.6s
  3. 792 Hz (G5), sustain 0.5s, decay 0.8s
- Gain each: 0.5–0.6
- Total sequence ~2.5s

**Implementation:**
```javascript
const audioCtx = window.audioCtx || (window.audioCtx = new (window.AudioContext || window.webkitAudioContext)());

function playBell(frequency, duration, sustainGain, releaseTime, startTime = 0) { /* ... */ }
function playTransitionBell() { playBell(528, 1.2, 0.6, 0.9); }
function playAUMBell() { playBell(432, 0.7, 0.4, 0.5); }
function playCompleteChime() { /* sequence with setTimeout for 450ms gaps */ }
```

All functions must:
1. Call `audioCtx.resume()` at start
2. Wrap in `try { ... } catch (e) { console.warn('Audio failed:', e); }`

━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 7 — SVG RING SPEC
━━━━━━━━━━━━━━━━━━━━━━━━━

**SVG Structure:**
```svg
<svg viewBox="0 0 200 200" class="timer-ring">
  <circle class="ring-bg" cx="100" cy="100" r="90" />
  <circle class="ring-fill" cx="100" cy="100" r="90" />
  <text class="ring-text" x="100" y="100">MM:SS</text>
</svg>
```

**Geometry:**
- Radius `r = 90` (leaves padding in 200×200 viewBox)
- Circumference = `2 × π × 90 ≈ 565.48`
- `stroke-dasharray = 565.48`
- `stroke-dashoffset` = `circumference × (1 - remaining / total)`
- `stroke-linecap = round`

**Ring Size:**
- Actual rendered size controlled by CSS: `width` and `height` set to `min(70vw, 280px)`
- Container centers with `margin: 0 auto`

**Colors:**
- Ring background: `var(--ring-bg)` (light: #DDD0BC, dark: #3A3025)
- Ring fill: `var(--accent)` (light: #C8832A, dark: #D4922E)
- Warning state (last 10s): `color: var(--accent-glow)` (#E8A040) or CSS class `.ring-warning`
- Text: `var(--text)`

**AUM Chanting Mode:**
- Same ring, but `remaining` resets to 10 every round
- Progress bar at top shows cumulative progress across all 21 rounds

**Text Display:**
- Large monospace or tabular-nums font: `font-family: 'Cormorant Garamond', serif; font-size: clamp(2.5rem, 8vw, 4rem); font-weight: 600;`
- Format: `MM:SS` (zero-padded)
- Text anchor: `middle`, dominant-baseline: `central`

**Animations:**
- `stroke-dashoffset` transition: `0.2s ease-out` (only if `prefers-reduced-motion: no-preference`)
- Throb effect on ring with CSS `@keyframes` for gentle pulse (optional)

━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 8 — PAVAMANA MANTRA SPEC
━━━━━━━━━━━━━━━━━━━━━━━━━

**Google Font:** Tiro Devanabari for Sanskrit

**Pre-Practice Invocation (Invocation Screen):**

Sanskrit (Devanagari, centered):
```
ॐ असतो मा सद्गमय ।
तमसो मा ज्योतिर्गमय ।
मृत्योर्मा अमृतं गमय ।
ॐ शान्तिः शान्तिः शान्तिः ॥
```

Transliteration (Cormorant Garamond):
```
Om Asato Maa Sad-Gamaya |
Tamaso Maa Jyotir-Gamaya |
Mrtyor-Maa Amrtam Gamaya |
Om Shaantih Shaantih Shaantih ||
```

English meaning (Lato 300/400):
> "Lead me from the unreal to the Real.
> Lead me from darkness to Light.
> Lead me from death to Immortality.
> Om Peace, Peace, Peace."

Instruction: "Take a moment to recite this invocation before beginning." (class="instruction")

Font sizes:
- Sanskrit: `clamp(1.1rem, 3vw, 1.45rem)`
- Transliteration: `clamp(1rem, 2.5vw, 1.3rem)`
- English: `clamp(0.9rem, 2vw, 1.1rem)`

**Post-Practice Invocation (Completion Screen):**
Same text, positioned above the "Practice Complete" heading
Label: "Closing Invocation" (small, all-caps, letter-spacing)

**Line Breaks:** Use `<br>` or separate `<p>` with margin. Preserve visual line breaks matching the verse structure.

━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 9 — VISUAL DESIGN TOKENS
━━━━━━━━━━━━━━━━━━━━━━━━━

**CSS Variables (Light Mode):**
```css
:root {
  --bg: #F5EFE6;
  --bg-card: #FDF8F2;
  --text: #2C2416;
  --text-muted: #7A6A52;
  --accent: #C8832A;
  --accent-glow: #E8A040;
  --ring-bg: #DDD0BC;
  --border: #E0D5C5;
}
```

**Dark Mode (`prefers-color-scheme: dark`):**
```css
@media (prefers-color-scheme: dark) {
  :root {
    --bg: #1A1510;
    --bg-card: #231E17;
    --text: #F0E8DC;
    --text-muted: #A89070;
    --accent: #D4922E;
    --ring-bg: #3A3025;
  }
}
```

**Typography Stack:**

```css
/* Google Fonts import */
@import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@400;600&family=Lato:wght@300;400&family=Tiro+Devanabari&display=swap');

/* Font families */
--font-heading: 'Cormorant Garamond', Georgia, serif;
--font-body: 'Lato', system-ui, sans-serif;
--font-sanskrit: 'Tiro Devanabari', Georgia, serif;

/* Weights */
--font-light: 300;
--font-regular: 400;
--font-semibold: 600;
```

**Base Styles:**
- `box-sizing: border-box`
- `body`: `background: var(--bg); color: var(--text); font-family: var(--font-body); min-height: 100vh;`
- Container: `max-width: 800px; margin: 0 auto; padding: 1rem;`

━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 10 — RESPONSIVE BREPOINTS
━━━━━━━━━━━━━━━━━━━━━━━━━

**320px (minimum):**
- Container padding: 0.75rem
- Ring size: `min(65vw, 200px)`
- Fonts: clamp lower bounds tightened
- Buttons: stacked vertically, full-width

**375px (iPhone SE):**
- Container: standard 1rem
- Ring: `min(68vw, 220px)`
- Button spacing: 0.75rem

**414px (iPhone Plus):**
- Ring: `min(70vw, 240px)`

**768px (tablet portrait):**
- Container: `max-width: 720px`
- Buttons can be side-by-side if only 2; width: auto, min-width 160px
- Ring: `min(60vw, 260px)`

**1024px (tablet landscape / small desktop):**
- Ring: `min(50vw, 280px)`
- Content width: ~900px

**1280px+ (desktop):**
- Ring: 280px fixed
- Container: `max-width: 800px`

**Landscape Mobile (height < 500px):**
- Reduced ring size: `min(50vw, 200px)`
- Tighter vertical spacing: `gap: 0.5rem` between major blocks
- Smaller font sizes for headings

**Orientation Change:** Reload not needed; rem/vw units + clamp() handle scaling automatically. Test that ring remains circular and text does not overflow.

━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 11 — COMPLETE EDGE CASES LIST
━━━━━━━━━━━━━━━━━━━━━━━━━

**Edge Case 1: Skip Step on Last Step**
- Condition: User clicks Skip on final step (id 10)
- Expected: Button hidden or does nothing
- Implementation: Render Skip button only if `currentStepIndex < STEPS.length - 1`

**Edge Case 2: Restart Mid-Session**
- Condition: User clicks "Start New Session" from Completion (or Welcome while timer running)
- Expected: Current timer stops cleanly, no ghost intervals
- Implementation: In `startNewSession()`, call `timerEngine.destroy()` before resetting state

**Edge Case 3: AudioContext Blocked by Browser**
- Condition: Autoplay policy blocks AudioContext creation until user gesture
- Expected: Sounds fail silently; no crash
- Implementation: Create AudioContext only after first user gesture (button click). Wrap all playback in try/catch. Resume before play.

**Edge Case 4: Wake Lock Not Supported**
- Condition: Device/browser lacks Screen Wake Lock API
- Expected: App continues without error; just no wake lock
- Implementation: Feature-detect `navigator.wakeLock`. If unavailable, proceed normally. If available, request on timer start, release on completion/destroy.

**Edge Case 5: AUM Round 21 Completes**
- Condition: Round 21 finishes (210s elapsed), remainingTime = 0
- Expected: No "round 22" bell triggered
- Implementation: Round count tracked as `Math.floor((total - remaining) / 10)`; when roundIndex >= 21, suppress round bell

**Edge Case 6: User Pauses on Last Second**
- Condition: User clicks Pause when remainingTime = 0.5 (bell about to fire)
- Expected: Pause works; on Resume, timer continues from 0.5 and bell plays at 0
- Implementation: Pause sets `isPaused` but does not auto-advance. Advance only on `remainingTime <= 0` during tick.

**Edge Case 7: Tab Hidden Mid-Timer**
- Condition: User switches tabs (pagehide/visibilitychange)
- Expected: Timer continues correctly (setInterval may throttle but drift correction maintains accuracy)
- Implementation: Rely on Date.now()diff, not counter decrements. Optionally reduce interval frequency to conserve CPU.

**Edge Case 8: Screen Width 320px**
- Condition: Ultra-narrow viewport
- Expected: All text wraps, buttons stack, ring shrinks, no horizontal scroll
- Implementation: Test at 320px emulation. Use `overflow-x: hidden` on body. Clamp fonts aggressively. Button `min-height: 48px`.

**Edge Case 9: Skip to Shambhavi (Progress Bar Recalculate)**
- Condition: Preparatory steps skipped
- Expected: Progress bar starts at 0% for step 5, and percentage computed against shambhavi-only duration (1260s)
- Implementation: Compute progress based on cumulative elapsed across active step list, not raw step indices. Use filtered steps matching the actual session flow.

**Edge Case 10: Invocation Screen Appears for Both Flows**
- Condition: Both full path and skipped path must show invocation
- Expected: Always show invocation; no variation based on skip
- Implementation: Invocation screen always rendered; only the starting step index differs after it.

**Edge Case 11: Rapid Double-Click on Skip Button**
- Condition: User clicks Skip twice quickly
- Expected: Only one skip executes; no double-advance or crash
- Implementation: `skip()` should be idempotent or set `isSkipping` flag to ignore while animating. Debounce not necessary; just guard state: if `!intervalId` return.

**Edge Case 12: Session Complete — No Ghost Intervals**
- Condition: Last step completes; tick fires after `destroy()` called
- Expected: No errors, no extra state changes
- Implementation: In `tick()`, early return if `isPaused` or `intervalId === null`. In `destroy()`, set `intervalId = null` after clearing. Guard `onStepComplete` with `isRunning` flag.

**Edge Case 13: Dark Mode Text Contrast**
- Condition: System dark mode enabled
- Expected: All text meets WCAG AA (4.5:1)
- Implementation: Verify `--text` (#F0E8DC) vs `--bg` (#1A1510) ratio > 4.5. Test with contrast checker.

**Edge Case 14: Orientation Change (Portrait ↔ Landscape)**
- Condition: Mobile device rotates
- Expected: Layout adjusts without glitch; ring resizes smoothly
- Implementation: Use vw/vh units; avoid fixed px for ring size. No JS resize listener needed.

**Edge Case 15: Google Fonts Fails to Load**
- Condition: Network blocks fonts or slow
- Expected: Fallback fonts load; layout remains readable
- Implementation: Specify fallbacks in font-family stack. FOUT acceptable.

**Edge Case 16: User Refreshes Mid-Session**
- Condition: Page reload while timer running
- Expected: Session state not recoverable (no persistence required) → Welcome Screen on load
- Optional: implement localStorage save for recovery (not required)
- Implementation: On load, always start fresh; do not attempt state recovery unless explicitly requested later.

**Edge Case 17: Multiple Rapid Pause/Resume Clicks**
- Condition: User clicks Pause/Resume repeatedly
- Expected: No timer acceleration or duplicate intervals
- Implementation: In `pause()`, if already paused, return. In `resume()`, if running, return. Guard with `isPaused` flag.

━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 12 — ACCESSIBILITY REQUIREMENTS
━━━━━━━━━━━━━━━━━━━━━━━━━

- **ARIA Live Region:** Container for step announcements: `<div aria-live="polite" aria-atomic="true" class="sr-only" id="step-status"></div>`. Update text on step start and completion.
- **Button Labels:** All buttons have clear `aria-label` or inner text. Icons-only buttons (if any) must have `aria-label`.
- **Focus Management:** On screen change, move focus to first interactive element (e.g., "Begin Practice" button). Use `tabindex="-1"` on screen containers, `element.focus()` after transition.
- **Reduced Motion:** `@media (prefers-reduced-motion: reduce)`: disable SVG ring animation, replace with opacity fade transitions only.
- **Tap Targets:** Minimum 48×48px. Add padding around smaller buttons to meet this.
- **Color Contrast:** All foreground/background combinations meet 4.5:1. Verify both light and dark modes.
- **Keyboard Navigation:** Fully keyboard-accessible: Tab through buttons, Enter/Space activates.
- **Font Scalability:** Use `rem` units; respect user's browser font size settings. No `!important` to override.
- **Semantic HTML:** Use `<main>`, `<section>`, `<h1>`, `<button>` elements properly.

━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 13 — OPEN QUESTIONS FOR CODER
━━━━━━━━━━━━━━━━━━━━━━━━━

**Q1: Progress Bar Calculation**
- **Question:** Should the progress bar on the Active Timer screen reflect cumulative elapsed time across ALL steps (including skipped preparatory) or only the steps in the current session?
- **Recommended:** Only steps in current session. If preparatory skipped, progress starts at 0% and goes to 100% over the 5 shambhavi steps. This feels more natural to the user.
- **Implementation:** Generate `activeSteps` array based on skip flag (filter `STEPS` by phase if skipped), compute progress as `sum(completedDurations) + (totalDurations - remainingTime)` / `totalSessionDuration`.

**Q2: Landscape mobile layout ring size**
- **Question:** When `height < 500px` in landscape, should ring size reduce to `min(40vw, 180px)` or `min(50vw, 200px)`?
- **Recommended:** `min(50vw, 180px)` — small enough to fit content above/below, but still visible.
- **Implementation:** CSS media query `@media (max-height: 500px) and (orientation: landscape)`.

**Q3: Skip advisory auto-dismiss**
- **Question:** Should the Skip Advisory modal auto-dismiss after 2 seconds, or require explicit "Continue" click?
- **Recommended:** Auto-dismiss after 2s but also keep "Continue" button for immediate proceed. Shows message then fades out.
- **Implementation:** `setTimeout(() => modal.classList.add('hidden'), 2000)`. User can still click "Continue" to hide immediately.

**Q4: Wake Lock on which screens only**
- **Question:** Should wake lock be requested only when timer is running (Active Timer), or also during Invocation/Completion?
- **Recommended:** Only during Active Timer. Request on step start, release on session complete. No need on Invocation/Completion.
- **Implementation:** In `timerEngine.start()`, request wake lock. In `destroy()` or when session completes, release.

**Q5: AUM round bell sound design**
- **Question:** Should the AUM round bell also play a very subtle "ding" at the very start of each round (before the main bell), or is a single bell sufficient?
- **Recommended:** Single bell is sufficient. Simplicity over complexity.
- **Implementation:** Just `playAUMBell()` at round boundary (when remaining resets to 10).

**Q6: Font loading strategy**
- **Question:** Should we use `font-display: swap` (default) or `font-display: optional` to avoid invisible text if fonts fail?
- **Recommended:** `swap` — keeps content readable with fallbacks, flashes new font when loaded. Sanskrit text is critical; swap acceptable.
- **Implementation:** Google Fonts URL includes `&display=swap` (already in spec).

**Q7: Step description length**
- **Question:** Step descriptions provided in step spec are relatively long. Should we truncate or wrap on Active Timer screen?
- **Recommended:** Show full description, wrapped. If too long, truncate at 2 lines with `line-clamp: 2` and `title` attribute for full text. Keep UI uncluttered.
- **Implementation:** CSS `display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden;`.

**Q8: Progress bar style**
- **Question:** Should progress bar be a thin line (4px height) or a thicker accent bar?
- **Recommended:** Thin line, full width at top of Active Timer screen, below header if any. Height: 4px. Color: `var(--accent)`.
- **Implementation:** Fixed position or static in flow? Static below step name for simplicity.

---

**End of SPEC.md**
