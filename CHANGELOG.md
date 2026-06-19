# Changelog

## v18.1 — Corrections + anonymous answers + alignment fixes

**Reverted**
- Removed the accent line I added to teaching/statement/closing slides last round —
  that was a misread; the request was to remove the line, not restore it. Back to
  exactly how v17 had it.

**Bug fix**
- Reflection question text was rendering centered instead of left-aligned. Root cause:
  `#questionnaire` carries a static `locked` class that nothing ever removes, and the
  centering rule was scoped to `#questionnaire.locked .q-body` — which matches both the
  locked-state message AND the actual open questions, since both use `.q-body`. Rescoped
  the rule to `#q-locked-view` only. Reflection questions are left-aligned now, like
  everywhere else in the app.

**Changed**
- Scripture ('sc') slides on `/projector` (and the admin preview): center-aligned
  instead of left-aligned — the mark, reference, verse text, and takeaway all center
  on the slide now.
- Ask a Question drawer: added an "Submit anonymously" checkbox. When checked, the
  question is sent and displayed as "Anonymous" everywhere it shows up — admin's Live
  Questions panel, the Feature-on-Screen overlay, OBS outputs.

## v18 — Brightness, slide consistency, lesson picker, live Q&A overlay, polling

**Bug fix (important — affects live Q&A and the new polling feature)**
- The Supabase Realtime handler only ever called `handleMessage` when `!isAdmin`. That
  meant the admin's browser never processed a single realtime message from another
  device — same-device testing across two tabs (which uses `BroadcastChannel`, not
  Supabase) looked fine, but a real attendee on their own phone submitting a question
  never reached the admin live. Same bug would have silently broken poll votes. Removed
  the gate; `handleMessage` already routes correctly based on screen type and `isAdmin`
  inside each branch, so this was never needed.

**Visual fixes**
- Standby/loading screens (`/projector` and `/scriptures` before Start is pressed):
  background image brightness raised from .72 to .92, and the dark gradient overlay on
  top of it lightened, so the photo is actually visible behind the text instead of mostly
  reading as black.
- Teaching slide stage background and the cover slide background: +15% brightness
  (these are the two rules that actually win in projector mode — there were three
  layers of brightness overrides stacked on top of each other from earlier patches;
  edited the one that wins, removed the one that had gone dead).
- Added a small red accent line (`.sl-mk`) to the top of teaching ("te"), statement
  ("big"), and closing ("final") slides — the same treatment cover, scripture, and names
  slides already had. This is what existed before the point-number badge was removed in
  the v17 cleanup; it's back, just without the point number.

**Admin: lesson picker**
- `LESSON1_SLIDES`, `NOTES_L1`, `SCRIPTURE_MAP`, `VERSE_BANK`, and `QUESTIONS` changed
  from `const` to `let` so they can be swapped when switching lessons (every existing
  function that reads them keeps working unchanged — they just point at different data
  after a switch).
- Added `LESSON_LIBRARY`, a 4-entry registry (Lesson 1 enabled with real content, Lessons
  2-4 present but locked/empty placeholders) and `loadLesson(idx)` to switch the active
  lesson, rebuild the slide list/verse bank/questionnaire, and broadcast the switch to
  every connected screen.
- Added a "Lessons" picker to the admin right column — Lesson 1 shows as the active
  lesson, Lessons 2-4 show locked, matching the public hub's existing lock styling.
  When you add Lesson 2's content later, set `enabled:true` and fill in its
  `slides`/`notes`/`scriptureMap`/`verseBank`/`questions` in `LESSON_LIBRARY` (section 04b).

**Admin: live Q&A overlay**
- Added a gold-accented full-screen "Live Question" overlay on `/projector`, a compact
  banner on `/confidence` so the speaker knows what's on screen, and lower-third text on
  `/obslowerthirds` / `/obsslides`.
- The admin's existing Live Questions panel now has a "Feature on Screen" button per
  question — tap it to push that question to every connected screen. "Clear Featured
  Question" closes it everywhere.

**Admin: live polling (new)**
- New "Live Poll" panel in admin: type a question, type 2-6 comma-separated options,
  "Open Poll." Attendees see a poll card appear at the top of their hub and tap an
  option (one vote per device, enforced client-side, same trust model as the rest of
  this app — there's no login system).
- Admin sees a live tally with bars as votes come in.
- "Show Results" pushes a results overlay (bar chart) to the same screens the Q&A
  overlay uses. "Close Poll" ends it everywhere.
- Reuses the existing `sbSend`/`handleMessage` sync — no new backend, no database,
  matches everything else in this app.

**Needs live testing before Sunday — see the testing checklist below.** This round
touched the realtime message handler and added three stateful features with no way to
render or test them outside a real browser with a real Supabase project. The syntax has
been checked and the logic has been traced by hand, but "compiles cleanly" is not the
same as "works at the pulpit."

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
