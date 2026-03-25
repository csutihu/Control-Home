# ControlHome – Developer Deep Dive

## 🧠 Architecture Overview

ControlHome uses a modern mobile architecture:

- MVVM (ViewModel + StateFlow)
- Global device state store
- WebSocket-first updates
- Cache-first startup

---

## 🔄 Data Flow

Startup:
Cache → UI → HTTP refresh → WebSocket sync

Runtime:
WebSocket → Global Store → UI recomposition

---

## 🌐 WebSocket Model

- Single global WebSocket connection
- Not tied to screens
- Updates ALL devices in memory

### Benefits:
- Linked Tile Colors works everywhere
- No UI inconsistencies
- No per-screen subscriptions

---

## 🗂 Global Device Store

All devices stored in memory:

- Favorites
- Rooms (Plans)
- Categories

Used by:
- Favorites screen
- Rooms screen
- Devices screen
- Linked Tile Colors

---

## 🎨 Linked Tile Colors

### Concept:
UI-level override system

- Target device → visual tile
- Status device → condition source

### Flow:
Status change → Global store → UI recomposition → background override

### Key point:
No backend change required

---

## 🏠 Rooms Implementation

- Uses getplans + device mapping
- Header:
  - image OR color
  - adaptive text contrast

---

## 🧩 Devices Implementation

- Built from categories
- Category header:
  - color override
  - navigation

---

## ⚙️ Settings System

Uses DataStore:

- Room visuals
- Category visuals
- Linked rules

---

## 🎛 Color System

- HSV based
- Opacity included
- Shared logic across app

---

## ⚡ Performance

- Cache-first load (~0.5s)
- Minimal HTTP
- WS-driven updates

---

## 🧪 UI System

- Compose + Material3
- Tile-based layout
- Override layers for visuals

---

## 🔮 Future Ideas

- Condition-based rules (> < =)
- Per-device customization
- Tablet UI
