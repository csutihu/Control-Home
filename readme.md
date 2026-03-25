# ControlHome

## 🚀 Overview
ControlHome is a fast, mobile-first Domoticz client focused on **real-time control**, **clarity**, and **usability**.

Design principles:
- ⚡ WebSocket-first (real-time updates)
- 🚀 Cache-first startup (instant UI)
- 🎯 Optimistic UI (fast interaction feedback)
- 🎨 Highly customizable tiles

## Feedback

If you find a bug or have a feature request, please open an Issue.

---

# 🧭 Main Screens

## ⭐ Favorites
Default landing screen.

### Features
- Uses Domoticz favorites order
- Real-time updates (WebSocket)
- Instant UI feedback on tap

---

## 🏠 Rooms

### Navigation
- Swipe between rooms
- Arrow navigation also available

### Room Source
- Based on Domoticz **Plans (planid)**

⚠️ If a room name contains `_`, it is **ignored**

---

### Room Header (Top Tile)

Each room has a large customizable tile:

#### Menu (3 dots)
- **Set image** → background image
- **Set color** → background color
- **Set default** → reset

#### Visual Behavior
- Image OR color background
- Automatic text/icon color adaptation
- Semi-transparent overlay for readability

---

## 🧩 Devices

### Categories
Based on Domoticz device filters:
- All Devices
- Switches (light)
- Temperatures (temp)
- Utilities (utility)
- Weather (weather)

---

### Category Header

- Larger tile for category
- 3-dot menu:
  - Set color
  - Set default
- Swipe or arrows to navigate

---

### 🔍 Search

Each category includes a search field:

- Filters by device name
- Partial match supported
- Fast and responsive
- Accent-insensitive

---

# 🔘 Tiles

## Supported Types

- Switch (on/off)
- Dimmer (slider)
- Selector (multi-state)
- Sensor (temp, humidity, etc.)
- Contact sensor (open/closed, no toggle)

---

# 🔗 Linked Tile Colors

## Purpose
Allows a tile’s background color to be controlled by another device.

---

## Example Use Cases

### 🚪 Garage Door
- Button → trigger
- Sensor → real state

Result:
- Red when open
- Default when closed

---

### 🔥 Heating
- Thermostat tile reflects heating state
- Based on controlled device status

---

## How It Works

1. Select target device (tile)
2. Select status device
3. Define condition (ON/OFF)
4. Assign color

---

## Important

- Status device **does NOT need to be favorite**
- Works globally via WebSocket
- Instant updates

---

# 🎛 Custom Colors

Available in:
- Rooms
- Device categories

### Features
- HSV color picker
- Opacity slider (0–100%)
- Default ≈ 60% transparency

---

# ⚙️ Settings

## Server
- Domoticz URL
- Credentials

---

## Background
- Global background (color/image)
- Blur & dim options

---

## Tile Appearance
- Tile colors (base/on/off)
- Icon colors
- Font sizes
- Corner radius

---

## Linked Tile Colors
- Configure device relationships
- Define conditions and colors

---

## 🔘 Quick Settings (QS)

Android Quick Settings integration.

### Limits
- Max **6 devices**

---

### Supported Devices
- Switch
- Dimmer
- Selector

---

### Behavior

#### Tap
- Switch → toggle
- Dimmer → on/off
- Selector → default/next

---

#### Long Press
- Opens popup:
  - Dimmer → slider
  - Selector → options list

---

### Data Handling
- Uses HTTP refresh on open
- Uses cached state
- No background WebSocket

---

# 🎨 UI Behavior

## Status Bar
- App background extends under status bar
- Status bar color adapts to background

---

## Dynamic Text Color
- Text/icons adapt automatically:
  - Background color
  - Image brightness

---

# ⚡ Performance

- Cache-first startup (~0.5s)
- Immediate UI display
- WebSocket-driven updates
- Minimal HTTP usage

---

# 🧠 Architecture (Simplified)

WebSocket → Global Device Store → UI

- One shared state
- No duplicated data
- Instant updates across all screens

---

# 💬 Summary

ControlHome is a modern Domoticz client that provides:

- ⚡ Real-time experience
- 🎨 Deep customization
- 📱 Clean mobile UI
- 🧠 Smart data handling

It goes beyond a basic dashboard and becomes a powerful smart home control interface.

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
