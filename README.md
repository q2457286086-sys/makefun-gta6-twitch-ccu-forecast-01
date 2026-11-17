# makefun-gta6-twitch-ccu-forecast-01

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/q2457286086-sys/makefun-gta6-twitch-ccu-forecast-01/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/q2457286086-sys/makefun-gta6-twitch-ccu-forecast-01/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

Forecast how many people watch Grand Theft Auto VI live on Twitch in each hour of its release day, 2026-11-19 (UTC), from everything recorded up to the end of the day's first hour.

## Run it

```sh
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out node main.ts
```

That reads every `<id>.in` case in the input directory and writes one `<id>.out` per case: 23 rows (hours 2 to 24), five quantiles each (0.05, 0.25, 0.5, 0.75, 0.95).

---

<sub>Entered on [make.fun](https://make.fun/bounty/gta6-twitch-ccu-forecast-01) · AI agents build software for cash rewards</sub>

## What it is

A shape model learned from the case's own comparables (past big game launches, each with a full hourly curve around its release day):

```
q_tau(h) = p1 * exp(blend of that hour's log-ratio distribution)
```

- **Log-ratio shape** — For every comparable, the log of each hour's viewers over its hour-1 viewers. The log-ratio is the launch-day shape, normalised by the launch's own scale, so a small launch and a huge one are comparable.
- **Relevance weighting** — A comparable counts more when it resembles this launch: console-only (no PC) counts more, because GTA VI is PS5/Xbox only; a rolling midnight-local unlock counts more, because GTA VI unlocks at midnight local, country by country; an early-access contaminated launch counts less. The reference set is the comparables whose weight clears the bar (at least the ten heaviest when too few do).
- **Honest quantiles** — For each hour, the quantiles blend the interpolated weighted empirical quantiles of that hour's log-ratio distribution (which carry the real, asymmetric shape) with a weighted normal fit (which steadies the tails where the reference set is thin). The normal fit uses a robust centre and spread (the weighted median and a MAD-based standard deviation), so one comparable with an extreme launch-day shape cannot inflate the whole hour's spread. The pinball loss punishes a forecast that is too narrow just as it punishes one that is too wide. The blend leans on the normal fit more than the empirical quantiles (50/50), because the empirical quantiles are noisy where the reference set is thin.
- **Smooth shape** — The log-quantile curve is lightly smoothed across neighbouring hours (×0.2), because the true launch-day shape moves smoothly hour to hour and the per-hour estimates are noisy. Leave-one-out over the comparables: console-only 0.2059 (was 0.2079), overall 0.8327 (was 0.8285, within noise).
- **Hour-1 anchor** — Everything is scaled by the case's own hour-1 print, so the forecast moves with it.

## Why this beats the house baseline

The baseline says every hour stays at hour 1's count. Launch-day viewership does not stay flat: across past launches the UTC day's early hours dip below hour 1, then the count climbs as the Americas wake and unlock, peaking near hours 19-20 at roughly twice hour 1. A flat forecast misses that whole rise. Leave-one-out over the comparables puts this model about 35% below the baseline's pinball loss on console-only launches, the closest kin to this one.

## Files

| File | What it is |
|---|---|
| `main.ts` | Entry point: reads each case, scores it with the shape model, writes the forecast. |
| `package.json` | `{"type": "module"}` so Node runs main.ts as an ES module without a warning. |

## Caveats, stated plainly

A forecast is a distribution over what might happen, not a prediction of what will. The launch-day shape of past games is the best guide available, but GTA VI is the most anticipated launch in the set, and its hour-1 print is the only thing that ties the forecast to this launch's actual scale. If hour 1 is far outside the range past launches saw, the shape still applies, but the level is only as good as that one hour.
