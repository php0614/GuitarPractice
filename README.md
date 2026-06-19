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

The single-file HTML UI now mirrors the compact native iOS layout more closely: icon-labeled mode/transport buttons, icon-bearing collapsible headers, a tighter 430px app width, and a two-row practice-control strip with the Arp lock aligned on the second row. This refresh is visual only; the original HTML app functions and data flow are unchanged.

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

Both new modes follow the same flow as the originals: pre-roll, then a new prompt every *Maintain* kicks.

### Prompt Lock 🔒

The lock button (Practice Settings, next to the Rhythm difficulty menu) freezes the current prompt: the note/chord/scale/triad/rhythm keeps repeating instead of advancing every *Maintain* kicks. The countdown shows "🔒 Locked" while active. Selecting a **different** practice mode releases the lock automatically.

## Fretboard

- Realistic 2D neck: rosewood + grain, bone nut, metallic frets at true logarithmic spacing, pearl inlays, gauge-graded strings (wound look on E/A/D). Low E at the bottom, like TAB.
- **Tunings:** Standard, Drop D, Open G, Open D, DADGAD. Retuned strings are labeled in orange at the nut. All modes recompute their positions for the selected tuning (CAGED shapes are Standard-only and fall back to whole-neck triad tones in other tunings).
- **Markers:** orange ring = root, gold = chord/scale tone, cyan diamond = the scale's character note. Every dot is labeled with its note name (respects the *Flats* toggle).

## Arpeggio Preview (Shift+Space / Swipe up / Arp button)

While practicing, hit **Shift+Space** (desktop keyboard), **swipe up** (mobile), or tap **♪ Arp**:

- The current note/chord/scale/triad plays as an arpeggio on a small subtractive synth (two detuned saws → resonant lowpass with a plucky filter envelope). Volume: *Arp vol* in Practice Settings.
- In **Rhythm** mode it instead plays the displayed rhythm as chord stabs — an audio demo of the notation, rendered at the displayed rhythm timing and swing-aware so the feel matches the drums.
- The arpeggio always finishes within **two bars (half the sequencer cycle)** at the current BPM.
- Swipes that scroll the page (or a list) are ignored — only a clean upward flick triggers it.
- As each note sounds, the matching fretboard position flashes white — the pitches you hear are computed from the actual string/fret positions in the current tuning.

## Cloud Sync (Supabase)

Settings persist across devices via Supabase Storage:

- Project: `upqnxyllnenivehmallp.supabase.co`, bucket `csv-data` (the same project/bucket as the other PL apps), dedicated file **`guitar_practice_settings.csv`**.
- Format: `key,value` CSV with a single `config` row holding the RFC-4180-quoted JSON payload (same convention as `stock_pl_settings.csv`).
- Saved: BPM, 2x, meter, swing, drum volumes, sine/arp volumes, pre-roll, maintain, flats, rhythm difficulty, tuning, scale toggles, sequencer patterns, and all presets.
- **Auto-loads on startup** (cache-busted fetch, never stale). Saving is manual: *Cloud Sync → Save settings*. *Reload* re-fetches.
- The bucket and its anon read/write policies already exist; no Supabase dashboard setup is needed. If the file doesn't exist yet, the app silently uses defaults until the first save.

## Sequencer

Unchanged from before: 4 tracks (Kick/Snare/C-Hat/O-Hat), 6 meters, swing, 2x, per-track volume, presets (now also synced to the cloud). Practice prompts advance on **Kick** hits.

## Files

```
index.html              the whole app
percAudio/              drum samples + legacy presets.csv
CCBackup/               pre-rewrite backup (index_CCBackup.html)
```
