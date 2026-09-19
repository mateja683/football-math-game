# CLAUDE.md — Goal Maths

Context for continuing this project in Claude Code.

## What this is
A maths practice game for a 4-year-old, played with a parent. Football theme. UK English throughout ("maths", en-GB speech voice). Mobile-first, portrait phone is the main target; also works on tablet and desktop.

## Architecture
Everything lives in `index.html`: inline CSS, inline JS, no framework, no build, no dependencies apart from the Google Fonts stylesheet (Baloo 2, with system fallbacks). Keep it single-file unless there's a strong reason to split.

Screens are sibling `<main class="app">` elements toggled by `show(id)`: `start`, `play`, `end`, `prize`, plus the full-screen `#pong` overlay.

State is one object `S` (team, max, modes, voice, streak, prizes, controls, and per-match values). `save()` persists the settings part to localStorage under the key `goalmaths`. All storage access is wrapped in try/catch — keep it that way.

### Key functions
- `make()` — builds a random question for one of the enabled modes. Returns `{mode, key, ans, text, say, win, eq?, hi?, countSel?, ...}`.
  - `key` prevents the same question twice in a row.
  - `hi` is the upper bound for the numeric answer buttons (defaults to `S.max`).
  - `countSel` overrides which elements "Count with me" steps through.
- `next()` — renders the question: stage (balls/shirts/numbers), answer buttons, count button, speech.
- `options(ans, hi)` — the three numeric choices (answer plus near distractors, sorted ascending).
- `answer(v, el)` — right/wrong handling, streak logic, prize unlock.
- `goal(el)` — ball-flight animation, cheer, then next question / full time / prize screen.
- `countWithMe()`, `peek()` — hints.
- `shirt(team, label?)` — SVG shirt, optional number on it. Red-and-white stripes use the `#stripes` pattern in the hidden `<defs>`.
- `frame(n, gone, empty)` — balls in rows of five (ten-frame layout, deliberate: helps subitising).
- `Pong` — IIFE module for the two-player canvas game. `Pong.start(afterCallback)`. Bottom keeper = child's team, top keeper = `other(S.team)`. First to 3 (`WIN`). Pointer events map each touch to its half of the pitch; arrow-button mode uses the same `keys` set as the keyboard (A/D top, ←/→ bottom).
- Sound: `speak()` (speechSynthesis), `whistle()`, `cheer()`, `thud()` (Web Audio, no audio files).

### Adding things
- **New question type:** add a branch in `make()`, a render branch in `next()`, a chip with `class="chip mode" data-mode="..."` on the start screen, and a gentle wrong-answer line in `answer()`.
- **New team:** add an entry to `TEAMS` (`name`, `main`, `trim`, `confetti`), a `.team` button on the start screen, and a `.pbtn.<team>` style. If the kit isn't a plain colour, special-case it in `shirt()` and `drawPad()` like `red` (stripes) and `claret` (sleeves).

## Design rules to keep
- Wrong answers are never punished: the button wobbles and fades, and a kind voice line plays. No opponent goals, no timers.
- Questions are always read aloud (child can't read yet); 🔊 repeats.
- Big tap targets, sentence case, minimal text.
- Respect `prefers-reduced-motion` (animations are skipped; see the `reduced` flag).
- Teams are kit colours only. Don't draw real club crests, mascots or badges, or use club names in the UI.
- Safe-area insets are handled for phones with notches.

## Ideas discussed but not built
- A more forgiving streak (a miss knocks off a couple of bars instead of resetting to zero).
- Letting the grown-up choose their own team in the football game.
- A grown-ups-only lock on the direct football button.

## Testing
There are no automated tests. Before committing, at minimum:
- syntax-check the script: `node -e "const h=require('fs').readFileSync('index.html','utf8');new Function(h.split('<script>')[1].split('</script>')[0])"`
- open it on a real phone and try each game mode, a full match, and the football game in both control modes with two players.
