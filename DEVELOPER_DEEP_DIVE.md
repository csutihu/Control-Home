# ControlHome – Developer Deep Dive

## 🧠 Core Principles

ControlHome is built around a few core principles that should guide all future development:

- Single source of truth
- WebSocket-first runtime behavior where supported
- Cache-first startup
- Stateless UI rendering
- UI customization layered on top of unmodified device truth
- Domoticz order as baseline order everywhere
- HTTP-only domains must stay explicit and not be disguised as WS-backed

These are not incidental implementation choices. They are the basis of the current app behavior.

---

# 🔄 Full Data Flow

## Startup flow

```text
cached snapshot / cached auxiliary collections
   ↓
immediate UI render
   ↓
HTTP bootstrap refresh
   ↓
WebSocket sync activation for supported domains
```

This startup path is one of the strongest parts of the application UX.

## Runtime flow

```text
WS event / HTTP-only refresh / command result / fallback refresh
                     ↓
       repository + shared snapshot update
                     ↓
          ViewModel StateFlow emission
                     ↓
             Compose recomposition
```

History flow is intentionally separate:

```text
long press on tile
        ↓
HistoryRepository HTTP request
        ↓
DeviceHistory.Chart / DeviceHistory.Log / DeviceHistory.Unsupported
        ↓
DeviceHistoryDialog
```

Important observation:
- the UI should not invent device truth
- it should render what the shared model currently says, possibly with temporary optimistic state

---

# 🌐 WebSocket Model

## Design choice
The app uses a **single global WebSocket model** rather than separate screen-level sockets.

This matters because:
- Favorites, Rooms, and Devices need consistent state
- linked visual features depend on shared state
- users expect changes made in one place to appear everywhere immediately

## ActiveGroup
The WebSocket manager uses an active-group model to determine what is relevant at a given time.

Examples:
- Favorites
- Room(planId)
- Devices(category)
- None

## Lifecycle
Foreground:
- connection active
- snapshot + sync as needed

Background:
- WebSocket suspended/stopped

This is intentional. QS and cache behavior cover the background gap.

## Important boundary
Scenes / Groups and User Variables do **not** participate in this WS model.  
Do not try to force them into ActiveGroup semantics. They are HTTP-only by design.

---

# 🗂 Shared Device Model

The effective shared state is built from bootstrap data plus explicit HTTP-only side collections.

Conceptually it contains:
- all known standard devices
- per-idx access
- per-plan grouping
- per-category membership
- favorites ordering reference
- scenes / groups collection
- user variables collection

This shared model is consumed by:
- Favorites
- Rooms
- Devices
- Linked Tile Colors
- QS cache seeding

## Why this is important
This allows:
- cross-screen consistency
- no duplicate fetch logic
- easier future features such as reorder, device customization, advanced overlays

---

# 📦 Bootstrap Data

Bootstrap data is the real heart of the app’s data layer.

It now merges:
- plans
- category-filtered device lists
- favorites information
- scenes
- user variables

The important design outcome is that the app gets one coherent model instead of several disconnected lists.

## Key fields to preserve
- `plans`
- `devicesByIdx`
- `devicesOrdered`
- `devicesByPlan`
- `categoryIndex`
- `favoriteOrder`
- `scenes`
- `userVariables`

Any refactor touching bootstrap generation should be treated carefully.

---

# 🔄 Ordering Deep Dive

## Current rule
Domoticz order is the default order for:
- Favorites
- Rooms
- Devices
- Scenes
- User Variables

## Why this matters
The app already supports local customization. That only works cleanly if:

```text
default order = Domoticz order
custom order = local override
reset order = Domoticz order again
```

If any layer alphabetically reorders devices, this model breaks.

## Places where ordering must be protected
- repository merge
- `devicesOrdered`
- `devicesByPlan`
- scenes / variables list preservation
- ViewModel mapping functions
- screen page builders

---

# 🏠 Rooms Implementation Details

Rooms are based on Domoticz plans.

## Important behaviors
- rooms with `_` in the name are excluded
- room devices preserve Domoticz order
- room visuals are app-local only
- per-room reorder is supported and reset returns to Domoticz order

## Header feature set
- image
- color
- reset default
- adaptive text/icon contrast
- persistent image access after restart

## Important implementation update
Header presentation is explicitly normalized:
- `Set image` clears color
- `Set color` clears image
- `Set default` clears both

This prevents half-hidden state combinations.

---

# 🧩 Devices Implementation Details

Devices screen is now category-driven plus explicit HTTP-only side categories.

## Categories
- All
- Switches
- Scenes
- Temperatures
- Utilities
- Weather
- User Variables

## Important behaviors
- standard category content comes from bootstrap category index
- All Devices uses merged global list
- Scenes comes from `getscenes`
- User Variables comes from `getuservariables`
- category search is accent-insensitive and real-time
- ordering is preserved from Domoticz
- local reorder works per category

## Architectural note
Search is a **view-layer filter**, not a sorting or mutation step.

## Scenes / Groups details
- `Scene` is on-only trigger behavior
- `Group` is on/off behavior
- no WS support
- command result is confirmed with HTTP refresh

## User Variables details
- read-only
- sensor-style rendering
- tag icon
- no Favorites integration

---

# ⭐ Favorites Implementation Details

Favorites is still the most UX-critical screen.

## Important behaviors
- default landing screen
- cache-first responsiveness
- Domoticz favorites order preserved
- WebSocket-backed live updates for supported device types
- local reorder override supported
- favorite Scenes / Groups can now appear in Favorites as well

## Architectural note
Favorite membership and default order belong to Domoticz.  
Local visual ordering override belongs to ControlHome.

That split keeps the architecture sane.

---

# 🎨 Linked Tile Colors

This is one of the app’s most differentiated features.

## Concept
A tile’s visual background is determined by another device state.

## Why the architecture works
Because:
- all relevant devices are already present in shared state
- linked rules are local settings
- the override is calculated at UI layer

## Flow
```text
status device changes
        ↓
shared state updates
        ↓
rule evaluation
        ↓
tile background override applied
```

## Important constraint
The status device does not need to be favorite.

---

# 🎛 Color & Visual Customization System

The app has evolved toward a reusable visual customization model.

## Existing areas
- Room header image/color
- Category header color
- Tile appearance settings
- Linked Tile Colors
- adaptive text field styling on visually dynamic screens

## Common UX patterns
- preset colors
- custom HSV selection
- opacity/alpha control
- adaptive contrast in rendering

Any new customization UI should reuse these patterns rather than invent new ones.

---

# 🔁 Shared Reorder System

A meaningful structural improvement in this development phase is the shared reorder implementation.

## Shared parts
- `ReorderMenuButton`
- `ReorderActionButtons`
- `ReorderDeviceGrid`

## Why this matters
Earlier, Favorites, Rooms, and Devices drifted apart in drag behavior and top-bar controls.
Now:
- drag/drop behavior is harmonized
- top-bar edit entry is harmonized
- reset/cancel/done controls are harmonized

This reduces future regressions and makes reorder fixes propagate across screens more easily.

---

# ⚙️ Settings System

## SettingsStore
Stores non-sensitive state:
- server host/port and flags
- background config
- tile appearance
- linked rules
- room visuals
- category visuals
- cache
- reorder overrides
- QS config/state
- connection revision support

## SecretStore
Stores sensitive data:
- password only

This split is now a core architectural decision.

---

# 🔐 SecretStore and Connection Hardening

## Secret handling
Problem that existed:
- EncryptedSharedPreferences could become unreadable after reinstall/restore due to Android Keystore mismatch

Current direction:
- catch crypto init failures
- clear corrupted secret store
- recreate safely
- exclude secret prefs from backup

## Connection reset handling
Another production-hardening area added in this phase:
- when server settings change, the app must not keep talking to the previous server
- WS must reset
- view model / connection keys must reflect connection-relevant fields
- cached state should not silently mask an old still-active connection forever

This is still an area worth continued testing.

---

# ⚠️ Error-State Handling

This area now has more concrete behavior.

## Distinct states
1. Password missing
2. Unauthorized
3. Server not available / offline

These are not the same thing and should not be flattened into one generic error.

## Implemented direction
- Password missing → navigate to Server Settings and explain re-entry
- Unauthorized → navigate to Server Settings and explain credential failure
- Server unavailable → navigate to Server Settings and explain server/network issue

## Why this matters
The app is cache-first, so “UI visible” does not automatically mean “connection healthy”.

---

# 🔘 Quick Settings (QS) Deep Dive

QS is intentionally designed as a semi-independent control surface.

## Supported device kinds
- toggle
- dimmer
- selector

## Interaction model
Tap:
- main action / toggle path

Long press:
- deeper popup-style controls
- dimmer slider
- selector options

## Data strategy
- uses cached per-device state
- refreshes with HTTP when QS opens
- does not rely on active background WS

This is important because the app itself may not be in foreground.

---



---

# 📈 History Implementation Deep Dive

History is now implemented as a long-press popup feature rather than a permanent tile expansion.

## Main files

Current implementation is centered around:
- `HistoryRepository.kt`
- `HistoryModels.kt`
- `DeviceHistoryDialog.kt`
- `TileActionsHost.kt`

## Model

The repository maps raw Domoticz responses into a small sealed model:

```kotlin
sealed class DeviceHistory {
    data class Chart(...)
    data class Log(...)
    data class Unsupported(...)
}
```

This keeps UI rendering independent from Domoticz's inconsistent history field names.

## Long press behavior

Tile long press opens the history dialog.

Important behavior:
- normal tap remains the primary control action
- long press is reserved for details/history
- switch, dimmer, selector and sensor tiles may all route to history where supported

## Chart UI

Current chart behavior:
- Day / Month range selector
- metric toggle rows
- metric values with units
- compact metric spacing
- dynamic Y-axis recalculation based on visible metrics
- line-end values
- tap tooltip for selected point
- no pinch zoom

The tooltip is intentionally simple:
- tap selects a point by X position
- vertical marker line is drawn
- active metric values are shown for the selected timestamp
- tapping the same point again clears the selection

## Log UI

Switch and dimmer history uses event-list rendering.

Current behavior:
- simple switch history hides meaningless `0%` level values
- dimmer history still shows meaningful non-zero percentage levels
- user/source information is displayed where Domoticz provides it

## Domoticz history endpoint mapping

### Switch / Dimmer / Selector

```text
/json.htm?type=command&param=getlightlog&idx=IDX
```

Displayed as event log.

### Temperature / Humidity / Pressure

```text
/json.htm?type=command&param=graph&sensor=temp&idx=IDX&range=day|month
```

Field mapping:
- `te` → Temperature, °C
- `hu` → Humidity, %
- `ba` → Pressure, hPa
- `ta` → Average
- `tm` → Minimum

### PPM / Air Quality

```text
/json.htm?type=command&param=graph&sensor=counter&idx=IDX&range=day|month
```

Field mapping:
- `co2` → PPM, ppm
- `v` → PPM fallback, ppm

This is based on observed Domoticz behavior for Air Quality / VOC-style devices where the device displays ppm but the graph result uses `co2`.

### P1 Smart Meter

```text
/json.htm?type=command&param=graph&sensor=counter&idx=IDX&range=day|month&method=1
```

Field mapping:
- `v1 + v2` → Power input, W
- `r1 + r2` → Power output, W
- `eu` → Today input, kWh
- `eg` → Today output, kWh
- current device data → Total input / Total output

Important note:
`v1/v2` and `r1/r2` represent tariff/time-zone split data. Only one may be active in a given interval, but the app sums them to get the displayed metric.

### Electricity / Solar Energy Meter

```text
/json.htm?type=command&param=graph&sensor=counter&idx=IDX&range=day|month&method=1
```

Field mapping:
- `v` → Usage, W
- `eu` → Today, kWh
- current device data → Total, kWh

## Default metric visibility

P1 and Energy Meter metrics are visible by default.

Reason:
- the metric rows support toggling
- units are shown
- users can hide noisy series as needed

## Scale behavior

Chart scale is recalculated from active series only.

Important cases:
- if humidity is disabled, temperature gets a useful temperature-only scale
- if large Total kWh lines are disabled, Power W becomes readable again
- positive values do not always force a zero baseline; values far from zero get a tighter useful range

## Current limitation

Dual Y-axis is not implemented yet.

Potential future use:
- Temperature / Humidity
- W / kWh
- On/Off / %

This should be treated as a chart rendering improvement, not a repository or API change.


# ⚡ Performance Deep Dive

Performance is not accidental. It comes from a few key decisions.

## 1. Cache-first startup
Users see content almost immediately.

## 2. WebSocket instead of polling where possible
Runtime load stays low while responsiveness stays high.

## 3. Explicit HTTP-only side domains
Scenes / Groups and User Variables do not pollute the main WS-backed device path.

## 4. Shared state
No unnecessary duplicated fetch paths.

## 5. UI-layer filtering
Search and overrides do not force architectural complexity in the data layer.

---

# 🧪 Debugging Guidance

## If ordering looks wrong
Check:
- bootstrap merge order
- `devicesOrdered`
- scenes / variables preservation
- ViewModel mapping functions
- screen builders for accidental sorting

## If WS looks wrong
Check:
- ActiveGroup selection
- foreground/background lifecycle hooks
- snapshot sync behavior
- command fallback refresh

## If auth or connection switching looks wrong
Check:
- server settings in DataStore
- password presence in SecretStore
- connection revision / key generation
- WS reset behavior after settings save
- unauthorized vs password-missing distinction

## If cache looks wrong
Check:
- cache record timestamps
- bootstrap serialization schema
- auxiliary scenes / variables fields
- whether cached UI is expected to appear before fresh HTTP

---

# 🧱 Architectural Constraints to Respect

Any future feature should preserve these constraints unless there is a deliberate redesign:

1. **Domoticz order remains baseline**
2. **UI overrides remain local by default**
3. **Secret storage stays separate**
4. **Cache-first startup stays intact**
5. **Global WS/shared state remains the primary consistency mechanism where supported**
6. **HTTP-only domains stay explicitly modeled as such**

These constraints are what keep current behavior coherent.

---

# 🔮 Strong Candidates for Future Development

The current architecture already supports several next-step features well:

- diagnostics / logs screen
- richer global status/error banners
- device-level visual customization
- multi-server profiles
- tablet-specific layouts
- camera preview handling
- dual-axis chart rendering for mixed-unit history views
- additional Domoticz domains beside devices/scenes/variables

The most important thing is not to break the baseline invariants while adding them.

---

# ✅ Developer Summary

ControlHome is no longer a simple collection of screens.  
It now has a meaningful internal architecture:

- shared bootstrap-based model
- global real-time state where supported
- explicit HTTP-only side collections
- cache-first startup
- separated secure storage
- preserved Domoticz ordering
- reusable reorder infrastructure
- extensible UI override layer

That makes it a strong base for continued development, provided future changes continue to respect the current architectural boundaries.


---

# 📱 Responsive Tile/Grid Deep Dive

## Current Responsive Layout

Current responsive column configuration:

- Phone portrait → 2 columns
- Phone landscape → 4 columns
- Tablet portrait → 4 columns
- Tablet landscape → 6 columns

Important direction:
- deterministic layout behavior
- consistent visual density
- tablet-first dashboard optimization

---

# 🧩 Width-Based Tile Size Evolution

## Current Active Model

The app now uses a width-only tile size architecture.

Supported modes:
- Default
- 1x1
- 2x1
- 3x1
- 4x1

## Historical Context

Double-height tile support was experimentally implemented.

Problems discovered:
- excessive whitespace
- unstable visual density
- inconsistent value between tile types
- reorder-mode spacing divergence
- tile-size-mode layout instability

Architectural conclusion:
artificial height reservation was the wrong abstraction layer.

Earlier model:

grid reserves height
→ tile attempts to fill reserved space

Current model:

tile content defines height
→ grid adapts naturally

This dramatically improved layout consistency.

---

# ⭐ Favorites Section Tile System

## Concept

Favorites now supports local structural section tiles.

Purpose:
- visual separation of logical device groups
- improved tablet readability
- lightweight dashboard organization

Example:

[Lighting]
Switches...

[Heating]
Thermostats...

[Energy]
P1 / inverter / utilities...

## Architectural Nature

Section tiles:
- are not Domoticz devices
- are not WS-backed
- are not command-capable
- are not server-synchronized

They are purely local UI structure elements.

## Shared Infrastructure Reuse

Section tiles intentionally reuse:
- reorder system
- tile size system
- visual customization system
- adaptive contrast system
- category-header visual language

This avoided introducing a second parallel customization architecture.

## Visual Design Direction

Section tiles intentionally resemble Devices category header tiles.

Shared characteristics:
- larger typography
- distinct corner radius
- centered title
- category-style colors

Purpose:
- visually separate structure from actionable device tiles
- improve dashboard readability

---

# 🔁 Shared Reorder/Grid Evolution

The reorder system evolved significantly during this phase.

Earlier:
- Favorites
- Rooms
- Devices
- reorder mode
- tile size mode

all behaved slightly differently.

Current shared behavior:
- harmonized drag/drop
- harmonized edit entry
- shared spacing logic
- shared tile-size behavior
- shared tile span calculations

This substantially reduced regression risk.

---

# 🎨 Visual Customization Expansion

The app now has an increasingly unified visual customization model.

Shared customization behavior now spans:
- room headers
- category headers
- linked tile colors
- section header tiles
- tile appearance settings

Common UX patterns:
- preset colors
- HSV customization
- opacity support
- adaptive contrast
- `...` menu driven editing

This consistency is becoming a major UX strength.

---

# 🔮 Next Likely Architectural Direction

The current architecture is now well-positioned for:
- long-press history popup overlays
- chart rendering
- tablet-oriented dashboards
- richer information tiles

Recent implementation outcome:
- history popup using Domoticz HTTP history APIs
- popup-based chart rendering instead of persistent oversized tiles
- switch/dimmer log rendering
- P1, Energy Meter, Temperature, Humidity, Pressure and PPM history support

Likely next refinement:
- optional dual-axis chart rendering for mixed-unit metrics
