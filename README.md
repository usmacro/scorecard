# USMacro scorecard — the ledger

This repository is the public archive of [usmacro.com/scorecard](https://usmacro.com/scorecard): every forward call the
site has published, dated the day it first appeared, and how each one was graded when its window ended. The site writes
it every night; nothing here is typed by hand, and nothing is ever removed.

## The file

`calls.jsonl` — one JSON object per line, append-only, in the order the lines were written. The ledger holds 27 lines:
20 calls, 3 resolutions and 4 corrections, the first call made on 2026-09-19.

**A call line** (`status` `pending`), written the night the site first showed the call:

```
{"basis":"capture","composite":"forward_stress","data_month":"2026-11","evidence":"https://usmacro.com/state-of-the-economy?date=2026-11-03","first_published":"2026-11-03","from":"healthy","from_rung":1,"h":"b6ee5c22472a2358f0c61b1b411a88b2badf3e22506f67cff80ed14d5ac3a6dd","id":"band-forward_stress-2026-11-03","live_score":"55.56","made_on":"2026-11-03","page":"/state-of-the-economy/methodology#band-crossing","reading":"live score 55.56 at the 15:20 UTC capture","rule":"A band crossing is a call the day the live reading first shows it. It held if the next month to settle after the crossing reads in the new band or beyond it in the same direction; it did not hold otherwise. The window ends when that month settles.","settles":"2026-10","status":"pending","subject":"Forward Stress","to":"neutral","to_rung":2,"type":"gauge_band","window":"on that month's settle date, the day its month-end score is written to the record, never restated after","window_end":"2026-11-05"}
```

- `id` — the call's key; a resolution carries the same `id`.
- `made_on` / `first_published` — the day the site first showed the call (never earlier; `basis` says `capture` when
  a stored capture of the live site shows it, `record` when it is read from the site's stored point-in-time record).
- `type`, `subject`, `from`, `to`, `reading`, `data_month` — what the call is.
- `rule` — the sentence that grades it, the engine's own, fixed on the line the day the call is made; `window` and
  `window_end` say when.
- `evidence` — the site's page where the call can be seen: as it stood that day where the page reads a date (the
  address ends `?date=` and the day), as it is today where the page has no dated view.
- `h` — the line's hash: sha256 over the previous line's `h` and this line without `h`.

**A resolution line** (`status` `resolved`) carries the same `id`, `resolved_on`, `outcome` (`held` or
`did_not_hold`), `resolving_reading` and `outcome_word` (the rule's own word). A call is graded once; a later revision
of the data never regrades it.

**A correction line** (`status` `corrected`) carries the same `id`, `field` (`evidence`), `old`, `new`, `reason` and
`corrected_on`: a link written wrong is corrected by a new line, and the page shows the new link with a note. The
call's own line stays as written.

## The append-only rule

A line is never edited. Each line's `h` chains to the line before (the first chains from 64 zeros), so a changed,
removed or inserted line breaks every hash after it. A resolution is a second line with the same `id`, never a change
to the first.

## How to verify

```
git clone https://github.com/usmacro/scorecard
cd scorecard
sha256sum calls.jsonl
cat hashes/$(ls hashes | tail -1)
```

The two hashes agree on the day the hash file was written; `hashes/YYYY-MM-DD.txt` holds the ledger's sha256 that
night. The chain itself: for each line, sha256 of (previous `h` + newline + the line's canonical JSON without `h`,
keys sorted, no spaces) must equal its `h`.

## Independent archive

The Wayback Machine holds a capture of the ledger taken 2026-11-01 — https://web.archive.org/web/20261101235506/https://raw.githubusercontent.com/usmacro/scorecard/main/calls.jsonl — and of that night's hash file — https://web.archive.org/web/20261101235512/https://raw.githubusercontent.com/usmacro/scorecard/main/hashes/2026-11-01.txt. The site asks for both after each night's push; the capture is the Internet Archive's own copy, outside this repository and the site.

## Incident

This repository showed test data from October 29, 2026 05:43 UTC to 16:46 UTC, and again from October 30 05:43 UTC until the restoring commit below. On the site's side nothing moved. Its own ledger — the file the site writes and serves — held its 11 lines throughout. The commits below were made by the site's test suite, run by the nightly check on
the production box, which reached this repository's clone and its deploy key; they are not the site's calls:

- `ed6a640` (2026-10-29T05:43:02Z) — "ledger 2026-10-28: 6 calls, 0 resolutions": six fixture call lines from the site's test suite, not the site's calls
- `2016943` (2026-10-29T05:43:03Z) — "ledger 2027-01-05: 0 calls, 3 resolutions": three fixture resolutions dated a day that had not come
- `97251bd` (2026-10-30T05:43:11Z) — "ledger 2026-10-28: 6 calls, 0 resolutions": the same six fixture lines, replacing the eleven real lines pushed on October 29
- `94904d4` (2026-10-30T05:44:13Z) — "ledger 2027-01-05: 0 calls, 3 resolutions": the three fixture resolutions again
- `588154e` (2026-10-30T05:44:39Z) — "archive: Wayback capture 2026-10-30": a Wayback Machine capture of the fixture ledger, named in the README of that commit

This repository's `calls.jsonl` and `hashes/` were restored from the site's ledger by the commit "ledger restored 2026-10-30: 11 calls after test-data commits (see README)". The history
keeps the commits above as they were: nothing was rewritten and nothing was force-pushed. The test suite can no longer
reach a clone, a key or a remote.

## The grading rules

[usmacro.com/scorecard#how-graded](https://usmacro.com/scorecard#how-graded) — one entry per source, the sentence that
grades each, served from the site's decision record. Counts, never probabilities.
