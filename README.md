# ControlHome

## 🚀 Overview
ControlHome is a fast, mobile-first Domoticz client focused on:

- ⚡ Real-time control where Domoticz supports it
- 🚀 Instant startup with cache-first rendering
- 🎯 Optimistic UI for fast interaction
- 🎨 Deep UI customization for tiles, headers, and backgrounds
- 🌐 Unified connection recovery with offline cache banner
- 🔁 Shared reorder and tile-size infrastructure across Favorites, Rooms, and Devices
- Supports local section/separator tiles for grouping favorite devices

This is **not a simple dashboard**.  
It is a full-featured mobile control interface designed for speed, clarity, and further extension.

---

# 🧭 Main Screens

## ⭐ Favorites (Default screen)

### Features
- Uses Domoticz **Favorites ordering** as the default baseline
- WebSocket real-time updates for normal devices
- Cache-first loading
- Optimistic UI behavior
- Local reorder override on top of Domoticz order
- Supports Domoticz-favorite **Scenes / Groups** as well as normal favorite devices

### Notes for further development
- Favorites remains the reference screen for perceived performance.
- Favorites membership belongs to Domoticz.
- Favorites default order comes from Domoticz.
- Local tile reorder should remain an override layer on top of Domoticz ordering, not a replacement for it.

---

## 🏠 Rooms

### Data Source
- Domoticz `getplans`
- Devices mapped by `PlanID` / `PlanIDs`

### Ordering
- Uses **Domoticz order (NOT alphabetical)**
- Preserved end-to-end (API → Repository → ViewModel → UI)
- Supports local per-room tile reorder with reset back to Domoticz order

### Navigation
- Swipe left/right
- Arrow buttons

### Room filtering
- If a room name contains `_`, it is ignored from the visible room set.

### Room Header (Top Tile)

Each room has a large customizable tile.

#### Menu
- Set image
- Set color
- Set default

#### Behavior
- Image OR color background, never both as active presentation mode
- Automatic contrast handling
- Overlay dim for readability
- Header title and controls adapt to the effective room header visual
- Image selection uses persistent document access, so the chosen image survives app restart

### Notes for further development
- The room header is visual customization only. It does not change Domoticz room structure.
- Room reorder is already implemented as an app-local visual override.
- Reset order should always return to Domoticz room device order.

---

## 🧩 Devices

### Categories
- All
- Switches
- Scenes
- Temperatures
- Utilities
- Weather
- User Variables

### Ordering
- Uses Domoticz JSON order as default
- Scene and variable categories are HTTP-driven, not WebSocket-driven
- Supports local reorder per category, with reset back to Domoticz order

### Category Header
- Large category tile
- Per-category color customization
- Left/right navigation between categories

### 🔍 Search
- Real-time filtering
- Accent-insensitive
- Per-category
- Partial name matching
- Search field styling adapts to the same dark/light title logic used by the screen header

### Scenes / Groups
- Loaded from Domoticz `getscenes`
- Added after **Switches**
- Scene tiles use power-style visuals
- `Scene` sends only `On`
- `Group` sends `On` / `Off`
- No WebSocket support, therefore action result is confirmed by HTTP refresh

### User Variables
- Loaded from Domoticz `getuservariables`
- Placed after **Weather**, at the end of Devices
- Read-only
- Displayed as sensor-style tiles
- Uses a tag-style icon

### Notes for further development
- Category lists should always inherit order from the merged Domoticz/bootstrap snapshot.
- Search is a UI filter only. It must never mutate the underlying order model.
- HTTP-only categories such as Scenes and User Variables should stay structurally separate from the WS-backed device categories.

---

# 🔘 Tiles

## Supported Types

- Switch
- Dimmer
- Selector
- Sensor
- Contact sensor (OPEN/CLOSED → no toggle)
- Scene / Group tile behavior on Devices and Favorites
- User Variable sensor tiles
- Long-press history popup where Domoticz exposes history

## Behavior

### Optimistic UI
- Immediate state change on tap where the device type supports control
- WS or fallback HTTP refresh confirms final state

### Edge case
- If backend rejects → tile reverts or reloads from truth source

### Contact sensor behavior
- Contact sensors are displayed as state indicators, not toggle controls.
- They can still be used by Linked Tile Colors as status source devices.

### Scene / Group behavior
- Scene tiles are activation-only and always send `On`
- Group tiles behave as `On` / `Off`
- Scene/Group status display still comes from Domoticz state

### Notes for further development
- Tile behavior is intentionally separated from storage and server structure.
- A future tile-specific customization layer should remain UI-only unless explicitly intended to write back to Domoticz.

---



---

# 📈 History

ControlHome supports a long-press history popup for supported devices.

## How to open

Long press a tile.

The popup can show either:
- a chart for numeric history
- an event log for switch/dimmer/selector style history
- an unsupported message when Domoticz does not expose suitable history for that device type

## Supported history types

### Numeric chart history
Supported examples:
- Temperature
- Humidity
- Pressure / barometer
- PPM / Air Quality
- P1 Smart Meter
- Electricity / solar energy meter

### Event log history
Supported examples:
- Switch
- Dimmer
- Selector

## History ranges

Current supported ranges:
- Day
- Month

Year view is intentionally not part of the current mobile UI.

## Chart features

- Day / Month selector
- visible metric toggles
- metric values with units
- dynamic Y-axis scaling
- chart range recalculates when metrics are enabled/disabled
- line-end values
- tap tooltip showing values at a selected point

Pinch zoom is intentionally not implemented at this stage.

## Important metric mappings

### Temperature / Humidity / Pressure
- Temperature → °C
- Humidity → %
- Pressure → hPa

### PPM / Air Quality
- PPM → ppm

### P1 Smart Meter
- Power input → W
- Power output → W
- Today input → kWh
- Today output → kWh
- Total input → kWh
- Total output → kWh

### Energy Meter
- Usage → W
- Today → kWh
- Total → kWh

## Notes for further development

A future dual Y-axis chart renderer could improve mixed-unit charts, for example:
- Temperature / Humidity
- W / kWh
- On/Off / %

The current implementation uses metric toggles and dynamic scaling instead.


# 🔗 Linked Tile Colors

## Purpose
Control a tile background using another device state.

## Use Cases
- Garage door button colored by door sensor state
- Thermostat tile reflecting real heating activity

## Rules
- Status device does **not** need to be favorite
- Global, works across screens
- WebSocket-driven where supported
- Implemented as a UI override layer, not server-side metadata

## Current behavior
- The UI uses linked rules to calculate visual overrides for tile backgrounds.
- Rules are stored in app settings.
- The underlying Domoticz device model remains unchanged.

---

# 🎛 Custom Colors

Available for:
- Rooms
- Device category headers
- Tiles
- Linked Tile Colors rules

## Features
- Preset colors
- HSV picker
- Opacity slider
- Adaptive contrast

---

# ⚙️ Settings

## Server
- Host / Port / SSL / self-signed handling
- Credentials (password stored in SecretStore)

### Special Handling
- Password is not backed up
- Missing password redirects to Server Settings with a clear message
- Unauthorized credentials redirect to Server Settings with a clear message
- Saving server settings now forces connection reset/rebuild behavior so the app does not keep talking to the previously active server

## Background
- Color or image
- Blur (Android 12+)
- Dim overlay

## Tile Appearance
- Colors
- Icon style
- Radius
- Typography

## Linked Tile Colors
- Rule editor
- Condition-based coloring

## 🔘 Quick Settings (QS)

Android QS tiles.

### Supported
- Switch
- Dimmer
- Selector

### Limits
- Up to 6 slots

### Behavior
Tap:
- Toggle / main action

Long press:
- Dimmer slider
- Selector options

### Data model
- Cache + HTTP refresh
- No WebSocket dependency in background
- Uses last known per-device state cache for quick rendering

---

# 🎨 UI Behavior

## Status Bar
- Edge-to-edge layout
- Dynamic icon color

## Dynamic Contrast
Based on:
- Background color luminance
- Image dim level

## Header adaptation
- Room header text/icons adapt to room visual
- Category header text/icons adapt to category background color
- Reorder menu icon ring follows the same light/dark contrast logic

---

# ⚡ Performance

- Cache-first startup
- Minimal HTTP calls
- WebSocket-driven updates where supported
- HTTP-only categories loaded lazily when entered

## Startup flow
- Load cached state first
- Render immediately
- Refresh in background
- Apply WS updates after snapshot sync

## Notes for further development
- Cache-first behavior is now a core part of perceived UX quality.
- Any future architectural refactor should protect startup responsiveness.

---

# 🔄 Data Flow

Startup:

```text
Cache → UI → HTTP refresh → WebSocket sync
```

Runtime:

```text
WebSocket / HTTP refresh → Global store → UI update
```

Additional HTTP-only flows:
- Scenes / Groups
- User Variables

These are merged into the app model without pretending they are WS-backed.

---

# 🌐 Connection Recovery & Offline Cache

ControlHome now uses a shared connection recovery model across Favorites, Rooms, and Devices.

## Startup / foreground recovery

When the app starts or returns from background:

```text
cached snapshot
   ↓
immediate UI render, if available
   ↓
central connection probe/recovery
   ↓
fresh bootstrap refresh when server is available
```

The recovery cycle uses a short retry sequence:
- 0 ms
- 400 ms
- 800 ms
- 1200 ms

Each probe has a short timeout so an unavailable LAN/server does not block the UI for a long time.

## Offline cached mode

If the server is not reachable but cached content exists:
- the main screen stays visible
- an `Offline - cached data` banner is shown
- `Retry` starts an explicit recovery attempt
- `Settings` opens Server Settings
- a quiet automatic probe runs in the background

The automatic probe does not hide the banner while checking.  
This prevents UI blinking during offline periods.

## Command failure handling

If a command fails because the server is unavailable:
- the app does not immediately navigate to Server Settings
- the shared offline/cache state is activated
- cached UI remains visible where possible

This keeps transient Wi-Fi or Domoticz restart issues from disrupting normal app navigation.

---

# 🔐 Security & Storage

## SecretStore
- Stores password separately
- Not included in backup
- Recreated safely if encrypted storage becomes invalid

## Backup rules
- Prevent restoring broken auth state
- Secret prefs excluded from backup
- Non-sensitive settings can remain restorable

---

# ⚠️ Error Handling

## Password Missing
- Redirects to Server Settings
- Requests password re-entry

## Unauthorized
- Redirects to Server Settings
- Shows authentication error message

## Server Not Available / Offline
- Startup and foreground recovery are coordinated centrally
- Existing cached data remains visible when possible
- If cache is available, the app shows an `Offline - cached data` banner instead of immediately forcing Server Settings
- The banner provides `Retry` and `Settings` actions
- Automatic offline retry uses a quiet single-probe loop so the banner stays stable and does not blink
- If no usable cache exists, repeated failed recovery can still navigate to Server Settings with a clear explanation
- A failed switch/dimmer/selector command does not automatically open Server Settings; the connection state is represented by the shared offline banner

## Current development status
- Broken encrypted prefs crash is fixed
- Missing password and unauthorized flows are implemented
- Server unavailability is handled through the shared connection recovery coordinator
- Offline cached operation is supported with a stable banner and silent background probes

---

# 🧠 Architecture Summary

```text
WebSocket + HTTP → Shared bootstrap/cache model → ViewModels → Compose UI
```

- Single source of truth
- No duplicated mutable screen state
- Instant startup from cache
- Local UI override layers on top of Domoticz truth

---

# 🧭 Development Guidance

These rules should be preserved in future development:

1. **Domoticz order is the default order everywhere**
   - Favorites
   - Rooms
   - Devices categories

2. **UI overrides remain local unless explicitly intended otherwise**
   - Linked Tile Colors
   - Room header visuals
   - Category header visuals
   - Local tile reorder

3. **Secret data stays separated from normal settings**
   - Password in SecretStore
   - UI/config in DataStore

4. **Cache-first startup must be protected**
   - It is one of the app’s strongest UX characteristics

5. **HTTP-only categories must stay explicit**
   - Scenes / Groups
   - User Variables

---

# 💬 Summary

ControlHome provides:

- ⚡ Real-time control where available
- 🎨 Full customization
- 🧠 A coherent internal architecture
- 📱 A fast mobile-first Domoticz UX

Designed for further extension without breaking baseline Domoticz behavior.


---

# 📱 Responsive Tile/Grid System

## Adaptive Grid Layout

Current responsive layout:

- Phone portrait: 2 columns
- Phone landscape: 4 columns
- Tablet portrait: 4 columns
- Tablet landscape: 6 columns

The grid system is intentionally deterministic and currently not user-configurable.

## Width-Based Tile Size System

Favorites, Rooms, and Devices now share a common width-based tile sizing system.

Supported tile sizes:
- Default
- 1x1
- 2x1
- 3x1
- 4x1

Important implementation direction:
- width-only resizing is the active architecture
- tile height is now content-driven
- experimental double-height reservation logic was intentionally removed from active UX usage

This significantly improved:
- spacing consistency
- reorder mode stability
- tile-size mode stability
- Compose grid behavior

## Section Header Tiles

Favorites now supports local application-defined section header tiles.

Purpose:
- visually separate logical device groups
- improve tablet dashboard readability
- provide lightweight organization without introducing hierarchy or sub-pages

Characteristics:
- Favorites-only feature
- local UI object
- not backed by Domoticz devices
- reorderable
- resizable
- customizable title and color
- persisted locally
- uses the same visual language as Devices category headers

Section tile editing supports:
- rename
- color customization
- delete
- shared `...` menu interaction pattern

Visual direction:
- centered title
- larger typography
- category-style radius
- category-style color palette

## Shared Reorder Evolution

Favorites, Rooms, and Devices now share:
- reorder menu behavior
- reorder action bars
- tile size edit mode
- spacing logic
- drag/drop infrastructure

This significantly reduced layout divergence between screens and edit modes.

## Reorder touch handling

Reorder mode now places a transparent drag layer over each tile.

Reason:
- switch tiles and dimmer tiles have their own interactive touch handling
- dimmers also contain slider interaction
- those inner gestures can cancel grid drag if they receive touches during reorder mode

Current behavior:
- reorder mode captures tile touch at the grid layer
- inner switch/dimmer/slider controls are passive while reordering
- normal tile interaction is unchanged outside reorder mode
- switch and dimmer reorder now behaves like selector/sensor/P1 tiles

---

---

# 📦 Play Store Publishing Notes

ControlHome is planned as a free app.

Recommended distribution path:
- GitHub remains the public project and issue/support location
- Google Play should start with internal or closed testing
- production release should follow after real-world testing
- the app should be uploaded to Play as an Android App Bundle (`.aab`), not as a standalone APK

Donation/support recommendation:
- keep the Play-distributed app free and without in-app donation buttons
- if project support is desired, use a `Support the project` section on GitHub instead of putting a PayPal donation flow inside the app


# 🖼 Pictures

The screenshots below are shown in filename creation order. Click any picture to open it in full size.

<table>
  <tr>
    <td><a href="Screenshot_20260325_224541_Control%20Home.jpg"><img src="Screenshot_20260325_224541_Control%20Home.jpg" width="180" alt="ControlHome screenshot 1"></a></td>
    <td><a href="Screenshot_20260325_224600_Control%20Home.jpg"><img src="Screenshot_20260325_224600_Control%20Home.jpg" width="180" alt="ControlHome screenshot 2"></a></td>
    <td><a href="Screenshot_20260325_224617_Control%20Home.jpg"><img src="Screenshot_20260325_224617_Control%20Home.jpg" width="180" alt="ControlHome screenshot 3"></a></td>
    <td><a href="Screenshot_20260325_224642_Control%20Home.jpg"><img src="Screenshot_20260325_224642_Control%20Home.jpg" width="180" alt="ControlHome screenshot 4"></a></td>
  </tr>
  <tr>
    <td><a href="Screenshot_20260405_000429_Control%20Home.jpg"><img src="Screenshot_20260405_000429_Control%20Home.jpg" width="180" alt="ControlHome screenshot 5"></a></td>
    <td><a href="Screenshot_20260405_001252_Control%20Home.jpg"><img src="Screenshot_20260405_001252_Control%20Home.jpg" width="180" alt="ControlHome screenshot 6"></a></td>
    <td><a href="Screenshot_20260405_001311_Control%20Home.jpg"><img src="Screenshot_20260405_001311_Control%20Home.jpg" width="180" alt="ControlHome screenshot 7"></a></td>
    <td><a href="Screenshot_20260405_002320_One%20UI%20Home-EDIT.jpg"><img src="Screenshot_20260405_002320_One%20UI%20Home-EDIT.jpg" width="180" alt="ControlHome screenshot 8"></a></td>
  </tr>
  <tr>
    <td><a href="Screenshot_20260606_201115_Control%20Home.jpg"><img src="Screenshot_20260606_201115_Control%20Home.jpg" width="180" alt="ControlHome screenshot 9"></a></td>
    <td><a href="Screenshot_20260606_201238_Control%20Home.jpg"><img src="Screenshot_20260606_201238_Control%20Home.jpg" width="180" alt="ControlHome screenshot 10"></a></td>
    <td><a href="Screenshot_20260606_201300_Control%20Home.jpg"><img src="Screenshot_20260606_201300_Control%20Home.jpg" width="180" alt="ControlHome screenshot 11"></a></td>
    <td><a href="Screenshot_20260606_201328_Control%20Home.jpg"><img src="Screenshot_20260606_201328_Control%20Home.jpg" width="180" alt="ControlHome screenshot 12"></a></td>
  </tr>
</table>
