# Changelog

## [v0.4.2] - 2026-11-06

### Fixed
- **Prevent Stale `Room.OverallMotion` from Locking Wireless Sensor Motion `on` (`rooms.py`, `merge.py`)**:
  - Removed the `RoomOverallMotion` fallback from `ResideoAccessory.motion` because the REST `/group/0/rooms` `Rooms[i].OverallMotion` field lags behind per-accessory `Sensor` push events (`OccupancyDet: false`), which previously prevented `binary_sensor.*_motion` from clearing when `OccupancyDet` turned `off`.
  - `ResideoAccessory.motion` on `Model 4354` sensors now strictly tracks live `MotionDet || OccupancyDet`, and live `Rooms` push events synchronize both `MotionDet` and `OccupancyDet`.

## [v0.4.1] - 2026-11-06

### Changed
- **Promote Room Priority Switches & Priority Mode Selector to Controls (`switch.py`, `select.py`)**:
  - Removed `EntityCategory.CONFIG` from `ResideoRoomPrioritySwitch`, `ResideoThermostatRoomPrioritySwitch`, and `ResideoPriorityTypeSelect` so they appear under **Controls** (instead of **Configuration**) on device pages and are automatically surfaced in Home Assistant room/area overview cards.

## [v0.4.0] - 2026-11-06

### Added
- **Room Priority Control (`switch` & `select`)**:
  - Added per-room priority switches (`switch.*_room_priority`) on every wireless room sensor sub-device and a built-in thermostat room priority switch (`switch.*_include_thermostat_room_in_priority`) to dynamically include or exclude rooms from temperature averaging (`SelectedRooms`).
  - Added a thermostat Priority Mode selector (`select.*_priority_mode`) to switch between `PickARoom` and `FollowMe`.
  - Added shared optimistic override coalescing (`priority_override`) in `ResideoDataUpdateCoordinator` so back-to-back room toggles compose into a single coherent `/priority` payload without clobbering each other during the cloud reconciliation window.
  - Added `sensor.*_active_priority_rooms` displaying the human-readable list of rooms currently participating in priority averaging.
- **EIM Duct Temperature Probes (`sensor`)**:
  - Exposed **Discharge duct temperature** (`DischargeTemperature`), **Return duct temperature** (`ReturnTemperature`), and **Duct temperature delta ($\Delta T$)** (`DischargeTemperature - ReturnTemperature`) when probes are wired to an Equipment Interface Module (EIM).
  - Added live SignalR stream merging (`_merge_duct_temperature`) for `DuctTemperature` events (`DischargeTemp` / `ReturnTemp`).
- **Whole-Home Humidifier & Dehumidifier Entities (`binary_sensor` & `sensor`)**:
  - Exposed live running binary sensors for **Humidifier** (`HumidifierRequest`) and **Dehumidifier** (`DehumidifierRequest`), plus **Dehumidification away active** (`DehumAway.Active`).
  - Exposed target setpoints (`HumidifierSetpoint`, `DehumidifierSetpoint`), **Humidifier deadband**, **Dehumidifier away setpoint**, **Dehumidifier away cool setpoint**, and **Windows protection** (frost level `1–10` and condensation protection percentage).
  - Added live SignalR stream merging (`_merge_humidification`) for `Humidification` push events.
- **HVAC Stage & Equipment Diagnostics (`binary_sensor` & `sensor`)**:
  - Exposed individual stage running binary sensors (`stage_1_active` through `stage_4_active` from `HeatAndCoolDemand.StagesOn[0..3]`), **Active demand mode** (`HeatAndCoolDemand.Mode`), **Active HVAC stages**, and **EIM status** (`Model 4353`).
- **Local Wall-Unit vs. Priority Averaged Readings (`sensor`)**:
  - Exposed **Thermostat local temperature**, **Thermostat temperature (actual)** (3-decimal precision thermistor reading), and **Thermostat local humidity** directly from the wall unit (`AccessoryId: 0`), distinct from the multi-room priority average (`DisplayedIndoorTemperature`).
- **Alert Thresholds & Maintenance Diagnostics (`sensor`)**:
  - Exposed **High/Low temperature alert thresholds** (`FaultThresholds.TemperatureHighThreshold` / `TemperatureLowThreshold`), **High/Low humidity alert thresholds** (`HumidityHighThreshold` / `HumidityLowThreshold`), **Air filter life** (`RestartDays`), **Backlight intensity**, and **Last connectivity change** timestamp.

### Fixed
- **Wireless Room Sensor (`Model 4354`) Motion & Occupancy Detection (`merge.py`, `rooms.py`)**:
  - Fixed off-by-one SignalR `Sensor` push routing on systems with a RedLINK Equipment Interface Module (`AccessoryId: 1`, `Type: "EI"`), where the first wireless room sensor (`AccessoryId: 2`) arrives under header `AccessoryId: 1`.
  - Fixed `binary_sensor.*_motion` remaining permanently `off` on `Model 4354` wireless room sensors (which report `MotionDet: false` while toggling `OccupancyDet: true` and room-level `OverallMotion: true`) by falling back to `OccupancyDet` / `RoomOverallMotion`.
  - Propagated room-level `OverallMotion` changes from `LiveFeed` `Rooms` events down to child accessories.
