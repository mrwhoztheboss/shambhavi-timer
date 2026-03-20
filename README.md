# Shambhavi Timer

A guided timer web app for the Shambhavi Mahamudra kriya practice. It provides step-by-step guidance through all ten steps with audio bells, visual countdown, and optional preparatory steps.

## Features

- Full kriya practice (~27 min) or Shambhavi-only (~21 min)
- Visual SVG countdown ring with real-time updates
- Audio cues: transition bells, AUM chanting bells, completion chime
- Session progress bar
- Screen wake lock to prevent device sleep
- Tab visibility handling (auto-pause when switching tabs)
- Light and dark mode support
- Accessible: keyboard navigation, ARIA live regions, reduced motion support
- Mobile responsive: works on all screen sizes and orientations
- No build tools: single HTML file, works offline after first load

## Usage

1. Open `index.html` in any modern browser
2. Optionally check "Skip preparatory steps" if you want only the core Shambhavi practice
3. Click "Start Practice", then "Begin Practice" on the invocation screen
4. The timer will guide you through each step automatically
5. Use Pause/Resume and Skip buttons as needed
6. Completion screen shows summary and closing invocation

## Steps

### Preparatory (optional)
1. Butterfly / Patangasana — 2 min
2. Shishupalasana Right Leg — 2 min
3. Shishupalasana Left Leg — 2 min
4. Nadi Vibhajan — 4 min

### Shambhavi Mahamudra (core)
5. Sukh Kriya — 6 min
6. AUM Chanting — 21 rounds × 10 sec
7. Vipareit Swasa — 4 min
8. Bandas – Fullness of Breath — 30 sec
9. Bandas – Emptiness of Breath — 30 sec
10. Observing Breath — 6 min

## Audio

All sounds are synthesized using the Web Audio API (no external files):

- **Transition bell** (528 Hz): plays when a step completes
- **AUM round bell** (432 Hz): plays at the start of each AUM chanting round
- **Completion chime**: three ascending tones (528 Hz → 660 Hz → 792 Hz)

Audio starts only after user interaction (clicking "Begin Practice").

## Technical

- Plain HTML, CSS, JavaScript (no frameworks)
- Single `index.html` file — everything is inline
- Web Audio API for bells and chimes
- Screen Wake Lock API to keep device awake during practice
- CSS custom properties for theming (light/dark)
- SVG ring with `stroke-dashoffset` animation
- Date-based timer engine (drift-corrected)
- Debounced button handlers
- Full keyboard accessibility

## Browser Support

Works in all modern browsers: Chrome, Firefox, Safari, Edge. Requires JavaScript and Web Audio support.

## Development

No build process needed. Simply edit `index.html`. To deploy, push to GitHub Pages or any static host.

## License

This is an open tool for Isha Yoga practitioners. No attribution required.
