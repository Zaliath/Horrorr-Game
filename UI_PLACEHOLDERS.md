# UI Placeholders & Deferred Pixel Overhaul Tracker

This document tracks every UI screen, icon, widget, or procedural control that is currently **deferred**, **using a temporary `AssetCatalog` placeholder**, or **awaiting dedicated pixel art** so the next UI overhaul pass knows exactly what needs custom artwork or wiring.

> **Source Catalog:** `ReplicatedStorage.Assets.PixelUI.AssetCatalog` (59 total image keys: 43 shared Lobby base assets + 16 Group-owned `382676491` Main-Game additions) and [`Assets/UI/PixelCRT/manifest.json`](Assets/UI/PixelCRT/manifest.json).  
> **Rule Reminder:** Whenever you add a placeholder UI element or replace one with final pixel art, update this file (see [`GEMINI.md`](GEMINI.md), [`CLAUDE.md`](CLAUDE.md), and [`README.md`](README.md)).

---

## 1. Deferred Place Wiring (Pixel Templates Ready, Controllers Pending)

| Place | Studio Template Paths | Current State | Work Needed |
| --- | --- | --- | --- |
| Lobby (`THE horror driving game`, `placeId: 124598324996118`) | `StarterGui.PixelLobby_TopBar`, `PixelLobby_Garage`, `PixelLobby_Items`, `PixelLobby_Emotes`, `PixelLobby_Dossier`, `PixelLobby_BayHUD`, `PixelLobby_Settlement`, `PixelLobby_Notification` | All 8 `PixelLobby_*` templates and `ReplicatedStorage.Assets.PixelUI` are built in Studio | Wire Lobby client controllers to bind to `StarterGui.PixelLobby_*` templates and `UITheme` pixel helpers instead of procedural GUIs |

---

## 2. Missing Custom Pixel Art Using Temporary Catalog Fallbacks (`The game part`, `placeId: 93445054108906`)

| UI Element | Studio Hierarchy / Path | Controller / Owner | Temporary Fallback Used | Needed Custom Art for Next UI Pass |
| --- | --- | --- | --- | --- |
| 12 Monster Bestiary Portraits | `StarterGui.PixelGame_WristPDA.BottomRightDock.Body.Page_Guide.GuideList.EntryTemplate.PortraitFrame.Icon` (and Lobby `PixelLobby_Dossier`) | `WristMenuController` | Shares `Icon_MonsterUnknown`, `Icon_Skull`, and `Tile_DitherStatic` | 12 unique 32×32 dithered pixel monster portraits (one per monster) |
| Secondary Monster Threat HUDs | Runtime `PlayerGui.ForestHunterHUD`, `StareMonsterHUD`, `PorchMonsterHUD`, `HitchhikerHUD` | `ForestHunterController`, `StareMonsterController`, `PorchMonsterController`, `HitchhikerController` | Separate runtime `ScreenGui`s instead of unified `PixelGame_MonsterAlerts` banners | Consolidate remaining secondary monster prompts/alerts into `StarterGui.PixelGame_MonsterAlerts` |

---

## 3. Procedural / Non-Pixel UI Elements Temporarily Retained (`The game part`)

| UI Element | Studio Hierarchy / Path | Controller / Owner | Current Implementation | Planned Pixel Replacement |
| --- | --- | --- | --- | --- |
| Wrist PDA Admin Console Controls | `StarterGui.PixelGame_WristPDA.BottomRightDock.Body.Page_Admin` (`AdminCommandScroll`, `AdminDropdownOverlay`, `AdminViewSwitchBtn`, `Persp_1..3`) | `WristMenuController` | Clears `CmdRow_Template` and generates procedural `UITheme.WireButton` dev/admin cards dynamically | Migrate admin command cards and dropdown rows to pixel templates (using `CmdRow_Template` / `OptionTemplate` image styling) |
| Custom Proximity Prompts | Runtime `BillboardGui`s managed by `PromptUIController` | `PromptUIController` | Circular modern keybadge + dual-arc hold ring + rounded pill action text | Replace with 16-bit square pixel keycap frame + segmented pip hold bar matching `PixelGame_MonsterAlerts.BottomAlertStack` |
| Hotbar Mini-Bar Fills | `StarterGui.PixelGame_Hotbar.HotbarRoot.Slots.Slot_1..4.FillTrack.Fill` | `HotbarController` | Flat `Frame` fill bar inside pixel card | Replace with micro segmented pixel pips (`Pip_*`) or pixel-bordered progress slice |
| Flip Van Cooldown Overlay | `StarterGui.PixelGame_Speedometer.FlipVanContainer.FlipVanBtn.CooldownFill` | `VanController` | Flat semi-transparent cooldown overlay `Frame` | Replace with pixel dither cooldown sweep / pip countdown indicator |
| Parcel Damage Crack Overlays | `StarterGui.PixelGame_ShiftHUD.PackageConditionHUD.WireframeParcel.Crack1` & `Crack2` | `ShiftController` | Rotated flat `Frame` diagonal lines | Swap in dedicated pixel crack overlays over `Icon_ParcelWireframe` |
| Remote Van Feed Mini Boost Fill | `StarterGui.PixelGame_WristPDA...LiveVanFeedContainer.MiniTelemetryBox.MiniBoostTrack.MiniBoostFill` | `WristMenuController` | Flat `Frame` fill bar inside `MiniBoostTrack` | Replace with micro segmented `Pip_*` row |

---

## 4. Intentionally Procedural / Diegetic / Reverted Elements (Keep As-Is)

These layers are fullscreen optical/camera VFX, 3D world-space diegetic monitors, or smooth calibrated / procedural terminal layers intentionally kept or reverted per user design and should **not** be replaced with 2D pixel frame or pip templates:
- **Reverted P.A.R.A.L.L.A.X. Procedural CRT Terminal Death Screen (`DeathScreenController` -> `PlayerGui.DeathScreenGui`):**
  - Intentionally reverted from `StarterGui.PixelGame_DeathScreen` (removed from `StarterGui`) back to the procedural CRT `DeathScreenGui` (`Phase0Intro` oscilloscope flatline, `Phase1Terminal` PowerShell typewriter, 4-quadrant `Phase2Overview` with responsive `Scale` layout + `UITextSizeConstraint`, and `SpectatorOverlay`).
- **Reverted Smooth Calibrated Gauge & Progress Bars (Keep Pre-Overhaul Style):**
  - `StarterGui.PixelGame_Stamina.StaminaBarContainer.BarTrack` (`StaminaFill`, `DrainGhost`, `SliderThumb`, `RecoveryTick` — `SprintController`)
  - `StarterGui.PixelGame_Speedometer.SpeedoPanel.BoostRow.BoostTrack` (`BoostFill`, `BoostThumb`) & `FuelRow.FuelTrack` (`FuelFill` — `VanController`)
  - `StarterGui.PixelGame_WristPDA.BottomRightDock.Body.Page_Status.ECGCard.HpTrack` (`HpFill` — `WristMenuController`)
  - `StarterGui.PixelGame_MonsterAlerts.TopAlertStack.ChaseBanner.MeterBg` (`Fill` — `ChaseMonsterController`)
  - `StarterGui.PixelGame_MonsterAlerts.BottomAlertStack.DoorMashHUD.MashBarFrame` (`MashFill` — `DoorMonsterController`)
- **Fullscreen screen-space vignettes, flashes, and cinematics:** `WindowMonsterFX`, `ChaseMonsterFX` (`RearVignette`, `RamFlash`, `SwallowBlackout`), `DoorMonsterHUD` (`EdgeVignette`, `RipFlash`), `HorrorAtmosphereOverlay`, `LightningFlashGui`, and `ReviveCinematicGui`.
- **3D world-space `SurfaceGui`s:** `DeliveryVan.CabInterior.TrackerScreen` (`ShiftController` GPS radar) and `BrokenBorder_*` (`WindowMonsterController` shattered glass rim cracks).
