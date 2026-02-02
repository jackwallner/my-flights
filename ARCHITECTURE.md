# my-flights Architecture

> **Live Site:** https://jackwallner.github.io/my-flights/  
> **Service:** https://github.com/jackwallner/flight-tracker-service  
> **This Repo:** Display layer for flight-tracker-service

## What This Is

GitHub Pages site that displays flight data from the Mac tracker service. The tracker runs as a LaunchAgent on your Mac, polls FlightRadar24, and exports data here.

**Don't confuse with:** [overhead-flights](https://github.com/jackwallner/overhead-flights) (standalone client-side tracker)

## System Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  FlightRadar24  │────▶│  tracker.mjs    │────▶│  AWTRIX Clock   │
│     API         │     │  (Mac service)  │     │  (192.168.5.56) │
└─────────────────┘     └────────┬────────┘     └─────────────────┘
                                 │
                    ┌────────────┼────────────┐
                    ▼            ▼            ▼
            ┌──────────┐  ┌──────────┐  ┌──────────┐
            │ flights  │  │  flights │  │flights-  │
            │ .json    │  │  -web    │  │ web.html │
            │(history) │  │  .json   │  │(website) │
            └──────────┘  └────┬─────┘  └────┬─────┘
                               │             │
                               │  sync-to-   │
                               │  pages.sh   │
                               ▼             ▼
                    ┌─────────────────────┐
                    │   my-flights/       │◀── You are here
                    │   (GitHub Pages)    │
                    └─────────────────────┘
```

## File Structure

### Running Service (edit in flight-tracker-service repo)
```
~/flight-tracker-service/           ← The actual running service
├── tracker.mjs                      ← Main tracker (Node.js)
├── api-fr24.mjs                     ← FlightRadar24 API client
├── aircraft-db.mjs                  ← Aircraft database lookup
├── aircraft_db.json                 ← 520k aircraft database
├── channel.mjs                      ← AWTRIX clock integration
├── flights-web.html                 ← Website template
├── sync-to-pages.sh                 ← Syncs to this repo
├── run.sh                           ← Launch script
├── flights-web.json                 ← Current flight data
├── flights.json                     ← Flight history log
└── tracker.log                      ← Service logs
```

### Website (this repo)
```
my-flights/                          ← This repo
├── index.html                       ← Copied from flights-web.html
├── flights-web.json                 ← Copied from service
├── flights.json                     ← Flight history
├── README.md                        ← User docs
└── ARCHITECTURE.md                  ← This file
```

## How It Works

1. **Service polls** FlightRadar24 every 10s for flights within 1.5 NM radius
2. **Database lookup** enriches with aircraft type, airline, registration (520k records)
3. **Session tracking** creates new entry for each overhead pass (5-min buffer)
4. **AWTRIX displays** 3-screen sequence for close flights (≤1.5 NM)
5. **Data exports** to `flights-web.json` continuously (throttled 5s)
6. **Sync script** copies files every 2 min and pushes to GitHub Pages

## Configuration

All configuration happens in the **flight-tracker-service** repo:

```bash
cd ~/flight-tracker-service

# Edit configuration
cp .env.example .env
# Edit .env with your coordinates

# Or set environment variables
export TRACKER_LAT=45.625280431872
export TRACKER_LON=-122.52811167430798
export TRACKER_RADIUS_NM=1.5
```

## Session-Based Tracking

Each overhead pass creates a **separate entry** in the flight history:

- **Session Key**: `callsign_YYYY-MM-DDTHH` (hour-level grouping)
- **Buffer Window**: 5 minutes
- Same flight passing 10 minutes later = separate table row
- Private planes doing circuits = each pass tracked separately

This prevents old flight data from being overwritten when the same flight passes by on different days.

## Data Format

### `flights-web.json` (Current Flight)
```json
{
  "closestApproach": {
    "distance": 0.77,
    "altitude": 36000,
    "speed": 406,
    "timestamp": "2026-01-31T07:16:48.584Z",
    "lat": 45.6281,
    "lon": -122.5101,
    "precision": "tracked"
  },
  "flight": {
    "callsign": "ASA691",
    "aircraftType": "737",
    "origin": "PSP",
    "destination": "SEA",
    "firstSeen": "2026-01-31T07:16:48.581Z"
  },
  "overheadScore": 0.77,
  "isOverhead": true,
  "status": "tracking",
  "timestamp": "2026-01-31T07:16:48.584Z"
}
```

### `flights.json` (History)
```json
[
  {
    "callsign": "ASA691",
    "flightNumber": "AS691",
    "aircraftType": "B739",
    "origin": "PSP",
    "destination": "SEA",
    "firstSeen": "2026-01-31T07:16:48.581Z",
    "lastSeen": "2026-01-31T07:16:48.581Z",
    "closestDistance": 0.77,
    "closestAltitude": 36000,
    "closestSpeed": 406,
    "_sessionKey": "ASA691_2026-01-31T07"
  }
]
```

## Development

### To edit the tracker:
```bash
cd ~/flight-tracker-service
# Edit any .mjs file
./run.sh
```

### To edit the website:
```bash
cd ~/flight-tracker-service
# Edit flights-web.html
git add flights-web.html
git commit -m "Update website template"
```

The template is automatically copied here by `sync-to-pages.sh`.

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Site shows old data | Check sync script is running, or run `./sync-to-pages.sh` manually |
| No new flights | Check tracker log: `tail ~/flight-tracker-service/tracker.log` |
| AWTRIX not showing | Check channel.mjs, restart service |

## Related

- **flight-tracker-service:** https://github.com/jackwallner/flight-tracker-service (the service that feeds this site)
- **overhead-flights:** https://github.com/jackwallner/overhead-flights (standalone client tracker)
