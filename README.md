# Guitar Practice Web App

A single-file (`index.html`) beat-synced practice trainer: a drum step sequencer flashes random prompts (notes, chords, scales, triads, rhythms) that you execute on guitar before the next prompt lands. Works on desktop and mobile (iOS Web Audio quirks handled).

## Running

Serve the folder over HTTP (the drum samples in `percAudio/` are fetched, so `file://` won't load them):

```bash
# Windows / macOS
cd GuitarPractice
python -m http.server 8000
# open http://localhost:8000
```

Or deploy the folder to any static host (GitHub Pages, Netlify).

## Visual Layout Refresh

The single-file HTML UI now mirrors the compact native iOS layout more closely: icon-labeled mode/transport buttons, icon-bearing collapsible headers, and a two-row practice-control strip with the Arp lock aligned on the second row.

Desktop HTML starts in a horizontal layout by default. The **Tall/Wide** button in Practice Settings switches between horizontal and vertical layouts. In horizontal mode the 22-fret guitar fretboard owns the full top row; in vertical or portrait-width mode the fretboard returns to the original 15-fret range. Both layouts keep the sheet visualizer, Note-to-Arp button block, and Scale Toggles together at the top, with the buttons attached tightly below the score. Everything below the horizontal fretboard uses three columns: sheet visualizer plus the Note-to-Arp button block with Scale Toggles below it, Practice Settings plus Practice Calendar, and Sequencer plus volume sliders, Presets, and Settings. Landscape dropdown spacing is compact so closed headers and opened section contents sit close together like the portrait layout.

## Practice Modes

Tap a mode button to start it (playback starts automatically after the pre-roll). Tap the same button again to stop.

| Mode | Prompt | What it trains |
|------|--------|----------------|
| **Note** | Random note name + staff notation; every position lights up on the fretboard | Note recall, fretboard mapping, sight reading |
| **CAGED** | Random root + CAGED form; the actual grip is drawn on the fretboard (Standard tuning) | CAGED system, chord shells |
| **Chord** | Random jazz chord + rootless voicing hint; chord tones across the whole neck | Jazz harmony, voicing recall |
| **Scale** | Random root + scale; scale tones across the neck, root in orange, the scale's *character note* as a cyan diamond (e.g. Lydian ♯4, Dorian ♮6, Mixolydian ♭7) | Scale fluency, modal color |
| **Triad** *(new)* | Random root × quality (Maj/min/dim/aug) × inversion × string set, e.g. "B♭ min · 1st inv · Str 4-3-2"; the exact 3-note shape is drawn on the fretboard in any tuning | The "triads not scales" drill — voice leading, neck-wide chord-tone mastery |
| **Rhythm** *(new)* | A freshly generated one-bar rhythm in real notation (difficulty: Easy/Medium/Hard in Practice Settings) plus a chord to comp; an orange playhead tracks the bar as the sequencer plays | Rhythmic sight reading, time feel — the most-cited weakness of self-taught players |

Both new modes follow the same flow as the originals: pre-roll, then a new prompt after the full sequencer grid has repeated for the selected *Maintain* cycle count.

### Prompt Lock 🔒

The lock button (Practice Settings, next to the Rhythm difficulty menu) freezes the current prompt: the note/chord/scale/triad/rhythm keeps repeating instead of advancing after the *Maintain* cycle count. The countdown shows "🔒 Locked" while active. Selecting a **different** practice mode releases the lock automatically.

## Fretboard

- Realistic 2D neck: rosewood + grain, bone nut, metallic frets at true logarithmic spacing, pearl inlays, gauge-graded strings (wound look on E/A/D). Low E at the bottom, like TAB.
- Horizontal landscape layout spans 22 frets, with markers and fret labels through the 21st fret. Vertical and portrait-width layouts show the original 15-fret fretboard.
- **Tunings:** Standard, Drop D, Open G, Open D, DADGAD. Retuned strings are labeled in orange at the nut. All modes recompute their positions for the selected tuning (CAGED shapes are Standard-only and fall back to whole-neck triad tones in other tunings).
- **Markers:** orange ring = root, gold = chord/scale tone, cyan diamond = the scale's character note. Every dot is labeled with its note name (respects the *Flats* toggle).
- Staff notation graphics are 20% larger than the earlier compact layout for better note and staff readability.

## Arpeggio Preview (Shift+Space / Swipe up / Arp button)

While practicing, hit **Shift+Space** (desktop keyboard), **swipe up** (mobile), or tap **♪ Arp**:

- The current note/chord/scale/triad plays as an arpeggio on a small subtractive synth (two detuned saws → resonant lowpass with a plucky filter envelope). Volume: *Arp vol* in Practice Settings.
- In **Rhythm** mode it instead plays the displayed rhythm as chord stabs — an audio demo of the notation, rendered at the displayed rhythm timing and swing-aware so the feel matches the drums.
- While the sequencer is running, the Arp trigger is queued to the first beat of the next grid cycle. When the sequencer is stopped or paused, Arp playback starts immediately.
- The arpeggio always finishes within **two bars (half the sequencer cycle)** at the current BPM.
- Arpeggio playback is always remapped to frets 1-15, even when the horizontal 22-fret board is visible.
- Swipes that scroll the page (or a list) are ignored — only a clean upward flick triggers it.
- As each note sounds, the matching fretboard position flashes white — the pitches you hear are computed from the actual string/fret positions in the current tuning.

## Cloud Sync (Supabase)

Settings persist across devices via Supabase Storage:

- Project: `upqnxyllnenivehmallp.supabase.co`, bucket `csv-data` (the same project/bucket as the other PL apps), dedicated file **`guitar_practice_settings.csv`**.
- Format: `key,value` CSV with a single `config` row holding the RFC-4180-quoted JSON payload (same convention as `stock_pl_settings.csv`).
- Saved: BPM, 2x, meter, swing, drum volumes, sine/arp volumes, pre-roll, maintain, flats, rhythm difficulty, tuning, scale toggles, sequencer patterns, and all presets.
- **Auto-loads on startup** (cache-busted fetch, never stale). Saving is manual: *Cloud Sync → Save settings*. *Reload* re-fetches.
- The bucket and its anon read/write policies already exist; no Supabase dashboard setup is needed. If the file doesn't exist yet, the app silently uses defaults until the first save.

## Google Calendar Practice History

The **Settings** section stores a browser OAuth Client ID and target calendar name/ID/`primary`. The **Practice Calendar** section stores each practice locally first, shows week/month history, syncs missing `Guitar Practiced` events from Google Calendar, and can remove the current page-session practice from both local history and Google Calendar.

Only one practice history item and one Google Calendar event are created per full page launch. After page exit or the 10-minute idle timeout closes the session, starting playback again in the same page load will not create another history item; reload the page to start a new practice-history session.

Google Calendar setup is the same browser OAuth pattern as FlexStructure: enable Google Calendar API, create a Web application OAuth Client ID with the app's served origin authorized, enter that Client ID in Settings, enter the calendar name/ID, then use **Sign in with Google**.

## Sequencer

Unchanged from before: 4 tracks (Kick/Snare/C-Hat/O-Hat), 6 meters, swing, 2x, per-track volume, presets (now also synced to the cloud). Practice prompts advance by full sequencer-grid cycles.

## Memo Mode

The transport **Arp** button is now an icon-only **Play Guitar** button at half width, beside a new icon-only **Memo** button (a pencil over a fretboard). **Memo** toggles Memo mode, which disables Grid Sequencer playback.

In Memo mode, click/tap notes on the fretboard canvas to mark them (click again to remove; unlimited notes). Marked notes use the standard overlay dots — the detected chord root in orange, the rest in gold. As notes are marked, the score visualizer names the most plausible chord via a faithful template-matching engine (`detectChord(midis, flats)`) covering triads, 6ths, 7ths, sus/add, altered and tension chords through 13ths, preferring the complete chord over inversion / power-chord readings (low-to-high Ab F C is named **Fm**, not F5/Ab). Marked notes also render on the staff. The chord engine is identical to the iOS app's.

**Play Guitar** in Memo mode strums all marked notes at once, a single time. **Stop** clears the marked notes.

The camera **Capture** button at the top-right of the fretboard panel (available in every mode) exports the fretboard plus the current overlay to a **transparent PNG** (RGBA, no page background) that always renders the full **22-fret** board even in portrait, and auto-downloads it.

### Score visualizer & capture refinements

In the horizontal/landscape layout the score visualizer is a **grand staff** — a treble staff over a bass staff (added below) sharing one continuous position axis, with middle C between them — so a much wider range of notes is shown. `.app.layout-horizontal .display-frame` grows to fit both staves. Portrait keeps the single treble staff.

The captured PNG renders with square corners and extra top/left margin (the top open-string label is no longer clipped), and the detected chord name is drawn over it in white (white fill + white outline, no background) next to the root note.

## Files

```
index.html              the whole app
percAudio/              drum samples + legacy presets.csv
CCBackup/               pre-rewrite backup (index_CCBackup.html)
```
