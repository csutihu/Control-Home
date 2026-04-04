# ControlHome

## 🚀 Overview
ControlHome is a fast, mobile-first Domoticz client focused on:

- ⚡ Real-time control where Domoticz supports it
- 🚀 Instant startup with cache-first rendering
- 🎯 Optimistic UI for fast interaction
- 🎨 Deep UI customization for tiles, headers, and backgrounds

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
- Redirects to Server Settings when the app determines the server is unavailable
- Cache-first behavior still remains important for perceived continuity

## Current development status
- Broken encrypted prefs crash is fixed
- Missing password and unauthorized flows are implemented
- Server unavailability handling is integrated with Settings navigation

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

# Screenshots

<p align="center">
  <a href="Screenshot_20260325_224451_Control Home.jpg"><img src="Screenshot_20260325_224451_Control Home.jpg" width="30%" /></a>
  <a href="Screenshot_20260325_224839_Control Home.jpg"><img src="Screenshot_20260325_224839_Control Home.jpg" width="30%" /></a>
  <a href="Screenshot_20260325_224530_Control Home.jpg"><img src="Screenshot_20260325_224530_Control Home.jpg" width="30%" /></a>
  <a href="Screenshot_20260325_224541_Control Home.jpg"><img src="Screenshot_20260325_224541_Control Home.jpg" width="30%" /></a>
  <a href="Screenshot_20260325_224600_Control Home.jpg"><img src="Screenshot_20260325_224600_Control Home.jpg" width="30%" /></a>
  <a href="Screenshot_20260325_224617_Control Home.jpg"><img src="Screenshot_20260325_224617_Control Home.jpg" width="30%" /></a>
  <a href="Screenshot_20260325_224642_Control Home.jpg"><img src="Screenshot_20260325_224642_Control Home.jpg" width="30%" /></a>
</p>
