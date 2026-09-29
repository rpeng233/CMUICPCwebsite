# agents.md — CMU ICPC results site

Static site: `index.html` (season schedule + list of finished contests) plus one
scoreboard per contest, named `YYYYMMDD<Contest>.html` (or `.pdf` for older ones).
No build step; edit the HTML directly.

## How CMU teams are displayed in a scoreboard

Reference implementation: `20260912ECprelim2.html` (also applied to
`20260907ECprelim1.html`). Any new scoreboard must follow these rules.

1. **Row marker.** Every CMU row gets `class="cmu"` on the `<tr>` (replacing any
   other row class such as `stand0x`/`solver`). Keep existing `id`s
   (`cmu-team-01`, `cmu-toad`, ...) so deep links keep working.

2. **Unnumbered.** The rank cell of a CMU row is empty (`<td class="rk"></td>`,
   or `<td class="stnd"></td>` in the raw-QOJ layout). CMU teams never carry a
   rank, official or not.

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

       /* prelim2-style layout */
       tbody tr.cmu td.rk, tbody tr.cmu td.nm, tbody tr.cmu td.st, tbody tr.cmu td.na { background: #d2a679; }
       /* raw QOJ layout (td.stnd = rank, name, "-" cells, stats) */
       tr.cmu td.stnd { background: #d2a679; }

   Do not colour the whole row (the old `class="solver"` approach).

5. **Team name format.** `CMU-<team> (First Last, First Last, First Last)` —
   prefix `CMU-`, a space before the opening parenthesis, members in English
   order separated by `, `. When the QOJ handle differs from the team name,
   keep `data-original-name="<handle>"` and a `title="Original: <handle>. ..."`
   on the name cell.

6. **Projected rows** (a CMU team that did not take part but whose result is
   estimated): insert a `class="cmu"` row at the median of the teams with the
   same solve count, with an empty rank and empty penalty/dirt cells. Problems
   deemed solved but not actually submitted get an accepted-coloured cell with
   `&nbsp;<br>&nbsp;` instead of `+`/time. Append
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
order: `Mon D, YYYY, *<A HREF="<file>">Name</A>::: S1 (R1), S2 (R2), ...` where
`S` = problems solved and `R` = the rank position in the original ranklist.
`+` after a rank means the team was unofficial (virtual / multi-keyboard) and
was mixed into the original ranks; `(projected, R+)` marks an estimated row.
