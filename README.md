# Gångis

A small single-page app (in Swedish) for learning the multiplication tables 1–10.

**▶ [Spela Gångis](https://sebastianueckert.github.io/gangis/)**

Or open `index.html` in a browser. No build step and no install needed.

- **The garden** (start screen): a generated top-down pixel garden built on the 10×10 table. Fact a·b grows at row a, column b, with numbered stakes along the edges; tapping a plant ties planting strings along its row and column and shows the fact. 7·8 and 8·7 grow the same (randomly chosen) plant, so the garden is mirrored along the diagonal.
- **Vattna trädgården**: 20 questions, picked mostly from the facts that need it. Only these rounds make plants grow.
- **Öva tabeller**: relaxed practice of the tables you pick. Missed questions come back, and dot hints are available.
- **Utforska tabellerna**: an interactive 10×10 table with dot pictures and a tip for each table.
- **Trädgårdens besökare**: everything that has come to stay in the garden.

## How the garden works

- **Size** of a plant = the best greenness its fact has ever reached (below): seed < 20 %, sprout 20–40 %, plant 40–60 %, bud 60–99 %, flower at 100 %. Plants never shrink.
- **Thirst**: when a fact's current greenness falls more than 15 points below its best, the plant droops: pale cracked soil and yellowed leaves. A couple of good answers perk it up.
- **Watering**: any finished round waters the garden for the day. One missed day per week counts as rain and doesn't break the day streak.
- **Visitors** move in when a whole row flowers (one animal per row). **Decorations**: bench (record 25 %), bird box (7 days in a row), pond with a frog (50 %), hedgehog (14 days), apple tree (75 %), greenhouse (100 %). All are permanent.

## How the speed score works

Each fact (7·8 and 8·7 count as one, 55 facts) has an estimate of her *thinking time*:

- One answer scores `time to OK − typing allowance`, capped at `C = 8 s`. A wrong answer scores `C`.
- The typing allowance is her running average time on the easy ×1 (one-digit answers) and ×10 (two-digit answers) questions.
- Typo corrections don't count: if she presses ⌫, the time is the time until her first key plus the time from her last ⌫ to OK.
- The estimate is a weighted average, `E ← 0.7·E + 0.3·score`, starting at `C`. One correct answer can raise `E` by at most 1.5 s, so a single slow moment barely shows.
- Without practice, the estimate fades back towards `C` with a half-life of 2 days.
- A square is fully green at `E ≤ 1 s`. "Grönt nu" is the average greenness over all 55 facts; the record (🏆) is the best value ever reached and never fades.

The parameters are first guesses and live in the `SP` object in `index.html`. The last 20 raw attempts per fact are stored too, so the scoring can be retuned later without losing history.

Progress is saved in the browser (localStorage).
