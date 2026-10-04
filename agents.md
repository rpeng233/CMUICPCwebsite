# agents.md — CMU ICPC results site

Static site: `index.html` (season schedule + list of finished contests) plus one
scoreboard per contest, named `YYYYMMDD<Contest>.html` (or `.pdf` for older ones).
No build step; edit the HTML directly.

## Scoreboard files

Scoreboards are rewritten into the compact markup of `20260912ECprelim2.html`
rather than saved as raw QOJ pages (a raw save is ~3.5 MB; compact is ~0.1–0.3 MB):

- one inline `<style>` block (copy it from prelim2), `<h1>` contest name,
  `<title>QOJ<id> - <contest name></title>`;
- `<thead>`: `<th>Rank.</th><th>Username</th>`, one `<th class="p">` per
  problem (`class="p0"` if nobody solved it) with `Letter<br>solved/submits`,
  then `<th>Solved</th><th>Penalty</th><th>Dirt</th>`;
- one `<tr>` per team, one line each, cells `td.rk` (rank), `td.nm` (name),
  per problem `td.ac` (accepted, `+k<br>h:mm`), `td.fs` (first solve),
  `td.wa` (rejected, `-k<br>h:mm`) or `td.na` (`-`, not attempted), then
  three `td.st` (solved, penalty, dirt);
- no per-row `title=` tooltips / registration data; team names are
  `<School>-<team> (First Last, ...)` as text only;
- team names are translated: Chinese QOJ handles
  `学校 - 队名 - 姓名, 姓名, 姓名` become `<School>-<Team name in English>
  (Given Surname, ...)` (school = usual English name/abbreviation, team name
  translated, members in pinyin, English order); handles in other scripts
  (Japanese, Russian) are translated too, Latin-script handles are kept.
  Say "Team names translated/transliterated from the QOJ handles" in the
  footnote (CCPC prelim, UCup 5-3);
- a browser "save page" (`<name>.htm` + `<name>_files/` with QOJ's css/js) is
  only the raw input: convert it and delete both, never commit them;
- only the top of the ranklist is kept: cut at a solve-count boundary so
  that every team within one solve of the weakest CMU team is included
  (prelim1: all teams with >= 6 solves, 613 of 2643 rows), and say so in the
  footnote (`Rows shown: QOJ ranks 1–N ...`).

### Scoreboards that come as official + unofficial PDFs (BOCA, e.g. LAC)

Merge both into one file (`20260307LAC26.html`) in the same compact markup and
delete the PDFs:

- one ranklist ordered by solved desc, then penalty asc; official teams keep
  their official rank in `td.rk`, unofficial teams get an empty rank cell and
  no other marker (no `[CCL]`/"unofficial" tag, no class) — only CMU rows get
  `class="cmu"` and the highlight;
- official team names stay as in the source (`[School] team`); unofficial
  names drop the division tag, CMU teams become `CMU-Team N (First Last, ...)`
  (rosters are not in the PDFs; take them from index.html / the coach);
- BOCA cell `k/t` -> `td.ac` `+(k-1)<br>h:mm`, `k/-` -> `td.wa` `-k` (no time
  is available for rejected runs); penalty is copied from the source total,
  first-solve marks and header `solved/submits` are computed from the rows
  shown; say so in the footnote, together with the team counts of each
  source and the scoreboard URL.

## How CMU teams are displayed in a scoreboard

Reference implementation: `20260912ECprelim2.html` (also applied to
`20260907ECprelim1.html`). Any new scoreboard must follow these rules.

1. **Row marker.** Every CMU row gets `class="cmu"` on the `<tr>` (other rows
   have no class). Keep existing `id`s (`cmu-team-01`, `cmu-toad`, ...) so
   deep links keep working.

2. **Unnumbered when unofficial.** When CMU took part unofficially (online
   qualifiers it cannot advance in, virtual/multi-keyboard runs, projected
   rows) the rank cell of a CMU row is empty (`<td class="rk"></td>`) and
   rule 3 applies. When CMU took part officially (UCup stages, onsite
   regionals) every row keeps its source rank, CMU rows included; only the
   `class="cmu"` highlight marks them, and the footnote says "CMU teams took
   part officially and keep their QOJ ranks" (UCup 5-3).

3. **Other ranks exclude CMU teams and preserve the original ties.** Ranks are
   competition-style (rank = 1 + number of teams strictly better), so for every
   non-CMU team

       new rank = original rank − (number of CMU teams with a strictly smaller original rank)

   Tied teams keep sharing a rank and the following gap is kept. Do not
   re-sort rows; CMU rows stay in their original position in the table.

4. **Highlight colour `#d2a679`,** applied only to the non-result cells of a CMU
   row: rank, team name, unattempted problems and the summary columns
   (Solved / Penalty / Dirt). Per-problem result cells keep their normal
   accepted / first-solve / rejected colours so the row is still readable.

       tbody tr.cmu td.rk, tbody tr.cmu td.nm, tbody tr.cmu td.st, tbody tr.cmu td.na { background: #d2a679; }

   Do not colour the whole row.

5. **Team name format.** `CMU-<team> (First Last, First Last, First Last)` —
   prefix `CMU-`, a space before the opening parenthesis, members in English
   order separated by `, `. When the QOJ handle differs from the team name,
   keep `data-original-name="<handle>"` and a `title="Original: <handle>. ..."`
   on the name cell.

6. **Projected rows** (a CMU team that did not take part but whose result is
   estimated): insert a `class="cmu"` row at the median of the teams with the
   same solve count, with an empty rank and empty penalty/dirt cells. Problems
   deemed solved but not actually submitted get `<td class="ac proj">PROJ</td>`:
   accepted colour, the text `PROJ` instead of `+`/time, and a diagonal X drawn
   across the whole cell (the `td.proj` rule in the `<style>` block, two
   `linear-gradient` layers over `#dfffdf`). Append
   `<span class="tag">(projected, ...)</span>` to the team name explaining
   what was actually done and what is assumed. Projected rows do not shift
   anyone's rank (they were never in the original ranklist).

7. **Footnote.** After the table add `<p class="foot">` stating, in this order:
   any projected/absent CMU team and where its row was placed; "All CMU teams
   are unnumbered; other ranks exclude CMU teams and preserve the original
   ties."; the range of rows shown (`Rows shown: QOJ ranks 1–N.`); and the
   source (`Source: qoj.ac/results/QOJ<id>.`).

## index.html

Under "Finished Contests", each contest is a `<li>` in reverse chronological
order: `Mon D, YYYY, *<A HREF="<file>">Name</A>::: S1 (R1), S2 (R2), ...`, one
entry per CMU team, where `S` = problems solved and `R` = the number of
non-CMU teams that finished strictly ahead of that team, i.e. the rank shown
on the last non-CMU row above it in the scoreboard (one less if that row is
tied with the CMU team). Other CMU teams are never counted, so two CMU teams
adjacent in the ranklist get the same `R`. `+` after a rank means the team
was not physically present at the contest (online / virtual participation);
`(projected, R+)` marks an estimated row.

When a row has no displayed rank (an unofficial team in a merged
official+unofficial scoreboard), it is skipped when looking for the last
ranked row above the CMU team: LAC26 = `*6 (6+), *6 (8+)` (CMU-Team 1 sits
between official ranks 6 and 7, CMU-Team 5 between 8 and 9).

Official participations (UCup stages, onsite regionals) list the source rank
with no `+`: UCup 5-3 = `5 (68), 4 (110)`. `+` and the row-above rule are for
unofficial rows only; `*` (cannot advance) is kept for qualifiers CMU is not
eligible for (EC/CCPC preliminaries).

Examples: prelim2 = `7 (49+), 6 (69+), 6 (projected, 80+)` (rows sit between
displayed ranks 49/50, 69/70 and 80/81); prelim1 = `9 (54+), 7 (142+), 7 (142+)`
(QOJ ranks 55 — tied with the row above at 55 —, 144 and 145; the 145 team has
one non-CMU team fewer ahead than its QOJ rank suggests because rank 55 was CMU,
and the CMU team at 144 is not counted).
