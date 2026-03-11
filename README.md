# Relay Spy

A web tool for inspecting and debugging Nostr relay events in real time.

Try it out: https://relayspy.5t34k.com/

## What it does

Relay Spy connects to any Nostr relay via WebSocket and displays every event as it arrives. Events are shown in **detection order** (the order your client sees them), not by their `created_at` timestamps. This matters when you're debugging relay behavior, event propagation, or verifying that your app is publishing events correctly.

Each event is displayed with:

- **Kind badge** with human-readable name (60+ NIP kind types mapped)
- **npub** and hex pubkey
- **Inline tag preview** showing tag name:value pairs at a glance
- **Event ID verification** - recomputes the SHA-256 hash and flags mismatches
- **Event size** in bytes
- **Color-coded left border** by kind category (ephemeral, replaceable, parameterized replaceable, etc.)

Click any event to expand its full details: all fields, tags table, content (with auto-detected JSON pretty-printing), note ID (bech32), latency, signature, and raw JSON with syntax highlighting.

## Getting started

1. Download `index.html`
2. Open it in a browser
3. Enter a relay URL and hit Connect (or press Enter)

The default relay is `ws://localhost:3334`. For public relays use `wss://` (e.g. `wss://relay.damus.io`).

## Features

### Filters

Filter events by any combination of:
- **Kind** (comma-separated, e.g. `1,4,30023`)
- **Pubkey** (hex or npub prefix)
- **Event ID** (hex prefix)
- **Content** (full-text search)
- **Tag name** (e.g. `p`, `e`, `t`)
- **Tag value** (search within tag values)
- **ID validity** (valid/invalid)

#### Multi-row filters with AND/OR logic

Click **+ Add Filter Row** to create compound filters. Each row's fields are AND'd together. Between rows, choose **AND** or **OR**:

```
Row 1: kind=1, tag name=t, tag value=nostr
                    OR
Row 2: kind=4, pubkey=abc123...
```

This matches events that are either (kind 1 with a `t` tag containing "nostr") OR (kind 4 from that pubkey).

AND groups are evaluated first, then OR'd together - so `A AND B OR C AND D` means `(A AND B) OR (C AND D)`.

### Custom Labels

Create named labels with filter criteria to visually tag matching events. Labels appear as colored badges on each event row.

- Define labels using any combination of kind, pubkey, content, event ID, ID validity, and multiple tag conditions
- Tag conditions support the same AND/OR grouping logic as filters
- Labels are **saved to localStorage** and persist across sessions
- Filter the event list to show only events matching a specific label
- Each label gets an auto-assigned color from a 16-color palette

**Example:** Create a label called "My App" with tag name `d` and tag value `myapp-` to instantly spot your app's events in a busy relay.

### Export

Export captured events at any time in three formats:

- **JSON** - full structured data including matched labels, npub/note IDs, kind names, latency, verification status, and all label definitions
- **CSV** - one row per event, opens in spreadsheets
- **Raw JSONL** - one raw Nostr event per line, useful for replaying to another relay

Export **all events** or just the **currently filtered** set.

### Other

- **Pause/Resume** - freeze the live feed to inspect events; new events are buffered and flushed on resume
- **Event counter and rate** - running total and events/second
- **Relative timestamps** that update live ("3s ago", "2m ago")
- **Copy raw JSON** per event with one click
- **bech32 display** - npub for pubkeys, note1 for event IDs

## Sample use cases

### Verifying your app's events

Connect to the relay your app publishes to. Create a custom label matching your app's pubkey or a specific tag your app uses. Every event your app creates will be highlighted, and you can expand each one to verify the kind, tags, content, and structure are exactly what you expect.

### Debugging tag structure

The inline tag preview on each event row lets you quickly scan for malformed or missing tags without expanding every event. Filter by tag name to isolate specific tag types (e.g. all events with a `p` tag).

### Monitoring a relay

Point Relay Spy at your own relay to watch all traffic in real time. Use the rate counter to gauge activity. Filter for specific kinds to focus on what matters.

### Catching invalid events

Filter by **ID Valid = Invalid** to find events where the computed hash doesn't match the claimed event ID. This catches serialization bugs, relay tampering, or malformed events.

### Exporting for analysis

Capture a session of events, then export as CSV to analyze in a spreadsheet - sort by kind, group by pubkey, look for patterns in tag usage, or chart event rates over time.

## Technical notes

- Single HTML file, zero dependencies, runs entirely in the browser
- Uses the browser's native WebSocket and Web Crypto (SubtleCrypto) APIs
- Subscribes to relays with an open filter `{}` to receive all events
- bech32 encoding (npub/note) implemented in pure JS
- Event ID verification uses the NIP-01 serialization: `sha256(json([0, pubkey, created_at, kind, tags, content]))`

## License

MIT

---

Vibed by [5t34k](https://nostrgate.com/#npub1x5t34kxd79m657qcuwp4zrypy9t8t4e6yks5zapjvau29t0xvgaqakh2p2)
