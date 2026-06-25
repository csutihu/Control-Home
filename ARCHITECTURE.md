# ControlHome – Architecture

## 🧱 High-Level Architecture

```text
Domoticz API
    │
    ├── HTTP bootstrap / refresh / command fallback
    ├── HTTP-only side channels
    │      ├── Scenes / Groups
    │      └── User Variables
    └── WebSocket (real-time updates for supported device domains)
            │
            ▼
      Repository Layer
            │
            ▼
      Shared Bootstrap / Cache Model
            │
            ├── Favorites data
            ├── Rooms data
            ├── Devices/category data
            ├── Scenes data
            ├── User Variables data
            ├── QS state cache
            └── UI override inputs
            │
            ▼
        ViewModels (StateFlow)
            │
            ▼
          Compose UI
```

This architecture is deliberately shaped around **fast startup**, **shared state**, **preserved Domoticz ordering**, and **minimal duplicated logic**.

---

# 🔄 Data Layers

## 1. API Layer
Responsibilities:
- Talk to Domoticz HTTP endpoints
- Manage WebSocket connection where applicable
- Provide raw DTOs and WS payloads

Key properties:
- Retrofit + OkHttp
- HTTP is used for bootstrap, refresh, commands, Scenes, User Variables, and history
- WebSocket is used for supported real-time device domains only
- Scenes / Groups and User Variables are **HTTP-only**
- History is **HTTP-only** and intentionally separate from the WS runtime model

---

## 2. Repository Layer
Responsibilities:
- Normalize Domoticz data
- Merge category-based device results into a consistent bootstrap model
- Preserve ordering
- Apply WS patches to the current snapshot
- Seed cache and QS state
- Maintain extra HTTP-only collections such as Scenes and User Variables inside the same shared cache model
- Map Domoticz history responses into chart/log models for the history popup

Important rule:
- **Repository owns the canonical ordering logic**
- UI must not reorder unless explicitly implementing a local visual override feature

---

## 3. Shared Bootstrap / Cache Model
This is the effective shared in-memory model used throughout the app.

Contains:
- Device inventory
- Per-plan device mapping
- Category membership index
- Favorite order baseline
- Scenes list
- User Variables list
- Current bootstrap state

Why this matters:
- Favorites, Rooms, Devices, and Linked Tile Colors depend on the same truth base
- Scenes/Variables can be integrated without pretending they are normal WS-backed devices
- Cache-first startup still works while allowing HTTP-only side collections

---

## 4. ViewModel Layer
Responsibilities:
- Translate repository/bootstrap data into UI models
- Expose StateFlow-based screen state
- Handle optimistic UI updates
- Apply fallback HTTP refresh where WS cannot confirm in time
- Handle HTTP-only categories lazily

ViewModels currently of interest:
- FavoritesViewModel
- RoomsViewModel
- DevicesViewModel

History is loaded on demand from the UI action host and is deliberately not part of the normal bootstrap or WS state.

Important update:
- Favorites now merges Domoticz-favorite normal devices with Domoticz-favorite Scenes / Groups
- DevicesViewModel now owns lazy HTTP loading for Scenes / Groups and User Variables

---

## 5. UI Layer
Responsibilities:
- Stateless rendering
- Consume ViewModel state only
- Apply presentation-only transforms such as filtering/search
- Apply visual overrides such as linked tile colors and custom header visuals
- Provide local reorder interaction while keeping Domoticz order as baseline

Important rule:
- UI filters may reduce visible items but must not mutate the canonical order model

---

# 🧠 State Management

ControlHome uses:
- StateFlow
- cache-first bootstrap
- global WebSocket-backed state where supported
- explicit HTTP-only side state for Scenes / Groups and User Variables

Design target:
- **Single source of truth**
- **No duplicated mutable screen models**
- **No screen-specific reimplementation of device state**

---

# 🔄 Ordering Strategy

## Current design rule
**Domoticz order = default app order**

This now applies consistently to:
- Favorites
- Rooms
- Devices categories
- Scenes category
- User Variables category

## Why this matters
Local tile reorder depends on this invariant:
- default = Domoticz order
- local override = ControlHome-only custom order
- reset = back to Domoticz order

## Corrected approach
Ordering is now preserved end-to-end:
- API result order
- repository merge order
- ViewModel mapping order
- screen rendering order

---

# 🌐 WebSocket Architecture

## Design
- Single global WebSocket connection
- Shared across app features
- Not tied to one screen’s local lifecycle model

## WS states
Typical states include:
- Disconnected
- Connecting
- Syncing
- Active
- Suspended

## ActiveGroup concept
The app scopes WS behavior using an `ActiveGroup` abstraction.

Examples:
- Favorites
- Room(planId)
- Devices(category)
- None

## Important limitation
Scenes / Groups and User Variables are **not part of the WS model**.  
They remain explicit HTTP-driven categories.

This is architecturally important because it avoids pretending those domains have real-time guarantees they do not have.

---

# 📦 Bootstrap Snapshot Architecture

The bootstrap snapshot is a critical foundation.

It combines:
- Domoticz plans
- Domoticz favorites ordering metadata
- Domoticz category-based device responses
- cached or refreshed Scenes / Groups collection
- cached or refreshed User Variables collection

The result becomes a unified `BootstrapData` model containing:
- plans
- devicesByIdx
- devicesOrdered
- devicesByPlan
- categoryIndex
- favoriteOrder
- scenes
- userVariables

This bootstrap model is then:
- cached
- published
- transformed into HomeData and UI state

---

# 🏠 Rooms Architecture

Rooms are based on:
- plans from Domoticz
- per-plan device mapping using `PlanID` / `PlanIDs`

Important architectural rules:
- rooms with `_` in name are excluded
- room ordering follows Domoticz plan order
- room device ordering follows Domoticz device order
- room visual customization is stored locally only

Current room customization scope:
- image
- header color
- reset to default

Implementation updates:
- room image selection uses persistent document access
- room header presentation is mutually exclusive: image or color
- image and color state transitions are explicitly normalized

---

# 🧩 Devices Architecture

Devices view is built from category-based data plus HTTP-only side collections.

Sections:
- All
- Switches
- Scenes
- Temperatures
- Utilities
- Weather
- User Variables

Important architectural rules:
- standard device category membership comes from Domoticz filter responses
- Scenes / Groups come from `getscenes`
- User Variables come from `getuservariables`
- ordering follows Domoticz/bootstrap order
- search filters only visible items
- no alphabetical resort in UI

## Scenes / Groups
- HTTP-only
- lazy-loaded on entry
- command result confirmed by HTTP refresh
- Scene sends only `On`
- Group sends `On` / `Off`

## User Variables
- HTTP-only
- lazy-loaded on entry
- read-only
- rendered as sensor-style UI tiles

---

# ⭐ Favorites Architecture

Favorites remains special because:
- it is the default landing screen
- it is the most performance-sensitive screen
- Domoticz favorites order is meaningful to users

Architectural rule:
- favorite membership belongs to Domoticz
- favorite ordering baseline belongs to Domoticz
- local reorder is an app-only override layer

Important update:
- Favorites now supports Domoticz-favorite Scenes / Groups in addition to normal favorite devices
- This keeps the ControlHome default Favorites view closer to real Domoticz behavior

---

# 🎨 UI Override Layer

## Linked Tile Colors
Implemented as a UI override layer.

Flow:

```text
device state
   ↓
linked rule evaluation
   ↓
tile background override
```

Important property:
- does not mutate the underlying device model
- does not require backend changes
- works because screens consume shared state

## Header Visual Customization
Also a UI-only override system:
- Room header image/color
- Category color

Again:
- persistent locally
- not written back to Domoticz

## Reorder UI
The three major views now share a common reorder component model:
- shared reorder menu button
- shared reorder action buttons
- shared drag/drop reorder grid

This reduced divergence between Favorites, Rooms, and Devices while preserving local screen-specific behavior.

---

# ⚙️ Settings Architecture

Settings are split into two storage domains:

## 1. DataStore / SettingsStore
Stores non-sensitive app state:
- server host/port flags
- background settings
- tile appearance
- linked tile color rules
- room/category visual settings
- caches
- reorder overrides
- QS state cache
- connection revision / reset helpers

## 2. SecretStore
Stores sensitive secret data:
- password

This split is now a structural requirement.

---

# 🔐 Security, Backup, and Connection Reset

A key improvement in recent iterations is backup-safe secret handling plus stronger connection reset behavior.

## SecretStore hardening
- encrypted prefs can become invalid after reinstall/restore
- app now recovers by forcing password re-entry instead of crashing
- secret prefs remain excluded from backup/restore

## Connection reset hardening
When server settings change:
- connection revision is bumped
- caches can be invalidated where needed
- WS is explicitly reset
- navigation/view model keys use connection-sensitive fields

This reduces the chance of the app continuing to talk to a previously active Domoticz server after settings were changed.

---

# ⚠️ Error Handling Architecture

Current app-state model distinguishes:
- Password missing
- Unauthorized
- Server not available / offline

Implemented UX direction:
- Password missing → navigate to Server Settings
- Unauthorized → navigate to Server Settings
- Server unavailable with no usable cache → navigate to Server Settings with clear message
- Server unavailable with usable cache → keep the current UI visible and show the offline cache banner
- Offline cached mode provides `Retry` and `Settings` actions
- Automatic offline probing is quiet and does not hide the banner while checking
- Command failures caused by connection loss are represented by the shared offline state, not by immediate Settings navigation

This split is important because “UI is visible” does not automatically mean “current connection is healthy”.  
The UI can intentionally remain visible in cached/offline mode while the connection coordinator probes for recovery.

---



---

# 📈 History Architecture

History is implemented as an on-demand, HTTP-only popup flow.

## Entry point

History is opened from device tiles by long press.

Design goal:
- keep normal tile controls fast and compact
- avoid permanently increasing tile height
- show detailed historical information only when requested

## Data flow

```text
Tile long press
   ↓
TileActionsHost
   ↓
HistoryRepository
   ↓
Domoticz HTTP history endpoint
   ↓
DeviceHistory model
   ↓
DeviceHistoryDialog
```

## Supported history presentation types

### Chart history
Used for numeric metrics:
- Temperature
- Humidity
- Pressure / barometer
- PPM / CO2 / VOC-style air quality history
- P1 Smart Meter
- Electricity / solar energy meter

### Log history
Used for event-style devices:
- Switch
- Dimmer
- Selector
- Contact sensor, where applicable

### Unsupported history
Unsupported types are handled explicitly with a user-facing message instead of failing silently.

Current unified message direction:
- `History is not available for this device type.`
- `History is not available for this item.`

## Range support

Current UI-supported ranges:
- Day
- Month

Year view is intentionally not exposed in the mobile UI because it is less useful on phone/tablet screens.

## Chart interaction

Current chart behavior:
- selectable Day / Month range
- metric rows below the chart
- metrics can be toggled on/off
- chart Y range is recalculated from the currently visible metrics
- compact metric list with units
- line-end value display
- tap-on-chart tooltip showing values at the selected point
- no pinch zoom by design

The current position is that pinch zoom is not worth the added complexity for this mobile control app. Deeper analysis belongs on a larger screen.

## Metric mapping

### Temperature / Humidity / Pressure

Domoticz endpoint:

```text
/json.htm?type=command&param=graph&sensor=temp&idx=IDX&range=day|month
```

Known fields:
- `te` → Temperature, °C
- `hu` → Humidity, %
- `ba` → Pressure, hPa
- `ta` → Average temperature in aggregated views
- `tm` → Minimum temperature in aggregated views

### PPM / Air Quality

Domoticz endpoint:

```text
/json.htm?type=command&param=graph&sensor=counter&idx=IDX&range=day|month
```

Known fields:
- `co2` → PPM, ppm
- `v` → PPM fallback, ppm

This is intentionally handled as a special history mapping because Air Quality / VOC-style devices do not expose history through the same `sensor=temp` path.

### P1 Smart Meter

Domoticz endpoint:

```text
/json.htm?type=command&param=graph&sensor=counter&idx=IDX&range=day|month&method=1
```

Known fields:
- `v1 + v2` → Power input, W
- `r1 + r2` → Power output, W
- `eu` → Today input, kWh
- `eg` → Today output, kWh
- current device fields → Total input / Total output

Notes:
- `v1/v2` and `r1/r2` represent tariff/time-zone split data and must be summed
- export values may appear negative and are normalized where needed
- all P1 metrics are visible by default and can be toggled individually

### Electricity / Solar Energy Meter

Domoticz endpoint:

```text
/json.htm?type=command&param=graph&sensor=counter&idx=IDX&range=day|month&method=1
```

Known fields:
- `v` → Usage, W
- `eu` → Today, kWh
- current device field → Total, kWh

All energy meter metrics are visible by default and can be toggled individually.

## Chart scale rule

The Y-axis is calculated dynamically from visible series.

Important behavior:
- disabling a metric recalculates the visible range
- values near zero may anchor to zero
- values far from zero use a tighter range so small changes remain visible
- this is important for examples such as 20–22 °C temperature or mixed humidity/temperature charts

## Future optional improvement

A dual Y-axis may be useful later for:
- °C / %
- W / kWh
- on-off / %

This was intentionally not implemented yet because the current toggle + dynamic-scale approach is already usable.



# 🌐 Connection Recovery Architecture

Connection recovery is now centralized.

Main component:
- `ConnectionRecoveryCoordinator`

Shared state:
- `Idle`
- `Recovering`
- `Online`
- `Unavailable(issue, failureId, hasCachedData)`

## Purpose

The coordinator prevents Favorites, Rooms, and Devices from implementing separate server-check and settings-navigation behavior.

Important architecture decision:
- main screens do not directly navigate to Server Settings from generic device command errors
- they report connection-type failures to the coordinator
- the coordinator owns offline/cache state
- `NavGraph` owns the final global UI reaction

## Startup / foreground recovery

The full recovery cycle is used for:
- app startup
- foreground return
- Server Settings `Done`
- explicit `Retry`

The quick retry schedule is:
- 0 ms
- 400 ms
- 800 ms
- 1200 ms

Each probe is intentionally short.

If a probe succeeds:
- fresh bootstrap data is refreshed and cached
- the coordinator moves to `Online`

If all probes fail:
- with cache → `Unavailable(..., hasCachedData = true)`
- without cache → `Unavailable(..., hasCachedData = false)` and Settings can be opened

## Offline cached mode

When cache exists and the server is unavailable:
- the app keeps cached UI visible
- `OfflineCacheBanner` is shown
- the banner exposes `Retry` and `Settings`

Automatic background retry uses a separate quiet method:
- `probeWhileOffline()`

Important rule:
- quiet probes must not change the visible state on failure
- the banner must remain stable and must not blink
- only successful probe + refresh transitions to `Online`

## Shared repository relationship

Favorites, Rooms, and Devices now operate on the same repository foundation for the active server connection.

This avoids the previous divergence where one screen could think the server was unavailable while another screen still displayed cached or refreshed content.
# ⚡ Performance Strategy

Performance is based on a few deliberate principles:

## Cache-first startup
- read cached snapshot
- draw immediately
- refresh afterwards

## WebSocket-first runtime where supported
- avoid repeated polling
- use fallback refresh only when necessary

## Explicit HTTP-only categories
- Scenes / Groups and User Variables do not burden the general WS path
- they are loaded when needed

## Minimal recomposition
- keep UI state narrow
- patch only affected devices where possible

## QS independence
- QS must work without depending on active foreground WS

---

# 🧪 Testing Strategy Guidance

## Ordering regressions
Check:
- repository merge order
- ViewModel mapping
- screen-level sort/filter logic
- reset order paths for Favorites / Rooms / Devices

## SecretStore regressions
Test:
- reinstall
- password deletion
- backup/restore-like scenarios

## WS regressions
Check:
- ActiveGroup transitions
- foreground/background lifecycle
- snapshot + WS sync interplay

## HTTP-only category regressions
Check separately:
- Scenes / Groups first load
- command-after-refresh behavior
- User Variables lazy load

## Offline/auth states
Check separately:
- password missing
- unauthorized credentials
- server unavailable

---

# 🔮 Extensibility

The current architecture is well positioned for future growth.

Good candidates:
- multi-server profiles
- diagnostics/logs screen
- richer linked tile conditions
- device-level visual customization
- camera preview surfaces
- dual-axis chart rendering for selected mixed-unit history views

Important rule for extending:
- protect the invariants:
  - Domoticz order is baseline
  - UI overrides stay local unless explicitly server-writing
  - cache-first startup remains intact
  - secret data remains separated from normal settings
  - HTTP-only categories remain explicit

---


# 📦 Release / Store Readiness Notes

Current public-distribution direction:
- GitHub is the primary project/documentation location
- Google Play distribution should start with internal or closed testing
- production release should follow after real-world testing
- Play release should use Android App Bundle (`.aab`)

Policy-sensitive design direction:
- keep the Play-distributed app free
- avoid in-app PayPal/donation prompts
- keep project support/donation links outside the app, preferably in GitHub documentation
- describe clearly that ControlHome connects only to the user-configured Domoticz server and does not operate a cloud backend
# ✅ Architectural Summary

ControlHome currently stands on a solid architectural foundation because it now has:

- shared bootstrap/cache state
- global WS-backed consistency where Domoticz supports it
- explicit HTTP-only side domains for Scenes / Groups and User Variables
- clean separation between sensitive and non-sensitive storage
- preserved Domoticz ordering
- shared reorder infrastructure
- UI-only override architecture for advanced visual features

That combination makes the app both fast today and realistically extensible tomorrow.


---

# 📱 Responsive Grid Architecture

## Current Layout Strategy

The tile grid system now uses explicit responsive column counts.

Current configuration:
- phone portrait → 2 columns
- phone landscape → 4 columns
- tablet portrait → 4 columns
- tablet landscape → 6 columns

Important design direction:
- deterministic layouts
- no runtime auto-density logic
- current UX direction avoids exposing manual grid tuning in settings

---

# 🧩 Tile Size Architecture

Favorites, Rooms, and Devices now share a unified width-based tile size override system.

Supported modes:
- Default
- 1x1
- 2x1
- 3x1
- 4x1

Important architectural decision:
- width-only resizing is the active architecture
- tile height is now primarily content-driven
- earlier double-height reservation logic was intentionally removed

Reason:
the previous height-reservation approach caused:
- spacing instability
- wasted vertical space
- reorder-mode inconsistencies
- tile-size-mode inconsistencies

Current direction:
- tile content defines height
- grid adapts naturally
- width overrides remain explicit UI metadata

---

# ⭐ Favorites Section Tile Architecture

Favorites now supports local section header tiles.

These tiles are:
- application-owned UI elements
- not mapped to Domoticz entities
- persisted locally in SettingsStore
- merged into Favorites rendering order

Purpose:
- lightweight visual grouping
- improved tablet readability
- dashboard structure without hierarchy

Important constraints:
- no WS participation
- no Domoticz synchronization
- no command behavior
- no backend persistence

Shared infrastructure reuse:
- reorder system
- tile size system
- visual customization system
- adaptive contrast system
- category-header visual language

---

# 🔁 Shared Reorder/Grid Infrastructure

The reorder system evolved into a shared infrastructure layer.

Shared capabilities:
- reorder mode
- tile size mode
- local UI tiles
- harmonized edit controls
- shared spacing calculations

Important debugging outcome:
many earlier spacing problems were caused by:
- hidden height reservation logic
- inconsistent grid spacing rules between edit modes

The current implementation is significantly more stable.

## Reorder input capture

Reorder mode now uses a transparent drag-capture layer over each tile.

Reason:
- switch and dimmer tiles contain interactive touch handlers
- dimmer tiles contain a slider
- those inner controls can otherwise consume or cancel long-press drag gestures

Current rule:
- in reorder mode, the grid-level drag layer owns touch input
- tile controls are effectively passive
- normal tap, long-press history and slider behavior remain unchanged outside reorder mode

This makes switch/dimmer reorder behavior consistent with selector, sensor, thermostat and P1 tiles.
