# Goal Maths

A football-themed maths game for a 4-year-old, played together with a grown-up on a phone or tablet.

Answer a question right and the ball flies into the net. Five goals wins the match. Ten right-first-time answers in a row wins a two-player football mini game (the "prize match"), which can also be opened straight from the home screen.

## Running it

It's a single static file with no build step and no dependencies.

- Open `index.html` in a browser, or
- serve the folder locally: `npx serve .` or `python3 -m http.server`, then open the printed address on your phone (same Wi-Fi), or
- host it free on GitHub Pages: repo Settings → Pages → deploy from branch `main`, folder `/ (root)`.

Speech (questions read aloud) uses the browser's built-in voices, so how it sounds depends on the device.

## What's in it

**Games**
- Counting — how many balls?
- Adding — two groups, how many altogether?
- Taking away — some balls roll away, how many are left?
- Who has more? — two teams of shirts, tap the bigger team

**Harder games**
- Missing number — numbered shirts with one blank (up to 20 on the "up to 10" level)
- Make 5 or 10 — balls plus empty spaces, how many more?
- Quick look — balls flash then hide
- Bigger number — two numerals, tap the bigger one

**Settings** (remembered in the browser via localStorage): team, numbers up to 5 or 10, which games, voice on/off, prize-match controls (slide a finger or arrow buttons).

**Teams** are kit colours only: blue and yellow, red and white stripes, claret and blue. No club crests, mascots or names are used.
