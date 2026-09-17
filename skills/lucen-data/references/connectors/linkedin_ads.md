# LinkedIn Ads

## Tables and grain

| Table | Grain | Use for |
|---|---|---|
| `gold_linkedin_ads_performance` | campaign (or served pivot) / day | The pack's LinkedIn delivery fact |

`list_tables` column descriptions name the exact pivot (campaign, creative,
etc.) this org's dataset uses.

## Additive / non-additive

Spend, impressions, clicks are additive within the served grain. If the
table exposes `reach`, treat it as non-additive across days and breakdowns.

## Ratios

The server lints `SUM`/`AVG` of stored ratios; the finding is a caveat,
not a block.

Recompute CTR/CPC/CPM from summed components. Do not AVG stored ratios.

## Do not join / do not sum

There is one LinkedIn gold fact. Do not invent a second grain by joining
it to Meta or Google tables unless the user asked for a combined view and
you can align dates and currency honestly.

## Attribution

LinkedIn's served click attribution is whatever the dataset config pinned.
Do not assume Meta's 7d_click window.

## Typical questions → table

| Question | Table |
|---|---|
| Campaign spend / CTR | `gold_linkedin_ads_performance` |
