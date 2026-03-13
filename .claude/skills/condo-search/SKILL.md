---
name: condo-search
description: Search HouseSigma for condos and condo townhouses in Brampton and Mississauga. Filters for 2+ bedrooms, sale price under $600,000, sigma estimated price under $600,000, and within 5km of a GO train station (hard filter). Ranks by bathrooms, then maintenance cost, then sigma estimated price. Reports top 20. Persists results to disk and tracks new vs. previously found listings.
allowed-tools: Bash(agent-browser:*)
---

# Condo Search — Brampton & Mississauga

Maintain a persistent ranked list of qualifying condos at `/workspace/group/recommended_listings.json`. On each run: prune sold/delisted listings, add newly found ones, re-rank, then report the full list (max 20) marking what's new.

## When to use

- User says "find condos", "search for apartments", "condo search", "look for condos in Brampton/Mississauga", "update condo list", "any new condos"

## GO Train Stations (Mississauga & Brampton)

Use these coordinates to calculate proximity (straight-line distance in km). Report the nearest one per listing.

| City | Station | Line | Latitude | Longitude |
| :--- | :--- | :--- | :--- | :--- |
| Brampton | Mount Pleasant GO | Kitchener | 43.6750 | -79.8222 |
| Brampton | Brampton Innovation District GO | Kitchener | 43.6868 | -79.7623 |
| Brampton | Bramalea GO | Kitchener | 43.7011 | -79.6917 |
| Mississauga | Malton GO | Kitchener | 43.7051 | -79.6384 |
| Mississauga | Lisgar GO | Milton | 43.5909 | -79.7876 |
| Mississauga | Meadowvale GO | Milton | 43.5979 | -79.7548 |
| Mississauga | Streetsville GO | Milton | 43.5762 | -79.7088 |
| Mississauga | Erindale GO | Milton | 43.5714 | -79.6644 |
| Mississauga | Cooksville GO | Milton | 43.5838 | -79.6235 |
| Mississauga | Dixie GO | Milton | 43.6074 | -79.5772 |
| Mississauga | Clarkson GO | Lakeshore West | 43.5019 | -79.6247 |
| Mississauga | Port Credit GO | Lakeshore West | 43.5559 | -79.5866 |

## recommended_listings.json Format

```json
[
  {
    "id": "housesigma-listing-id-or-url-hash",
    "url": "https://housesigma.com/...",
    "address": "123 Main St, Unit 405, Mississauga",
    "type": "Condo Apt",
    "bedrooms": 2,
    "bathrooms": 2,
    "list_price": 549900,
    "sigma_estimate": 537000,
    "maintenance_fee": 420,
    "nearest_go_station": "Cooksville GO",
    "nearest_go_distance_km": 2.1,
    "added_at": "2026-03-10",
    "last_seen": "2026-03-10",
    "is_new": true
  }
]
```

## Procedure

### Step 1 — Load existing listings

Read `/workspace/group/recommended_listings.json`. If it doesn't exist, start with an empty array. Collect all existing listing URLs.

### Step 2 — Verify existing listings (prune sold/delisted)

For each existing listing, open its URL:
```bash
agent-browser open "<listing_url>"
agent-browser wait --load networkidle
agent-browser snapshot -c
```

Check if the listing is still active (for sale). Signs it's sold/delisted: status shows "Sold", "Conditional", "Terminated", or the page returns a 404/redirect. Remove any sold or delisted listings from the array. Update `last_seen` for still-active ones to today's date.

### Step 3 — Search for new listings

**Mississauga:**
```bash
agent-browser open "https://housesigma.com/on/mississauga/?type=condo-apt,condo-townhouse&bed_min=2&price_max=600000&status=sale"
agent-browser wait --load networkidle
agent-browser snapshot -i
```

**Brampton:**
```bash
agent-browser open "https://housesigma.com/on/brampton/?type=condo-apt,condo-townhouse&bed_min=2&price_max=600000&status=sale"
agent-browser wait --load networkidle
agent-browser snapshot -i
```

If URL filters don't apply correctly, use the UI filters manually. Scroll and load more results — aim for at least 20–30 listings per city.

For each listing card, extract the URL. Skip any URL already in the existing listings array.

### Step 4 — Extract new listing details

For each new listing URL (not already in recommended_listings.json), open the page and extract:
- **Address**
- **List price**
- **Sigma estimated price** (look for "Sigma Est.", "HouseSigma Estimate", or similar)
- **Bedrooms**
- **Bathrooms**
- **Maintenance fee** (monthly)
- **Property type** (Condo Apt or Condo Townhouse)
- **Listing URL**

### Step 5 — Filter new listings

Keep only new listings where:
1. Property type is Condo Apt or Condo Townhouse
2. Bedrooms ≥ 2
3. List price ≤ $600,000
4. Sigma estimated price ≤ $600,000 (exclude if unavailable or over limit)

### Step 6 — Calculate GO station proximity and apply distance filter

For each new qualifying listing, calculate straight-line distance (km) to every GO station using the Haversine formula (or approximate: 1 degree lat ≈ 111 km, 1 degree lng ≈ 79 km at this latitude). Record the nearest station and distance.

**Hard filter: exclude any listing where the nearest GO station is more than 5km away.** There is no soft fallback — if a listing doesn't meet the 5km threshold it is dropped entirely. Set `is_new: true` and `added_at` to today's date on listings that pass.

### Step 7 — Merge and re-rank

Merge new qualifying listings into the array (after setting `is_new: false` on all previously existing listings that are still active, and `is_new: true` on newly added ones).

Sort the full array by:
1. **Bathrooms** — descending (more = better)
2. **Maintenance fee** — ascending (lower = better); treat missing/null as 999
3. **Sigma estimated price** — ascending (lower = better)

Keep all qualifying listings (no truncation in storage). Write back to `/workspace/group/recommended_listings.json`.

### Step 8 — Report

Return a formatted ranked list of the top 20 listings from recommended_listings.json, sorted by the ranking above. Mark new listings with 🆕.

---

**Condo Search Results — Brampton & Mississauga**
*Filters: Condo Apt / Condo Townhouse · 2+ bed · List price ≤ $600K · Sigma est. ≤ $600K*
*X listings total · Y new this run · Z removed (sold/delisted)*

**#1 🆕 — 123 Main St, Unit 405, Mississauga**
- Type: Condo Apt
- Bedrooms: 2 · Bathrooms: 2
- List price: $549,900
- Sigma estimate: $537,000
- Maintenance: $420/mo
- Nearest GO: Cooksville GO (2.1 km)
- [View on HouseSigma](https://housesigma.com/...)

**#2 — 456 Queen St W, Unit 12, Brampton**
- Type: Condo Townhouse
- Bedrooms: 3 · Bathrooms: 2
- List price: $589,000
- Sigma estimate: $571,000
- Maintenance: $385/mo
- Nearest GO: Brampton GO (3.8 km)
- [View on HouseSigma](https://housesigma.com/...)

---

If no listings match all criteria after filtering, report how many were found before each filter step so the user understands which filter is most restrictive.
