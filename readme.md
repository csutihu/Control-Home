# 📱 ControlHome

A modern, minimalista, WebSocket-first Android kliens a Domoticz rendszerhez.

ControlHome egy gyors, mobilra optimalizált, MVVM alapú Android alkalmazás, amely valós idejű (WebSocket) kommunikációval csatlakozik a Domoticz szerverhez.

---

## 🚀 Fő jellemzők

- ⚡ Cache-first betöltés (gyors indulás)
- 🔄 WebSocket-alapú élő frissítés
- 📱 Mobil UX optimalizálás (Favorites-first szemlélet)
- 🧠 Egységes architektúra (Home / Favorites / Room konzisztens működés)
- 🔌 Domoticz API kímélése (minimális polling)
- 🧩 MVVM + StateFlow architektúra
- 🎨 Jetpack Compose + Material 3 UI

---

## 🏗 Architektúra

Az alkalmazás MVVM mintát követ:

- **ViewModel**
- **Repository**
- **StateFlow**
- **Unidirectional Data Flow**

Kommunikáció:

- REST API (Retrofit)
- WebSocket (`/json` endpoint, `domoticz` subprotocol)
- Snapshot + Stream modell

---

## 🔌 WebSocket működés

### Állapotgép

- `Disconnected`
- `Connecting`
- `Syncing`
- `Active`
- `RetryWait`
- `Suspended`

### ActiveGroup rendszer

| Képernyő | ActiveGroup |
|----------|------------|
| Favorites | Favorites |
| Room | Room(planId) |
| Home | None |
| Settings | None |

### Snapshot + Stream modell

Belépéskor:

1. WebSocket connect
2. Syncing állapot
3. HTTP snapshot
4. Snapshot OK → Active
5. Ezt követően push update-ek

---

## 🧠 Cache stratégia

| Típus | Cache |
|-------|-------|
| Favorites | JSON payload cache (hash + TTL) |
| Home | Plans + Devices cache |
| Room | PlanID alapú cache |

TTL: 5 perc

---

## 📂 Projekt struktúra

nav/
vm/
repo/
ws/
ui/
settings/

---


### Fontosabb részek

- `DomoticzWebSocketManager.kt` → WS state machine
- `DomoticzRepository.kt` → REST + cache
- `HomeViewModel.kt` → Room WS + bulk kezelés
- `FavoritesViewModel.kt` → Favorites WS kezelés
- `SettingsStore.kt` → DataStore alapú beállításkezelés

---

## 🧩 Támogatott eszköztípusok

- Toggle
- Dimmer
- Selector
- Thermostat
- Sensor
- SecurityPanel

---

## 🔐 Security kezelés

- SecStatus váltás
- PIN fallback
- Pending protection (2s window)

---

## ⚙️ Beállítások

### Server Settings

- Host
- Port (dinamikus, nem fix)
- SSL
- Allow self-signed
- Requires login
- Username / Password

### Background Settings

- Default
- Color
- Image
- Blur
- Dim overlay

---

## 🏁 Állapot

ControlHome jelenleg:

- Stabil WebSocket működés
- Partial UI update optimalizálás
- Cache-first architektúra
- Minimal polling
- Mobilra optimalizált

Ez már nem polling dashboard, hanem valódi push-alapú kliens.

---
## 📸 Screenshot

<img src="Screenshot_20260216_221609_Control Home.jpg" width="350"/>




