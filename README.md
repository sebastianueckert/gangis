# Gångis

A small single-page app (in Swedish) for learning the multiplication tables 1–10.

**▶ [Spela Gångis](https://sebastianueckert.github.io/gangis/)**

Or open `index.html` in a browser. No build step and no install needed.

**Install it as an app:** open the link on a phone or tablet and add it to the home screen (Safari: share button → *Lägg till på hemskärmen*; Chrome: menu → *Installera app*). It then opens full screen with its own icon and works offline. On iPhone/iPad the installed app keeps its own saved progress, separate from the Safari tab, so install it before she starts playing.

The look: a pixel-art garden (in the spirit of Stardew Valley and Minecraft) inside a bright, rounded interface. The garden is alive: plants sway in the wind, visitors flutter and buzz, buds about to open twinkle, and after a round the plants she cared for grow one by one in front of her (with a small burst at each new stage). With "reduce motion" switched on in the device settings, the garden stays still.

- **The garden** (start screen): a generated top-down pixel garden built on the 10×10 table. Fact a·b grows at row a, column b, with numbered stakes along the edges; tapping a plant ties planting strings along its row and column and shows the fact. 7·8 and 8·7 grow the same (randomly chosen) plant, so the garden is mirrored along the diagonal.
- **Sköt om trädgården**: 20 questions, picked mostly from the facts that need it. Thirsty plants come first: each one is asked early in the round (up to 12 of the 20 questions), and plants that are still dry after that come up more often. Every round also asks at least one bud (at a random point in the round), most likely one that is about to open, so a flower can bloom. A missed question comes back once, 3 questions later, on top of the 20; that second try counts like any answer, so a fast correct one makes up for the miss. Only these rounds make plants grow. The report shows the plants she cared for (before → after) and how many buds can open next round.
- **Öva tabeller**: relaxed practice of the tables you pick. Missed questions come back, and dot hints are available.
- **Utforska tabellerna**: an interactive 10×10 table with dot pictures and a tip for each table. Facts that are hard for her right now are shown in red (both 7·8 and 8·7), and the explanation says so when she taps one. A fact counts as hard once she has answered it at least twice and either missed it in one of her last 3 tries without making up for it since (a correct answer with less than 3 s thinking time clears a miss), or her typical thinking time on it is 3 s or more (`HARD_*` in `SP`). A flowering plant is therefore only red if the fact really has slipped.
- **Trädgårdens besökare**: everything that has come to stay in the garden.

## How the garden works

- **Size** of a plant = the best greenness its fact has ever reached (below), plus a little for every correct answer however slow (+4 % each, at most +40 % and never past 55 %, so buds and flowers still need speed). It is drawn continuously: it gains size and leaves with every bit of progress, so each good round visibly changes the garden. Stages: seed < 20 %, sprout 20–40 %, plant 40–60 %, bud 60–99 % (the bud swells and colours up), flower at 100 %. Plants never shrink.
- **About to open**: buds whose current greenness is at least 85 % (about two fast answers from flowering) sparkle.
- **Thirst**: when a fact's current greenness falls more than 15 points below its best, the plant droops: pale cracked soil and yellowed leaves. A couple of good answers perk it up, and the next watering round asks thirsty plants first.
- **Days in a row**: any finished round counts for the day and the start screen shows today's rounds. One missed day per week counts as rain. The streak is only a counter; no rewards depend on it.
- **Visitors**: one new animal each time garden maturity (the average size of all 55 plants) passes another 10 %. It lands on the plant that grew most that round and lives there. **Full rows**: when a whole row flowers she knows that table; the row and column stakes get a rosette. **Decorations**: bench (record 25 %), bird box (10 flowering plants), pond with a frog (50 %), hedgehog (25 flowering plants), apple tree (75 %), greenhouse (100 %). All are permanent.

## How the speed score works

Each fact (7·8 and 8·7 count as one, 55 facts) has an estimate of her *thinking time*:

- One answer scores `time to OK − typing allowance`, capped at `C = 8 s`. A wrong answer scores `C`.
- The typing allowance is her running average time on the easy ×1 (one-digit answers) and ×10 (two-digit answers) questions.
- Typo corrections don't count: if she presses ⌫, the time is the time until her first key plus the time from her last ⌫ to OK.
- The estimate has two parts, both on a log scale (1 s vs 2 s weighs like 4 s vs 8 s):
  - **Typical time** `m`: her usual thinking time on the fact. Only answers move it, never the passing of time.
  - **Uncertainty** `v`: how sure we are about `m`. Every answer makes it smaller, and it grows again while the fact isn't practised.
- An answer moves the typical time by the share `v / (v + NOISE²)`. When we're unsure (a new fact, or after a break), one answer counts for a lot, so one fast answer after a week off brings the fact straight back. When we're sure, it counts for less, though always at least 1/4 (the uncertainty never drops below `V_MIN`). A fact's first answer counts for 1/3. A single slow correct answer counts as at most 1.5 s slower than the cautious time below, so one distracted moment barely shows.
- Without practice the uncertainty grows by `DRIFT` per half-life. Each fact has its own half-life: it starts at 2 days, grows ×1.6 when she answers fast (≤ 1.5 s thinking time) at least half a half-life after the last time she saw that fact, and halves after a wrong answer (limits 1–60 days). Well-known facts stay certain for a long time, shaky ones don't. The info panel for a plant says roughly how many days it lasts before it gets thirsty.
- The garden counts with the **cautious time** `e^(m + √v)`, meaning "we're fairly sure she's at least this fast". A fact is fully green when the cautious time is 1.8 s or less. Because the uncertainty never drops below its floor, the cautious time is always about 20 % above her typical time, so a flower needs a typical time of about 1.5 s. "Grönt nu" is the average greenness over all 55 facts; the record (🏆) is the best value ever reached and never fades; it is also raised when the app opens if "Grönt nu" is above it.

What that means for each plant:

| | Comes from | Changes when |
|---|---|---|
| **Greenness** (current) | `(C − cautious time) / (C − 1.8 s)` | falls while the fact isn't practised (the uncertainty grows), rises with answers |
| **Size** | the best greenness ever reached right after an answer, plus the effort bonus | only answers make it grow; it never shrinks |
| **Thirst** | how far the current greenness is below the best (`(best − greenness − 0.15) / 0.45`) | grows with time off, and one fast answer usually fixes it |

Saves from the earlier model (one estimate that faded back towards `C`) are converted once when the app loads: the old estimate becomes the typical time, sizes, effort, half-lives, the record and stored answers are kept, and the old save is copied to `gangis-v1-before-uncertainty` in localStorage.

The parameters are first guesses and live in the `SP` object in `index.html`. The last 20 raw attempts per fact are stored too, so the scoring can be retuned later without losing history.

## Switching rows off

The ⚙ button at the top opens *Vilka rader är med?*, where rows (tables) are switched on and off, so a younger child can start with only some tables. It is open to the child on purpose: the full garden needs all ten rows, which is a goal in itself. A fact is in play when its row or its column is on: with only the 2- and 5-tables on, 2·7 is still asked (and its plant grows at 2·7 and 7·2), but 7·8 is not. Plots out of play turn into lawn: nothing is asked there, nothing grows there and no visitors land there (visitors already in the garden move to a plant in a row that is on). The stakes of switched-off rows are greyed out, and *Öva tabeller* only offers the rows that are on.

100 % still means the whole 10×10 table. "Grönt nu", the record and garden maturity count the plants in play over all 55 plots, so with fewer rows the garden can only get part of the way (19 of 55 plants with rows 2 and 5, so at most 35 %, 3 of 10 visitors and 2 of 6 decorations). The settings screen says how far the garden can get, and the visitors page marks visitors and decorations that need more rows. Visitors and decorations already earned are kept when rows are switched off, and answers already given are kept, so switching a row back on brings its plants back as they were. At least one row always stays on.

Progress is saved in the browser (localStorage). Tap the Gångis logo five times for a hidden parent view with typing allowance, per-fact typical and current cautious thinking times, half-lives and a "Kopiera data" button. To start over, tap "Hej, …!" in the news line on the start screen and choose "Börja om från början".
