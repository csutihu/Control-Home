ControlHome – Architecture
🧱 High-Level Architecture
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
This architecture is deliberately shaped around fast startup, shared state, preserved Domoticz ordering, and minimal duplicated logic.
---
🔄 Data Layers
1. API Layer
Responsibilities:
Talk to Domoticz HTTP endpoints
Manage WebSocket connection where applicable
Provide raw DTOs and WS payloads
Key properties:
Retrofit + OkHttp
HTTP is used for bootstrap, refresh, commands, Scenes, and User Variables
WebSocket is used for supported real-time device domains only
Scenes / Groups and User Variables are HTTP-only
---
2. Repository Layer
Responsibilities:
Normalize Domoticz data
Merge category-based device results into a consistent bootstrap model
Preserve ordering
Apply WS patches to the current snapshot
Seed cache and QS state
Maintain extra HTTP-only collections such as Scenes and User Variables inside the same shared cache model
Important rule:
Repository owns the canonical ordering logic
UI must not reorder unless explicitly implementing a local visual override feature
---
3. Shared Bootstrap / Cache Model
This is the effective shared in-memory model used throughout the app.
Contains:
Device inventory
Per-plan device mapping
Category membership index
Favorite order baseline
Scenes list
User Variables list
Current bootstrap state
Why this matters:
Favorites, Rooms, Devices, and Linked Tile Colors depend on the same truth base
Scenes/Variables can be integrated without pretending they are normal WS-backed devices
Cache-first startup still works while allowing HTTP-only side collections
---
4. ViewModel Layer
Responsibilities:
Translate repository/bootstrap data into UI models
Expose StateFlow-based screen state
Handle optimistic UI updates
Apply fallback HTTP refresh where WS cannot confirm in time
Handle HTTP-only categories lazily
ViewModels currently of interest:
FavoritesViewModel
RoomsViewModel
DevicesViewModel
Important update:
Favorites now merges Domoticz-favorite normal devices with Domoticz-favorite Scenes / Groups
DevicesViewModel now owns lazy HTTP loading for Scenes / Groups and User Variables
---
5. UI Layer
Responsibilities:
Stateless rendering
Consume ViewModel state only
Apply presentation-only transforms such as filtering/search
Apply visual overrides such as linked tile colors and custom header visuals
Provide local reorder interaction while keeping Domoticz order as baseline
Important rule:
UI filters may reduce visible items but must not mutate the canonical order model
---
🧠 State Management
ControlHome uses:
StateFlow
cache-first bootstrap
global WebSocket-backed state where supported
explicit HTTP-only side state for Scenes / Groups and User Variables
Design target:
Single source of truth
No duplicated mutable screen models
No screen-specific reimplementation of device state
---
🔄 Ordering Strategy
Current design rule
Domoticz order = default app order
This now applies consistently to:
Favorites
Rooms
Devices categories
Scenes category
User Variables category
Why this matters
Local tile reorder depends on this invariant:
default = Domoticz order
local override = ControlHome-only custom order
reset = back to Domoticz order
Corrected approach
Ordering is now preserved end-to-end:
API result order
repository merge order
ViewModel mapping order
screen rendering order
---
🌐 WebSocket Architecture
Design
Single global WebSocket connection
Shared across app features
Not tied to one screen’s local lifecycle model
WS states
Typical states include:
Disconnected
Connecting
Syncing
Active
Suspended
ActiveGroup concept
The app scopes WS behavior using an `ActiveGroup` abstraction.
Examples:
Favorites
Room(planId)
Devices(category)
None
Important limitation
Scenes / Groups and User Variables are not part of the WS model.  
They remain explicit HTTP-driven categories.
This is architecturally important because it avoids pretending those domains have real-time guarantees they do not have.
---
📦 Bootstrap Snapshot Architecture
The bootstrap snapshot is a critical foundation.
It combines:
Domoticz plans
Domoticz favorites ordering metadata
Domoticz category-based device responses
cached or refreshed Scenes / Groups collection
cached or refreshed User Variables collection
The result becomes a unified `BootstrapData` model containing:
plans
devicesByIdx
devicesOrdered
devicesByPlan
categoryIndex
favoriteOrder
scenes
userVariables
This bootstrap model is then:
cached
published
transformed into HomeData and UI state
---
🏠 Rooms Architecture
Rooms are based on:
plans from Domoticz
per-plan device mapping using `PlanID` / `PlanIDs`
Important architectural rules:
rooms with `_` in name are excluded
room ordering follows Domoticz plan order
room device ordering follows Domoticz device order
room visual customization is stored locally only
Current room customization scope:
image
header color
reset to default
Implementation updates:
room image selection uses persistent document access
room header presentation is mutually exclusive: image or color
image and color state transitions are explicitly normalized
---
🧩 Devices Architecture
Devices view is built from category-based data plus HTTP-only side collections.
Sections:
All
Switches
Scenes
Temperatures
Utilities
Weather
User Variables
Important architectural rules:
standard device category membership comes from Domoticz filter responses
Scenes / Groups come from `getscenes`
User Variables come from `getuservariables`
ordering follows Domoticz/bootstrap order
search filters only visible items
no alphabetical resort in UI
Scenes / Groups
HTTP-only
lazy-loaded on entry
command result confirmed by HTTP refresh
Scene sends only `On`
Group sends `On` / `Off`
User Variables
HTTP-only
lazy-loaded on entry
read-only
rendered as sensor-style UI tiles
---
⭐ Favorites Architecture
Favorites remains special because:
it is the default landing screen
it is the most performance-sensitive screen
Domoticz favorites order is meaningful to users
Architectural rule:
favorite membership belongs to Domoticz
favorite ordering baseline belongs to Domoticz
local reorder is an app-only override layer
Important update:
Favorites now supports Domoticz-favorite Scenes / Groups in addition to normal favorite devices
This keeps the ControlHome default Favorites view closer to real Domoticz behavior
---
🎨 UI Override Layer
Linked Tile Colors
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
does not mutate the underlying device model
does not require backend changes
works because screens consume shared state
Header Visual Customization
Also a UI-only override system:
Room header image/color
Category color
Again:
persistent locally
not written back to Domoticz
Reorder UI
The three major views now share a common reorder component model:
shared reorder menu button
shared reorder action buttons
shared drag/drop reorder grid
This reduced divergence between Favorites, Rooms, and Devices while preserving local screen-specific behavior.
---
⚙️ Settings Architecture
Settings are split into two storage domains:
1. DataStore / SettingsStore
Stores non-sensitive app state:
server host/port flags
background settings
tile appearance
linked tile color rules
room/category visual settings
caches
reorder overrides
QS state cache
connection revision / reset helpers
2. SecretStore
Stores sensitive secret data:
password
This split is now a structural requirement.
---
🔐 Security, Backup, and Connection Reset
A key improvement in recent iterations is backup-safe secret handling plus stronger connection reset behavior.
SecretStore hardening
encrypted prefs can become invalid after reinstall/restore
app now recovers by forcing password re-entry instead of crashing
secret prefs remain excluded from backup/restore
Connection reset hardening
When server settings change:
connection revision is bumped
caches can be invalidated where needed
WS is explicitly reset
navigation/view model keys use connection-sensitive fields
This reduces the chance of the app continuing to talk to a previously active Domoticz server after settings were changed.
---
⚠️ Error Handling Architecture
Current app-state model distinguishes:
Password missing
Unauthorized
Server not available / offline
Implemented UX direction:
Password missing → navigate to Server Settings
Unauthorized → navigate to Server Settings
Server unavailable → navigate to Server Settings with clear message
Cache-first startup remains important for perceived continuity
This split is important because “UI is visible” does not automatically mean “current connection is healthy”.
---
⚡ Performance Strategy
Performance is based on a few deliberate principles:
Cache-first startup
read cached snapshot
draw immediately
refresh afterwards
WebSocket-first runtime where supported
avoid repeated polling
use fallback refresh only when necessary
Explicit HTTP-only categories
Scenes / Groups and User Variables do not burden the general WS path
they are loaded when needed
Minimal recomposition
keep UI state narrow
patch only affected devices where possible
QS independence
QS must work without depending on active foreground WS
---
🧪 Testing Strategy Guidance
Ordering regressions
Check:
repository merge order
ViewModel mapping
screen-level sort/filter logic
reset order paths for Favorites / Rooms / Devices
SecretStore regressions
Test:
reinstall
password deletion
backup/restore-like scenarios
WS regressions
Check:
ActiveGroup transitions
foreground/background lifecycle
snapshot + WS sync interplay
HTTP-only category regressions
Check separately:
Scenes / Groups first load
command-after-refresh behavior
User Variables lazy load
Offline/auth states
Check separately:
password missing
unauthorized credentials
server unavailable
---
🔮 Extensibility
The current architecture is well positioned for future growth.
Good candidates:
multi-server profiles
diagnostics/logs screen
richer linked tile conditions
device-level visual customization
camera preview surfaces
history / chart views
Important rule for extending:
protect the invariants:
Domoticz order is baseline
UI overrides stay local unless explicitly server-writing
cache-first startup remains intact
secret data remains separated from normal settings
HTTP-only categories remain explicit
---
✅ Architectural Summary
ControlHome currently stands on a solid architectural foundation because it now has:
shared bootstrap/cache state
global WS-backed consistency where Domoticz supports it
explicit HTTP-only side domains for Scenes / Groups and User Variables
clean separation between sensitive and non-sensitive storage
preserved Domoticz ordering
shared reorder infrastructure
UI-only override architecture for advanced visual features
That combination makes the app both fast today and realistically extensible tomorrow.
