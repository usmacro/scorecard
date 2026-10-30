# USMacro scorecard — the ledger

This repository is the public archive of [usmacro.com/scorecard](https://usmacro.com/scorecard): every forward call the
site has published, dated the day it first appeared, and how each one was graded when its window ended. The site writes
it every night; nothing here is typed by hand, and nothing is ever removed.

## The file

`calls.jsonl` — one JSON object per line, append-only, in the order the lines were written. The ledger holds 11 lines:
11 calls and 0 resolutions, the first call made on 2026-09-19.

**A call line** (`status` `pending`), written the night the site first showed the call:

```
{"basis":"capture","composite":"momentum","data_month":"2026-10","evidence":"https://usmacro.com/state-of-the-economy?date=2026-10-15","first_published":"2026-10-15","from":"stable","h":"4dd5ae4945d3c0cebb039355d6095bf8eb535518b60b2089b1f48f152caea771","id":"trajectory-momentum-2026-10","made_on":"2026-10-15","page":"/state-of-the-economy/methodology","reading":"stable, score 58.48","rule":"The outcome is the same rule applied to months m → m+3 on today's revised data: the score change, against the engine's own band. Held: the same state. Opposite: improving against deteriorating. Moved: any other change.","status":"pending","subject":"Momentum","to":"stable","type":"trajectory","window":"when the composite's month m+3 is on the revised record","window_end":"2027-01"}
```

- `id` — the call's key; a resolution carries the same `id`.
- `made_on` / `first_published` — the day the site first showed the call (never earlier; `basis` says `capture` when
  a stored capture of the live site shows it, `record` when it is read from the site's stored point-in-time record).
- `type`, `subject`, `from`, `to`, `reading`, `data_month` — what the call is.
- `rule` — the sentence that grades it, the engine's own, fixed on the line the day the call is made; `window` and
  `window_end` say when.
- `evidence` — the site's address with the day, where the call can be seen as it stood.
- `h` — the line's hash: sha256 over the previous line's `h` and this line without `h`.

**A resolution line** (`status` `resolved`) carries the same `id`, `resolved_on`, `outcome` (`held` or
`did_not_hold`), `resolving_reading` and `outcome_word` (the rule's own word). A call is graded once; a later revision
of the data never regrades it.

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

The Wayback Machine holds a capture of the ledger taken 2026-10-30 — https://web.archive.org/web/20261030155234/https://raw.githubusercontent.com/usmacro/scorecard/main/calls.jsonl — and of that night's hash file — https://web.archive.org/web/20261030235515/https://raw.githubusercontent.com/usmacro/scorecard/main/hashes/2026-10-30.txt. The site asks for both after each night's push; the capture is the Internet Archive's own copy, outside this repository and the site.

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
