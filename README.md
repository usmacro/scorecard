# USMacro scorecard — the ledger

This repository is the public archive of [usmacro.com/scorecard](https://usmacro.com/scorecard): every forward call the
site has published, dated the day it first appeared, and how each one was graded when its window ended. The site writes
it every night; nothing here is typed by hand, and nothing is ever removed.

## The file

`calls.jsonl` — one JSON object per line, append-only, in the order the lines were written. The ledger holds 9 lines:
6 calls and 3 resolutions, the first call made on 2026-10-03.

**A call line** (`status` `pending`), written the night the site first showed the call:

```
{"basis":"capture","composite":"momentum","data_month":"2026-10","evidence":"https://usmacro.com/state-of-the-economy?date=2026-10-15","first_published":"2026-10-15","from":"stable","h":"249780a55bbe0674899084a21f3544db4c1ef99bf18aa7c933bba5082a2fa53f","id":"trajectory-momentum-2026-10","made_on":"2026-10-15","page":"/state-of-the-economy/methodology","reading":"improving, score 66","rule":"The outcome is the same rule applied to months m → m+3 on today's revised data: the score change, against the engine's own band. Held: the same state. Opposite: improving against deteriorating. Moved: any other change.","status":"pending","subject":"Momentum","to":"improving","type":"trajectory","window":"when the composite's month m+3 is on the revised record","window_end":"2027-01"}
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

Not yet archived independently: the first Wayback Machine capture follows the first push.

## The grading rules

[usmacro.com/scorecard#how-graded](https://usmacro.com/scorecard#how-graded) — one entry per source, the sentence that
grades each, served from the site's decision record. Counts, never probabilities.
