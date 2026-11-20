# Devlog

## 2026-11-19 — Antigravity
- **Summary of Changes:** Renamed `GAME DESIGN.md` to `GAME_DESIGN.md` to match the filename referenced in `GEMINI.md`, `CLAUDE.md`, and `README.md`. Added `.gitignore` for `.DS_Store` and `Thumbs.db` and untracked `.DS_Store`.
- **Studio Paths Modified/Created:** None (repository housekeeping only).
- **New Dependencies:** None.
- **Next Steps / Known Issues:** `GAME_DESIGN.md` is currently empty and ready to be populated with the core game design doc.

## 2026-11-19 — Riplatki (via Claude)
- **Summary of Changes:** Documentation only, no game code changed. Populated `GAME_DESIGN.md` with a draft based on the live Studio places (verified items only; TODO sections left for the team). Added the baseline of existing systems below so nobody duplicates them.
- **Studio Paths Modified/Created:** None.
- **Existing Systems Found (baseline, "The game part", placeId 93445054108906):**
  - Framework: `ServerScriptService.Server` (Bootstrap, Core/{ServiceList,Scheduler,Net}, Services/{DataService,MonetizationService,ReplicationService,AntiExploitService}), `ProfileStore`, `BanService`, `AntiCheat`, `Cmdr`.
  - Game scripts: `ServerScriptService.DeliveryChute`, `ServerScriptService.StartAreaShops`, `Workspace.DeliveryVan.VanDrive`, `StarterPlayer.StarterPlayerScripts.VanDriverClient`.
  - Shared: `ReplicatedStorage.Shared` (Loader, Enums, Types, Util, NetDefs, Config), `ReplicatedStorage.Packages` (Trove, Signal, SignalPlus, Janitor, DataServiceTyped).
- **New Dependencies:** None created. Already in use, do not recreate:
  - RemoteEvents (`ReplicatedStorage.Remotes`): `DevRemote`, `OpenShop` (server to client, arg: ShopId string).
  - CollectionService tags: `DeliveryBox` (spawned boxes), `ShopPrompt` (ProximityPrompts carrying a `ShopId` attribute).
  - ShopIds: `VanUpgrades`, `PersonalUpgrades`, `Weapons`, `Stamina`, `Gear`.
  - Attributes: DeliveryVan `MaxCargoBoxes`, `CargoBoxes`, `Scale`, `DriveSign`, `SteerSign`; DeliveryBox `InVan`.
  - Collision groups: `VanBody`, `VanWheel`.
  - DataStore keys: none found in game scripts (store names are presumably set inside `DataService`; not yet checked).
- **Next Steps / Known Issues:**
  - Shop UI is not built: nothing on the client listens for `OpenShop`.
  - Verify `VanDrive` does not anchor van-attached parts (Moving Van Physics Rule). Not checked yet.
  - `GAME_DESIGN.md` is a draft: the team must fill in the TODO sections (core loop, threats, economy, progression).
  - "THE horror driving game" (placeId 124598324996118) was only inspected at the ServerScriptService/ReplicatedStorage level; it has the same framework but not the van/start-area content.

## 2026-11-19 — Claude (second session)
- **Summary of Changes:** Session start sync (pulled, read docs, inspected live Studio via the Server datamodel because Studio was in Play mode). Merged the planned design from the project owner (core loop, procedural map, monsters, upgrades, co-op) into the existing GAME_DESIGN.md draft as "Planned" sections. No Studio changes were made.
- **Studio Paths Modified/Created:** None.
- **New Dependencies:** None created. Note: a Play-mode (Server) view showed extra remotes in `ReplicatedStorage.Remotes` (`ClientReady`, `PingSample`, `Notification`, and randomly named `VYUCVTYJIDVADSUTPCV`) that the Edit-mode baseline above does not list; they may be created at runtime. Verify in Edit mode before reusing or removing them.
- **Next Steps / Known Issues:** Inspect `Workspace.DeliveryVan` and `ForestRoadMap` in detail before building the procedural generator. Confirm what the `VYUCVTYJIDVADSUTPCV` remote is for before adding new remotes. Git is not on PATH on this machine; Claude used GitHub Desktop's bundled git (`%LOCALAPPDATA%\GitHubDesktop\app-3.7.0\resources\app\git\cmd\git.exe`), and the `app-3.7.0` folder name changes when GitHub Desktop updates.

## 2026-11-19 — Antigravity
- **Summary of Changes:** Built the seeded HorrorKit procedural map system and end-to-end delivery shift gameflow (`Prep` → `Active` → `Return` → payout & next shift), live in-cab `TrackerScreen` radar/address/ETA/speed updates, `ShiftHUD`, package grab/carry/drop/deliver flow, and fixed `AntiExploitService` to exempt seated van players (boost speed 140 > 120 cap). Also identified the previously mysterious runtime remote (`VYUCVTYJIDVADSUTPCV`) as `ServerScriptService.AntiCheat` renaming `ReplicatedStorage.Remotes.DevRemote` at runtime.
- **Studio Paths Modified/Created:**
  - `ReplicatedStorage.Shared.NetDefs` (modified)
  - `ServerScriptService.Server.Services.AntiExploitService` (modified)
  - `ServerScriptService.Server.Core.ServiceList` (modified)
  - `ServerStorage.Modules.ProceduralRoadGraph` (created)
  - `ServerStorage.Modules.ChunkBuilder` (created)
  - `ServerScriptService.Server.Services.MapService` (created)
  - `ServerScriptService.Server.Services.PackageService` (created)
  - `ServerScriptService.Server.Services.ShiftService` (created)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.ShiftController` (created)
  - `StarterPlayer.StarterPlayerScripts.Client.Core.ControllerList` (modified)
- **New Dependencies:**
  - Remotes (`NetDefs` / `ReplicatedStorage.Remotes`): `ShiftState` (S2C), `ShiftAction` (C2S: `"StartShift"`, `"DropPackage"`, `"ReturnToWarehouse"`, `"ResetVan"`).
  - Tags: `EmergencyBox` (auto-loaded shortfall cargo boxes, also tagged `DeliveryBox`).
  - Attributes: Player `CarryingPackage` (boolean); DropoffPad `StopHouseId` (number), `StopAddress` (string); DeliveryBox `DroppedBox` (boolean).
  - Runtime Folders/Parts: `Workspace.ProceduralMap`, `Workspace.StartAreaGround`, `ServerStorage.LegacyHandbuiltMap` (runtime archive of `ForestRoadMap`, `Houses`, `MapModels`).
  - `leaderstats`: `Cash`, `Deliveries`, `Shifts` (`IntValue`s).
- **Next Steps / Known Issues:**
  - Legacy `ForestRoadMap`, `Houses`, and `MapModels` remain intact in Edit mode and are archived to `ServerStorage.LegacyHandbuiltMap` only at runtime.
  - `HorrorKit_Houses` (`Default` CollisionFidelity) blocks stepping directly onto porches (~4.2 studs out); `DeliverPrompt` uses `MaxActivationDistance = 14` with `RequiresLineOfSight = false` so front-of-porch delivery works reliably.
  - Monster systems, Shop UI (`OpenShop`), and `DataService` persistent cash/upgrade saves remain to be hooked into `ShiftService` / `MapService`.

## 2026-11-19 — Antigravity (box pickup bounds & prompt filtering)
- **Summary of Changes:** Restricted van package pickups (`InVan == true`) so `PickupBoxPrompt` only appears when the player's `HumanoidRootPart` is physically inside `DeliveryVan.Cargo` (hidden outside the van and on the roof), enforced the same cargo-bounds check server-side in `PackageService`, and hid all `DeliveryBox` `PickupBoxPrompt`s on the client while the player is carrying a box (`CarryingPackage == true`) without disabling van door, seat, delivery, dispatch, or shop prompts.
- **Studio Paths Modified/Created:**
  - `ReplicatedStorage.Shared.Util.VanCargo` (created)
  - `ServerScriptService.Server.Services.PackageService` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.ShiftController` (modified)
- **New Dependencies:** None (uses existing `DeliveryBox` tag, `InVan`/`DroppedBox` attributes, and player `CarryingPackage` attribute).
- **Next Steps / Known Issues:** Verified in Play mode across all 5 scenarios (outside van, roof, inside cargo, carrying, and dropped/returned); Studio returned to `Edit` mode.

## 2026-11-20 — Antigravity (Cmdr admin commands & Backpack toggle)
- **Summary of Changes:** Added Cmdr admin commands (`fly`/`togglefly`, `unfly`, `setspeed`/`speed`/`ws`/`walkspeed`, `setcash`/`cash`/`setmoney`), added `ShiftService:SetCash` so cash edits sync both `leaderstats.Cash` and `ShiftHUD`, exempted flying players and server-set `WalkSpeed` values in `AntiExploitService`, disabled Roblox's default `Enum.CoreGuiType.Backpack` inventory UI so pressing `` ` ``/`~` opens Cmdr cleanly, and guarded `CmdrController` startup until `StarterGui.Cmdr` finishes replicating so `Window` never yields on missing GUI children.
- **Studio Paths Modified/Created:**
  - `ServerScriptService.Cmdr.Commands.fly` & `flyServer` (created)
  - `ServerScriptService.Cmdr.Commands.unfly` & `unflyServer` (created)
  - `ServerScriptService.Cmdr.Commands.setspeed` & `setspeedServer` (created)
  - `ServerScriptService.Cmdr.Commands.setcash` & `setcashServer` (created)
  - `ServerScriptService.Server.Services.ShiftService` (modified)
  - `ServerScriptService.Server.Services.AntiExploitService` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.CmdrController` (modified)
- **New Dependencies:**
  - Player attributes: `AdminFly` (boolean), `AdminFlySpeed` (number), `AdminWalkSpeed` (number).
- **Next Steps / Known Issues:** Verified in Play mode (`Backpack` CoreGui disabled, `setspeed`, `setcash`, `fly`, `unfly`) with zero console errors or warnings; Studio returned to `Edit` mode.

## 2026-11-20 — Antigravity (VanController, ShopService, ChuteService & Driving Feel Overhaul)
- **Summary of Changes:**
  - Converted standalone van client scripts (`VanDriverClient`, `VanFirstPerson`, `VanSeatSwitch`, `VanControls`) into a unified `Loader`-managed `VanController` with public API & signals (`OnSeatChanged`, `OnChaseCamChanged`, `GetVan`, `GetSeatedRole`, `IsDriving`, `IsInVan`, `IsChaseCamActive`, `SetChaseCamBlocked`, `GetSpeedStuds`, `GetSpeedMph`, `GetBoostPercent`, `SwitchSeat`, `ToggleHeadlights`, `ToggleCabLight`, `SetHorn`), preserving `VanStunned` support untouched.
  - Added GTA-style hold-`V` third-person chase camera in `VanController` (smooth spring blend, horizontal heading follow, optional RMB mouse look-around with auto-recenter, wall/ceiling occlusion raycast, clean restore to first-person on release).
  - Smoothed driving feel and fixed getting stuck at bad angles: 4-wheel + center ground checks, speed-sensitive non-linear steering curve (`38°` standstill → `14°` top speed), gentle engine coasting instead of harsh brake lock on throttle release, self-righting roll/pitch upright assist, low-speed crawl & curb-climb assist when wedged, raised `ChunkBuilder.GROUND_TOP_Y` (`0.20` → `0.86`, cutting off-road curb step from `1.26` to `0.60` studs), and made `Prop_DeadGrass_Cluster` and `Prop_TrashDebris_Pile` `CanCollide = false`.
  - Converted `ServerScriptService.StartAreaShops` → `ShopService` and `ServerScriptService.DeliveryChute` → `ChuteService` (registered in `ServiceList`, accessible via `Loader.Get(...)`) and removed the 6 replaced standalone scripts.
  - Updated `GEMINI.md`, `CLAUDE.md`, and `README.md` to standardize on `Loader`-managed Services/Controllers and avoid standalone scripts unless literally needed.
- **Studio Paths Modified/Created:**
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController` (created)
  - `StarterPlayer.StarterPlayerScripts.Client.Core.ControllerList` (modified)
  - `ServerScriptService.Server.Services.ShopService` (created)
  - `ServerScriptService.Server.Services.ChuteService` (created)
  - `ServerScriptService.Server.Core.ServiceList` (modified)
  - `ServerStorage.Modules.ChunkBuilder` (modified)
  - Removed replaced standalone scripts: `ServerScriptService.StartAreaShops`, `ServerScriptService.DeliveryChute`, `StarterPlayer.StarterPlayerScripts.VanDriverClient`, `VanFirstPerson`, `VanSeatSwitch`, `VanControls`.
- **New Dependencies:** None (reuses existing `OpenShop`, `DeliveryBox`, `ShopPrompt`, `VanStunned`, `SwitchSeat`, `VanControl`).
- **Next Steps / Known Issues:** Verified in Play mode (`Loader` healthy, `VanController` HUDs & `DriverSeat` session, `ShopService` `OpenShop` firing, `ChuteService` lamp & `MaxCargoBoxes = 24`, `grassTop = 0.86`, `0` colliding clutter, zero console errors) and visually inspected bay-to-road seam. `WindowMonsterController` was left untouched and can optionally migrate to `Loader.Get("VanController")` in a future pass.
