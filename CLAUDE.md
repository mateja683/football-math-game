# CLAUDE.md — Goal Maths

Context for continuing this project in Claude Code.

## What this is
A learning game for a 4-year-old, played with a parent: maths, reading and writing. Football theme throughout (goals, prize match), even for reading/writing. UK English throughout ("maths", en-GB speech voice). Mobile-first, portrait phone is the main target; also works on tablet and desktop.

## Architecture
Everything lives in `index.html`: inline CSS, inline JS, no framework, no build, no dependencies apart from the Google Fonts stylesheet (Baloo 2, with system fallbacks). Keep it single-file unless there's a strong reason to split.

Screens are sibling `<main class="app">` elements toggled by `show(id)`: `category` (the very first screen — reading/writing/maths), `start` (maths setup, reached from category), `play`, `end`, `prize`, `gate` (parent multiplication check), `reading`, `writing`, `wordEnd` (shared round-complete screen for reading and writing), plus the full-screen `#pong` overlay and the `#timeout` session-lock overlay (the latter isn't part of `show()` — it's toggled directly and sits above everything, `z-index:100`).

State is one object `S` (team, allGames, modes, voice, streak, prizes, controls, locked, and per-match values). `save()` persists the settings part to localStorage under the key `goalmaths`. All storage access is wrapped in try/catch — keep it that way. Maths difficulty is hardcoded via the `MAX_NUM` constant (20), not a user setting — there's no "numbers up to" UI any more. Reading/writing state (`RD`, `WR`) is separate and not persisted — each round always starts fresh.

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
- `lockSession()` — fires from a `setTimeout(lockSession, SESSION_LIMIT_MS)` set once at load (20 minutes). Sets `S.locked`, freezes Pong, aborts any active speech recognition, shows `#timeout`. Checked as a guard at the top of `kickoff()`, `parentGate()`, `readStart()`, `writeStart()`, and `Pong.start()`, and in the Pong keydown listener. Only cleared by reloading the page (the timer isn't persisted).
- `readStart()`/`readNext()`/`readMark(correct)` — reading game (screen `reading`, state `RD`). Shows a word from `READ_WORDS` (3–4 letters) in large print — deliberately **not** spoken aloud, unlike maths questions, since the whole point is the child decoding it themselves. `recognizer` (Web Speech API, feature-detected from `SpeechRecognition`/`webkitSpeechRecognition`) drives a hold-to-listen 🎤 button (`pointerdown` → `recognizer.start()`, release → `recognizer.stop()`) that's the only control shown by default when supported. It only ever auto-*confirms* a correct read; a mismatch just invites another hold, never marks it wrong. `#readAskGrownup` is a small always-present link that reveals the manual `#readRight`/`#readAgain` buttons (`#readJudge`) for that word if the mic keeps struggling; it resets to mic-only on the next word. When `recognizer` is unavailable, the mic/link are hidden and `#readJudge` shows directly instead — that's the sole fallback path in that case.
- `writeStart()`/`writeNext()`/`writeMark(correct)` — writing game (screen `writing`, state `WR`). Shows a picture (`WORDS3`/`4`/`5`, up to 5 letters) with the word hidden until "Show hint"; the child writes on the `#wc` canvas (plain pointer-event freehand drawing, Apple Pencil/touch/mouse all work via Pointer Events same as `Pong`). There's no handwriting OCR — a backend or ML model would be needed for that and would contradict the single-file/no-dependencies architecture, and wouldn't be reliable for a young child's handwriting anyway — so `#writeRight`/`#writeAgain` (parent-judged) is the only marking mechanism, matching the app's "played with a parent" design.
- `wordEnd(kind, msg, replay)` — shared round-complete screen for both reading and writing (`WORD_ROUND` = 8 words/round); `replay` is `readStart` or `writeStart` depending which just finished.
- Sound: `speak()` (speechSynthesis), `whistle()`, `cheer()`, `thud()` (Web Audio, no audio files).

### Adding things
- **New question type:** add a branch in `make()`, a render branch in `next()`, a chip with `class="chip mode" data-mode="..."` inside `#modePicker` on the start screen, and a gentle wrong-answer line in `answer()`. It's automatically included when "All games" is on since that reads from `ALL_MODES` — add it there too.
- **New team:** add an entry to `TEAMS` (`name`, `main`, `trim`, `confetti`), a `.team` button on the start screen, and a `.pbtn.<team>` style. Kit colours only — no real club names, crests, or mascots in the UI (see below); match a real kit's *colours* if asked, but label it generically (e.g. `navy`/"Blue and white" for a Chelsea-style kit). If the kit isn't a plain colour, add it to `STRIPED` (vertical stripes, like `red`/`garnet`) or `SLEEVED` (contrast sleeve panels, like `claret`/`amber`) instead of special-casing `shirt()`/`drawPad()` directly — both are already generic over those two lists.
- **New reading/writing words:** add `{w, e}` (word, emoji) to `WORDS3`/`WORDS4`/`WORDS5`. Keep emoji unambiguous — it's the only "picture" support, no image assets.

## Design rules to keep
- Wrong answers are never punished: the button wobbles and fades, and a kind voice line plays. No opponent goals, no in-question timers (the session limit is a screen-time guard, not gameplay pressure — it never counts down visibly or rushes an answer). Speech recognition in the reading game follows the same rule: a mismatch never marks the child wrong, it only asks the grown-up to check.
- Maths questions are always read aloud (child can't read yet); 🔊 repeats. Reading-game words are the deliberate exception — never auto-spoken, since reading them is the point.
- Big tap targets, sentence case, minimal text.
- Respect `prefers-reduced-motion` (animations are skipped; see the `reduced` flag).
- Teams are kit colours only. Don't draw real club crests, mascots or badges, or use club names in the UI — even when asked to match a specific real club, use its colours under a generic name (see "Adding things" above).
- Safe-area insets are handled for phones with notches.

## Ideas discussed but not built
- A more forgiving streak (a miss knocks off a couple of bars instead of resetting to zero).
- Letting the grown-up choose their own team in the football game.
- A configurable "numbers up to" level again (currently hardcoded to `MAX_NUM = 20`).
- Difficulty levels / word-length picker for reading and writing (currently a fixed 3–4 letter pool for reading, 3–5 for writing, no way to narrow it).
- True handwriting recognition for the writing game (would need a backend/ML model — see `writeMark` above for why that's out of scope here).

## Testing
There are no automated tests. Before committing, at minimum:
- syntax-check the script: `node -e "const h=require('fs').readFileSync('index.html','utf8');new Function(h.split('<script>')[1].split('</script>')[0])"`
- open it on a real phone and try each game mode, a full match, and the football game in both control modes with two players.
