# Gångis

A small single-page app (in Swedish) for learning the multiplication tables 1–10.

**▶ [Spela Gångis](https://sebastianueckert.github.io/gangis/)**

Or open `index.html` in a browser. No build step and no install needed.

The look: a pixel-art garden (in the spirit of Stardew Valley and Minecraft) inside a bright, rounded interface. The garden is alive: plants sway in the wind, visitors flutter and buzz, buds about to open twinkle, and after a round the plants she cared for grow one by one in front of her (with a small burst at each new stage). With "reduce motion" switched on in the device settings, the garden stays still.

- **The garden** (start screen): a generated top-down pixel garden built on the 10×10 table. Fact a·b grows at row a, column b, with numbered stakes along the edges; tapping a plant ties planting strings along its row and column and shows the fact. 7·8 and 8·7 grow the same (randomly chosen) plant, so the garden is mirrored along the diagonal.
- **Sköt om trädgården**: 20 questions, picked mostly from the facts that need it. A missed question comes back once, 3 questions later, on top of the 20; that second try counts like any answer, so a fast correct one makes up for the miss. Only these rounds make plants grow. The report shows the plants she cared for (before → after) and how many buds can open next round.
- **Öva tabeller**: relaxed practice of the tables you pick. Missed questions come back, and dot hints are available.
- **Utforska tabellerna**: an interactive 10×10 table with dot pictures and a tip for each table.
- **Trädgårdens besökare**: everything that has come to stay in the garden.

## How the garden works

- **Size** of a plant = the best greenness its fact has ever reached (below), drawn continuously: it gains size and leaves with every bit of progress, so each good round visibly changes the garden. Stages: seed < 20 %, sprout 20–40 %, plant 40–60 %, bud 60–99 % (the bud swells and colours up), flower at 100 %. Plants never shrink.
- **About to open**: buds whose current greenness is at least 85 % (about two fast answers from flowering) sparkle.
- **Thirst**: when a fact's current greenness falls more than 15 points below its best, the plant droops: pale cracked soil and yellowed leaves. A couple of good answers perk it up.
- **Days in a row**: any finished round counts for the day and the start screen shows today's rounds. One missed day per week counts as rain. The streak is only a counter; no rewards depend on it.
- **Visitors**: one new animal each time garden maturity (the average size of all 55 plants) passes another 10 %. It lands on the plant that grew most that round and lives there. **Full rows**: when a whole row flowers she knows that table; the row and column stakes get a rosette. **Decorations**: bench (record 25 %), bird box (10 flowering plants), pond with a frog (50 %), hedgehog (25 flowering plants), apple tree (75 %), greenhouse (100 %). All are permanent.

## How the speed score works

Each fact (7·8 and 8·7 count as one, 55 facts) has an estimate of her *thinking time*:

- One answer scores `time to OK − typing allowance`, capped at `C = 8 s`. A wrong answer scores `C`.
- The typing allowance is her running average time on the easy ×1 (one-digit answers) and ×10 (two-digit answers) questions.
- Typo corrections don't count: if she presses ⌫, the time is the time until her first key plus the time from her last ⌫ to OK.
- The estimate is a weighted average, `E ← 0.7·E + 0.3·score`, starting at `C`. One correct answer can raise `E` by at most 1.5 s, so a single slow moment barely shows.
- Without practice, the estimate fades back towards `C`. Each fact has its own half-life for this: it starts at 2 days, grows ×1.6 when she answers fast (≤ 1 s thinking time) at least half a half-life after the last time she saw that fact, and halves after a wrong answer (limits 1–60 days). Well-known facts fade slowly, shaky ones quickly. The info panel for a plant says roughly how many days it lasts before it gets thirsty.
- A fact is fully green at `E ≤ 1.5 s` (raised from 1 s; stored sizes are rescaled once when the limit changes). "Grönt nu" is the average greenness over all 55 facts; the record (🏆) is the best value ever reached and never fades.

The parameters are first guesses and live in the `SP` object in `index.html`. The last 20 raw attempts per fact are stored too, so the scoring can be retuned later without losing history.

Progress is saved in the browser (localStorage). Tap the Gångis logo five times for a hidden parent view with typing allowance, per-fact thinking times, half-lives and a "Kopiera data" button. To start over, tap "Hej, …!" in the news line on the start screen and choose "Börja om från början".
