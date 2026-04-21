# RollingGo UV Reference

> Use this file for uv / uvx / Python environments.
> Interaction style and branching logic live in `../SKILL.md`.

## Latest-first prefix

```bash
uvx --refresh --from rollinggo@latest rollinggo
```

## API Key Setup

Resolution order: `--api-key` flag → `RollingGo_API_KEY` env var. If the host drops it between runs, see [claw-host-env.md](claw-host-env.md).

## Three commands this skill uses most

### 1. Find candidate hotels: `search-hotels`

```bash
uvx --refresh --from rollinggo@latest rollinggo search-hotels \
  --origin-query "high-end hotels near West Lake worth watching for price drops" \
  --place "West Lake Hangzhou" \
  --place-type "<check --help for valid values>" \
  --check-in-date 2026-05-01 \
  --stay-nights 2 \
  --adult-count 2 \
  --size 5
```

### 2. Inspect a specific hotel: `hotel-detail`

```bash
uvx --refresh --from rollinggo@latest rollinggo hotel-detail \
  --hotel-id 123456 \
  --check-in-date 2026-05-01 \
  --check-out-date 2026-05-03 \
  --adult-count 2 \
  --room-count 1
```

### 3. Translate fuzzy preferences: `hotel-tags`

```bash
uvx --refresh --from rollinggo@latest rollinggo hotel-tags
```

## Typical order of operations for this skill

### Already booked

1. collect hotel name, one disambiguating locator, dates, occupancy, and room type
2. use `hotel-detail --name` or `search-hotels` to resolve candidates if `hotelId` is not already known
3. confirm the matched property by name plus city/area and any available address or brand signal; if still ambiguous, ask the user to choose
4. use `hotel-detail` to inspect current pricing and cancellation rules for the confirmed property
5. ask for the booking platform, exact plan, or booked price only if needed
6. if the agent has `Heartbeat`, `Cron`, or another scheduled-task capability, create the recurring watch directly
7. if no scheduled capability exists, produce a `Watch Task Summary`

### Not booked yet

1. use `hotel-tags` for fuzzy preferences
2. use `search-hotels` to get 3 to 5 candidates
3. once the user picks one, use `hotel-detail`
4. if the agent has `Heartbeat`, `Cron`, or another scheduled-task capability, create the recurring watch directly; otherwise turn it into a `Watch Task Summary`

## Key rules

- `uvx` here needs `--from rollinggo`
- use `rollinggo@latest` so uv resolves the latest published release
- `--place-type` must come from `search-hotels --help`
- `hotel-detail` does not support `--format table`
- do not rely on fuzzy `--name` matching alone; confirm the property before price comparison
- if there is no real price-history data, do not invent volatility percentages
- if the agent has `Heartbeat` or `Cron`, prefer using them for recurring price checks instead of stopping at a one-off summary

## Troubleshooting

- **`rollinggo` not found:** use `uvx --refresh --from rollinggo@latest rollinggo ...`
- **Missing API key:** set `RollingGo_API_KEY`, pass `--api-key`, or if you are inside any claw-style host, follow [claw-host-env.md](claw-host-env.md) and inject the same key through that host's config layer
- **No results:** relax stars, tags, distance, budget, then dates
