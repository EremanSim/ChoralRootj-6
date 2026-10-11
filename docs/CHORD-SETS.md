# Chord sets

ChoralRoot J-6 adds 100 **chord sets** to the ChoralRoot chord engine. A chord set gives each of the 12 keys
(C, C#, D … B) a chord that someone already voiced, usually with upper extensions, slash basses and a deliberate
voice-leading between neighbouring keys. Pressing one key plays a full chord, the way the chord mode of a
Roland AIRA J-6 does. Without a set, ChoralRoot builds every chord from the same formulas (MAJ, MIN, M7, m7 …), so
all the chords have the same shape and the same register. With a set they don't.

The 100 sets follow the "Chord Set List" printed in the Roland J-6 owner's manual (version 1.02): the same numbering,
genres, chord names and notes. See [Where the sets come from](#where-the-sets-come-from) below.

## Playing them on the FM-1

**Options > Chord Set** (the entry after Reset Settings; SELECT moves through Options, KNOB 1 sets the value):

| Value | What a root key does |
| --- | --- |
| Off (default) | as before: a key alone is a single note, the chord keys (DIM MIN MAJ SUS, 6 m7 M7 9) build the chord |
| 1 … 100 | a key alone plays that set's chord for its pitch class |

- **The chord keys still work.** Hold a chord key (or latch it with LOCK) and the root key plays the chord block's
  chord as before. Let go of the chord keys and the root plays the set's chord again. This lets you mix
  the set with your own chords.
- **Octaves.** The roots D4 … C#5 play every chord of the set at the pitches the list gives. A root an octave above
  or below that plays the same chord an octave higher or lower. OCT- / OCT+ move it by octaves too.
- **Transpose, voicing, bass, performance.** KNOB 1's voicing rotates the set's voicing (lowest note up an octave,
  or highest note down an octave), the transpose moves it, and the bass plays the pitch class of the
  voicing's lowest note, so a slash chord gets its slash bass. Strum, Arp, Pattern and Harp play it like any chord.
- **The screen.** The top line shows `Set 12 Jazz Min`. The chord name is the set's own (`Dm7b5`, `C#M9/C`,
  `Abm6addb13`), transposed when you transpose.
- **Pads and the looper.** A chord pad keeps the set it was recorded from. A loop records the key and plays it
  back through the set selected when it plays (when Chord Set is Off then, through the last set that was on).
  So you can record a progression once and change the set under it.
- **Saved.** The set is part of the session (cr_session_t `chord_set`). It comes back after you switch the unit off
  and on. Reset Settings turns it Off.
- **MIDI.** The notes go out on the performance channel as usual. A DAW sees the chord, not the key.

## Playing them in a browser

`web/j6/index.html` is a stand-alone chord player that needs no FM-1 and runs from any static web server or
GitHub Pages (`j6/`):

- the 100 sets, a set browser filtered by genre, each key labelled with its chord;
- an original Web Audio synth with 8 sounds (poly pad, brass, strings, electric piano, pluck, organ, bell, lo-fi
  keys), chorus, delay and reverb;
- 14 styles: held chord, strum up and down, five arpeggios, pulse, house offbeat, syncopated, Charleston, broken
  (Alberti) and bass + chord;
- octave, transpose, voicing, hold (latch) and bass doubling;
- a 16-step chord sequencer with record, rest and step lengths from 1/16 to one bar;
- Web MIDI in (a MIDI keyboard plays the key's chord) and out where the browser allows it;
- computer keys `A W S E D F T G Y H U J` for C … B, `Z`/`X` for octave, `,`/`.` for the set, `Space` to play.

The firmware itself also runs in the browser (`tools/emu/web`, published as `emu/`). There, Options > Chord Set
works as it does on the unit.

## Where the sets come from

`data/chordsets/j6_chord_set_list.txt` is the only source. It holds two lines per set: the chord names for the 12
keys and their notes, top to bottom, as the manual prints them. `tools/gen_chordsets.py` reads it and writes:

- `firmware/src/cr_chordsets.h`: the tables the engine includes (4.8 KB of notes and 16.8 KB of names in flash);
- `web/j6/chordsets.js`: the same data for the browser player.

The generator sorts each voicing low to high and removes repeated notes. It names the three interval-stack sets
(14 to 16), which have no names in the list, and fixes two obvious typos (`F#FM7` becomes `F#M7`; the cut-off
`AbMb5/` becomes `AbMb5/#9`). Other oddities in the published list stay as published: a few names don't match their
notes (set 59 lists set 60's voicings), and one voicing is printed out of order. `python3 tools/gen_chordsets.py
--check` fails when an output is stale. `tests/run_cr_tests.sh` and the `web` workflow run it.

### Copyright and trademarks

- The data is a list of facts: which notes sound together and what musicians call that chord. Chord names, chord
  voicings and chord progressions are basic musical material that every musician uses. This project includes no
  Roland software, firmware, samples, sound data, presets, artwork or text from the manual beyond those chord
  names and note names.
- The sound engines, the chord engine, the sequencer, the player and every line of code here are this project's
  own (GPL-3.0, see [LICENSING.md](../LICENSING.md)). The J-6 list was used as a reference and as inspiration.
- ChoralRoot J-6 is not affiliated with, endorsed by or sponsored by Roland Corporation. "Roland", "AIRA" and
  "J-6" are trademarks of Roland Corporation, used here only to say which instrument's chord list the sets follow.
- The list lives in one file on purpose. If a rights holder objects, removing `data/chordsets/` and regenerating
  removes the sets and leaves the engine as it is.

## The engine side

`cr_set_chord_set(c, n)` (cr_engine.h) selects the set (0 is Off) for new gestures. A voice captures the set at
note-on (`cr_voice_t.cs`). With no chord key in its spec, `crx_build_base` takes the set's voicing
(`cr_chord_set_chord`) instead of the chord tables. Its quality is `CR_Q_SET`. Pads store the set in `ext`, and the
looper's events carry `CR_Q_SET` with the key as the root. `cr_chord_info` reports `set` and splits the set's name
into root, quality mark (`m`, `dim`) and superscript for the screen. `tests/cr_engine_test.c s_chordsets` covers
the tables, octaves, transpose, chord-key override, bass, pads, loop voices and Off. `tools/emu/scripts/cr_chordset.txt`
covers Options > Chord Set in the whole firmware (`tools/emu/web/test_web_emu.mjs`).
