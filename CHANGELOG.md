# Changelog

## v17 — Master Template Cleanup

**Bug fix**
- Restored `question_submit` and `q_unlock` message handling. These existed in the
  original `handleMessage` but were dropped when the `v13` patch replaced
  `window.handleMessage` without carrying them forward. Effect of the bug: live attendee
  questions never reached the admin's Live Questions panel, and unlocking the
  questionnaire from admin never reached attendees in real time (it only worked on page
  reload, via `localStorage`). Both now work live again.

**Organization**
- Added 9 labeled sections to `index.html`: SERIES CONFIG, THEME CONFIG, LESSON DATA,
  SCRIPTURE DATA, APP STATE, ROUTING/INIT, UI/RENDERERS/CONTROLS, OBS+TEACHING SLIDE LOGIC,
  CONFIDENCE MONITOR LOGIC. See README.md for the full section map and what each one is
  safe to edit.
- Added `SERIES_CONFIG` (title, subtitle, speaker, passwords, Supabase channel ID, QR URL)
  as the single place to edit series identity for a future series.
- Added `THEME` (colors + background image) and `applyTheme()`, wired into the existing
  CSS variables (`--bg`, `--w`, `--red`, `--gold`, etc.) with identical default values —
  zero visual change, but color/background is now a one-object edit instead of a CSS hunt.
  Added `--bg-image` and `--green`/`--greend` CSS variables so the background photo and the
  Start-button green are themeable the same way everything else already was.

**Dead code removed** (each verified unreachable — overridden before first use, confirmed
no caller resolves to it — see inline comments left in their place pointing to the live
version)
- Old `renderSlide` — pre-dated the fix that removed the large point-number badge from
  teaching slides. Superseded by the `v13` patch version, which is what actually renders.
- Old `slidePlainTitle`, `showOBS`, `showOBSSlide` — superseded by `v13` patch versions.
- Old `handleMessage` — superseded by `v13`'s version (itself wrapped by `v16`).
- Old `confidenceNextLabel`, `renderConfidenceScripture`, `updateConfidence`,
  `updateConfidenceScripture`, `clearConfidenceScripture` — superseded by the `v16` patch
  versions, which also implement "skip support scripture slides when previewing the next
  teaching slide," something the old version never did.
- Old `broadcastScripture`, `pushRawScripture` — superseded by `v16` versions (which add
  `auto`/`manual` flags to the synced message, currently unused metadata).

**Verified clean, no change needed**
- No "Slide 1/15"-style indicators leaking into public outputs.
- No misspellings of Previous / Confidence / Projector.
- No stray debug labels.

## v16 — Confidence Monitor Cleanup (prior)
- Confidence monitor: correct current/next slide, skip support scripture slides when
  determining the next main slide, fit scripture text in the box, stop auto-scripture
  pushes from hijacking the monitor.

## v13 — Teaching/OBS Cleanup (prior)
- Removed the point-number badge from teaching slides.
- OBS full output: show scripture on scripture slides, show the clean main point
  otherwise, stop every Auto-P2 verse push from hijacking it after a slide change.

## v10–v12 (prior)
- Cover/projector stability fixes, Start/mobile cleanup, standby/sync/mobile fixes,
  mobile zoom lock.
