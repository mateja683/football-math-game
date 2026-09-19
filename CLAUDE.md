# CLAUDE.md — Goal Maths

Context for continuing this project in Claude Code.

## What this is
A maths practice game for a 4-year-old, played with a parent. Football theme. UK English throughout ("maths", en-GB speech voice). Mobile-first, portrait phone is the main target; also works on tablet and desktop.

## Architecture
Everything lives in `index.html`: inline CSS, inline JS, no framework, no build, no dependencies apart from the Google Fonts stylesheet (Baloo 2, with system fallbacks). Keep it single-file unless there's a strong reason to split.

Screens are sibling `<main class="app">` elements toggled by `show(id)`: `category` (the very first screen — maths vs writing), `start` (maths setup, reached from category), `play`, `end`, `prize`, `gate` (parent multiplication check), `writing` (coming-soon placeholder), plus the full-screen `#pong` overlay and the `#timeout` session-lock overlay (the latter isn't part of `show()` — it's toggled directly and sits above everything, `z-index:100`).

State is one object `S` (team, allGames, modes, voice, streak, prizes, controls, locked, and per-match values). `save()` persists the settings part to localStorage under the key `goalmaths`. All storage access is wrapped in try/catch — keep it that way. Difficulty is currently hardcoded via the `MAX_NUM` constant (20), not a user setting — there's no "numbers up to" UI any more.

### Key functions
- `make()` — builds a random question for one of the enabled modes (`pick(activeModes())`). Returns `{mode, key, ans, text, say, win, eq?, hi?, countSel?, ...}`. Uses `MAX_NUM` as the top of the range throughout.
  - `key` prevents the same question twice in a row.
  - `hi` is the upper bound for the numeric answer buttons (defaults to `MAX_NUM`).
  - `countSel` overrides which elements "Count with me" steps through.
- `activeModes()` — returns `ALL_MODES` when `S.allGames` is true, else `S.modes` (the manually-picked subset). Picking any individual mode chip flips `S.allGames` off.
- `next()` — renders the question: stage (balls/shirts/numbers), answer buttons, count button, speech.
- `options(ans, hi)` — the three numeric choices (answer plus near distractors, sorted ascending).
- `answer(v, el)` — right/wrong handling, streak logic (first-try-correct count for the current round).
- `goal(el)` — ball-flight animation, cheer, then next question or `endOfRound()` once `ROUND_LEN` (10) questions are done.
- `endOfRound()` / `fullTime(hits)` / `prizeUnlocked(hits, resume)` — a round is 10 questions; `PRIZE_THRESHOLD` (9) first-try-correct unlocks the prize match.
- `parentGate(cb)` — grown-up-only multiplication check (factors 2–12) shown before the direct "Two-player football" button; `cb` runs on a correct answer.
- `countWithMe()`, `peek()` — hints.
- `shirt(team, label?)` — SVG shirt, optional number on it. Red-and-white stripes use the `#stripes` pattern in the hidden `<defs>`.
- `frame(n, gone, empty)` — balls in rows of five (ten-frame layout, deliberate: helps subitising). At `MAX_NUM`=20 this can render up to 4 rows; keep an eye on this if the range ever grows further.
- `Pong` — IIFE module for the two-player canvas game. `Pong.start(afterCallback)`, `Pong.freeze()` (cancels the animation loop without running the after-callback; used by the session lock). Bottom keeper = child's team, top keeper = `other(S.team)`. First to 3 (`WIN`), no rematch — winning always ends the session (`pback` → `stop()` → `after()`); playing again means re-earning or re-gating a new one. Pointer events map each touch to its half of the pitch; arrow-button mode uses the same `keys` set as the keyboard (A/D top, ←/→ bottom). `askExit()` shows an in-canvas "Leave the match?" confirm on the ✕ button mid-play instead of exiting instantly. Keepers shrink (`padWEff`, floored at `MIN_PAD_SCALE`) the longer a rally stays at `maxSpeed()` without a goal, via `fullSpeedStreak` (reset in `serve()`).
- `lockSession()` — fires from a `setTimeout(lockSession, SESSION_LIMIT_MS)` set once at load (10 minutes). Sets `S.locked`, freezes Pong, shows `#timeout`. Checked as a guard at the top of `kickoff()`, `parentGate()`, and `Pong.start()`, and in the Pong keydown listener. Only cleared by reloading the page (the timer isn't persisted).
- Sound: `speak()` (speechSynthesis), `whistle()`, `cheer()`, `thud()` (Web Audio, no audio files).

### Adding things
- **New question type:** add a branch in `make()`, a render branch in `next()`, a chip with `class="chip mode" data-mode="..."` inside `#modePicker` on the start screen, and a gentle wrong-answer line in `answer()`. It's automatically included when "All games" is on since that reads from `ALL_MODES` — add it there too.
- **New team:** add an entry to `TEAMS` (`name`, `main`, `trim`, `confetti`), a `.team` button on the start screen, and a `.pbtn.<team>` style. If the kit isn't a plain colour, special-case it in `shirt()` and `drawPad()` like `red` (stripes) and `claret` (sleeves).
- **Writing games:** `#writing` is currently just a coming-soon placeholder reached from `#category`. Not designed yet.

## Design rules to keep
- Wrong answers are never punished: the button wobbles and fades, and a kind voice line plays. No opponent goals, no in-question timers (the 10-minute *session* limit is a screen-time guard, not gameplay pressure — it never counts down visibly or rushes an answer).
- Questions are always read aloud (child can't read yet); 🔊 repeats.
- Big tap targets, sentence case, minimal text.
- Respect `prefers-reduced-motion` (animations are skipped; see the `reduced` flag).
- Teams are kit colours only. Don't draw real club crests, mascots or badges, or use club names in the UI.
- Safe-area insets are handled for phones with notches.

## Ideas discussed but not built
- A more forgiving streak (a miss knocks off a couple of bars instead of resetting to zero).
- Letting the grown-up choose their own team in the football game.
- Writing games (category exists on the home screen, but the games themselves aren't designed yet).
- A configurable "numbers up to" level again (currently hardcoded to `MAX_NUM = 20`).

## Testing
There are no automated tests. Before committing, at minimum:
- syntax-check the script: `node -e "const h=require('fs').readFileSync('index.html','utf8');new Function(h.split('<script>')[1].split('</script>')[0])"`
- open it on a real phone and try each game mode, a full match, and the football game in both control modes with two players.
