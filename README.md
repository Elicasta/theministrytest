# The Ministry — Sermon Presentation System

A static, single-file live teaching/presenter app for church services. One `index.html`
drives every screen — the big projector, a side scripture display, a confidence monitor
for the speaker, OBS broadcast outputs, and a desktop + mobile presenter controller — all
kept in sync over Supabase Realtime (with a same-device `BroadcastChannel` fallback).

This is v17, a cleanup/master-template pass on the stable v16 build. **No new features,
no rewrite.** See `CHANGELOG.md` for exactly what changed.

## Routes

| Route | What it is |
|---|---|
| `/projector` | Main big-screen teaching projector. |
| `/scriptures` | Side-screen scripture display. KJV primary, RVR 1960 underneath. |
| `/confidence` | Confidence monitor for the speaker — current slide, next main slide, timer, notes, scripture. |
| `/obslowerthirds` | OBS browser-source lower third (broadcast-style). |
| `/obsslides` | OBS browser-source full slide output (scripture or clean main point). |
| `/admin` and `/mobile` | **Not separate routes.** Both resolve to the landing page. Click **Admin**, enter the admin password, and you're in the desktop controller. From there, click **Mobile Mode** to get the phone-sized controller. This is the existing behavior — it was intentionally left alone in this cleanup (see "Routing" below). |

Legacy query-string routes (`?projector=1`, `?screen=2`, `?obs=lower`, `?obs=full`) still
work as a fallback — `vercel.json` rewrites every path to `index.html`, and the app reads
`location.pathname` first, then falls back to the query string.

## How to duplicate this for a new series

1. Copy the whole project folder.
2. Open `index.html` and edit, in order:
   - **Section 01 — SERIES CONFIG**: title, subtitle, speaker, attendee password, admin
     password, Supabase channel ID (must be unique per series — this is the Realtime
     "room" name), and the QR code URL text.
   - **Section 02 — THEME CONFIG**: the `THEME.colors` object and `THEME.backgroundImage`.
     `applyTheme()` pushes these straight into the CSS variables the whole app already
     uses, so you should not need to touch any CSS rule.
   - **Section 03 — LESSON DATA**: replace `LESSON1_SLIDES` and `NOTES_L1` with the new
     lesson's slides and speaker notes.
   - **Section 04 — SCRIPTURE DATA**: replace `SCRIPTURE_MAP`, `VERSE_BANK`, and
     `QUESTIONS` (the workbook/questionnaire content) for the new lesson.
3. Replace `assets/ministry-bg.jpeg` (or point `THEME.backgroundImage` at a new file) and
   the two QR images in `assets/`.
4. Search for any literal `Elder Eli Castaneda`, `Matthew 10`, or `The Ministry` strings
   you want changed that live inside slide/markup data you're rewriting anyway in step 2
   — these are content, not config, so they're not centralized.
5. Deploy.

**Do not touch:** sections 05–09 (state, routing/init, renderers, OBS/teaching-slide logic,
confidence monitor logic). These are the engine. If something about the engine itself
needs to change for every series, that's a real feature request — flag it, don't patch it
silently into one series's copy.

## Section map (what's actually in `index.html`)

```
01. SERIES CONFIG          — edit per series
02. THEME CONFIG           — edit per series
03. LESSON DATA            — edit per series
04. SCRIPTURE DATA         — edit per series
04b. LESSON LIBRARY        — edit per series (Lesson 2-4 placeholders live here)
05. APP STATE              — do not edit
06. ROUTING / INIT         — do not edit
07. UI, RENDERERS & CONTROLS — do not edit
08. OBS + TEACHING SLIDE LOGIC (active patch) — do not edit
09. CONFIDENCE MONITOR LOGIC (active patch)   — do not edit
10. LIVE Q&A OVERLAY + POLLING (new, not a patch) — do not edit
```

## Adding Lesson 2 (or 3, or 4)

1. Find `LESSON_LIBRARY` (section 04b). Lesson 2's entry currently has `enabled:false`
   and `null` for everything.
2. Build that lesson's slides/notes/scripture map/verse bank/questions the same way
   Lesson 1's are built in sections 03-04 (you can write them as their own `let
   LESSON2_SLIDES = [...]` etc. above `LESSON_LIBRARY`, then reference them in the entry).
3. Set `enabled:true`.
4. The admin "Lessons" picker will show it as clickable instead of locked. Clicking it
   switches the whole app over — slides, verse bank, questionnaire, and every connected
   screen.
5. Separately, update the matching `lcard` on the public hub (it's static markup, not
   yet wired to `LESSON_LIBRARY`) so attendees see it unlock too.

## Live Q&A overlay

Admin's Live Questions panel has a "Feature on Screen" button per question. Tapping it
pushes that question full-screen to `/projector` (gold accent, distinct from the red
scripture overlay), as a small banner on `/confidence`, and into the lower-third/full
text on the OBS outputs. "Clear Featured Question" closes it everywhere.

## Live polling

Admin's "Live Poll" panel: type a question, type 2-6 comma-separated options, "Open
Poll." It shows up as a card at the top of the attendee hub. Each device gets one vote
per poll (no login system, so this is enforced client-side — same trust level as the
rest of the app's audience features). Admin sees live tallies; "Show Results" pushes a
bar-chart overlay to the same screens the Q&A overlay uses.

Don't run a Q&A overlay and a poll results overlay at the same time — clear one before
opening the other, since they share the same on-screen real estate on `/projector`.

### Patch layers (read this before touching JS)

This file grew through a series of in-place patches (`v10` through `v16`), each one a
`<style>`/`<script>` block added near the bottom that **overrides** an earlier function by
redefining the same global name. That pattern is still here in sections 08 and 09 — it
works, and rewriting it into a single clean pass was judged out of scope for this cleanup
(see "What this pass did not do," below). If you're debugging the confidence monitor or
the OBS output and the behavior doesn't match what you see in section 07, check section 09
or 08 first — the last definition in document order is the one that runs:

- `renderSlide`, `slidePlainTitle`, `showOBS`, `showOBSSlide`, `handleMessage` →
  **section 08** has the final word.
- `updateConfidence`, `updateConfidenceScripture`, `clearConfidenceScripture`,
  `broadcastScripture`, `pushRawScripture` → **section 09** has the final word (and section
  09 also wraps `handleMessage` again, on top of section 08's version, so the confidence
  monitor doesn't get hijacked by auto-scripture pushes).

## Deploying to Vercel

1. Push to a GitHub repo (or drag-and-drop deploy via the Vercel dashboard).
2. Import the repo in Vercel. No build step — it's static. `vercel.json` already rewrites
   every path to `index.html` so the clean routes above work.
3. If you use the Supabase Realtime sync, set `window.SB_KEY` in section 01's area (search
   `SB_KEY`) to your Supabase anon public key, or wire it through an environment variable
   if you'd rather not commit it.
4. `api/waitlist.js` and `api/workbook-submit.js`, and `content/series/...`, are **not**
   called by the live app — see "What this pass did not do" below before relying on them.

## Using this with ProPresenter / a dual-projector setup

- Point your **main projector** (ProPresenter or a browser window) at `/projector`.
- Point your **side/confidence screen** at `/scriptures` (audience-facing scripture) or
  `/confidence` (speaker-facing monitor), depending on what that screen is for.
- Run the **admin controller** from a laptop at the root URL → Admin. Use **Mobile Mode**
  from a phone for hands-free Next/Previous from the pulpit.
- For OBS: add `/obslowerthirds` and/or `/obsslides` as Browser Sources. Toggle green
  screen mode on `/obsslides` with the on-screen control if you're keying it over a camera
  feed instead of using the built-in background.

## Testing before going live

Run through this exact checklist (from the original cleanup brief) before every service:

1. Open `/projector`, `/scriptures`, and `/confidence` in three tabs/screens.
2. Open the root URL → Admin.
3. Confirm all outputs show standby/loading screens.
4. Press **Start**.
5. Confirm Projector goes to slide 1.
6. Confirm Scriptures syncs when a slide has scripture.
7. Confirm Confidence shows the current slide and the correct **next main slide** (it
   should skip support-scripture slides, not show one as if it were the next teaching
   point).
8. Press **Next**.
9. Confirm the Projector 1 scripture overlay clears.
10. Confirm Scriptures (Projector 2) still follows scripture sync.
11. Manually push a scripture overlay to Projector 1 from the verse bank.
12. Confirm Projector 1 shows KJV only, centered.
13. Confirm Scriptures shows KJV/RVR together.
14. Confirm Confidence shows fitted scripture text, not huge/overflowing text.
15. Test Mobile Mode on an actual phone-sized viewport.
16. Test `/obslowerthirds`.
17. Test `/obsslides` with and without green-screen mode.
18. **New this pass:** from a second device/tab acting as an attendee, submit a question
    through the "Ask" drawer and confirm it shows up in the admin's Live Questions panel.
    Then unlock the questionnaire from admin and confirm the attendee tab's questionnaire
    actually unlocks without a page reload. (This was broken before this pass — see
    CHANGELOG.)

## What this pass did and did not do

**Did:**
- Organized the file into the 9 labeled sections above.
- Added `SERIES_CONFIG` and `THEME` as the single editable surface for series identity
  and color/background — wired into the existing CSS variables, zero visual change.
- Fixed a real bug: `question_submit` and `q_unlock` message handling was silently
  dropped when the `v13` patch replaced `handleMessage` and didn't carry those two
  branches forward. Live Q&A-to-admin and live questionnaire-unlock were broken. Restored.
- Removed five blocks of dead code (the pre-`v13`/pre-`v16` versions of `renderSlide`,
  `slidePlainTitle`, `showOBS`, `showOBSSlide`, `handleMessage`, `updateConfidence`, and
  friends) that were silently overridden and never actually ran. Verified each one's full
  call chain before deleting — see the inline comments left in their place.
- Confirmed no leftover "Slide 1/15" indicators, no misspellings of Previous / Confidence
  / Projector, and no stray debug labels — this file was already clean on those points.

**Did not do** (flagged as out of scope, not silently skipped):
- Did **not** physically reorder functions into the 9 sections. Sections 05–09 are real,
  contiguous regions of the file, but section 07 is one large pile of UI/control functions
  rather than further split into renderers vs. admin vs. mobile. Splitting that apart for
  real would mean moving ~800 lines of interdependent code with no live Supabase
  environment to test against — that's a rewrite risk, not a cleanup, and it was explicitly
  out of scope ("do not break live output" is priority one).
- Did **not** touch `content/series/the-ministry/` (a disabled markdown-lesson-loader
  experiment) or `api/waitlist.js` / `api/workbook-submit.js` (serverless functions for
  email capture and workbook email delivery). None of these are called by `index.html` —
  they're inert. They look like the start of a markdown-loader/database direction, which
  the cleanup brief explicitly said not to build. Left in place untouched rather than
  deleted, in case they're intentional groundwork — but worth a deliberate decision
  (finish wiring them, or delete them) rather than leaving them as ambient clutter.
- Did **not** do a full dead-CSS sweep. Spot checks didn't turn up anything live and
  breakable, but a confident "delete this rule" pass needs the rendered app in front of
  you, not a text read of 600+ lines of CSS.
