# RollingGo NPX Reference

> Use this file for npm / npx / Node environments.
> Interaction style and branching logic live in `../SKILL.md`.

## Latest-first prefix

```bash
npx --yes --package rollinggo@latest rollinggo
```

## API Key Setup

Resolution order: `--api-key` flag → `RollingGo_API_KEY` env var.

If the host is any claw shell, desktop wrapper, or compatible runtime that manages agent env separately, read [claw-host-env.md](claw-host-env.md) first. In OpenClaw-family hosts, prefer `skills.entries.rollinggo-hotel-price-watch`; in other hosts, map the same `RollingGo_API_KEY` into that host's per-skill env or secret store.

## Three commands this skill uses most

### 1. Find candidate hotels: `search-hotels`

Required:
- `--origin-query`
- `--place`
- `--place-type`

```bash
npx --yes --package rollinggo@latest rollinggo search-hotels \
  --origin-query "high-end hotels near West Lake worth watching for price drops" \
  --place "West Lake Hangzhou" \
  --place-type "<check --help for valid values>" \
  --check-in-date 2026-05-01 \
  --stay-nights 2 \
  --adult-count 2 \
  --size 5
```

### 2. Inspect a specific hotel: `hotel-detail`

Prefer `--hotel-id` and include dates plus occupancy so the room-plan output is meaningful.

```bash
npx --yes --package rollinggo@latest rollinggo hotel-detail \
  --hotel-id 123456 \
  --check-in-date 2026-05-01 \
  --check-out-date 2026-05-03 \
  --adult-count 2 \
  --room-count 1
```

### 3. Translate fuzzy preferences: `hotel-tags`

Use this when the user says things like "design-forward", "family-friendly", "great breakfast", or "brand-heavy".

```bash
npx --yes --package rollinggo@latest rollinggo hotel-tags
```

## Typical order of operations for this skill

### Already booked

1. Collect hotel name, one disambiguating locator, dates, occupancy, and room type
2. Use `hotel-detail --name` or `search-hotels` to resolve candidates if `hotelId` is not already known
3. Confirm the matched property by name plus city/area and any available address or brand signal; if still ambiguous, ask the user to choose
4. Use `hotel-detail` with the confirmed hotel to get current pricing and cancellation terms on available rooms
5. Ask for the user's booked price and their booking's cancellation deadline if not provided
6. If the agent has `Heartbeat`, `Cron`, or another scheduled-task capability, create the recurring watch directly
7. If no scheduled capability exists, produce a `Watch Task Summary`

### Not booked yet

1. Use `hotel-tags` for fuzzy preferences
2. Use `search-hotels` to get 3 to 5 candidates
3. Once the user picks one, use `hotel-detail` to get current pricing and cancellation terms
4. If the user wants to keep watching, follow the watch path: if the agent has `Heartbeat`, `Cron`, or another scheduled-task capability, create the recurring watch directly; otherwise turn it into a `Watch Task Summary`
5. If the user wants to book now, provide the booking URL or hotel detail page link from the result and summarize the recommended room, current price, and cancellation terms

## Key rules

- `--place-type` must come from `search-hotels --help`
- `hotel-detail` does not support `--format table`
- `--check-out-date` must be later than `--check-in-date`
- prefer `hotelId` whenever you have it
- do not rely on fuzzy `--name` matching alone; confirm the property before price comparison
- if there is no real price-history data, do not invent drop percentages
- if the agent has `Heartbeat` or `Cron`, prefer using them for recurring price checks instead of stopping at a one-off summary

## Troubleshooting

- **Missing API key:** set `RollingGo_API_KEY`, pass `--api-key`, or if you are inside any claw-style host, follow [claw-host-env.md](claw-host-env.md) and inject the same key through that host's config layer
- **No results:** loosen stars, tags, distance, budget, then dates
- **No room plans in detail output:** this is a business result, not necessarily an error; try different dates, occupancy, or hotel
