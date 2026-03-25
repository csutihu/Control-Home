# ControlHome – Architecture

## 🧱 High-Level Architecture

```
Domoticz API
    │
    ├── HTTP (initial load)
    └── WebSocket (real-time)
            │
            ▼
    Global Device Store
            │
            ├── Favorites
            ├── Rooms
            ├── Devices
            └── Linked Overrides
            │
            ▼
         UI (Compose)
```

---

## 🔄 Data Layers

### 1. API Layer
- Retrofit + OkHttp
- HTTP + WebSocket

### 2. Data Layer
- Global device state
- Cached data

### 3. UI Layer
- Compose screens
- Tile rendering

---

## 🧠 State Management

- StateFlow
- Single source of truth

---

## 🎨 UI Override Layer

Linked Tile Colors:

```
Device state
   ↓
Rule engine
   ↓
Tile background override
```

---

## ⚙️ Settings Flow

DataStore → Flow → UI

---

## 📱 Screens

- Favorites
- Rooms
- Devices
- Settings

---

## ⚡ Performance Strategy

- Cache-first
- WS updates
- Minimal recomposition

---

## 🔮 Extensibility

- Easy to add new tile types
- Extend override system
- Add new conditions
