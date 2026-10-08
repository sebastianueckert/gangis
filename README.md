# Gångis

A small single-page app (in Swedish) for learning the multiplication tables 1–10.

**▶ [Spela Gångis](https://sebastianueckert.github.io/gangis/)**

Or open `index.html` in a browser. No build step and no install needed.

- **Gröna rutan** (main mode): a 10×10 board where each square turns from grey to green as she answers that fact correctly and quickly. 20 questions per round, picked mostly from the greyest squares.
- **Öva tabeller**: relaxed practice of the tables you pick. Missed questions come back, and dot hints are available.
- **Utforska tabellerna**: an interactive 10×10 table with dot pictures and a tip for each table.
- **Min djursamling**: collect 30 animals by doing well.

## How Gröna rutan scores

Each fact (7·8 and 8·7 count as one, 55 facts) has an estimate of her *thinking time*:

- One answer scores `time to OK − typing allowance`, capped at `C = 8 s`. A wrong answer scores `C`.
- The typing allowance is her running average time on the easy ×1 (one-digit answers) and ×10 (two-digit answers) questions.
- Typo corrections don't count: if she presses ⌫, the time is the time until her first key plus the time from her last ⌫ to OK.
- The estimate is a weighted average, `E ← 0.7·E + 0.3·score`, starting at `C`. One correct answer can raise `E` by at most 1.5 s, so a single slow moment barely shows.
- Without practice, the estimate fades back towards `C` with a half-life of 2 days.
- A square is fully green at `E ≤ 1 s`. "Grönt nu" is the average greenness over all 55 facts; the record (🏆) is the best value ever reached and never fades.

The parameters are first guesses and live in the `SP` object in `index.html`. The last 20 raw attempts per fact are stored too, so the scoring can be retuned later without losing history.

Progress is saved in the browser (localStorage).
