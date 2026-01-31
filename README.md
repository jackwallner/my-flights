# my-flights ✈️

[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-Live-blue?logo=github)](https://jackwallner.github.io/my-flights/)

> Personal flight tracker display showing aircraft that fly overhead. A server-fed companion to [overhead-flights](https://github.com/jackwallner/overhead-flights).

**Live Site:** https://jackwallner.github.io/my-flights/

![Screenshot](docs/screenshot.png)

## What is this?

A GitHub Pages site that displays flight data collected by a local tracker service running on your Mac. Unlike [overhead-flights](https://github.com/jackwallner/overhead-flights) which fetches data directly from the browser, this system uses a dedicated service to:

- Poll FlightRadar24 API every 10 seconds
- Track closest approach for each aircraft
- Display flights on an AWTRIX pixel clock
- Export data to this website automatically

## System Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  FlightRadar24  │────▶│  tracker.mjs    │────▶│  AWTRIX Clock   │
│     API         │     │  (Mac service)  │     │  (192.168.5.56) │
└─────────────────┘     └────────┬────────┘     └─────────────────┘
                                 │
                                 ▼
                    ┌─────────────────────┐
                    │  flights-web.json   │
                    │  index.html         │
                    └──────────┬──────────┘
                               │ sync (2 min)
                               ▼
                    ┌─────────────────────┐
                    │   my-flights/       │◀── You are here
                    │   (GitHub Pages)    │
                    └─────────────────────┘
```

## Quick Start

### 1. Set up the Tracker Service

```bash
# Clone the service (separate repo)
git clone https://github.com/jackwallner/flight-tracker-service.git ~/flight-tracker-service
cd ~/flight-tracker-service

# Install dependencies
npm install

# Configure your location
export TRACKER_LAT=45.625280431872
export TRACKER_LON=-122.52811167430798
export TRACKER_RADIUS_NM=1.5

# Start the service
./run.sh
```

### 2. Configure GitHub Pages Sync

Edit `sync-to-pages.sh` with your repo URL:

```bash
# Replace with your GitHub Pages repo
GITHUB_REPO="git@github.com:yourusername/my-flights.git"
```

### 3. Deploy to GitHub Pages

```bash
# The service auto-syncs every 2 minutes
# Or force a sync:
./sync-to-pages.sh
```

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `TRACKER_LAT` | (required) | Your latitude |
| `TRACKER_LON` | (required) | Your longitude |
| `TRACKER_RADIUS_NM` | 1.5 | Detection radius in nautical miles |
| `POLL_INTERVAL` | 10 | Seconds between API polls |
| `AWTRIX_IP` | 192.168.5.56 | Your AWTRIX clock IP |

### macOS LaunchAgent Setup

Create `~/Library/LaunchAgents/com.yourname.flight-tracker.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.yourname.flight-tracker</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/node</string>
        <string>/Users/YOURNAME/flight-tracker-service/tracker.mjs</string>
    </array>
    <key>EnvironmentVariables</key>
    <dict>
        <key>TRACKER_LAT</key>
        <string>45.625280431872</string>
        <key>TRACKER_LON</key>
        <string>-122.52811167430798</string>
        <key>TRACKER_RADIUS_NM</key>
        <string>1.5</string>
    </dict>
    <key>WorkingDirectory</key>
    <string>/Users/YOURNAME/flight-tracker-service</string>
    <key>StandardOutPath</key>
    <string>/Users/YOURNAME/flight-tracker-service/tracker.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/YOURNAME/flight-tracker-service/tracker.error.log</string>
    <key>KeepAlive</key>
    <true/>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

Load it:
```bash
launchctl load ~/Library/LaunchAgents/com.yourname.flight-tracker.plist
```

## Data Format

The site displays flights from `flights-web.json`:

```json
{
  "flights": [
    {
      "callsign": "ASA123",
      "airline": "Alaska Airlines",
      "aircraftType": "Boeing 737-800",
      "registration": "N512AS",
      "route": "SEA → LAX",
      "closestDistance": 0.8,
      "closestAltitude": 8500,
      "closestSpeed": 245,
      "firstSeen": "2026-01-30T14:23:00.000Z",
      "lastSeen": "2026-01-30T14:28:00.000Z",
      "flightType": "commercial"
    }
  ],
  "stats": {
    "totalFlights": 42,
    "closestFlight": 0.3,
    "todayCount": 12
  },
  "lastUpdated": "2026-01-30T14:30:00.000Z"
}
```

## Comparison: my-flights vs overhead-flights

| Feature | my-flights | overhead-flights |
|---------|------------|------------------|
| Data source | Local service + FR24 | OpenSky API (browser) |
| Server needed | Yes (Mac service) | No |
| AWTRIX display | ✅ Yes | ❌ No |
| Flight history | Persistent JSON | Session only |
| Closest approach | Pre-calculated | Real-time tracking |
| Multiple locations | No | Yes |
| Setup complexity | Higher | Lower |

## Related Projects

- **[overhead-flights](https://github.com/jackwallner/overhead-flights)** - Standalone client-side tracker (no server needed)
- **[flight-tracker-service](https://github.com/jackwallner/flight-tracker-service)** - The Mac service that feeds this site

## License

MIT
