# Flight Tracker Comparison

Choosing the right flight tracker for your needs.

---

## Quick Decision Tree

```
Do you want AWTRIX display?
├── YES → Use my-flights + flight-tracker-service
│
└── NO → Do you want persistent flight history?
    ├── YES → Use my-flights + flight-tracker-service
    │
    └── NO → Use overhead-flights (simplest)
```

---

## Feature Comparison

| Feature | my-flights | overhead-flights |
|---------|------------|------------------|
| **Server required** | ✅ Yes (Mac service) | ❌ No |
| **AWTRIX display** | ✅ Yes | ❌ No |
| **Flight history** | ✅ Persistent | ❌ Session only |
| **Closest approach** | ✅ Pre-calculated | ✅ Real-time |
| **Setup complexity** | Medium | Low |
| **Hosting** | GitHub Pages | GitHub Pages |
| **Data source** | FlightRadar24 API | OpenSky API |
| **Polling rate** | Every 10 seconds | Every 5 seconds |
| **Aircraft database** | 520k records | Basic |
| **Multiple locations** | ❌ No | ✅ Yes |
| **Mobile friendly** | ✅ Yes | ✅ Yes |

---

## my-flights + flight-tracker-service

**Best for:** Enthusiasts who want the full experience

**Pros:**
- Persistent flight history across sessions
- AWTRIX clock integration
- Rich aircraft data (type, registration, route)
- Closest approach automatically calculated
- "Set and forget" service runs 24/7

**Cons:**
- Requires a Mac running the service
- More complex setup
- Single location only

**Setup time:** ~30 minutes

---

## overhead-flights

**Best for:** Casual users, quick setup, multiple locations

**Pros:**
- Zero server requirements
- Works entirely in browser
- Can check any location instantly
- Simple deployment
- Lower API usage (direct browser calls)

**Cons:**
- No persistence between sessions
- No AWTRIX integration
- Less aircraft detail
- Requires browser to be open

**Setup time:** ~5 minutes

---

## Architecture Differences

### my-flights (Server-Fed)

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  FlightRadar24  │────▶│  Mac Service    │────▶│   GitHub Pages  │
│     API         │     │  (Persistent)   │     │   (Display)     │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                │
                                ▼
                        ┌─────────────────┐
                        │  AWTRIX Clock   │
                        └─────────────────┘
```

### overhead-flights (Client-Side)

```
┌─────────────────┐     ┌─────────────────┐
│   OpenSky API   │◀────│  Browser (JS)   │
│                 │     │  (GitHub Pages) │
└─────────────────┘     └─────────────────┘
```

---

## When to Use Which

### Use **my-flights** if:
- You have a Mac that can run 24/7
- You want AWTRIX clock notifications
- You care about flight history
- You want rich aircraft information
- You track a single location

### Use **overhead-flights** if:
- You want zero maintenance
- You check multiple locations
- You don't need persistence
- You don't have a Mac server
- You want something running in 5 minutes

---

## Can I Use Both?

Yes! They serve different purposes:

- **my-flights** at home → Your personal flight log with AWTRIX alerts
- **overhead-flights** on phone → Check flights anywhere, anytime

Both can be deployed to GitHub Pages simultaneously.
