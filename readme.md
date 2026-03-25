# ControlHome

## 🚀 Overview
ControlHome is a fast, mobile-first Domoticz client designed for real-time control and high usability.

Built with:
- WebSocket-first architecture
- Cache-first startup
- Jetpack Compose UI

---

## ✨ Key Features

### ⚡ Real-Time Updates
- WebSocket-based global device state
- Updates all devices regardless of screen
- No unnecessary polling

---

### 🧠 Smart UI Architecture
- Unified device state (global store)
- Optimistic UI (instant feedback)
- Automatic refresh after startup (~0.5s)

---

### 🎨 Linked Tile Colors
Control one tile’s background using another device’s state.

**Use cases:**
- Garage door: trigger + real state
- Heating: active / inactive indication

**How it works:**
- Status device (sensor)
- Target device (tile)
- ON/OFF → color override

---

### 🏠 Rooms Screen
- Swipe between rooms
- Room header supports:
  - Set image
  - Set color
  - Set default
- Adaptive text color based on brightness

---

### 🧩 Devices Screen
- Categories:
  - All
  - Switches
  - Temperatures
  - Utilities
  - Weather
- Category header:
  - Set color
  - Set default
  - Navigation arrows

---

### 🎛 Custom Color System
- HSV color picker
- Opacity slider (0–100%)
- Default ≈ 60% transparency
- Consistent across Rooms and Devices

---

### ⭐ Favorites
- Fast tile-based view
- Order from Domoticz favorites
- Real-time updates

---

### ⚙️ Settings

#### Server
- Domoticz URL
- Credentials

#### Background
- Color or image
- Blur + dim options

#### Tile Appearance
- Tile colors (on/off/base)
- Icon colors
- Font sizes

#### Linked Tile Colors
- Add rules:
  - Target device
  - Status device
  - Trigger value
  - Color

#### Quick Settings (QS)
- Up to 6 tiles
- Direct device control
- HTTP refresh on open

---

## ⚡ Performance

- Cache-first startup
- < 0.5s initial UI
- Minimal HTTP usage
- WebSocket-driven updates

---

## 🧱 Architecture

### Data Flow
WebSocket → Global Store → UI

### API
- getdevices (categories)
- getplans (rooms)
- favorites (ordering)

---

## 🛠 Developer Notes

- MVVM + StateFlow
- Retrofit + OkHttp
- Compose + Material3
- DataStore for settings

---

## 🔮 Roadmap

- Advanced Linked Tile conditions
- Per-device customization
- Tablet layout
- UI animations

---

## 💬 Summary

ControlHome provides a fast, flexible, and modern Domoticz experience with real-time updates and powerful UI customization.
