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

## 2026-11-20 — Antigravity (6 Rigged PS1 Horror Monster Models + Docs Sync)
- **Summary of Changes:**
  - Built 6 distinct, low-poly retro PS1 atmospheric horror monster models in Blender (matching the pale-flesh / crimson-joint / void-black-shroud / bone-mask reference style), each organized in its own Blender Collection (`Monster1_Chaser` through `Monster6_Skinwalker`), Metric `1.0` scale, origin grounded at bottom-center between feet `(0, 0, 0)`, facing `-Y`, all transforms applied, outward-recalculated normals, and non-overlapping Smart UV Project (`island_margin = 0.02`):
    - `Monster1_Chaser` (Road Pursuer — `4.86 × 1.92 × 3.90 m`, `1,662` verts / `1,839` faces): towering hunched sprint predator with jagged black hood, oval porcelain mask, exposed crimson ribs, dangling rags, and spider-long clawed arms.
    - `Monster2_Window` (Passenger Window Ambush — `1.37 × 1.40 × 2.09 m`, `1,217` verts / `1,324` faces): lean hunched door/window clinger with unhinged needle-tooth jaw, dorsal spines, and hooked talons.
    - `Monster3_Stare` (Passenger Seat / Rear-View Apparition — `0.72 × 0.50 × 2.63 m`, `1,086` verts / `1,147` faces): tall gaunt rigid humanoid in tattered black shroud with oversized hollow black sockets and glowing pale pinpoint eyes.
    - `Monster4_Gremlin` (Dashboard / Headlight Saboteur — `0.98 × 1.05 × 1.18 m`, `1,103` verts / `1,291` faces): small crouched 3-eyed hood/wiper saboteur with twin jagged ears, crimson back spikes, and spindly wire-shredding claws.
    - `Monster5_Cargo` (Back-of-Van Stowaway — `1.66 × 1.87 × 1.40 m`, `1,935` verts / `2,128` faces): crouched spider-like cargo crawler sized for the van cargo bay (`1.40 m` tall), crooked 6-eyed porcelain mask with mandibles, 4 arching spider-chitin back appendages, and box-ripping foreclaws.
    - `Monster6_Skinwalker` (Road Decoy / Stalker — `1.14 × 0.63 × 2.87 m`, `2,023` verts / `1,976` faces): tall uncanny humanoid/deer hybrid with bleached deer skull, asymmetrical branching antlers, hollow headlight-glint sockets, slack toothed jaw, tattered black pelt mantle, overly long motionless arms, and digitigrade hooved deer legs.
  - Rigged every monster with an exact 16-bone Roblox R15-compatible Armature (`HumanoidRootPart` → `LowerTorso` → `UpperTorso` → `Head`, plus `Left/Right` Upper/Lower Arm, Hand, Upper/Lower Leg, Foot) with clean part-aware vertex weights and `0` unskinned vertices across all 6 models.
  - Exported each monster individually as `.fbx` (`Selected Objects Only`, `Apply Transform` enabled, `Leaf Bones` disabled, `axis_forward='-Z'`, `axis_up='Y'`) and saved the master scene to `Assets/Monsters/GraveyardExpress_Monsters.blend`.
  - Updated `GAME_DESIGN.md` with Monster #6 (`Skinwalker`) and asset references under `Assets/Monsters/`.
  - Updated `GEMINI.md`, `CLAUDE.md`, and `README.md` with team rules to keep `GAME_DESIGN.md` updated whenever new ideas are proposed and to sync AI instruction files across agents whenever workflow rules change.
- **Studio Paths Modified/Created:** None (Blender asset creation and repository docs only; no Roblox Studio scripts edited).
- **Repository Files Added/Modified:**
  - `Assets/Monsters/Monster1_Chaser.fbx` (created)
  - `Assets/Monsters/Monster2_Window.fbx` (created)
  - `Assets/Monsters/Monster3_Stare.fbx` (created)
  - `Assets/Monsters/Monster4_Gremlin.fbx` (created)
  - `Assets/Monsters/Monster5_Cargo.fbx` (created)
  - `Assets/Monsters/Monster6_Skinwalker.fbx` (created)
  - `Assets/Monsters/GraveyardExpress_Monsters.blend` (created)
  - `GAME_DESIGN.md`, `GEMINI.md`, `CLAUDE.md`, `README.md`, `DEVLOG.md` (modified)
- **New Dependencies:** Shared Blender PBR material slots (`Mat_PaleFlesh`, `Mat_CrimsonFlesh`, `Mat_VoidBlack`, `Mat_BoneMask`, `Mat_DarkBone`, `Mat_GlowEye`); 16-bone R15 skeleton naming ready for Roblox Studio 3D Importer & Animation Editor.
- **Next Steps / Known Issues:** Import the 6 `.fbx` models into Roblox Studio via the **3D Importer** (`Avatar` / `R15` rig), store them under `ReplicatedStorage.Assets.Monsters`, author animations in Animation Editor, and connect them to `MonsterService` / client monster controllers.

## 2026-11-20 — Antigravity (Agent Self-Improvement Loop Setup)
- **Summary of Changes:**
  - Created `LESSONS.md` to establish a cross-agent self-improvement loop across Antigravity and Claude sessions, and seeded it with initial verified rules + reasons mined from `DEVLOG.md` and saved session logs (Roblox Studio MCP datamodel/timeout pitfalls, `AntiCheat` runtime remote renaming, `Cmdr`/Service-Controller conventions, `AntiExploitService` exemptions, `HorrorKit_Houses` porch collision bounds, Blender MCP field/orphan-purge gotchas, and Git non-interactive push / Windows PATH handling).
  - Updated `GEMINI.md`, `CLAUDE.md`, and `README.md` so every new agent reads `LESSONS.md` before starting, audits recent saved run logs every run or so to capture failures/wasted tokens/corrections as short rules with reasons, and fixes or removes any rule that turns out wrong.
- **Studio Paths Modified/Created:** None (repository workflow & documentation only).
- **Repository Files Added/Modified:**
  - `LESSONS.md` (created)
  - `GEMINI.md`, `CLAUDE.md`, `README.md`, `DEVLOG.md` (modified)
- **New Dependencies:** None.
- **Next Steps / Known Issues:** Every future Antigravity and Claude session should read `LESSONS.md` on startup and keep it pruned and updated as new issues are discovered or resolved.

## 2026-11-20 — Riplatki (via Claude) (Cab door furniture now rides with the doors)
- **Summary of Changes:** Fixed door decorations on the delivery van not sticking to the doors. `VanDrive` now attaches all cab door furniture, door seam trim, and cab livery stripes to the appropriate door, and applies the existing door-open collision handling to the added parts. Body trim remains attached to the chassis; rear doors and the sliding door are unchanged. No parts are anchored at runtime; new parts are welded to door roots with `WeldConstraint`s.
- **Studio Paths Modified/Created:**
  - `Workspace.DeliveryVan.VanDrive` (modified, "The game part" placeId 93445054108906)
- **New Dependencies:** None (no new remotes, tags, attributes or DataStore keys).
- **Next Steps / Known Issues:** Verified in Play mode: all 68 door parts are welded to their own door and sampled furniture moved with zero drift; no van-related console errors. Not visually inspected in-game from a player camera. Rear hinge barrels (`RearDoors.RearHinge*`) and running boards (`Cab.StepL/R`) deliberately stay on the body.


## 2026-11-20 — Riplatki (via Claude) (Verified existing driver/passenger animation system, no code changes)
- **Summary of Changes:** No code changed. Asked to add avatar hand/leg animations in the van (hands on the steering wheel, feet on gas/brake, hands reaching for door handles, visible to all players). Inspected live Studio and found this already exists and is registered: `VanAnimController` (client) plus the `VanDrive` pose relay and `announceReach`. It was not mentioned in the devlog before, so it is documented here as baseline.
  - Wheel turns and gas/brake pedals move from `PoseSteer` / `PoseGas` / `PoseBrake` van attributes (driver client relays input via `DeliveryVan.VanControl` "Pose"; every client smooths and animates).
  - Driver avatar grips the wheel with both hands and works the pedals (right foot gas/brake, left foot dead pedal). R15 uses `IKControl`; R6 (this game's current rig) uses a Motor6D aim solver in `PreSimulation`.
  - Door handle reach: `VanDrive` sets Player attributes `VanReachDoor` (`CabL`, `CabR`, `Slide`, `RearL`, `RearR`) and `VanReachSeq` when a handle prompt is triggered; each client reaches the nearest hand toward that handle for ~1.2s.
- **Studio Paths Modified/Created:** None.
- **New Dependencies:** None created. In use, do not recreate: Player attributes `VanReachDoor`, `VanReachSeq`; van attributes `PoseSteer`, `PoseGas`, `PoseBrake`; Attachments `VanAnimGripR/L`, `VanAnimFootR/L`, `VanAnimReachTarget`.
- **Next Steps / Known Issues:** Play-mode check (R6 avatar seated in `DriverSeat`): both hands within ~0.2 studs of the wheel grips, right foot ~0.13 and left foot ~0.44 studs from their pedals, zero `VanAnimController` warnings. Door reach was triggered via the attributes but not cleanly confirmed (the van rolled away during the test) and nothing was inspected visually from a player camera. R6 arms are one rigid limb, so they point at far handles instead of bending (only R15 bends via IK). Passengers do not get hand/foot posing in their seat; only the driver does.

## 2026-11-20 — Riplatki (via Antigravity) (Standardized attribution and commit conventions)
- **Summary of Changes:** Updated `GEMINI.md` to formally document attributing all devlog entries to `Riplatki (via Antigravity)` and tagging commits with `(Riplatki)` to match `CLAUDE.md`.
- **Studio Paths Modified/Created:** None.
- **Repository Files Added/Modified:** `GEMINI.md`, `DEVLOG.md`.
- **New Dependencies:** None.
- **Next Steps / Known Issues:** Ready for next gameplay or Studio task.

## 2026-11-20 — Riplatki (via Antigravity) (First-person limb visibility, passenger pose, wheel/pedal & door reach overhaul)
- **Summary of Changes:**
  - Fixed avatar limbs being completely invisible in first-person camera mode (`Enum.CameraMode.LockFirstPerson`) by setting `LocalTransparencyModifier = 0` on arms and legs each frame in `RenderStepped` for seated players while keeping `Head` hidden (`LTM = 1`), allowing players to clearly see their own hands and legs on the wheel and pedals.
  - Hardened R6 limb aiming in `VanAnimController` with fallback tip vectors (`Vector3.new(0, -1.5, 0)` for arms, `Vector3.new(0, -2.0, 0)` for legs) so avatars missing attachments like `LeftGripAttachment` or `RightFootAttachment` no longer abort rig initialization.
  - Added passenger seat resting poses (`VanAnimPassGripR/L` on knees/lap and `VanAnimPassFootR/L` on floorboards) and replicated passenger seating detection across clients.
  - Disambiguated `Workspace.DeliveryVan.Seats.FrontPassengerSeat` from rear cargo bench seats in `VanDrive` and `VanAnimController`, fixing the right cab door ("Ride shotgun") and seat switching ([C]).
  - Polished door handle reaches (`VanReachDoor` & `VanReachSeq`) for driver door (`CabL`), passenger door (`CabR`), sliding door (`Slide`), and rear doors (`RearL`/`RearR`): smooth reach in, handle hold, and ease-out return for both seated occupants and nearby players outside the van.
  - Updated `solveRigs()` execution to run on both `PreSimulation` and `RenderStepped` to prevent engine animation updates from overriding transforms before rendering.
- **Studio Paths Modified/Created:**
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanAnimController` (modified)
  - `Workspace.DeliveryVan.VanDrive` (modified)
  - `Workspace.DeliveryVan.Seats.FrontPassengerSeat` (renamed from `PassengerSeat`)
- **Repository Files Added/Modified:**
  - `LESSONS.md` (modified)
  - `DEVLOG.md` (modified)
- **New Dependencies:**
  - Attachments on van/chassis: `VanAnimPassGripR`, `VanAnimPassGripL`, `VanAnimPassFootR`, `VanAnimPassFootL`.
- **Next Steps / Known Issues:** Verified in live Play mode across all scenarios: driver first-person hands turning steering wheel through 150° lock, pedal travel between gas/brake (3.2 stud foot shift), passenger lap pose and `CabR` reach, and outside sliding door reach. Studio returned cleanly to `Edit` mode.

## 2026-11-20 — Riplatki (via Antigravity) (Adjusted cab seating posture: torso back, head/camera forward)
- **Summary of Changes:**
  - Moved the seated torso back naturally against the seat backrest by reducing `CAB_SEAT_SHIFT` from `0.5` to `0.15` in `VanDrive`, stopping the chest from being shoved forward into the wheel and camera.
  - Dynamically leaned the avatar's `Neck` joint forward (~`0.45` studs along `-Z` in torso space) in `VanAnimController` while seated, positioning the head and eyes naturally forward over the chest (like a real driver over the steering wheel) and smoothly returning to default standing posture upon getting out.
  - Configured first-person camera in `VanController` to keep `CameraSubject` anchored to `Humanoid` with a forward eye offset (`hum.CameraOffset = Vector3.new(0, 0.05, -0.3)`), so looking down in first person looks cleanly past the chest and shows only the lower half of the torso (lap/waist) and the legs/pedals without occlusion.
  - Kept `UpperTorso` hidden (`LTM = 1`) and `LowerTorso` visible (`LTM = 0`) for R15 avatars in first person.
- **Studio Paths Modified/Created:**
  - `Workspace.DeliveryVan.VanDrive` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanAnimController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController` (modified)
- **Repository Files Added/Modified:**
  - `LESSONS.md` (modified)
  - `DEVLOG.md` (modified)
- **New Dependencies:** None.
- **Next Steps / Known Issues:** Verified in Play mode: looking down now displays the lower torso, thighs, and pedals cleanly with zero chest obstruction; looking ahead frames the wheel, dashboard, and windshield naturally; standing up restores normal neck alignment.

## 2026-11-20 — Antigravity (Fix Van Seat Trap / Movement Lock & Replicate Van Animations to All Players)
- **Summary of Changes:**
  - **Fixed Players Getting Trapped / Unable to Move in & Exiting the Van:**
    - Diagnosed two root causes: (1) Roblox's `AvatarAbilities` / `ControllerManager` (`CharacterContext.JumpAction`) intercepts `Space` before `InputBegan` and skips `Humanoid.Jump`, trapping seated players in `DriverSeat`/`FrontPassengerSeat` with `ActiveController = nil`; (2) inside the enclosed cab, standing up immediately triggered `Seat` touch-to-sit re-seating.
    - Added a server-authoritative `"LeaveSeat"` action on `DeliveryVan.VanControl` in `VanDrive` that temporarily disables `seat.Disabled`, removes `SeatWeld`, clears `hum.Sit = false`, waits for `hum.SeatPart == nil`, and steps the player cleanly outside the open cab door via `char:PivotTo(...)` with wall-clearance raycasting.
    - Added reliable `Space` / `ButtonA` exit detection in `VanController` (edge-detected in the seated `RenderStepped` loop plus `JumpRequest` and `InputBegan`) and restored `VanController`'s cab first-person `CameraOffset = Vector3.new(0, 0.05, -0.3)` (resetting on exit and during hold-`V` chase cam) plus clean third-person zoom release on exit.
    - Fixed `[C]` seat switching (`SwitchSeat`) in `VanDrive` to disambiguate `FrontPassengerSeat` from the 7 rear cargo `PassengerSeat`s, apply `CAB_SEAT_SHIFT = 0.15` to both cab seats, and cleanly wait past Roblox's post-standup sit cooldown before seating into the opposite cab seat.
    - Raised `Workspace.SpawnLocation` flush with `Workspace.Factory.Main_Floor_Slab` (`top = 1.93` vs `slabTop = 1.91`) so avatars no longer spawn slightly clipped into the floor.
  - **Made Van Animations Replicated & Visible to All Players:**
    - Fixed the `VanAnimController:tryBind()` race condition where `refs.control` (`DeliveryVan.VanControl`) cached as `nil` permanently before `VanControl` replicated, causing zero `"Pose"` events to ever reach the Server.
    - Replaced `VanDrive`'s dropping rate-limiter with a `Heartbeat`-flushed `pendingPose` queue (`PoseSteer`, `PoseGas`, `PoseBrake`, `PoseBoost`) plus `0.5s` heartbeat resends from `VanAnimController` so final `{0, 0, 0}` resting poses and late-joiners never desync.
    - Moved neutral `ClosedC0` attributes (`SteerJoint`, `BrakePedalJoint`, `AccelPedalJoint`) and fixed `VanAnim*` target `Attachment`s (`VanAnimGripR/L`, `VanAnimFootR/L`, `VanAnimPassGripR/L`, `VanAnimPassFootR/L`, `VanAnimReachTarget`) onto `VanDrive` (Server) so every client shares identical neutral targets even across `StreamingEnabled` or mid-turn joins.
    - Replicated authoritative seat occupancy (`DeliveryVan` attributes `DriverUserId`/`PassengerUserId` and Player attribute `VanSeatRole`), upgraded R6 limb aiming in `VanAnimController` to an idempotent `Motor6D.C0` solver (relative to saved `defaultC0`, canceling `motor.Transform`) that works identically for local and throttled remote players, fixed headless-driver visibility during hold-`V` chase cam, and synced `DashController` BOOST/BRK displays for passengers via `PoseBoost`/`PoseBrake`.
- **Studio Paths Modified/Created:**
  - `Workspace.DeliveryVan.VanDrive` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanAnimController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.DashController` (modified)
  - `Workspace.SpawnLocation` (repositioned flush with `Factory.Main_Floor_Slab`)
- **Repository Files Modified:** `LESSONS.md`, `DEVLOG.md`.
- **New Dependencies:**
  - `DeliveryVan` attributes: `DriverUserId`, `PassengerUserId`, `PoseBoost` (in addition to existing `PoseSteer`, `PoseGas`, `PoseBrake`).
  - `Player` attribute: `VanSeatRole` (`"Driver"` | `"Passenger"` | `nil`).
  - Joint attribute: `ClosedC0` on `SteerJoint`, `BrakePedalJoint`, `AccelPedalJoint`.
  - `VanControl` remote action: `"LeaveSeat"` (C2S, valid for both driver and passenger).
- **Next Steps / Known Issues:**
  - Verified live in Play mode: `DriverSeat` pose relay (`PoseSteer=0.7, PoseGas=0.7` -> clean `0, 0` flush), `[C]` seat switch to `FrontPassengerSeat`, `Space` exit cleanly stepping outside the door (`State=Running`, `CM.Active=GroundController`, `Neck.C0` & `Head.LTM` restored) followed by free walking with `W`, and a simulated remote driver dummy observed from outside tracking full `150°` wheel turn and gas→brake foot travel with zero van script errors.
  - Note: During the final Play run, Studio console logged an unrelated parse error from a live Team Create edit in `ServerStorage.Modules.Monsters.WindowMonster:290` (`Ambiguous syntax: ... use ';' to separate statements`), which blocked `Server.Bootstrap` / `ServiceList` from loading; left untouched to avoid stepping on active teammate monster work.

## 2026-11-20 — Riplatki (via Antigravity) (3D Articulated Window Monster, 4-Phase Animation & FrontPassengerSeat Fix)
- **Summary of Changes:**
  - Replaced the invisible 2D `SurfaceGui` silhouette (which was occluded behind `SideGlassL/R`'s `Enum.Material.Glass`!) with a real, opaque 110-part **3D articulated Window Monster model** stored at `ReplicatedStorage.Assets.Monsters.Monster2_Window` matching the PS1 horror aesthetic: hunched void-black shrouded body with exposed crimson ribs and dorsal spines, gaunt bone-pale skull with sunken sockets and pinpoint glowing `Neon` eyes + face `PointLight`, hinged unhinging lower jaw packed with needle teeth, and two articulated spidery arms with hooked bone talons.
  - Upgraded `WindowMonsterController` to animate the 3D monster via single-call `workspace:BulkMoveTo` (zero physics load on the moving van, synchronized across all clients via replicated attributes, and auto-compatible with an imported Blender FBX replacement at `ReplicatedStorage.Assets.Monsters.Monster2_Window`):
    - **Climb-up:** claws hook onto the window sill and the skull rises from below the door to eye level.
    - **Stalk & Tap:** peers side-to-side into the cab, creeps closer without clipping the glass, periodically raps/taps claws on the window pane with spatial audio cues, unhinges its needle-tooth jaw wider, and flares its eyes & light from pale bone-white to pulsing blood crimson.
    - **Repelled climax (`WindowThreatEvent = "Repelled"`):** violently recoils away from the slamming roller blind and tumbles off the side of the van into the dark.
    - **Attack / Breach climax (`WindowThreatEvent = "Attack"`):** violently lunges straight through the window into the cab with slashing claws and wide-open jaw before yanking back as the van stalls.
  - Migrated `WindowMonsterController` onto `Loader.Get("VanController")` and `Loader.Get("AudioController")`, fixed stun camera shake so it preserves `VanController`'s seated forward `CameraOffset`, refined the side vignette so the monster stays clearly visible through the window, and added a 0.28s client debounce on `WindowCover` to prevent `AntiExploit` `MinInterval` warnings when mashing `[Z]`/`[X]`.
  - Fixed `WindowMonster` and `WindowMonsterController` looking up `"PassengerSeat"` (which hit rear cargo seats) instead of `"FrontPassengerSeat"`, restoring passenger `[Z]`/`[X]` cover controls.
  - Added server climax event attributes (`WindowThreatEvent`, `WindowThreatEventSide`, `WindowThreatEventTime`, `WindowThreatEventSeq`), `MonsterService:TriggerWindowMonster` / `ForceWindowAttack` / `ClearWindowMonster`, and the `Cmdr` admin command `windowmonster` (`wm`, `spawnwindow`) for instant testing (`windowmonster L|R|random|attack|clear`).
- **Studio Paths Modified/Created:**
  - `ReplicatedStorage.Assets.Monsters.Monster2_Window` (created)
  - `ServerStorage.Modules.Monsters.WindowMonster` (modified)
  - `ServerScriptService.Server.Services.MonsterService` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.WindowMonsterController` (modified)
  - `ServerScriptService.Cmdr.Commands.windowmonster` & `windowmonsterServer` (created)
- **Repository Files Added/Modified:**
  - `LESSONS.md` (modified)
  - `DEVLOG.md` (modified)
- **New Dependencies:**
  - Replicated `DeliveryVan` attributes: `WindowThreatEvent`, `WindowThreatEventSide`, `WindowThreatEventTime`, `WindowThreatEventSeq`.
  - Optional custom SFX names supported in `ReplicatedStorage.Assets.Sounds`: `WindowMonsterAppear`, `WindowMonsterTap`, `WindowMonsterRepel`, `WindowMonsterAttack` (falls back cleanly to built-in `rbxasset://sounds/` spatial cues).
- **Next Steps / Known Issues:** Verified visually in live Play mode via inside-cab screenshot: skull, glowing crimson eyes, unhinging needle-tooth jaw, ribs, and gripping spidery hands render cleanly through `SideGlassL` with zero glass clipping and zero console errors (the earlier `WindowMonster:290` syntax note above was immediately resolved during the same session). Studio returned to `Edit` mode. Remember to **Commit Drafts** in Team Create.

## 2026-11-20 — Antigravity (Non-Collidable Roadside Infrastructure & Trees in Procedural Map)
- **Summary of Changes:**
  - Updated `ChunkBuilder` so all `HorrorKit_Infrastructure` (guardrails `Infra_Guardrail_Straight`/`Infra_Guardrail_Broken`, `Infra_Streetlamp`, `Infra_UtilityPole`, `Infra_RoadBollard_Glow`, `Infra_ChevronSign`, `Infra_PowerWire_20m`, plus any future assets in that kit) and all `HorrorKit_Props` (`Prop_DeadTree_A`, `Prop_DeadTree_B`, `Prop_DeadGrass_Cluster`, `Prop_Mailbox_Rusted`, `Prop_WoodFence_Segment`, `Prop_TrashDebris_Pile`, plus any future props) spawn with `CanCollide = false` and `CanQuery = false` (also prevents hold-`V` chase-cam and exit raycasts from snagging on poles/trees).
  - Kept driveable surfaces (`GrassGround`, `HorrorKit_Roads`) and buildings (`HorrorKit_Houses` + `DropoffPad`s) fully collidable (`CanCollide = true`, `CanQuery = true`).
  - Updated `MapService.archiveLegacyMap()` to also set `CanCollide = false` and `CanQuery = false` on the 9 loose hand-placed `Workspace.Tree` models that sit in/near the procedural road zone at runtime.
- **Studio Paths Modified/Created:**
  - `ServerStorage.Modules.ChunkBuilder` (modified)
  - `ServerScriptService.Server.Services.MapService` (modified)
- **New Dependencies:** None.
- **Next Steps / Known Issues:** Verified in live Play mode across 36 active `ProceduralMap` chunks (`0` colliding roadside infrastructure, `0` colliding trees/props, `0` colliding loose `Workspace.Tree` parts; all roads, ground slabs, houses, and dropoff pads remain collidable; zero script errors). Studio returned cleanly to `Edit` mode.

## 2026-11-20 — Riplatki (via Antigravity) (Window Monster Forced Look, Camera Shake, Head-Banging, Growing Cracks & Lethal In-Van Breach)
- **Summary of Changes:**
  - **Forced Camera Look & Escalating Camera Shake (`WindowMonsterController`):**
    - Added `isLocalPlayerInsideVan()` detection covering seated cab occupants (`DriverSeat`, `FrontPassengerSeat`), rear cargo bench `PassengerSeat`s, and players standing anywhere inside the van hull.
    - While `WindowMonster3D` is active (`Threat` or `Attack`) for players inside the van, `RunService:BindToRenderStep("WindowMonsterCam", Camera.Value + 25)` smoothly locks camera look onto the monster's skull (`VanController:SetChaseCamBlocked(true)`) while layering continuous terror tremor + heavy impact rotational/positional shake on every head-bang and shatter.
    - Unlocks cleanly on repel (smooth blend back), van exit, death, or respawn (`CharacterAdded`), and removed `onStunChanged()`'s `hum.CameraOffset` override so `VanController`'s seated forward `CameraOffset` is never clobbered.
  - **Violent Head-Banging & 4-Stage Growing Spiderweb Cracks (`WindowMonster` & `WindowMonsterController`):**
    - Replaced idle tapping with synchronized `ClimbSeconds = 0.85` emergence followed by `BangCount = 5` violent skull windup-and-slam impacts over `ThreatSeconds = 9`, driving `WindowCrackL/R` (`0..4`) and `WindowBangSeq` from server `_tickThreat`.
    - Added pre-built 4-stage crisp spiderweb fracture overlays (`CracksLayer` on `WindowCoverL/R`'s `SurfaceGui` at `ZIndex = 2` behind `Shade` `ZIndex = 10` with `LightInfluence = 0.55` to eliminate Bloom washout), 7-piece glass chips popping on each impact, layered thud + crack spatial cues, and 4s delayed crack cleanup when repelled.
  - **Window Shatter, 3D Glass Shards & Lethal In-Van Rampage (`WindowMonster` & `WindowMonsterController`):**
    - On the 5th head-bang (`_fail()` / `WindowThreatEvent = "Attack"`), the server breaks `SideGlassL/R` (`WindowBrokenL/R = true`, `Transparency = 1`, `CanCollide = false`), the client bursts 26 tumbling 3D `GlassShard` parts inward (inheriting van velocity, zero collisions/van-physics load), and the monster vaults straight through the broken window into `DeliveryVan` local space—mauling the closest cab seat (`0.72s`), slashing the opposite cab seat (`1.15s`), bursting past the bulkhead to sprint down the cargo aisle (`1.50s–2.45s`) with a final hull sweep at `2.35s` to kill everyone inside the van, and auto-repairing the broken window after `6.0s` so respawned players can continue the shift.
- **Studio Paths Modified/Created:**
  - `ServerStorage.Modules.Monsters.WindowMonster` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.WindowMonsterController` (modified)
- **Repository Files Modified:**
  - `LESSONS.md` (modified)
  - `DEVLOG.md` (modified)
- **New Dependencies:**
  - Replicated `DeliveryVan` attributes: `WindowCrackL`, `WindowCrackR`, `WindowBangSeq`, `WindowBrokenL`, `WindowBrokenR`, `WindowThreatClimb`, `WindowThreatBangCount`.
  - Optional custom SFX names supported in `ReplicatedStorage.Assets.Sounds`: `WindowMonsterBang`, `WindowMonsterCrack`, `WindowMonsterShatter`.
- **Next Steps / Known Issues:** Verified end-to-end in live Play mode: forced camera lock (`CamDot` up to `1.000`) in both cab & rear cargo bench, `WindowBangSeq` `0->4` and `Attack` shatter, visual `screen_capture` confirming razor-sharp Stage 3 spiderweb cracks over the monster's glowing skull with zero bloom washout, `26` flying 3D `GlassShard`s, monster pathing through cab (`X=3.3->-1.6`) and cargo (`Z=-1.1->+16.6`), lethal kills for both `DriverSeat` (`0.77s`) and rear cargo `PassengerSeat` (`1.67s`), `6.0s` glass auto-repair, and zero script console errors. Studio returned to `Edit` mode. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Focus:** `WindowMonster` Directional Pre-Arrival Audio/Visual Cues (Cargo Wall Denting) & Timed Forced-Look Release
- **Changes Made (in live Roblox Studio via MCP):**
  - `game.ServerStorage.Modules.Monsters.WindowMonster`:
    - Added synchronized `ApproachSeconds = 2.6` pre-arrival telegraph (`_startApproach`, `_clearApproach`, `_finishApproach`) replicating `WindowApproachSide`, `WindowApproachStart`, and `WindowApproachDuration` on `DeliveryVan` before the monster pops up at the cab glass.
    - Added early cover repel during the `Approach` phase (`_finishApproach` resolves directly to `WindowThreatEvent = "Repelled"` when the threatened side's window cover is already held shut).
    - Wired approach cleanup/transitions through `_trySpawn`, `ForceSpawn`, `ForceAttack`, `Update`, and `Reset`.
  - `game.StarterPlayer.StarterPlayerScripts.Client.Controllers.WindowMonsterController`:
    - Added 3D directional audio emitter `ApproachAudioAnchor_<Side>` racing strictly down the threatened side (`relX = -5.30` Left / `+5.30` Right, `relZ = 13.0 -> -7.5`) with EQ-filtered dull metallic thud (`t = 0`, `WindowMonsterApproachThud`) followed by 10 accelerating footsteps (`stepIdx 0..9`, `WindowMonsterFootstep` + `WindowMonsterStepThump`), synchronized with an exterior scrambling `Monster2_Window` at `relX = sx * 6.45`.
    - Added rear-cargo inward-bending wall dents (`WallDent`, `WallDentCore`, `WallDentCreaseT`, `WallDentCreaseB`, `DentDust` via `spawnWallDent`, `applyDentFrame`, `stepWallDents`) on `Interior.LinerL`, `Interior.LinerR_Front`, `Interior.LinerR_Rear`, and closed `SlidingDoor.DoorPanel` (`maxDepth = 0.72` / `0.84` heavy, `curDepth * 1.15` thickness anchored inside the inner face to guarantee zero exterior poke-through, skipping open `SlidingDoor`), updated in `chassis.CFrame` space on `RenderStepped`.
    - Updated `CAM_STEP_NAME` (`WindowMonsterCam`, `Camera.Value + 25`) with `INITIAL_LOOK_LOCK_DURATION = 0.95` (`threatLookStartedAt`, `forcedLookReleased`) so players inside the van are pulled to face the monster for the initial `0.95s` jump-pop and can freely look away afterward while camera shake continues, adding `CAM_PRE_STEP_NAME` (`WindowMonsterCamPre`, `Camera.Value - 5`) calling `stripAppliedShake()` before Roblox's `CameraModule` (`200`) to prevent camera aim drift.
- **Next Steps / Known Issues:** Verified in live Play mode: Left/Right spatial emitters (`AppAnchorX = -5.30` / `RightDents = 7`), cargo bay visual `screen_capture` confirming inward `WallDent` / `WallDentCore` / `WallDentCreaseT/B` geometry cleanly bounded on `Interior.LinerL` with zero z-fighting or exterior poke-through, early cover repel from `FrontPassengerSeat` (`WindowApproachSide = "R"` -> `Remotes.WindowCover:FireServer("R")` -> `WindowThreatEvent = "Repelled"`), initial `0.95s` forced look -> free look away in `DriverSeat` (`InitialLockDot = 1.000`, `PulledBackDuringLockDot = 1.000`, `FreeLookAwayFwdDot = 0.950`), and zero script console errors. Studio returned to `Edit` mode. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Focus:** Fix Broken Jump & Team Test Movement Lock (AvatarAbilities / ControlState Beta Bypass + `ClientBootstrap` Path)
- **Root Cause:** Even after switching `StarterPlayer.LuaCharacterController` from `Default` to `Legacy`, Roblox Studio's active experimental **AvatarAbilities / CCL** beta still injected `AbilityManagerActor`, `ControllerManager`, `ControlState`, and forced `Humanoid.EvaluateStateMachine = false` on spawned characters. Because `EvaluateStateMachine == false`, `PlayerModule`'s `AvatarAbilitiesInterface:isEnabled()` returned `true`, causing `ControlModule:characterMove` and `InputReplication.writeInputToCS` to ignore `JumpAction` (`Space`) in favor of `AbilityAction1` (stuck at `Enabled = false`, breaking jump: `dY = 0.00`, `JumpReq = false`) and route movement strictly through `ControlState` (which breaks on Team Test when clients/cloud server mismatch beta state). Additionally, `ClientBootstrap` waited on `StarterPlayer.StarterPlayerScripts.Client` instead of `script.Parent`.
- **Changes Made (in live Roblox Studio via MCP):**
  - `StarterPlayer.LuaCharacterController`: Set to `Enum.CharacterControlMode.Legacy`.
  - `StarterPlayer.StarterPlayerScripts.Client.ClientBootstrap`: Updated `clientFolder` lookup from `StarterPlayer:WaitForChild("StarterPlayerScripts"):WaitForChild("Client")` to `script.Parent :: Folder`.
  - `ServerScriptService.LegacyCharacterFix` (`Script`, created): Disables scripts inside `AbilityManagerActor`, strips `AbilityManagerActor`, `ControllerManager`, `ControlState`, `PlayerControlState`, and root sensors (`FloorSensor`, `LadderSensor`, `BuoyancySensor`), restores `HumanoidRootPart.CollisionGroup = "Default"` (from `CharacterRootParts`), and enforces `Humanoid.EvaluateStateMachine = true` plus `Enum.HumanoidStateType.Jumping` enabled across spawns and property changes—allowing `PlayerModule` to run its classic `Humanoid:Move` / `Humanoid.Jump` + `JumpAction` path cleanly.
  - `StarterPlayer.StarterPlayerScripts.LegacyCharacterClientFix` (`LocalScript`, created): Client-side companion disabling `AbilityManagerActor` client scripts (`ClientInitialize`) and keeping `Humanoid.EvaluateStateMachine = true` and `Jumping` enabled locally.
- **Repository Files Modified:** `DEVLOG.md`, `LESSONS.md`.
- **Next Steps / Known Issues:** Verified in live Play mode: Client & Server both report `EvalSM=true`, `AMA=nil`, `CM=nil`, `CS=nil`, `FloorS=nil`, `LadderS=nil`, `BuoyS=nil`, `HRP.CG=Default`, `JumpEnabled=true`; real keyboard Jump (`Space`) + Walk (`W`) measured `dY=7.31`, `dXZ=8.22`, `HumJumpState=true`, `JumpReq=true`; `DriverSeat` entry + `Space` exit restored `SeatPart=nil`, `Sit=false`, `CameraMode=Classic`, followed by post-exit jump (`dY=7.29`, `HumJumpState=true`) and unobstructed reverse walk (`dXZ=9.97`); server respawn (`LoadCharacter()`) preserved classic state with zero errors/warnings from any game or `PlayerModule` script. Studio returned to `Edit` mode. Remember to **Commit Drafts** in Team Create (`ClientBootstrap`, `LegacyCharacterFix`, `LegacyCharacterClientFix`) before starting a fresh **Team Test**, and teammates can also disable the experimental **Avatar Abilities** beta in **File → Beta Features**.

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Focus:** Van Tasks #1 & #2 — Monster-Blocked Cab Doors & Power Roll-Down Windows with Arm Animations
- **Changes Made (in live Roblox Studio via MCP):**
  - `Workspace.DeliveryVan.VanDrive`:
    - **#1 Monster-Blocked Doors:** Added `isDoorBlockedByMonster(side)` checking `WindowThreatSide == side` and a bounded `3.0s` breach window on `WindowThreatEvent == "Attack"` (using `WindowThreatEventTime`, listening to `WindowThreatSide`, `WindowThreatEvent`, and `WindowThreatEventSeq`). When active on a side, any open door on that side slams shut, `setOpen(true)` is blocked, the door handle `ProximityPrompt` shows `"Door jammed!"` with a `2.2°` rattle tween + `DoorJammedSfx` + `Notification` toast (`"Something is holding the door shut!"`), and `exitVanSeat` (`LeaveSeat`) refuses to open or eject out that side (`"Something is blocking that door! Switch seats [C] or shut the cover!"`).
    - **#2 Roll-Down Cab Windows:** Replaced static `WeldConstraint`s on `SideGlassL/R` (`owner[glass] = glass`) with movable `Weld`s (`WindowJointL`, `WindowJointR`; `ClosedC0`, `DownDirLocal`, `MaxTravel = 3.05`), added matching `WindowSwitchR` + `WindowSwitchLedR` on the right armrest (`CabDetail.Doors`), replicated neutral hand targets (`VanAnimWindowSwitchL`, `VanAnimDriverWindowSwitchR`, `VanAnimWindowSwitchR`), added `[R]` `WindowPromptL/R` armrest prompts + `VanControl("RollWindow", side)` (handled before the driver-only guard, `0.35s` debounce), `Heartbeat` window sliding + `WindowMotorSfx` (`action_falling.mp3` / optional `Assets.Sounds.WindowRollMotor`) + `syncGlassCollision`, replicated van/player attributes (`WindowRollL/R`, `WindowRollTargetL/R`, `VanWindowAnimSwitch`, `VanWindowAnimDir`, `VanWindowAnimSeq`), and automatic reset on `WindowBrokenL/R` repair.
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController`: Added `KEY_WINDOW` (`R`), `KEY_WINDOW_OTHER` (`T` for driver controlling right window), `VanController:ToggleWindow(side?)`, and updated the bottom-left controls panel to list `[R] Window`.
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanAnimController`: Added `winSwitch*/winTip*` switch targets and `Rig.winW/winSide/winDir`, animating subtle downward/upward press offsets (`-0.08` / `+0.06` studs with motor tremor) in `step(dt)` and blending `reachL`/`reachR` smoothly onto the active switch while keeping the opposite hand anchored on the wheel/lap (visible in first person and replicated to all players).
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.WindowMonsterController`: Added `getWindowFrameCF(glass, side)` reading `WindowJoint<side>.Part0.CFrame * ClosedC0` (lines 1351+, 1781, 1806) so the Window Monster always anchors cleanly to the window opening even when the side glass is rolled down.
- **Repository Files Modified:** `DEVLOG.md`, `LESSONS.md`.
- **Next Steps / Known Issues:** Verified in live Play mode after the final edit: Left window roll (`base=2.93 -> mid RollL=0.30, Sw=L, DistL=0.45, gripR=0.17 -> down RollL=1.00, relY=2.80 -> up RollL=0.00, relY=5.85`), rolled-down `windowmonster L` frame anchoring (`MonsterHeadRelY=5.76`), blocked door & `LeaveSeat` (`ActL="Door jammed!"`, `ActR="Open door"`, `SeatBlk=DriverSeat`, `DoorL=-0.7` mid-rattle, `"Something is blocking that door!"` toast, unjamming cleanly to `"Open door"` on clear), Passenger Right window roll (`base=2.48 -> mid RollR=0.36, Sw=R, DistR=0.35 -> down RollR=1.00, relY=2.80 -> up RollR=0.00`), plus prior visual `screen_capture` (`Van_RollDownWindow_FP`), `-68° -> 0°` open-door slam, `[R]` keybind test, and `SwitchSeat` -> Right exit. Zero script or audio errors in `LogService` / `get_console_output`. Studio returned to `Edit` mode; paused after #1–#2 awaiting user go-ahead for #3–#4. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Focus:** Van Tasks #3 & #4 — Slower Window Cover Pull-Up & Arm-Braced Seat-Switch Lift-and-Slide Animation
- **Changes Made (in live Roblox Studio via MCP):**
  - `ServerStorage.Modules.Monsters.WindowMonster`:
    - **#3 Slower Window Cover Pull-Up:** Added `CoverPullUpSeconds = 1.35` and `Raising: boolean` to `CoverState`. When `RequestCover` is invoked from `FrontPassengerSeat`, `WindowCover<Side> = true` replicates immediately so every client starts the slower `1.35s` roller-blind pull-up animation, while `c.Up = true`, active threat `self:_repel()`, and the `CoverHoldSeconds = 8` hold timer engage once the `1.35s` pull-up completes (and `_finishApproach` / `_trySpawn` respect `c.Up or c.Raising`).
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.WindowMonsterController`:
    - Replaced the instant `0.42s` blind tween with `BLIND_UP_TWEEN = TweenInfo.new(1.35, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut)` for pulling covers up and `BLIND_DOWN_TWEEN = TweenInfo.new(0.75, Enum.EasingStyle.Quad, Enum.EasingDirection.In)` when dropping.
  - `Workspace.DeliveryVan.VanDrive`:
    - **#4 Arm-Braced Seat-Switch Lift & Slide:** Added replicated seat-switch hand bracing attachments on `chassis` (`VanAnimConsoleBraceL`, `VanAnimConsoleBraceR`, `VanAnimSeatBraceL`, `VanAnimSeatBraceR`). Replaced the stand-up-and-teleport `SwitchSeat` handler with a `0.68s` smooth lift-and-slide across the center console (`SwitchSlideWeld`, `+0.62` stud vertical lift arc `math.sin(u * math.pi)`, `-7°` forward pitch, `±5°` lateral roll). By transferring `SeatWeld` to `SwitchSlideWeld` and setting `hum.Sit = false` at `t = 0` while pre-assigning `VanSeatRole`, Roblox's `~0.18s` post-unsit cooldown expires during the `0.68s` slide so `target:Sit(hum)` attaches `target.SeatWeld` (`VanShifted = true`) immediately on completion with zero stand-up or teleport.
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController`:
    - Updated `refreshSeatState` and `hookCharacter` (`VanSwitchActive` listener) so first-person van camera (`LockFirstPerson`) and HUD remain completely uninterrupted throughout `[C]` seat switches.
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanAnimController`:
    - Added `consoleBraceL/R`, `seatBraceL/R`, and dynamic `switchHandR/L` + `switchFootR/L` targets. While `VanSwitchActive == true`, both arms visibly plant and push down on the seat cushion and center console (`pushDip = -0.14 * sin(u*pi)`) to lift the player across to the steering wheel (when moving to Driver) or lap (when moving to Passenger), while feet step across the floorboard (`+0.38` stud step arc) and first-person limbs remain visible (`LocalTransparencyModifier = 0`).
- **Repository Files Modified:** `DEVLOG.md`, `LESSONS.md`.
- **Next Steps / Known Issues:** Verified in live Play mode: slower cover pull-up progression (`L_Cover: t=0.25:0.06 | t=0.70:0.52 | t=1.10:0.92 | t=1.50:1.00`; `CoverR_Repel: t=0.25:0.03 | t=0.70:0.41 | t=1.10:0.87 | t=1.48:1.00`), bidirectional `[C]` seat switch trajectories (`P2D`: `x = +2.21 -> +1.76 -> -0.51 -> -2.06 -> -2.21`, `y = 2.97 -> 3.15 -> 3.55 (+0.58 lift) -> 3.04 -> 2.97`, hands bracing at `rY=2.41..3.15, lY=2.52..2.98` before rising to wheel `3.66..3.76`, landing on `DriverSeat`; `D2P`: `x = -2.21 -> -1.97 -> +0.41 -> +1.30 -> +2.21`, `y = 2.97 -> 3.07 -> 3.56 (+0.59 lift) -> 3.35 -> 2.97`, hands dropping from wheel `3.60` to console brace `2.54..2.86` and settling on lap `2.35`, landing on `FrontPassengerSeat`, with `cam = LockFirstPerson` throughout). Zero game script errors. Studio returned to `Edit` mode; paused after #3–#4 awaiting user go-ahead for #5–#6. Remember to **Commit Drafts** in Team Create.


---

## 2026-11-20 — justhyne1205 (via Antigravity)
- **Summary of Changes:**
  - Built 6 new low-poly dead tree variants in Blender (`DeadTree_01_Fork`, `DeadTree_02_Spire`, `DeadTree_03_Leaner`, `DeadTree_04_Twin`, `DeadTree_05_Snag`, `DeadTree_06_Crook`; ~152–235 tris each) matching `Prop_DeadTree_A` / `Prop_DeadTree_B`, imported them into Roblox Studio, scaled 5.3× (`24.1–48.4` studs), matched their dark charcoal bark (`RGB(52, 54, 58)`, `Plastic`), anchored them beside the reference trees, and mirrored all 6 into `ReplicatedStorage.Assets.MapModels.HorrorKit_Props`.
  - Upgraded the procedural map forest (`MapService` + `ChunkBuilder`) to spawn all 8 dead tree variants across a 3-cell-deep (`300` stud) forest border (`ChunkBuilder.FOREST_BORDER_RADIUS = 3`, `MAX_SPAWNS_PER_TICK = 8`) that scales progressively denser and larger farther from the road: Ring 1 roadside `6–9` trees (`0.88×–1.12×`, `CanCollide = false` within `95` studs of road center so the van never snags), Ring 2 mid-forest `14–18` trees (`~1.15×–1.55×`, solid blockers), and Ring 3 deep forest `22–28` trees (`~1.50×–2.00×`, up to ~97 studs tall, solid blockers) plus pooled invisible `ForestBoundary` walls along exposed Ring 3 outer edges to prevent running off the map.
  - Added `Assets/Props/DeadTrees.blend` and `Assets/Props/DeadTrees.glb` to the repo and documented the deep forest containment and planned far-from-van hunter monster in `GAME_DESIGN.md` (monster not implemented).
- **Studio Paths Modified/Created:**
  - `Workspace.MapModels.HorrorKit_Props.DeadTree_01_Fork` .. `DeadTree_06_Crook` (created)
  - `ReplicatedStorage.Assets.MapModels.HorrorKit_Props.DeadTree_01_Fork` .. `DeadTree_06_Crook` (created)
  - `ServerStorage.Modules.ChunkBuilder` (modified)
  - `ServerScriptService.Server.Services.MapService` (modified)
- **Repository Files Added/Modified:**
  - `Assets/Props/DeadTrees.blend`, `Assets/Props/DeadTrees.glb` (created)
  - `GAME_DESIGN.md`, `LESSONS.md`, `DEVLOG.md` (modified)
- **New Dependencies:** None (uses existing `ProceduralMap` pooling; adds internal pool key `__BoundaryWall` and `ChunkBuilder.FOREST_BORDER_RADIUS = 3`).
- **Next Steps / Known Issues:** Verified in live Play mode (`12` road chunks, `85` forest chunks, all 8 dead tree variants evenly distributed `141–180` each, `Ring1` avg `7.2` trees/chunk @ `45.1` studs → `Ring2` avg `15.1` @ `56.2` studs → `Ring3` avg `21.5` @ `65.5` studs max `96.7`, `0` colliding roadside trees, `1,204` solid deep-forest trees, `25` `ForestBoundary` outer walls, zero script errors). Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create (`ChunkBuilder`, `MapService`) and grant the group experience permission to use the 6 personal-account mesh assets (`119484560624615`, `133992692268823`, `136479370207102`, `90019402294940`, `101456467687813`, `98035104750823`) in Creator Dashboard before publishing.

---

## 2026-11-20 — justhyne1205 (via Antigravity)
- **Summary of Changes:**
  - Tuned procedural forest tree density and streaming performance to remove slight frame lag while keeping the full, imposing deep-forest look: trimmed per-chunk tree counts by ~25% (`Ring 1`: `5–7`, `Ring 2`: `10–13`, `Ring 3`: `15–19` with trunk spacing `18/15/12`), disabled `CastShadow` on distant `Ring 3` trees (explicitly reset on pool acquire in `ChunkBuilder`), and smoothed `MapService.MAX_SPAWNS_PER_TICK` from `8` → `5`.
- **Studio Paths Modified/Created:**
  - `ServerStorage.Modules.ChunkBuilder` (modified)
  - `ServerScriptService.Server.Services.MapService` (modified)
- **New Dependencies:** None.
- **Next Steps / Known Issues:** Verified in Play mode (`970` total active dead trees across `87` forest chunks — down ~25% from `1,295`; Ring 1 avg `5.9`, Ring 2 avg `10.9`, Ring 3 avg `14.8`; shadow casters cut from `1,295` to `542` with `428` far Ring 3 trees shadow-off; zero script errors). Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create (`ChunkBuilder`, `MapService`).

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Focus:** Hold-to-Pull-Up Window Covers & Fix Passenger-to-Driver Seat-Switch Camera Drop
- **Root Cause (Passenger -> Driver Camera Drop):**
  1. When an R6 avatar is already in a seated pose (with `Right Hip` and `Left Hip` rotated forward by `VanAnimController`), calling `target:Sit(hum)` computes `SeatWeld.C1 = CFrame.new(0, -0.76, 0) * CFrame.Angles(-math.pi/2, 0, 0)` from the bent-leg vertical extents instead of the canonical standing R6 `C1 = CFrame.new(0, -1.5, 0) * CFrame.Angles(-math.pi/2, 0, 0)`. Because `SwitchSeat` only restored `newSw.C0` without restoring `newSw.C1`, the driver's body and camera dropped down by `0.74` studs (`hrpY: 3.71 -> 2.97`, `camY: 5.41 -> 4.68`) upon landing in `DriverSeat`.
  2. Additionally, entering `DriverSeat` (`VehicleSeat`) caused Roblox's engine to switch `workspace.CurrentCamera.CameraSubject` from `Humanoid` to `DriverSeat`, ignoring `Humanoid.CameraOffset` (`CAB_FP_CAM_OFFSET`).
- **Changes Made (in live Roblox Studio via MCP):**
  - `Workspace.DeliveryVan.VanDrive`: Added `R6_SEAT_C1 = CFrame.new(0, -1.5, 0) * CFrame.Angles(-math.pi/2, 0, 0)` and `getCanonicalSeatC0(seat)`, enforcing both `C0` and `C1` in `bindCabSeatShift` and immediately on `newSw` after `target:Sit(hum)` in `SwitchSeat`.
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController`: Enforced `cam.CameraSubject = hum` and `hum.CameraOffset = CAB_FP_CAM_OFFSET` in `setFirstPersonLock(true)` and inside the first-person cab branch of `camConn` so switching into `DriverSeat` never switches `CameraSubject` to `VehicleSeat` or drops the camera.
  - `ServerStorage.Modules.Monsters.WindowMonster`, `ServerScriptService.Server.Services.MonsterService`, `ReplicatedStorage.Shared.NetDefs`, & `StarterPlayer.StarterPlayerScripts.Client.Controllers.WindowMonsterController`:
    - Converted window covers from a one-press toggle to a **hold-to-pull-up** mechanic (`NetController:FireServer("WindowCover", side, true/false)`, `NetDefs.WindowCover.MinInterval = 0.02`, `RateLimitPerSec = 16`).
    - Added `_tickCovers(dt)` in `WindowMonster.Update`: while holding `[Z]` / `[X]`, `c.Progress` advances at `1 / CoverPullUpSeconds` (`1.35s`) and the blind tweens linearly toward `1.0`. If the passenger releases the key before `c.Progress >= 1.0` (or leaves `FrontPassengerSeat`), `WindowCover<Side>` resets to `false` and the cover slides back down (`CoverDropSeconds = 0.75s`) without repelling. Once held all the way to `1.0`, `c.Up` latches to `true` for `CoverHoldSeconds = 8` and repels any threat on that side.
- **Repository Files Modified:** `DEVLOG.md`, `LESSONS.md`.
- **Next Steps / Known Issues:** Verified in live Play mode: early release at `0.55s` (`EarlyRelease_R: mid=0.38 -> dropped=0.00 (attr=false)`), full `1.48s` hold latch (`FullHoldLatch_L: top=1.00 -> afterRelease=1.00 (attr=true)`), and Passenger-to-Driver seat switch camera height (`P2D_CameraFix: beforeCamY=5.41 -> after(seat=DriverSeat,hrpY=3.71,headY=5.24,camY=5.42,subj=Humanoid,offY=0.05)`). Studio returned to `Edit` mode. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — justhyne1205 (via Antigravity)
- **Summary of Changes:**
  - Modeled 8 new modular HorrorKit-matching road formations & variants in Blender (`Assets/Roads/NewRoads.blend`, built at `1 stud = 0.28 m` matching `Road_Straight`'s `60`-stud width, `100`-stud cell, `0.85` deck / `1.20` curb profile, and `R=20` corner fillets): `Road_CrossIntersection` (`100×100`), `Road_Roundabout` (`100×100`), `Road_CulDeSac` (`92×100`), `Road_Bridge` (`61.1×100`), `Road_PulloffBay` (`86×100`), `Road_DamagedRough` (`60.7×100`), `Road_SCurve` (`160×200`, 2×2 cells), and `Road_WideTurn90` (`180×180`, 2×2 cells).
  - Exported all 8 as single-primitive meshes sharing one embedded 512×512 textured palette atlas (`Assets/Roads/NewRoads.glb`, `HorrorKit_RoadAtlas.png`), imported into Roblox Studio (`TextureID = rbxassetid://116369928377682`), scaled exact `1/0.28×` to exact stud sizes, set `CollisionFidelity = PreciseConvexDecomposition`, anchored, and lined them up beside the 4 existing roads in `Workspace.MapModels.HorrorKit_Roads`. Procedural map scripts were intentionally not modified yet.
- **Studio Paths Modified/Created:**
  - `Workspace.MapModels.HorrorKit_Roads.Road_CrossIntersection` (`rbxassetid://113709470661740`)
  - `Workspace.MapModels.HorrorKit_Roads.Road_Roundabout` (`rbxassetid://84769538109382`)
  - `Workspace.MapModels.HorrorKit_Roads.Road_CulDeSac` (`rbxassetid://78377713387098`)
  - `Workspace.MapModels.HorrorKit_Roads.Road_Bridge` (`rbxassetid://133376272696235`)
  - `Workspace.MapModels.HorrorKit_Roads.Road_PulloffBay` (`rbxassetid://125518970140506`)
  - `Workspace.MapModels.HorrorKit_Roads.Road_DamagedRough` (`rbxassetid://128238833357115`)
  - `Workspace.MapModels.HorrorKit_Roads.Road_SCurve` (`rbxassetid://71595213981898`)
  - `Workspace.MapModels.HorrorKit_Roads.Road_WideTurn90` (`rbxassetid://111725280710200`)
- **Repository Files Added/Modified:**
  - `Assets/Roads/NewRoads.blend`, `Assets/Roads/NewRoads.glb`, `Assets/Roads/HorrorKit_RoadAtlas.png` (created)
  - `LESSONS.md`, `DEVLOG.md` (modified)
- **New Dependencies:** Shared road atlas texture `rbxassetid://116369928377682` (plus the 8 MeshIds above; personal account `justhyne1205` — grant experience asset permissions in Creator Dashboard before publishing).
- **Next Steps / Known Issues:** Visually verified in Studio side-by-side with `Road_Straight` (`NewRoads_Overview`, `NewRoads_CloseComparison`). Ready to mirror into `ReplicatedStorage.Assets.MapModels.HorrorKit_Roads` and wire into `ProceduralRoadGraph` + `ChunkBuilder` when requested.

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Focus:** Van Tasks #5 & #6 — Disable Touch-to-Sit (Prompt-Only Seat Sitting with Tuned Distances) & Hide Van Prompts During `[V]` Chase Cam
- **Changes Made (in live Roblox Studio via MCP):**
  - `Workspace.DeliveryVan.VanDrive`:
    - **#5 Disable Touch-to-Sit + Seat ProximityPrompts on All 9 Seats:** Disabled automatic touch-to-sit across all 9 van seats (`DriverSeat`, `FrontPassengerSeat`, and all 7 rear cargo `PassengerSeat`s) by setting `seat.CanTouch = false` and `seat.Disabled = (seat.Occupant == nil)`, only enabling a seat inside `sitInSeat(seat, hum)` at the moment a seat prompt or `[C]` `SwitchSeat` sits the humanoid (and keeping `Disabled = false` while occupied so `DriverSeat` reads driving inputs). Added raised `SeatPromptAttachment` (`Vector3.new(0, 0.85, 0)`) and `SeatPrompt` (`ProximityPrompt`) to all 9 seats (`CAB_SEAT_PROMPT_DISTANCE = 6.5` on `DriverSeat`/`FrontPassengerSeat`, `CARGO_SEAT_PROMPT_DISTANCE = 5.5` on the 7 rear cargo `PassengerSeat`s, enabled whenever unoccupied and unblocked) so prompts reliably appear when standing near/over a seat without conflicting with opposite walls or door handles.
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController`:
    - **#6 Hide Van Prompts While Holding `[V]` Perspective:** Imported `ProximityPromptService` and updated `updateChaseCamState`, `leaveVanCamera`, and the `camConn` `RenderStepped` loop to set `ProximityPromptService.Enabled = false` while holding `[V]` (`Chase Cam`) and restore `ProximityPromptService.Enabled = true` when returning to first person or exiting the van.
- **Repository Files Modified:** `DEVLOG.md`.
- **Next Steps / Known Issues:** Verified in live Play mode: all 9 seats verified with `SeatPrompt` enabled and touch-to-sit disabled (`Seats=9, PromptsEnabled=9/9, NoTouch=9/9, DisabledEmpty=9/9`), standing directly over a rear cargo `PassengerSeat` does NOT auto-sit (`StandOverAutoSat=false`, `RearPromptDist=1.10/5.5`, `DriverDoorwayDist=3.42/6.5`), sitting via prompt works cleanly and hides the prompt while occupied (`AfterPromptSit: sat=true, ppEnabled=false`), and holding `[V]` in the van sets `ProximityPromptService.Enabled = false` (`HoldingV: keyDown=true ppsEnabled=false camType=Scriptable`). Zero game script errors. Studio returned to `Edit` mode; paused after #5–#6 awaiting user go-ahead for #7–#8. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — justhyne1205 (via Antigravity)
- **Summary of Changes:**
  - Mirrored all 7 active new HorrorKit road models (`Road_CrossIntersection`, `Road_Roundabout`, `Road_Bridge`, `Road_PulloffBay`, `Road_DamagedRough`, `Road_SCurve`, `Road_WideTurn90` — excluding `Road_CulDeSac`) into `ReplicatedStorage.Assets.MapModels.HorrorKit_Roads` (`PreciseConvexDecomposition` collision, object-pooled in `ChunkBuilder`).
  - Upgraded `ProceduralRoadGraph`, `ChunkBuilder`, and `MapService` to generate wider, longer multi-path looping road networks:
    - Detour loops now swing `2–3` cells (`200–300` studs) sideways and rejoin `5–7` cells ahead using both single-side `Road_TIntersection` splits and dual-side 3-path splits via `Road_Roundabout` and `Road_CrossIntersection` (zero dead ends; 100% of paths loop back to the main road).
    - Added full 2×2 multi-cell formation support (`WorldOffset`, `AnchorKey`, `MultiCellFiller` cells kept alive with their anchor in `MapService`) for sweeping `Road_SCurve` (`160×200`) and `Road_WideTurn90` (`180×180`) bends on both the main spine and wide detour corners.
    - Mixed `Road_Bridge`, `Road_PulloffBay` (with `BaySide`-aware infrastructure/forest avoidance), `Road_DamagedRough`, and `Road_Cracked` across straight segments, and added landmark lighting on `Road_Roundabout` and `Road_CrossIntersection`.
    - Kept 100% of delivery stop houses (`graph.Houses` + `DropoffPad`) strictly on straight main-spine cells while spawning optional decorative non-delivery `DecorHouse`s along side branches (with forest `avoidPoints` protection).
    - Extended `MapService` graph lookahead (`+22` steps, initial `28`) so 5–7 cell detour splits finalize completely before chunks build.
- **Studio Paths Modified/Created:**
  - `ReplicatedStorage.Assets.MapModels.HorrorKit_Roads.Road_CrossIntersection` .. `Road_WideTurn90` (7 MeshParts mirrored)
  - `ServerStorage.Modules.ProceduralRoadGraph` (modified)
  - `ServerStorage.Modules.ChunkBuilder` (modified)
  - `ServerScriptService.Server.Services.MapService` (modified)
- **Repository Files Modified:** `GAME_DESIGN.md`, `DEVLOG.md`.
- **New Dependencies:** None (uses existing `ProceduralMap` object pooling and road templates in `ReplicatedStorage.Assets.MapModels.HorrorKit_Roads`).
- **Next Steps / Known Issues:** Verified live in Play mode across 5 seeds × 100 steps (all 7 active new road pieces spawning: `20` Roundabouts, `22` CrossIntersections, `38` TIntersections, `45` WideTurn90, `35` SCurves, `93` Bridges, `101` PulloffBays, `122` DamagedRough, `38` decorative branch houses; `0` CulDeSacs, `0` dead ends, `0/149` branch delivery violations) and visually verified both `Road_SCurve` (`ProcMap_SCurveAndBridge`) and a live 3-path `Road_Roundabout` → `Road_CrossIntersection` dual loop (`ProcMap_3PathLoopOverview`) with pixel-perfect road seams, clear pull-off bays/bridges, dense dead forest, and zero console errors. Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create (`ProceduralRoadGraph`, `ChunkBuilder`, `MapService`).

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Focus:** Rear Cargo Passenger Seat `[C]` Switching to Looked-At / Mouse-Targeted Seat with Arm-Braced Bench Scoot Animation
- **Changes Made (in live Roblox Studio via MCP):**
  - `Workspace.DeliveryVan.VanDrive`:
    - Tagged all 7 rear cargo `PassengerSeat`s with `VanSeat = true` and `VanRearSeat = true`, wired `bindCabSeatShift(seat)` (using seat-local `Vector3.new(0, 0, -CAB_SEAT_SHIFT)` in `getCanonicalSeatC0(seat)` so both `-Z`-facing cab seats and `+X`-facing rear bench seats shift along their own forward `LookVector`), and replicated `VanSeatRole = "RearPassenger"` when occupied.
    - Updated `SwitchSeat` (`switchRemote.OnServerEvent:Connect(function(player, requestedTarget))`) to accept a target rear cargo `PassengerSeat`, computing distance-scaled duration (`0.56s..1.05s`) and multi-hop lift arcs (`hops = 1..3`, `hopSin = math.abs(math.sin(u * math.pi * hops))`, `liftY = 0.60 * hopSin`) across `SwitchSlideWeld` and replicating `VanSwitchFromLocal`, `VanSwitchToLocal`, and `VanSwitchHops`.
    - Updated `exitVanSeat` (`LeaveSeat`) so pressing `Space` in a rear cargo `PassengerSeat` steps the player `+1.85` studs inward into the interior cargo aisle (`relX ≈ -1.9`).
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController`:
    - Added `"RearPassenger"` to `SeatedRole` and implemented `findTargetSeatForSwitch(seat)` using both mouse-ray (`cam:ViewportPointToRay`) and camera-look raycast + cone alignment (`dot >= 0.78` or `perpDist <= 1.75` with `dot >= 0.40`) so pressing `[C]` in a rear cargo seat switches specifically to the unoccupied seat the player is looking at / pointing their mouse at (never a random seat).
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanAnimController`:
    - Added seated lap/floor limb posing (`sCF * CFrame.new(±0.60, 0.45, -1.10)` / `sCF * CFrame.new(±0.50, -1.35, -1.65)`) and first-person limb unveiling for `RearPassenger` seats, plus multi-hop rear-bench seat-switch arm bracing (`curWorld ± benchRight * 0.70 + upVec * 0.36 + inwardVec * 0.55 + pushDip`) and floorboard foot stepping (`stepArcR/L`).
- **Repository Files Modified:** `DEVLOG.md`.
- **Next Steps / Known Issues:** Verified in live Play mode: seated at rear cargo seat `z = -2.61`, aimed camera at `z = 5.65` (skipping 2 intermediate seats), pressed `[C]` -> landed directly on `seatRelZ = 5.65, hrp = (-3.62, 3.71, 5.65), role = RearPassenger`; then switched from `z = 5.65` to `z = 12.54` (`f0(z=5.65,y=3.71) -> f2(z=7.62,y=4.29,rY=3.07,lY=3.22) -> f4(z=11.67,y=4.10,rY=2.98,lY=2.73) -> f6(z=12.54,y=3.71,seatZ=12.54)`). Zero script console errors. Studio returned to `Edit` mode. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Focus:** Van Tasks #7 & #8 — Custom Circular ProximityPrompt UI + Accurate Handle/Switch/Seat Placements & 5 New Driver Interior Control Animations
- **Changes Made (in live Roblox Studio via MCP):**
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.PromptUIController` (created) & `StarterPlayer.StarterPlayerScripts.Client.Core.ControllerList`:
    - **#7 Custom Circular Proximity Prompt UI:** Created `PromptUIController` converting every `ProximityPrompt` in `Workspace` (and on `Workspace.DescendantAdded`) to `Enum.ProximityPromptStyle.Custom` and rendering a compact circular `BillboardGui` (`CircularPrompt_<Name>`) with a dark inner circle (`35×35 px`), centered keybind label (`E`, `F`, `R`, etc.), dual-arc `360°` `UIGradient` outer progress ring (`RightHalfClip` / `LeftHalfClip`) that fills smoothly during holds and flashes on trigger, and a compact action pill underneath (`ActionLabel`).
  - `Workspace.DeliveryVan.VanDrive`:
    - **#7 Accurate Prompt Attachments & Tight Distances:** Updated `prompt()` to parent prompts to centered `PromptAttachment`s on `Cab.DoorHandleL/R`, `SlidingDoor.DoorHandle`, and `RearDoors.RearHandleL/R` (`MaxActivationDistance = 5.5`), parented `WindowPromptL/R` to `SwitchPromptAttachment` on `CabDetail.Doors.WindowSwitch0` and `WindowSwitchR` on the door armrests (`MaxActivationDistance = 4.2`), and positioned `SeatPromptAttachment` right over the cushion center (`Vector3.new(0, 0.35, -0.15)`) on all 9 seats (`CAB_SEAT_PROMPT_DISTANCE = 5.5`, `CARGO_SEAT_PROMPT_DISTANCE = 4.8`).
    - **#8 Replicated Interior Control Targets & Pose Relay:** Created server-replicated attachments on the existing 3D interior parts inside `CabDetail`: `VanAnimHornPad` (on `Wheel.HubBody` at `Wheel.HornPad`), `VanAnimStalkL` (on `Wheel.StalkLTip` behind left side of wheel), `VanAnimShiftKnob` (on `Console.ShiftKnob`), `VanAnimHandbrake` (on `Console.HandbrakeGrip`), and `VanAnimDashBtn` (on `Dash.HazardBtn`). Extended `"Pose"` and `flushPose` to replicate `PoseDrift` and `PoseHandbrake` attributes on `DeliveryVan`.
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanAnimController`:
    - **#8 Driver Hand & Control Animations (FP + Replicated):**
      - **Honking (`[H]` / `HornOn`):** Right hand (`reachR`) leaves the 3-o'clock rim and presses into `VanAnimHornTip` at the center of the steering wheel (`(-2.09, 3.56, -8.66)`).
      - **Headlights (`[L]` / `HeadlightsOn`):** Left hand (`reachL`) reaches behind the left side of the wheel to flick `VanAnimStalkTip` (`(-3.10, 3.43, -8.81)`).
      - **Cab Light (`[K]` / `CabLightOn`):** Right hand (`reachR`) reaches forward to press `VanAnimDashTip` on the center dashboard (`(-0.61, 3.20, -8.61)`).
      - **Drifting (`[LeftShift]` / `PoseDrift`):** Right hand (`reachR`) grips `VanAnimShiftTip` (`Console.ShiftKnob`) and actively works the joystick with live steering and oscillation.
      - **Handbrake (`[B]` / `PoseHandbrake`):** Right hand (`reachR`) dynamically pulls and alternates between `VanAnimHandbrake` (`Console.HandbrakeGrip`) and `VanAnimShiftKnob` (`Console.ShiftKnob`).
- **Repository Files Modified:** `DEVLOG.md`.
- **Next Steps / Known Issues:** Verified in live Play mode: all 22 workspace `ProximityPrompt`s converted to `Custom` (`Prompts=22, Custom=22, VanAttachmentPrompts=16`), live circular prompt UI verified on `Cab.DoorHandleL.PromptAttachment` (`CircularPrompt_ProximityPrompt(key=E, act=Open door, ring=true)`), and driver hand animations verified (`Base: R(-1.55,3.61,-8.85) L(-2.87,3.61,-8.85)` -> `Horn[H]: R(-2.09,3.56,-8.66) dR=0.59` -> `Headlights[L]: L(-3.10,3.43,-8.81) dL=0.30` -> `CabLight[K]: R(-0.61,3.20,-8.61) dR=1.05`). Zero script console errors. Studio returned to `Edit` mode. All 8 van tasks (#1–#8) are now complete! Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Focus:** Replace Bottom-Left Driver Controls GUI with In-Car Circular ProximityPrompts on Cab Controls
- **Changes Made (in live Roblox Studio via MCP):**
  - `Workspace.DeliveryVan.VanDrive`:
    - Created in-car `DriverControl` `ProximityPrompt`s (`Style = Enum.ProximityPromptStyle.Custom`, `VanPromptRole = "DriverControl"`, `MaxActivationDistance = 4.6`, enabled only when `driverSeat.Occupant ~= nil`) mounted directly on the 3D interior controls inside `CabDetail`:
      - `HornPrompt` (`[H] Honk`) on `CabDetail.Wheel.HubBody.VanAnimHornPad`
      - `HeadlightsPrompt` (`[L] Headlights: ON/OFF`) on `CabDetail.Wheel.StalkLTip` behind the left side of the wheel
      - `CabLightPrompt` (`[K] Cab light: ON/OFF`) on `CabDetail.Dash.HazardBtn` on the center dashboard
      - `DriftPrompt` (`[SHFT] Drift`) on `CabDetail.Console.ShiftKnob`
      - `HandbrakePrompt` (`[B] Handbrake`) on `CabDetail.Console.HandbrakeGrip`
      - `BoostPrompt` (`[Q] Boost`) on `CabDetail.Wheel.SpokeBtn`
    - Also gated `WindowPromptL/R` `.Enabled` by cab seat occupancy (`refreshWindowPrompts`) so window switch prompts only enable when a cab seat is occupied.
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.PromptUIController`:
    - Added `isPromptAllowedForLocalPlayer` role filtering so `DriverControl` prompts are visible only to the seated driver and `WindowSwitch` prompts are visible only to the corresponding cab seat occupant, plus live golden ring highlight while holding a driver control key (`H`, `LeftShift`, `B`, `Q`).
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController`:
    - Disabled the bottom-left `VanControlsHint` text panel (`controlsPanel.Visible = false`) in favor of the in-car circular prompts on the physical cab controls.
- **Repository Files Modified:** `DEVLOG.md`.
- **Next Steps / Known Issues:** Verified in live Play mode: `BottomLeftGUI=false | VisibleCircularDriverPrompts(6)=CircularPrompt_BoostPrompt[Q:Boost] | CircularPrompt_CabLightPrompt[K:Cab light: OFF] | CircularPrompt_DriftPrompt[SHFT:Drift] | CircularPrompt_HandbrakePrompt[B:Handbrake] | CircularPrompt_HeadlightsPrompt[L:Headlights: ON] | CircularPrompt_HornPrompt[H:Honk]` (plus `CircularPrompt_WindowPromptL[R:Roll down window]` on the door armrest). Zero script console errors. Studio returned to `Edit` mode. Remember to **Commit Drafts** in Team Create.




---

## 2026-11-20 — justhyne1205 (via Antigravity)
- **Summary of Changes:**
  - Replaced the teleport-back-to-`(0,0)` shift reset with **endless forward Factory Depot progression** across `ProceduralRoadGraph`, `ChunkBuilder`, `MapService`, `ChuteService`, `ShiftService`, and `ShiftController`:
    - **Roadside Factory Depot Reservation & Relocation (`ProceduralRoadGraph.ReserveDepotAhead` + `MapService.ReserveAndRelocateDepot`):** When a shift ends (triggered **both** on completing all deliveries AND on the timer hitting `00:00`), reserves a clear `4×4` roadside lot `~6–8` cells ahead on a straight main-spine cell (clearing overlapping forest chunks and blocking future forest trees/guardrails/poles from the driveway), relocates the existing `Workspace.Factory` (`150` parts), `Workspace.StartArea` (`3,070` parts: chute, shops, decor), `Workspace.SpawnLocation`, and `Workspace.StartAreaGround` via a single `workspace:BulkMoveTo` call (zero 3,220-part cloning bloat), lays a flush `DepotDrivewayApron` across the curb, refreshes `ChuteService:RefreshBindings()`, and places a glowing green `DepotBeaconAnchor` + billboard inside the bay.
    - **Invisible Wall + Subtle Physical Barricade (`MapService:SetShiftGateActive`):** Spawns `Workspace.ShiftGateBarrier` (`ShiftGateWall` `130×36×4` invisible collision wall + 2 weathered steel/wood sawhorse barricades with slow-blinking amber warning beacons + 4 worn traffic cones) across the road just past the factory driveway exit (also active during initial Shift #1 Prep) so players can pull in and restock freely but cannot proceed until the next shift starts.
    - **All-Alive-Players Ready-Up Button (`ShiftService` + `ShiftController`):** Replaced the old `Return to Depot` teleport button and `Z < 135` auto-start with a compact `StartNextShiftBtn` HUD button during `Prep` (`ShiftAction = "ToggleReady"`, requiring every currently alive player `Humanoid.Health > 0` to ready up, displaying `0/N` / `Ready ✓ (R/N) — Cancel`, and automatically re-evaluating on `Humanoid.Died`, `CharacterAdded`, and `PlayerRemoving`). Once all alive players are ready, the gate drops and the next shift's delivery stops generate further ahead on the continuing road.
- **Studio Paths Modified/Created:**
  - `ServerStorage.Modules.ProceduralRoadGraph` (modified)
  - `ServerStorage.Modules.ChunkBuilder` (modified)
  - `ServerScriptService.Server.Services.MapService` (modified)
  - `ServerScriptService.Server.Services.ChuteService` (modified)
  - `ServerScriptService.Server.Services.ShiftService` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.ShiftController` (modified)
- **Repository Files Modified:** `GAME_DESIGN.md`, `DEVLOG.md`.
- **New Dependencies:**
  - `ShiftAction` action: `"ToggleReady"` (replaces `"ReturnToWarehouse"` teleport; `"StartShift"` aliases to ready toggle).
  - `ShiftState` snapshot fields added: `TimedOut`, `GateActive`, `ReadyCount`, `AliveCount`, `ReadyUserIds`.
  - Runtime workspace instances: `Workspace.ShiftGateBarrier`, `Workspace.DepotBeaconAnchor`, `Workspace.ProceduralMap.DepotDrivewayApron`.
- **Next Steps / Known Issues:** Verified end-to-end live in Play mode: Shift #1 initial Prep gate + `Start Shift #1 (0/1)` click dropped barrier cleanly; completed Stop #1 (`$100`) and let timer expire naturally at `00:00` → confirmed zero teleport back to `(0,0)`, Factory + StartArea + SpawnLocation + ChuteMouth relocated cleanly onto West roadside lot at `(460.9, -1109.6)` with `0` overlapping trees, `ShiftGateBarrier` (`39` parts, `wallCollide = true`) spawned past the driveway, visually verified via `RelocatedDepot_StreetView` and `RelocatedDepot_AerialView`, drove van into relocated bay → beacon cleared, `6` prompts intact inside relocated `StartArea`, `Start Next Shift (0/1)` appeared, clicking ready dropped `ShiftGateBarrier` and launched `SHIFT #2 (0/4 stops)` seamlessly down the continuing road with zero console errors. Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create (`ProceduralRoadGraph`, `ChunkBuilder`, `MapService`, `ChuteService`, `ShiftService`, `ShiftController`).

---

## 2026-11-20 — justhyne1205 (via Antigravity)
- **Summary of Changes:**
  - Sealed out-of-bounds player escapes at the starting factory (`MapService`): widened `ShiftGateWall` to `180` studs, added `StartAreaBoundary` with tall invisible wing-wall seals (`StartBayWingSeal_L` / `StartBayWingSeal_R` spanning `Z = 136..180`, `36` studs tall) closing the 4-stud side gaps between `Bay_WingWall_L/R` and `ShiftGateWall`, plus tall South/West/East invisible perimeter walls around the expanded `680×540` `StartAreaGround`.
  - Added `StartAreaForest` (`~215` dead trees across all 8 variants + dead grass clusters, growing denser and taller farther out, with zero trees inside the factory footprint) wrapping the left, right, and rear behind the starting factory; cleanly destroyed on Shift #2+ factory relocation (replaced by a right-sized `DepotGroundPad` cleanly behind the roadside anchor so grass never Z-fights across the road).
- **Studio Paths Modified/Created:**
  - `ServerScriptService.Server.Services.MapService` (modified)
- **Repository Files Modified:** `DEVLOG.md`.
- **New Dependencies:** Runtime workspace folders `Workspace.StartAreaForest`, `Workspace.StartAreaBoundary`, and `Workspace.ProceduralMap.DepotGroundPad`.
- **Next Steps / Known Issues:** Verified in Play mode: collision probes confirmed all 3 escape vectors blocked (`gapTest_EastPastWingL = StartBayWingSeal_L`, `gapTest_WestPastWingR = StartBayWingSeal_R`, `gapTest_NorthPastGate = ShiftGateWall`), `0` trees inside factory, visually verified full wrap-around dead forest behind and beside the factory (`StartFactory_ForestFullWrap`), zero console errors. Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create (`MapService`).

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Summary of Changes:**
  - Built **Monster #2 — The Road Chase Monster** using the tall, antlered deer-skull **Skinwalker model (`Monster6_Skinwalker`)**:
    - **Articulated 3D Skinwalker Model (`ReplicatedStorage.Assets.Monsters.Monster6_Skinwalker`):** Constructed a 135-part, 15-bone articulated Skinwalker rig (`ProceduralSkinwalkerRig = true`, `HipHeight = 4.5`) with a bleached deer skull, asymmetrical branching antlers (`13` beam/tine segments), hollow eye sockets with glowing neon eyes + `PointLight`, hinged needle-toothed lower mandible (`Jaw`), tattered dark pelt shroud over an exposed ribcage & dorsal spine (`Torso`), overly long clawed arms (`UpperArmL/R`, `ForearmL/R`, `HandL/R`), and digitigrade cloven-hooved legs (`UpperLegL/R`, `LowerLegL/R`, `FootL/R`).
    - **Server Chase Tug-of-War State Machine (`ServerStorage.Modules.Monsters.ChaseMonster` + `MonsterService`):**
      - Spawns `115` studs behind the van's rear bumper (`Spawning` → `Chasing` → `Ramming` / `Evaded`) and syncs replicated attributes on `Workspace.DeliveryVan` (`ChaseActive`, `ChaseState`, `ChaseDistance`, `ChaseLateral`, `ChaseSpeed`, `ChaseStartTime`, `ChaseEvent`, `ChaseEventTime`, `ChaseEventSeq`, `ChaseRamCount`).
      - Compares the van's signed forward velocity against the Skinwalker's sprint speed (`BaseChaseSpeed = 74` studs/s + shift scaling; van cruising top speed is `95` studs/s and Boost `[Q]` is `140` studs/s): closes in when the van slows down, stops, crashes, or is stunned, and falls back when the driver maintains high speed or boosts until evaded at `175` studs.
      - Includes **Horn Flinch Counterplay (`[H]`)** once per chase within `48` studs (`ChaseEvent = "HornFlinch"`, slowing the Skinwalker and pushing it back `+9` studs), on-foot road slash hazard for players caught outside the van, and a **Rear-Bumper Ram Climax (`ChaseDistance <= 2.5`)** that physically jolts the van chassis, deducts `10s` from the shift clock, briefly stuns the van, and pops open the rear cargo doors (`VanDrive` listener on `ChaseEvent == "Ram"`).
    - **Client 60 FPS Digitigrade Gallop Animator & Audio (`ChaseMonsterController` + `ControllerList`):**
      - Drives a 60 FPS 15-bone procedural sprint/gallop cycle behind the van via `workspace:BulkMoveTo` with downward road-surface raycasting so cloven hooves track ground height, forward-reaching talons as `ChaseDistance` drops under `40` studs, 3D spatial hoofbeat thuds + antler clicks + deer-skull bellows, rear-proximity tension vignette & HUD pursuit meter, and zero-drift camera tremor (while leaving `[V]` Chase Cam completely unrestricted so players can look back at the pursuing Skinwalker).
    - **Admin Testing Command (`chasemonster` / `cm` / `skinwalker`):** Added `ServerScriptService.Cmdr.Commands.chasemonster` and `chasemonsterServer` (`spawn`, `close`, `ram`, `evade`, `clear`).
- **Studio Paths Modified/Created:**
  - `ReplicatedStorage.Assets.Monsters.Monster6_Skinwalker` (created)
  - `ServerStorage.Modules.Monsters.ChaseMonster` (created)
  - `ServerScriptService.Server.Services.MonsterService` (modified)
  - `ServerScriptService.Cmdr.Commands.chasemonster` (created)
  - `ServerScriptService.Cmdr.Commands.chasemonsterServer` (created)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.ChaseMonsterController` (created)
  - `StarterPlayer.StarterPlayerScripts.Client.Core.ControllerList` (modified)
  - `Workspace.DeliveryVan.VanDrive` (modified — rear cargo doors pop open on `ChaseEvent == "Ram"`)
- **Repository Files Modified:** `DEVLOG.md`.
- **Next Steps / Known Issues:** Verified in live Play mode (`d0=80.0(Spawning) -> d1=70.1(Chasing) -> d2=46.9(Chasing)`, `ClientModel=SkinwalkerChaser3D (Parts=135)`, `RamReply=Forced Skinwalker Chase Monster rear-bumper ram attack. | Event=Ram | RamCount=1 | RearDoorPrompt='Close rear doors'`, `EvadeEvent=Evaded`). Zero console errors. Studio returned to `Edit` mode. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Summary of Changes:**
  - Overhauled the Road Chase Monster into a **Colossal Broken-Legged Skinwalker Crawler** larger than the delivery van (`~26.9 × 23.6 × 22.6` studs):
    - **Colossal Broken-Legged Rig with Wrapping Fingers (`ReplicatedStorage.Assets.Monsters.Monster6_Skinwalker`):** Rebuilt `Monster6_Skinwalker` as a 168-part, 17-bone articulated crawler (`ColossalCrawlerRig = true`, `HipHeight = 3.4`, `UpperArmLength = 11.5`, `ForearmLength = 12.5`) featuring a massive bleached deer skull & towering branching antlers, unhinging toothed `Jaw` exposing a glowing crimson `GulletCavern` / `GulletCoreGlow` throat cavity, huge 24-stud IK arms, articulated `FingersL`/`FingersR` bone groups (4 two-segment clawed fingers per hand that splay while reaching and curl around props/van), and fractured trailing hind legs (`UpperLegL/R`, `LowerLegL/R`, `FootL/R`).
    - **Forest Emergence & Woods Tree-Toppling Intro (`ChaseMonster` + `ChaseMonsterController`):** Spawns inside the roadside woods (`ChaseSpawnSide = "L" | "R"`, `ForestSpawnLateral = 42`) with a `2.3s` emergence phase (`ChaseState = "Spawning"`), unleashing an echoing forest roar + camera tremor and parting/toppling nearby trees crashing toward the highway (`triggerForestEmergence`, `ToppledTreesCount = 6` verified).
    - **2-Bone Analytical Arm IK, Environment Prop Grabbing & Ragdoll Dragging Legs (`ChaseMonsterController`):**
      - Scans nearby `ProceduralMap`, `StartAreaForest`, `StartArea`, and `Factory` parts ahead of each shoulder for **Houses/Buildings, Lamp Posts/Poles, Trees, and Guardrails** (with road-curb fallback), solves 2-bone analytical IK (`solveTwoBoneArmIK`) to plant the hand onto the surface, curls `FingersL`/`FingersR` around the target, and tilts/strains grabbed trees, poles, and guardrails as it hauls itself forward (`haulSurge` speed pulses in `ChaseMonster`).
      - Simulates `UpperLegL/R -> LowerLegL/R -> FootL/R` with a 60 FPS Verlet/pendulum ragdoll solver (`stepRagdollLeg`, `LegDragVerified = true`) clamped to road raycast elevation so both broken legs drag, fishtail, and bounce across the asphalt.
    - **"Grab the Van & Shove It Down Its Throat" Catch Climax (`ChaseMonster` + `ChaseMonsterController` + `chasemonster`):** On catch (`ChaseDistance <= 4.5` or `cm ram` / `cm devour`), both giant IK hands clamp onto the sides/roof of `DeliveryVan` (`GrabKindL = "Van"`, `GrabKindR = "Van"`, `-88°` finger curl), `Jaw` unhinges `-82°` to expose its throat, rear cargo doors fly open, and the van is yanked backward/upward before wrenching free (with `cm devour` alias added in `ServerScriptService.Cmdr.Commands.chasemonster` / `chasemonsterServer`).
- **Studio Paths Modified/Created:**
  - `ReplicatedStorage.Assets.Monsters.Monster6_Skinwalker` (rebuilt)
  - `ServerStorage.Modules.Monsters.ChaseMonster` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.ChaseMonsterController` (modified)
  - `ServerScriptService.Cmdr.Commands.chasemonster` (modified)
  - `ServerScriptService.Cmdr.Commands.chasemonsterServer` (modified)
- **Repository Files Modified:** `DEVLOG.md`.
- **Next Steps / Known Issues:** Verified in live Play mode via Cmdr (`chasemonster spawn 75`, `chasemonster devour`, `chasemonster clear`): confirmed forest spawn lateral ease (`41.97 -> 31.19`), `ToppledTrees=6`, 168-part colossal rig (`26.9 × 23.6 × 22.6` studs), environment prop IK tracking (`GrabL/GrabR`) & ragdoll leg dragging (`LegDrag=true`), devour climax (`State=Ramming`, `GrabL=Van`, `GrabR=Van`, `JawRelPitch=-82.0 deg`, `FingerRelCurl=-88.0 deg`, `RearAction='Close rear doors'`), and clean despawn/prop restore after `clear`. Zero script console errors. Studio returned to `Edit` mode. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Summary of Changes:**
  - Scaled the Colossal Skinwalker Chase Monster up ~2.5× larger (`46.5 × 44.0 × 127.4` template extents; `66.3 × 52.6 × 92.4` live posed extents) so it visibly dwarfs `DeliveryVan`, reaches deep into the treeline to grab trees/poles/houses, and physically lifts and shoves the van into its open throat instead of merely bumping the rear bumper:
    - **True Colossal Rig (`ReplicatedStorage.Assets.Monsters.Monster6_Skinwalker`):** Rebuilt with 172 descendants (`HipHeight = 8.2`, `UpperArmLength = 28.0`, `ForearmLength = 30.0`, `UpperLegLength = 15.0`, `LowerLegLength = 14.5`, `FootLength = 11.5`), a `14.5`-stud-wide deer skull with `44+`-stud branching antlers, a `13.5 × 12.5 × 14.0` `GulletCavern` throat cavity lined with `ThroatRibRing` arches, `8.8`-stud articulated `FingersL`/`FingersR` talons, and massive broken hind legs.
    - **Van-Grab & Throat-Devour Physics (`ChaseMonster` + `VanDrive` + `VanController`):**
      - Replaced the rear-bumper ram with a `2.4s` **Van Grab & Throat-Shove** sequence (`GrabDistance = 16.0`, pulling `ChaseDistance` from `12.0` down to `-5.5` studs inside the open throat).
      - Added `BoostTriggeredAt` tracking on `BoostPrompt.Triggered` (`VanDrive`) and cooperative `ChaseState == "Ramming"` lifting/backward-pulling in `VanController` (`vel = -look * 18 + Vector3.new(0, 7.5, 0)`) while leaving Boost (`[Q]`) unblocked by `VanStunned` during the grab so the driver can blast free early with `[Q]` Boost or `[H]` Horn; otherwise the crunch deals `35` damage and hurls the van free before rebounding to `68` studs.
    - **Colossal 58-Stud Arm IK & Emergence Scaling (`ChaseMonsterController`):** Expanded `triggerForestEmergence` (`110`-stud radius, up to `8` trees toppled/parted), `findEnvironmentAnchor` (`68`-stud scan ahead, `56.5`-stud IK reach, `18`-stud grab height, stronger `42°–48°` tree/pole bending), `-86°` jaw unhinge during devouring, and `-92°` claw wrap around the van roof/sides (`GrabKindL/R = "Van"`).
- **Studio Paths Modified/Created:**
  - `ReplicatedStorage.Assets.Monsters.Monster6_Skinwalker` (rebuilt at colossal scale)
  - `ServerStorage.Modules.Monsters.ChaseMonster` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.ChaseMonsterController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController` (modified)
  - `Workspace.DeliveryVan.VanDrive` (modified)
- **Repository Files Modified:** `DEVLOG.md`.
- **Next Steps / Known Issues:** Verified in live Play mode (`chasemonster spawn 90`, `chasemonster devour`, `chasemonster clear`): `Ext=66.3x52.6x92.4`, `Toppled=8`, `Grabs=House(BinBack),LampPost(LiftPost)`, `LegDrag=true`, `State=Ramming Dist=12.0->5.6`, `VanLift_dY=4.43`, `FwdDelta=-14.91`, `JawPitch=-86.0`, `FingerCurl=-92.0`, `Hands=Van/Van`, and zero script console errors. Studio returned to `Edit` mode. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Summary of Changes:**
  - Made the Colossal Skinwalker Chase Monster **much faster** and implemented full **Eat-the-Van** swallowing physics (`ChaseMonster`, `VanController`, `ChaseMonsterController`):
    - **High-Speed Pursuit (`ChaseMonster` + `ChaseMonsterController`):** Raised `BaseChaseSpeed = 104` studs/s (`MaxBaseChaseSpeed = 122`, `CatchupBonusMax = 16`, `StoppedClosingCap = 82`, `SpawnIntroSeconds = 1.25`), forcing active `[Q]` Boost / `[H]` Horn counterplay, and doubled the client IK crawl cadence (`crawlFreq = 2.4..4.4` cycles/s, fast reach phase `u < 0.30`).
    - **Full Eat-the-Van Sequence (`ChaseMonster` + `VanController`):**
      - During `ChaseState == "Ramming"`, the server temporarily takes physics ownership of `DeliveryVan` (`AlignPosition` + `AlignOrientation`, zero anchored parts) while `VanController` zeroes wheel motor torque and yields velocity overrides (`beingDevoured`) without blocking `[Q]` Boost break-free.
      - **`"Grabbing"` (`0.0s..1.0s`):** Lifts the van high up in front of the colossal open jaws (`LiftY ~ +18` studs) with a `0.25s..1.0s` Boost `[Q]` / Horn `[H]` break-free window during natural chases (`cm devour` forces full swallow).
      - **`"Swallowing"` (`1.0s..2.7s`):** Pitches the van nose-up (`+28°..+52°`) and shoves it rear-first through the teeth straight into `GulletCavern` (`ChaseDistance = 8.0 -> -10.0`).
      - **`"Swallowed"` (`2.7s..4.5s`):** Snaps the jaw shut (`JawPitch = -6°`), hides the swallowed van (`VanSwallowed = true` -> `LocalTransparencyModifier = 1` on `DeliveryVan`), deducts `25s`, kills players inside/near the van (`hum.Health = 0`), triggers `SwallowBlackout`, clutches/wipes around the bulging throat (`GrabKind = "Throat"`), and cleanly restores the van upright on the road upon departing (`_cleanupDevourConstraints`).
- **Studio Paths Modified/Created:**
  - `ServerStorage.Modules.Monsters.ChaseMonster` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.ChaseMonsterController` (modified)
- **Repository Files Modified:** `DEVLOG.md`.
- **Next Steps / Known Issues:** Verified in live Play mode: speed test closed `78.2` studs in `0.95s` (`101.5 -> 23.3`, `ChaseSpeed = 82.0`); full swallow curve verified across all 3 phases (`On_Grabbing: LiftY=18.02, JawPitch=-86.0`, `On_Swallowing: NoseY=0.45, AnchorZ=-28.1`, `On_Swallowed: VanSwallowed=true, VanLTM=1.0, JawPitch=-6.0, HandL=Throat, Dist=-10.0, AnchorZ=-8.5`), and clean constraint/visibility reset after `clear` (`AfterSw=false, AfterLTM=0.0, AfterAP=false`). Zero script console errors. Studio returned to `Edit` mode. Remember to **Commit Drafts** in Team Create.



---

## 2026-11-20 — justhyne1205 (via Antigravity)
- **Summary of Changes:**
  - **Expanded Procedural Road Generator with 6 Advanced 100%-Looping Macro Formations (`ProceduralRoadGraph`):**
    - Replaced the single repetitive rectangular bypass loop with a balanced, non-repeating library of 6 distinct macro formations using only existing `HorrorKit_Roads` (zero dead ends; `Road_CulDeSac` never spawned) and existing `HorrorKit_Houses` (`House_A_Cabin`, `House_B_Suburban`, `House_C_Ranch`) strictly as non-delivery decorative houses (`DecorHouse`) along side branches:
      1. `DeepForestWing` ("Out-and-Back Sweep"): `4–5` cells (`400–500` studs) deep into the dense woods around sweeping `Road_WideTurn90` bends with decorative cabins on outer & horizontal legs.
      2. `StaggeredZSplit`: asymmetrical dual-side branches where one side splits early (`gz0..gz0+5`) and the opposite side splits 3 cells later (`gz0+3..gz0+8`).
      3. `CrossroadsVillage`: compact `5×5` ghost-town neighborhood with alternating `Road_Roundabout` / `Road_CrossIntersection` hubs, parallel side streets, 3 cross-alleys, and `4–6` decorative abandoned houses.
      4. `LongBackroadBailouts`: `9`-cell-long forest backroad (`300` studs out) with sweeping `Road_WideTurn90` ends, cabins, and a mid-way `Road_Bridge` bailout lane back to the main road.
      5. `DoubleRoundaboutFigure8`: two asymmetric dual-loop `Road_Roundabout` / `Road_CrossIntersection` lobes separated by a center `Road_Bridge` straight stretch with alternating wide (`3`-cell) and tight (`2`-cell) sweeping outer arcs.
      6. `SCurveChicaneSplit`: curvy outer route chaining `Road_WideTurn90` + `Road_SCurve` + `Road_WideTurn90` paired against a shorter `Road_DamagedRough` / `Road_Cracked` inner spine road.
    - Widened lateral branch bounds (`BRANCH_MIN_GX = -11`, `BRANCH_MAX_GX = 11`), kept 100% of actual delivery stops (`graph.Houses` + `DropoffPad`) strictly on straight main-spine cells, extended house-eval buffer (`#SpineOrder - 13`) and `ReserveDepotAhead` lookahead (`+32`), and prevented `_straightRun` from counting `Turn90` or reusing prior formation cells.
  - **Decorative Yard Accents & Gentle Forest Tree Trim (`ChunkBuilder`, `MapService`):**
    - Added optional non-collidable `Prop_Mailbox_Rusted` and `Prop_WoodFence_Segment` yard accents to `DecorHouse` lots in `ChunkBuilder.BuildRoadChunk` (no `DropoffPad` or `StopHouseId`).
    - Gently reduced forest tree counts by a couple trees per chunk (~18–20% lighter for smoother FPS while keeping the woods thick and ominous): Ring 1 `4–6` (`minTrunkSpacing = 19`), Ring 2 `8–11` (`16`), Ring 3 `12–16` (`13`), and trimmed `MapService.StartAreaForest` from `215` → `175` trees. Kept progressive height scaling, solid deep-forest collision (`>= 95` studs), Ring 3 `CastShadow = false`, and outer `ForestBoundary` walls intact.
    - Extended `MapService` streaming window (`SPAWN_RADIUS = 850`, `DESPAWN_RADIUS = 950`, lookahead `currentSpine + 28`, initial `ExtendToStep(34)`).
- **Studio Paths Modified/Created:**
  - `ServerStorage.Modules.ProceduralRoadGraph` (modified)
  - `ServerStorage.Modules.ChunkBuilder` (modified)
  - `ServerScriptService.Server.Services.MapService` (modified)
- **Repository Files Modified:** `GAME_DESIGN.md`, `DEVLOG.md`.
- **New Dependencies:** None.
- **Next Steps / Known Issues:**
  - Verified with a 10-seed × 120-step graph audit: all 6 formations spawned evenly (`DeepForestWing=15`, `StaggeredZSplit=11`, `CrossroadsVillage=17`, `LongBackroadBailouts=11`, `DoubleRoundaboutFigure8=14`, `SCurveChicaneSplit=15`), `0` dead ends, `0` CulDeSacs, `0` branch delivery violations, `0` seam/mask mismatches, `373` main-spine delivery houses, `202` decorative branch houses, and `10/10` `ReserveDepotAhead` passes.
  - Verified live in Play mode with aerial `screen_capture`s of both `CrossroadsVillage` and `LongBackroadBailouts` (`StartAreaForest = 175` trees, `ProceduralMap` avg `8.6–8.7` trees/chunk, zero console errors). Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Summary of Changes:**
  - Upgraded the Colossal Skinwalker Chase Monster with **88-stud long-reach arms**, **much faster real pursuit speed**, deliberate **forward torso pull-lunges**, and a **full 3D destruction wake** (`Monster6_Skinwalker`, `ChaseMonster`, `ChaseMonsterController`):
    - **88-Stud Reach IK Arms & Claws (`ReplicatedStorage.Assets.Monsters.Monster6_Skinwalker`):** Rebuilt arms to `UpperArmLength = 42.0`, `ForearmLength = 46.0` (`172` descendants; `46.5 × 44.0 × 162.1` template extents; `108.1 × 63.7 × 100.4` live posed extents) with thickened `GiantPalm` (`6.0 × 2.2 × 7.8`), `ThumbTalon` (`8.5`), and `12`-stud articulated wrapping talons (`FingersL`/`FingersR`).
    - **Faster Pursuit & Lunge Velocity (`ServerStorage.Modules.Monsters.ChaseMonster`):** Increased `SpawnDistance = 160`, `EvadeDistance = 240`, `ForestSpawnLateral = 72`, `BaseChaseSpeed = 124` studs/s (`MaxBaseChaseSpeed = 142`, `CatchupBonusMax = 24`, `StoppedClosingCap = 105`, `SpawnIntroSeconds = 1.15`, `HornFlinchSlow = 46`, `HornFlinchPushback = 18`), with stronger haul lunge pulses (`haulSurge = 0.62..1.38`).
    - **Long-Reach Prop IK, Forward Pull-Lunge & 3D Destruction Engine (`ChaseMonsterController`):**
      - Updated `ARM_L1 = 42.0`, `ARM_L2 = 46.0`, `crawlFreq = 1.15..1.85` Hz, `20`-stud high reach arcs (`u < 0.34`), `10.5`-stud forward torso lunge surges on each pull, and `92`-stud prop scanning (`86`-stud max IK reach, `58`-stud road fallback).
      - Built 60 FPS `BulkMoveTo` client debris (`MAX_DEBRIS = 90` with gravity & spin) and fading 3D destruction scars (`MAX_SCARS = 36`) across houses (`HouseBreachHole`, `BrokenClawGash`, `3.5°` jolt, flying planks/masonry), trees (flat snap + bark splinters), lamp posts/poles (`80°–88°` snap + Neon spark streaks + flash light), guardrails (`68°–84°` tear + metal shards), and road surfaces (`RoadCrater`, `UpturnedAsphaltSlab`, continuous `22`-stud wake scarring & `0.18s` wake destruction sweep), all cleaned up cleanly on `clear` / despawn.
- **Studio Paths Modified/Created:**
  - `ReplicatedStorage.Assets.Monsters.Monster6_Skinwalker` (rebuilt with 88-stud arms)
  - `ServerStorage.Modules.Monsters.ChaseMonster` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.ChaseMonsterController` (modified)
- **Repository Files Modified:** `DEVLOG.md`.
- **Next Steps / Known Issues:** Verified in live Play mode (`Ext = 108.1 × 63.7 × 100.4`, `Toppled = 8`, `Grabs = House(Window_East_Upper_03), Road(RoadCurb)`, `Scars = 5 (24 parts)`, `Debris = 90`, `Props = 5` -> clean `Scars = 0`, `Debris = 0` on `clear`, and active `"Chasing"` closing speed verified at `135.8 -> 37.3` = `98.5` closed in `1.01s` / `97.5` studs/s rate, `maxSpd = 105.0`). Zero script console errors. Studio returned to `Edit` mode. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — justhyne1205 (via Antigravity)
- **Summary of Changes:**
  - **Cleaned Up Baked Static Clutter on `TrackerScreen` (`Workspace.DeliveryVan.CabInterior.TrackerScreen.SurfaceGui.Screen.Radar`):**
    - Removed the 10 fake baked-in decorative clutter instances (`6` fake static green dots, `2` fake static red dots, fake `"UNK SIGNAL"` label, and frozen `42°` green diagonal line) while preserving the radar rings, crosshairs, compass ticks, `N / E / S / W` labels (renamed to `CompassN/E/S/W`), and `DashController`'s rotating `Sweep`.
  - **Upgraded In-Van GPS Radar to Real-Time Heading-Up Mode (`ShiftController` & `ShiftService`):**
    - Switched radar math in `ShiftController` to **Heading-Up** orientation updated smoothly every frame on `RenderStepped` (zero per-frame allocations): projects target offsets into the van's horizontal `(Forward, Right)` frame (`Forward = up`, `Right = right`), keeps a crisp pure-Frame cyan-white upward GPS chevron (`VanArrow`) centered at `0°` (avoiding missing Unicode font glyphs on `SurfaceGui`), and orbits `CompassN/E/S/W` smoothly around the radar rim as the van turns.
    - **Active Shift — Next Delivery House:** Renders the current active delivery stop as a bright pulsing **green dot** (`RGB 65, 255, 115`) with a crisp mint-white stroke and pulsing `TargetPing` ring (clamped cleanly to the outer rim when beyond `RADAR_RANGE = 900` studs).
    - **Active Shift — Remaining Stops:** Added `RemainingStops` to `ShiftService`'s `ShiftState` snapshot (plus automatic re-broadcast when a streamed `DropoffPad` refines `PadWorldPos`) and rendered remaining uncompleted stops as smaller, dimmer green dots (`RemainingDot_1..N`) so drivers/navigators can read upcoming stops at a glance.
    - **Return Phase — Next Factory Depot:** Renders the relocated Next Factory Depot as a distinct pulsing **gold dot** (`RGB 255, 205, 55`) with matching gold ping ring while hiding remaining stop dots.
    - **Prep Phase:** Cleanly hides all target dots and ping rings until the shift begins.
- **Studio Paths Modified/Created:**
  - `Workspace.DeliveryVan.CabInterior.TrackerScreen.SurfaceGui.Screen.Radar` (modified — removed 10 fake static clutter elements, named `CompassN/E/S/W`)
  - `ServerScriptService.Server.Services.ShiftService` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.ShiftController` (modified)
- **Repository Files Modified:** `GAME_DESIGN.md`, `DEVLOG.md`.
- **New Dependencies:**
  - `ShiftState` snapshot field: `RemainingStops` (`{ { HouseId: number, Address: string, Pos: Vector3 } }`).
- **Next Steps / Known Issues:**
  - Verified live in Play mode across `Prep` (`TargetDotVis=false`, `RemVis=0`, `CompassE` at top while parked facing East), `Active` at East/North/West headings (confirmed accurate Heading-Up dot projection & rotating `N/E/S/W`, bright green active dot + 2 smaller dimmer remaining stop dots + crisp pure-Frame cyan-white van chevron via in-cab `screen_capture`), and `Return` (confirmed pulsing gold `NEXT FACTORY DEPOT` dot with `RemVis=0` via in-cab `screen_capture`). Zero console errors. Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Summary of Changes:**
  - **Fixed Van Seats Prematurely Kicking Players Out Across All 9 Seats (`VanDrive`, `VanController`, `AntiExploitService`):**
    - **Root Cause:** Keeping empty seats at `seat.Disabled = true` and toggling `seat.Disabled = false` on the exact same frame as `seat:Sit(hum)` triggered a Roblox client/server replication & ownership race that broke `SeatWeld`, causing `Occupant` listeners to immediately slam `seat.Disabled = true` and eject players. Additionally, `DeliveryVan.ModelStreamingMode` defaulted to `Default` under `Workspace.StreamingEnabled = true`, and `VanController:LeaveSeat()` lacked a post-sit grace window and `VanSwitchActive` guard.
    - **`Workspace.DeliveryVan.VanDrive`:** Registered non-collidable `"VanSeat"` `CollisionGroup`, added `configurePromptOnlySeat(seat)` keeping `seat.Disabled = false` permanently while disabling touch-to-sit via `CanTouch = false`, `CanCollide = false`, and stripping any `TouchTransmitter`; cleared `hum.PlatformStand` in `sitInSeat`; added `exitToken` + `switchingPlayers` guard in `exitVanSeat`; enforced `DeliveryVan.ModelStreamingMode = Enum.ModelStreamingMode.Persistent` (`Edit` + runtime).
    - **`StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController`:** Added `seatedAt` tracking and a `0.35s` post-sit grace window + `VanSwitchActive` guard in `LeaveSeat()`.
    - **`ServerScriptService.Server.Services.AntiExploitService`:** Exempted `VanSwitchActive == true` from speed-cap flags during `[C]` seat switches.
  - **Made Colossal Skinwalker Chase Monster Much Faster so Players Must Use Boost `[Q]` (`ChaseMonster`, `VanController`, `ChaseMonsterController`):**
    - **`ServerStorage.Modules.Monsters.ChaseMonster`:** Raised `BaseChaseSpeed = 134` studs/s (`MaxBaseChaseSpeed = 148`, `ShiftSpeedStep = 3.0`, `CatchupBonusMax = 32` easing smoothly above `vanForwardSpeed > 105`, `CloseMercySlow = 0`, `StoppedClosingCap = 155`, `SpawnIntroSeconds = 0.85`, `SpawnDistance = 155`, `EvadeDistance = 225`) and tightened `haulSurge = 0.88..1.12x` so its sprint speed never dips below the van's `MAX_SPEED = 95`.
    - **`VanController` & `ChaseMonsterController`:** Upgraded Boost `[Q]` to `BOOST_SPEED = 158`, `BOOST_PUSH = 58`, `BOOST_DRAIN = 24` (`~4.17s` tank while keeping `MAX_SPEED = 95`), and updated client crawl cadence (`crawlFreq = 1.35..2.40` Hz, `pullSurgeZ = 12.5`).
- **Studio Paths Modified/Created:**
  - `Workspace.DeliveryVan` (`ModelStreamingMode = Enum.ModelStreamingMode.Persistent`)
  - `Workspace.DeliveryVan.VanDrive` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController` (modified)
  - `ServerScriptService.Server.Services.AntiExploitService` (modified)
  - `ServerStorage.Modules.Monsters.ChaseMonster` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.ChaseMonsterController` (modified)
- **Repository Files Modified:** `DEVLOG.md`, `LESSONS.md`.
- **Next Steps / Known Issues:** Verified end-to-end in live Play mode: all 9 seats confirmed `Disabled=false, CanTouch=false, CanCollide=false, CollisionGroup="VanSeat", TouchTransmitter=nil, StreamingMode=Persistent` with walk-over anti-touch (`WalkOverSeatPart=nil` on cab & rear seats), zero kicks across `DriverSeat` (`1.0s` idle + `1.56s` moving + `12s` Server/Client trace), `[C]` `SwitchSeat` (`DriverSeat -> FrontPassengerSeat`), moving rear `PassengerSeat` (`95` studs/s, `0.8s`, `kicks=0`), and intentional `LeaveSeat`; chase speed verified parked (`135.3–155.0` studs/s), `95` studs/s cruising (`140.7–170.2` studs/s closing `123.0 -> 95.6`), and `158` studs/s Boost (`128.0–140.9` studs/s pulling away `135.0 -> 150.5` with `seat=DriverSeat`). Zero script console errors. Studio returned to `Edit` mode. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Summary of Changes:**
  - **Reduced `WindowMonster` Spawn Frequency (`ServerStorage.Modules.Monsters.WindowMonster`):**
    - Increased `Config.FirstThreatDelay` (`18` -> `45`s), `AfterThreatDelayMin` (`14` -> `38`s), and `AfterThreatDelayExtra` (`14` -> `27`s), making Shift 1 spawns occur every `38–65s` (up from `14–28s`).
    - Updated `_scheduleNext()` cooldown scaling (`shrink = math.min(12, (shiftNumber - 1) * 3)`) with a higher minimum cooldown floor (`26`s vs `8`s, giving `26–53s` by Shift 5+).
    - Paused the spawn countdown in `Update()` whenever `DeliveryVan:GetAttribute("ChaseActive") == true` so the Window Monster never overlaps an active Road Chase Monster pursuit.
- **Studio Paths Modified/Created:**
  - `ServerStorage.Modules.Monsters.WindowMonster` (modified)
- **Repository Files Modified:** `DEVLOG.md`.
- **Next Steps / Known Issues:** Verified in live Play mode (`Config: First=45 Min=38 Extra=27 | Shift1=[38.06..64.95] | Shift5=[26.11..52.94] | PauseTest: init=45.0 duringChase=45.0 afterChase=40.0`) with zero game script console errors; Studio returned to `Edit` mode. Remember to **Commit Drafts** in Team Create.



---

## 2026-11-20 — Riplatki (via Claude)
- **Focus:** Van seat-switch bugs: (1) van jumps when switching seats with `[C]`, (2) camera snaps forward when switching to the driver seat.
- **Changes Made (in live Roblox Studio via MCP):**
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController` (modified): added a `VanSwitchLookKeep` render step (`Camera.Value + 1`). While the `[C]` slide runs (`VanSwitchActive == true` and `hum.SeatPart == nil`) it stores the camera look direction in chassis space, then re-applies it for `0.4s` after landing. **Camera snap root cause:** Roblox's `VehicleCamera` forces the view to yaw `0` / pitch `-18` whenever the avatar becomes seated in a `VehicleSeat` (`DriverSeat`); `FrontPassengerSeat` is a plain `Seat` so only passenger -> driver snapped. **Verified in Play mode** (yaw kept within about 0.5 degrees at +50 / -70 in both directions, e.g. `-70 -> -70.4`, `50 -> 49.6`).
  - `Workspace.DeliveryVan.VanDrive` (modified): in the `SwitchSeat` handler, all character parts are put in the `VanSeat` collision group and set `Massless = true` for the duration of the slide, restored `0.2s` after landing. **This did NOT fix the jump** (see below). It is harmless but unproven, so remove it if it turns out to be useless.
- **Van jump: root cause NOT found yet.** Server-side sampling of `DeliveryVan.PrimaryPart` during a parked switch showed the van lifting about `1.4` studs (peak `vy` roughly `6-8` studs/s, `av` up to about `0.7`) within the first ~0.6s of the slide, in BOTH directions (driver -> passenger and passenger -> driver). The player owns the van's network ownership (`GetNetworkOwner() == player`). The same size of jump happened before and after the collision-group/massless change, so character-limb depenetration and character mass are ruled out. Remaining suspects, not yet tested: (a) `hum.Sit = false` + `sw:Destroy()` at slide start letting the Humanoid state machine apply a jump/getting-up velocity to the welded `HumanoidRootPart` (which moves the whole van assembly), (b) `startDriveSession` / `stopDriveSession` in `VanController` on seat change (`restore()`, motor torque, drive loop velocity writes), (c) network ownership handoff. Suggested next test: while seated, keep the `SeatWeld` alive and slide with `SeatWeld.C0` instead of unseating, or `hum:ChangeState(Enum.HumanoidStateType.Physics)` during the slide, and A/B each against the same sampler.
- **Studio Paths Modified/Created:**
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController` (modified)
  - `Workspace.DeliveryVan.VanDrive` (modified)
- **New Dependencies:** None (reuses existing `VanSeat` collision group and `VanSwitchActive` player attribute).
- **Next Steps / Known Issues:** Van jump on `[C]` still open (see above). The Studio MCP connection dropped at the end of the session, so confirm both scripts above saved in Studio (the `VanController` edit was applied and tested in Play; the `VanDrive` edit was applied in Edit mode). Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Summary of Changes:**
  - **Big, Chaotic, Ultra-Long-Reach Movement & Near-Miss Swipes (`StarterPlayer.StarterPlayerScripts.Client.Controllers.ChaseMonsterController`):**
    - **Velocity-Lead Ultra-Long Forward Arm Scanner (`findEnvironmentAnchor`):** Added shoulder forward velocity prediction (`leadDist = math.clamp(worldSpeed * reachDuration, 18, 56)`), forward-biased corner/roof clamping (`reachAimPos = predictedShoulder + forwardDir * 24`), `28–87` stud forward reach filtering scored heavily toward farthest forward distance (`fwdProj * 1.65`), and a `78`-stud forward road curb fallback so both `88`-stud arms always reach aggressively out in front even at `134–170+` studs/s pursuit speed.
    - **Desperate "Just Barely Missing" Van-Swipe Lunges (`isSwipeLunge`):** Within `96` studs during `"Chasing"`, alternating arms execute a wide haymaker windup (`u < 0.26`, `±42–56` studs lateral), blast across the rear bumper `3.5–8.0` studs behind the van (`0.26 <= u < 0.72`, `sx * 38 -> -sx * 16` lateral sweep at full up to `87`-stud extension with wind-whoosh SFX and snapped claw curl), and overshoot-crash into buildings/trees/poles/asphalt (`u >= 0.72`) with heavy thuds, camera shake, and stumble recoil.
    - **Chaotic Stumbling Torso, Head & Building Impacts:** Slowed sweep cadence to weighty readable arcs (`crawlFreq = math.clamp(effectiveSpeed * 0.010, 1.05, 1.55)` Hz) paired with `16.5–25.5` stud forward pull lunges, multi-harmonic lateral staggering (`chaosLat`), damped spring stumble physics (`stumbleLat`, `stumbleRoll`, `stumblePitch`), violent shoulder-twisting torso yaw (`±28°`) / roll (`±22°`) / pitch (`-16°±11°`), widened `42`-stud house/wake crash radius, `6.5°` house structural jolts + extra plank/brick debris, and clean rip-loose release when blasting past planted anchors (`relAnchorFwd < -16`).
- **Studio Paths Modified/Created:**
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.ChaseMonsterController` (modified)
- **Repository Files Modified:** `DEVLOG.md`.
- **Next Steps / Known Issues:** Verified in live Play mode (`chasemonster close 85` & `close 48` while driving at `95` studs/s): `MaxForwardReach = 97.8` & `95.7` studs ahead of `Root` (`fwdL=78.1`, `fwdR=89.2`), `SwipeMissCount = 1` (`L=SwipeMiss` alternating with `R=Tree`/`Guardrail`/`LampPost`), `BuildingStumbles = 2`, `DestroyedPropsCount = 9`, `seat=DriverSeat` held solidly throughout, and zero game script console errors; Studio returned cleanly to `Edit` mode (`6807141d-6567-4853-a980-bec7b2f2d9ee`). Remember to **Commit Drafts** in Team Create.


---

## 2026-11-20 — justhyne1205 (via Antigravity)
- **Summary of Changes:**
  - **Creeping Trailing Fog & Fallen-Tree Debris Barrier (`MapService`):**
    - Tracked farthest forward spine progress (`maxReachedSpine`) and added a creeping `Workspace.BackFogBarrier` positioned `4` spine cells behind `maxReachedSpine` (activates once `maxReachedSpine >= 6`, always ahead of `DESPAWN_SPINE_BACK = 6` so players never see bare/despawned grass or void behind them).
    - Built `BackFogBarrier` from a tall invisible collision wall (`BackFogWall`, dynamically widened across parallel multi-path road formations so nobody can detour around it), 4 layered vertical-gradient dark-teal fog curtains (`SurfaceGui` + `UIGradient` dissolving cleanly to `1.0` transparent at the top with zero hard rectangular edge against the night sky), billowing dark `DarkFogMist` + `GroundMist` `ParticleEmitter`s, 5 fallen/sprawled dead trees (`HorrorKit_Props`), and mangled broken guardrails / wood fence splinters / trash heaps / tilted chevron signs (`HorrorKit_Infrastructure` / `HorrorKit_Props`). Kept purely diegetic with zero radar/HUD bump warnings.
    - Protected all chunks between `backBarrierSpine - 2` and the van from despawning, raised `MAX_SPAWNS_PER_TICK` (`5` → `8`, boosted to `14` while reversing), exposed `MapService:GetBackBarrierSpine()` / `GetMaxReachedSpine()`, and destroyed leftover `Workspace.StartAreaGround` in `ReserveAndRelocateDepot()` so no empty grass field remains behind after Shift #1.
  - **"No Turning Back" Missed Delivery Handling (`ShiftService` & `ShiftController`):**
    - Stored `SpineIndex` and `Missed` on each `StopRecord` and checked `MapService:GetBackBarrierSpine()` during `Phase == "Active"`: when the trailing fog wall advances past an active delivery house, that stop is automatically marked **MISSED** (forfeits its `$100` payout and shift on-time bonus while keeping completed stops paid), broadcasts `"MISSED STOP: <Address> — swallowed by the fog!"`, clears its beacon/prompt, and immediately advances `activeStopIndex` to the next stop (or triggers `Return` / Next Factory Depot approach if it was the final stop) so a shift can never soft-lock.
    - Excluded `Missed` stops from `RemainingStops` so the in-van `TrackerScreen` radar immediately removes missed stops and locks onto the next stop (or gold depot dot).
- **Studio Paths Modified/Created:**
  - `ServerScriptService.Server.Services.MapService` (modified)
  - `ServerScriptService.Server.Services.ShiftService` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.ShiftController` (modified)
- **Repository Files Modified:** `GAME_DESIGN.md`, `DEVLOG.md`.
- **New Dependencies:**
  - Runtime Model: `Workspace.BackFogBarrier` (`BackFogWall`, `BackFogSheet_1..4`, `BackFogEmitterBar`, fallen dead trees & roadblock debris).
  - `MapService` API: `MapService:GetBackBarrierSpine()`, `MapService:GetMaxReachedSpine()`.
  - `ShiftState` stop field: `Missed` (boolean).
- **Next Steps / Known Issues:**
  - Verified live in Play mode: `BackFogBarrier` stayed `nil` at start, spawned cleanly ~4 cells behind the van (`Parts=18`, `Emitters=2`, `FallenTrees=5`, `Debris=7`, dynamically widened `WallSize=(710, 44, 8)` across a multi-path formation), automatically marked overshot stops MISSED and advanced cleanly (`583 OLD MILL RD` → `984 OLD MILL RD` → `Return` `NEXT FACTORY DEPOT` with gold radar dot & `RemDots=0`), destroyed `Workspace.StartAreaGround` (`nil`) on depot relocation, visually verified the gradient-faded dark fog + fallen trees/debris via road-level `screen_capture` (zero hard sky edge, zero bare grass), and confirmed zero console errors. Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Summary of Changes:**
  - **Replaced Random Road Chase Timer with Condition-Only Spawning (`ShiftService`, `MonsterService`, `ChaseMonster`):**
    - **`ServerScriptService.Server.Services.ShiftService` & `MonsterService`:** Added `ShiftService:GetMonsterContext()` exposing live `Phase`, `TimeRemaining`, `CompletedStops`, `TotalStops`, `TimedOut`, `MissedAnyStop`, and `IncompleteShiftEnd`, and passed `ctx` into `MonsterService`'s `m:Update(dt, active, shiftNumber, ctx)` loop.
    - **`ServerStorage.Modules.Monsters.ChaseMonster`:** Removed `FirstChaseDelay` / `AfterChaseDelayMin` / `AfterChaseDelayExtra` random spawns and replaced them with 3 strict natural triggers (plus `ChaseSpawnReason` attribute + `PostChaseCooldown = 18s` breather; pauses while `WindowThreatSide`/`WindowApproachSide` is active; resets delivery delay & stillness on each completed delivery):
      1. **Standing Still Too Long (`"StandingStill"`):** Triggers after `StillTriggerSeconds = 22`s below `StillSpeedThreshold = 10` studs/s (warning at `15s`; drains `3.5×` while driving `>= 18` studs/s).
      2. **Taking Too Long (`"TakingTooLong"` / `"LowTime"`):** Triggers after `StopDelayTriggerSeconds = 70`s without completing a delivery (warning at `55s`), or once per shift when `TimeRemaining <= 35`s (`LowTimeTriggerSeconds`) with packages still undelivered.
      3. **Incomplete Shift End (`"IncompleteShift"`):** Triggers immediately upon entering `"Return"` when `TimedOut`, `MissedAnyStop`, or `CompletedStops < TotalStops` to hunt the van toward the Next Factory Depot, resetting cleanly upon reaching `"Prep"` — while clean on-time completed shifts never spawn the Chaser on the depot drive.
- **Studio Paths Modified/Created:**
  - `ServerScriptService.Server.Services.ShiftService` (modified)
  - `ServerScriptService.Server.Services.MonsterService` (modified)
  - `ServerStorage.Modules.Monsters.ChaseMonster` (modified)
- **Repository Files Modified:** `DEVLOG.md`, `GAME_DESIGN.md`.
- **New Dependencies:** `ShiftService:GetMonsterContext()`; replicated `DeliveryVan` attribute `ChaseSpawnReason`.
- **Next Steps / Known Issues:** Verified all 7 scenarios in live Play mode (`1_FastDrivingNoSpawn(act=false)`, `2_Still22s(act=true,reason=StandingStill)`, `3_TooLong70s(act=true,reason=TakingTooLong)`, `4_LowTime(act=true,reason=LowTime)`, `5_IncompleteReturn(act=true,reason=IncompleteShift)->7_DepotPrep(act=false)`, `6_CleanCompleteReturn(act=false)`, `LiveVan_ChaseSpawnReason=""`), zero game script console errors, Studio returned to `Edit` mode. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Summary of Changes:**
  - **Rebalanced Chase Monster Speed Just Below Full Van Speed (`ChaseMonster`, `ChaseMonsterController`):**
    - **`ServerStorage.Modules.Monsters.ChaseMonster`:** Re-tuned pursuit speeds against `VanController`'s `MAX_SPEED = 95` and `BOOST_SPEED = 158` so the Skinwalker closes in whenever the van stops, corners, or cruises under top speed, while steadily losing ground when the player holds full speed (`95` studs/s) or hits Boost `[Q]`:
      - Set `Config.BaseChaseSpeed = 88`, `ShiftSpeedStep = 1.0`, `MaxBaseChaseSpeed = 93`, `CatchupBonusMax = 14`, `StoppedClosingCap = 95`, `SpawnDistance = 125`, and `EvadeDistance = 195`.
      - Updated the `Chasing` rubberband in `ChaseMonster:Update` so `distBonus` (beyond `80` studs) eases smoothly to `0` across `vanForwardSpeed = 78..92` (`fullSpeedEase`), and tuned `haulSurge` to `0.90 + 0.20 * (math.sin(self._elapsed * 5.2) ^ 2)` (mean `1.0×`).
    - **`StarterPlayer.StarterPlayerScripts.Client.Controllers.ChaseMonsterController`:** Updated client fallback `chaseSpeed` to `88` and tuned `crawlFreq` to `math.clamp(effectiveSpeed * 0.012, 1.05, 1.50)` so the sweeping hand-over-hand cadence remains weighty at the new pursuit speed.
- **Studio Paths Modified/Created:**
  - `ServerStorage.Modules.Monsters.ChaseMonster` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.ChaseMonsterController` (modified)
- **Repository Files Modified:** `DEVLOG.md`.
- **New Dependencies:** None.
- **Next Steps / Known Issues:** Verified in Play mode across `Shift 1`, `Shift 5`, and `Shift 6`: from `110.00` studs on `Shift 1`, stopped (`0`, `1s`) closed to `22.31` (`-87.69`), `75` studs/s (`5s`) closed to `37.33` (`-72.67`), `84` studs/s (`5s`) closed to `79.00` (`-31.00`), full speed `95` studs/s (`5s`) opened to `144.67` (`+34.67`), and Boost `158` studs/s from `SpawnDistance = 125.00` hit `Evaded` at `196.27` within `2s`. On late shifts, `Shift 5` cornering at `84` studs/s (`5s`) closed to `62.81` (`-47.19`), while holding full speed `95` studs/s still pulled ahead (`Shift 5`: `124.65`, `+14.65`; `Shift 6`: `119.65`, `+9.65`). Zero game script console errors; Studio returned to `Edit` mode. Remember to **Commit Drafts** in Team Create.


---

## 2026-11-20 — justhyne1205 (via Antigravity)
- **Summary of Changes:**
  - **Fixed Sidepath Bypass Around Factory `ShiftGateBarrier` (`MapService` & `ProceduralRoadGraph`):**
    - Widened `ShiftGateWall` from `180` studs to full map-corridor width (`2048 × 48 × 8` studs, Roblox's max single-part span covering `±1024` studs across all lanes, sidepaths, and forest margins) so neither the van nor players on foot can ever bypass the shift gate before clicking **"Start Next Shift"**.
    - Added automatic matching sawhorse barricades & traffic cones across any parallel branch road cell intersecting the gate's `Z` plane, and updated `ProceduralRoadGraph:ReserveDepotAhead()` with a clean Pass-1 preference to place relocated Factory Depots on single-road straight stretches with zero parallel sidepaths (`20/20` clean single-road placements across a 10-seed audit).
  - **Pushed `BackFogBarrier` Further Back & Protected Active Factory Depots (`MapService`):**
    - Increased `BACK_BARRIER_LAG` from `4` → `8` cells (`~800+` studs behind `maxReachedSpine`) and `BACK_BARRIER_MIN_SPINE` from `2` → `3` (activates after `maxReachedSpine >= 11`) so the dark fog never feels claustrophobic or chases right behind the van's bumper.
    - Raised `SPINE_BACK` (`5` → `9`), `DESPAWN_SPINE_BACK` (`6` → `11`), `SPAWN_RADIUS` (`850` → `950`), and `DESPAWN_RADIUS` (`950` → `1100`) so reversing all the way back to the fog wall never exposes bare ground or chunk pop-in.
    - Clamped `BackFogBarrier` behind `currentDepotSpine - 4` while `ShiftGateBarrier` is active so the fog wall never encroaches onto the Factory Depot yard, and snapped `BackFogBarrier` heading cleanly to cardinal road axes.
- **Studio Paths Modified/Created:**
  - `ServerScriptService.Server.Services.MapService` (modified)
  - `ServerStorage.Modules.ProceduralRoadGraph` (modified)
- **Repository Files Modified:** `GAME_DESIGN.md`, `DEVLOG.md`.
- **New Dependencies:** None.
- **Next Steps / Known Issues:**
  - Verified with a 10-seed depot placement audit (`20/20` clean single-road depot placements) and live in Play mode (`ShiftGateWall = 2048×48×8 CanCollide=true`, `BackFogBarrier = false` at `Z = -550`, spawns `904` studs behind at `VanZ = -1510`, zero console errors). Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create.


---

## 2026-11-20 — Riplatki (via Claude) (Rear door leaf decorations now swing with the rear doors)
- **Summary of Changes:** Fixed exterior (and interior) decorations on the rear doors staying in place when the rear doors open. `RearDoors.LeafDetailL` and `LeafDetailR` (48 parts each: stencils, hazard chevrons, window bezels/bars, hinge straps, kick plates, handle guards, tape, liners, grab handles) were never assigned an owner in `VanDrive`, so they were welded to the chassis. `VanDrive` now assigns every part in `LeafDetail<side>` to its `RearDoor<side>` root, so the existing `WeldConstraint` pass welds them to the door leaf. No parts are anchored at runtime, all affected parts are non-colliding and massless.
- **Studio Paths Modified/Created:**
  - `Workspace.DeliveryVan.VanDrive` (modified, "The game part" placeId 93445054108906)
- **New Dependencies:** None.
- **Next Steps / Known Issues:** Verified in Play mode (Server): all 96 `LeafDetail` parts welded to their own `RearDoor` root; with `RearDoorL` swung 110 deg about its hinge a detail part moved 5.22 studs in world with 0.0001 max drift relative to the door. Sliding door check: all 41 `SlidingDoor` child parts (incl. the `XD_*` decor) were already welded to `DoorPanel` and moved 7.02 studs with 0.0001 drift, and no `ExtraDetail` part sits on the sliding door's footprint, so I found nothing left behind on the side door; if something still stays put there, tell me which piece. Not visually inspected from a player camera. Deliberately still on the body: `RearHinge*` barrels, tail/reverse lights, `ExtraDetail.Rear` (`RearHeader`, `HighBrake`, `PlateLight` straddle the door gap). Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — justhyne1205 (via Antigravity)
- **Summary of Changes:**
  - **Sealed `BackFogBarrier` Full-Map Width & Locked East-West Orientation (`MapService`):**
    - Fixed the bug where `BackFogBarrier` rotated 90° sideways on East/West lateral road jogs (and was narrower than the flanking forest), letting players drive right around the fog through the woods.
    - Locked `BackFogBarrier` strictly to East-West orientation facing North (`-Z`) and widened both the invisible collision wall (`BackFogWall_L/R`) and all 4 vertical-gradient dark fog curtains (`BackFogSheet_1..4_L/R`) to two `1,600`-stud halves (`3,200` studs total, `X = [-1639 .. +1561]`, extending hundreds of studs beyond both outermost forest boundaries) with a `600`-stud billowing mist emitter bar across the road and shoulders.
  - **Tuned `BackFogBarrier` Distance Slightly Further Back (`MapService`):**
    - Added real North-South `maxReachedGz` tracking (`BACK_BARRIER_LAG = 11`, `BACK_BARRIER_GZ_LAG = 10`) so East/West jogs never shrink the actual world distance behind the van (~`1,000–1,200` studs behind — comfortable breathing room without feeling too far away).
    - Raised active Factory Depot clamp to `8` cells (`currentDepotSpine - 8` / `currentDepotGz - 8`) while `ShiftGateBarrier` is active, and expanded streaming keep bounds (`SPINE_BACK = 12`, `DESPAWN_SPINE_BACK = 14`, `SPAWN_RADIUS = 1100`, `DESPAWN_RADIUS = 1250`, reverse spawn boost `16`).
- **Studio Paths Modified/Created:**
  - `ServerScriptService.Server.Services.MapService` (modified)
- **Repository Files Modified:** `GAME_DESIGN.md`, `DEVLOG.md`.
- **New Dependencies:** None.
- **Next Steps / Known Issues:**
  - Verified live in Play mode (`SpanX=[-1639 .. 1561]`, `Total=3200 studs`, `CanCollide=true/true`, `WallLook=(0, 0, -1)`, `DistBehind=1204 studs` at `VanZ=-1709`, zero console errors). Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Summary of Changes:**
  - **Press-Toggle `[V]` 3rd-Person Chase Cam & Hold `[LeftAlt / MMB / R3]` Look Back / Window Lean-Out (`VanDrive`, `VanController`, `VanAnimController`):**
    - **`Workspace.DeliveryVan.VanDrive`:**
      - Added `"RollWindowDown"` (shared `setWindowTarget` helper reused by `toggleWindow`; idempotently rolls `L`/`R` open if `winTarget[side] < 0.5`, skips if `WindowBroken`, blocks if `WindowCover` is up) and `"LeanBack"` `VanControl` remote handlers setting replicated `Player` attribute `VanLookBackSide` (`"L"`, `"R"`, or `""`).
      - Added Heartbeat/exit/seat-switch sanitization so `VanLookBackSide` clears automatically if the player leaves or switches seats, drops window roll below `0.55`, closes the cover, or faces an approaching/active Window Monster.
    - **`StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController`:**
      - Converted `[V]` / `ButtonSelect` from per-frame `IsKeyDown` hold polling into `ToggleChaseCam()` (`chaseCamToggled`), auto-resetting when exiting the van and clearing on `SetChaseCamBlocked(true)`. Updated `VanSeatHint` text to `[C] Switch seat  •  [V] 3rd Person  •  Hold [Alt/MMB] Look Back`.
      - Implemented Hold Look Back (`LeftAlt` / `MouseButton3` / `ButtonR3`):
        - **3rd Person:** Smoothly orbits `180°` in front of the van looking backward with raycast collision clamping.
        - **1st Person Cab (`Driver` `L` / `Passenger` `R`):** Blocks with a toast if `WindowCover` is raised; leans immediately if shattered or `WindowRoll >= 0.75`; otherwise fires `"RollWindowDown"` until `WindowRoll >= 0.70`, then leans camera outside looking back down the van flank with world raycast collision clamping, speed-scaled rushing wind SFX (`VanLeanWindSfx` in `SoundService`), and speed-scaled camera shake; leaves window down on release; snaps back inside with a camera jolt (`"GET BACK INSIDE!"`) if `WindowApproachSide` or `WindowThreatSide` activates on that side.
        - **`RearPassenger`:** Smoothly turns camera toward the rear cargo doors without touching cab windows.
      - Added public API `ToggleChaseCam()`, `IsLookingBack()`, and `IsLeaningOut()`.
    - **`StarterPlayer.StarterPlayerScripts.Client.Controllers.VanAnimController`:**
      - Added `VanAnimSillBraceL` / `VanAnimSillBraceR` door-sill attachments and replicated R6 (`RootJoint`) / R15 (`Waist`) + `Neck` procedural lean-out and backward look twist driven by `VanLookBackSide`, keeping the driver's right hand on the steering wheel (`gripR`) while bracing the outer hand on the open window sill.
- **Studio Paths Modified/Created:**
  - `Workspace.DeliveryVan.VanDrive` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanAnimController` (modified)
- **Repository Files Modified:** `DEVLOG.md`.
- **New Dependencies:** Replicated `Player` attribute `VanLookBackSide`; `SoundService.VanLeanWindSfx`; door-sill attachments `VanAnimSillBraceL` / `VanAnimSillBraceR`.
- **Next Steps / Known Issues:**
  - Verified in Play mode: `VanSeatHint` displayed `"[C] Switch seat  •  [V] 3rd Person  •  Hold [Alt/MMB] Look Back"`, `VanAnimSillBraceL/R` created (`true/true`), `RollWindowDown` rolled `WindowRollL` from `0.00` past `0.73` to `1.00`, `LeanBack(true)` replicated `VanLookBackSide="L"` and shifted `RootJoint.C0` to `(-1.85, 0.17, -0.17)`, releasing left `WindowRollL=1.00` and returned `RootJoint.C0` to `(-0.02, 0.00, -0.00)`, and `WindowCoverL=true`, `WindowApproachSide="L"`, `WindowThreatSide="L"`, and seat exit all auto-cleared `VanLookBackSide` to `""`. Zero game script errors (`get_console_output`), Studio returned to `Edit` mode (`VirtualInputManager:SendKeyEvent` lacks `RobloxScript` capability in `execute_luau`, so live keyboard/camera testing of `[V]` toggle and `[Alt]` hold feel should be done interactively in Play). Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — Copilot SDK
- **Summary of Changes:** Added a large, bright red anchored test part to confirm live Studio editing.
- **Studio Paths Modified/Created:** `Workspace.StudioEditTestRedPart`
- **New Dependencies:** None.
- **Next Steps / Known Issues:** Test part is non-colliding and can be removed after verification. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — Riplatki (via Antigravity)
- **Summary of Changes:**
  - **Built & Finished Monster #3 — The Driver-Side Door-Ripper (`Monster3_DoorRipper`, `DoorMonster`, `DoorMonsterController`):**
    - **`ReplicatedStorage.Assets.Monsters.Monster3_DoorRipper`:** Built articulated 16-child procedural horror rig (`Root`, `Torso`, `Head` with glowing amber->crimson eyes + `EyeLightAnchor` `PointLight`, unhinging toothed `Jaw`, `UpperArmL/R` `2.05`, `ForearmL/R` `2.15`, clawed `HandL/R`, `UpperLegL/R` `1.95`, `LowerLegL/R` `2.05`, taloned `FootL/R`).
    - **`ServerStorage.Modules.Monsters.DoorMonster`, `MonsterService`, & `NetDefs`:**
      - Added `NetDefs.DoorMash` (`RateLimitPerSec = 28`, `MinInterval = 0.025`) wired in `MonsterService:Start()` to `doorMonster:RequestMash(player)`.
      - Implemented full state machine (`Approach` -> `Threat` -> `Repelled` / `Ripping`): spawns naturally only while actively driving (`MinDriveSpeed = 24`) and mutually exclusive with `WindowMonster` / `ChaseMonster`; during `"Threat"`, `DoorThreatProgress` decays (`0.155..0.215`/s) with periodic yanks (`1.35s`) swinging `DoorThreatAngle` (`2°..62°`), while driver `[E]` mashes (`+0.095`) fight back to `1.0` (`_repel()`, slams door shut). On failure (`_triggerRip()`), flings door to `78°`, sets `VanCoastUntil` (`6.5s`, maintaining `>= 46` studs/s rolling velocity), deducts `10s` shift time, exempts movement via `AntiExploitService`, destroys `DriverSeat.SeatWeld`, and ejects the driver (`-35` HP, non-lethal above `10` HP) outside the left door while the van keeps rolling down the road.
      - Added Cmdr command `doormonster` / `dm` (`spawn | threat | rip | repel | clear`) and mutual-exclusion guards in `WindowMonster` & `ChaseMonster`.
    - **`Workspace.DeliveryVan.VanDrive` & `VanController`:**
      - Physically rotates `DoorLowerL` via `DoorThreatAngle`, blocks cab entry/exit, `[C]` seat switch out, and Look Back lean-out on `"L"` during `DoorThreatActive`/`DoorThreatState`, disables left `DoorPrompt` so `[E]` isn't intercepted during mash, and switches `VanDrive` / `VanController:stopDriveSession()` to free-rolling wheel torque (`1400` vs `60000`/`70000` brake) while `VanCoastUntil` is active.
    - **`StarterPlayer.StarterPlayerScripts.Client.Controllers.DoorMonsterController` & `ControllerList`:**
      - Registered controller driving analytical 2-bone arm & leg IK (`BulkMoveTo`) so hands stay locked onto moving `DoorHandleL` & `DoorLowerL` and feet plant on the running board, plus sprint-and-leap approach, tumbling repel, cab lunge rip, local mash prediction HUD (`PlayerGui.DoorMonsterHUD`) with `[E]`/`ButtonX`/touch button + pulsing vignette, zero-drift camera shake + brief `0.55s` shock glance, `SetChaseCamBlocked` safety, and shaped built-in spatial SFX.
- **Studio Paths Modified/Created:**
  - `ReplicatedStorage.Assets.Monsters.Monster3_DoorRipper` (created)
  - `ReplicatedStorage.Shared.NetDefs` (modified)
  - `Workspace.DeliveryVan.VanDrive` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController` (modified)
  - `ServerStorage.Modules.Monsters.DoorMonster` (created)
  - `ServerStorage.Modules.Monsters.WindowMonster` (modified)
  - `ServerStorage.Modules.Monsters.ChaseMonster` (modified)
  - `ServerScriptService.Server.Services.MonsterService` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.DoorMonsterController` (created)
  - `StarterPlayer.StarterPlayerScripts.Client.Core.ControllerList` (modified)
  - `ServerScriptService.Cmdr.Commands.doormonster` & `doormonsterServer` (created)
- **Repository Files Modified:** `GAME_DESIGN.md`, `DEVLOG.md`.
- **New Dependencies:** Replicated remote `NetDefs.DoorMash`; van attributes `DoorThreatActive/State/Start/Duration/Progress/Open/Angle/MashSeq/YankSeq/VictimUserId/EventSeq` & `VanCoastUntil`; `DoorLowerL` attribute `DoorHingeLocalL`.
- **Next Steps / Known Issues:**
  - Verified live in Play mode: `NetDefs.DoorMash` & `Remotes.DoorMash` active (`true/true`); forced `"Threat"` opened `DoorLowerL` (`DoorThreatAngle = 40.74°`, `2.715` studs door travel) and 7 `[E]` `RequestMash` inputs repelled (`DoorThreatState = "Repelled"`, `DoorThreatAngle = 0.00°`, `0.000` stud closed error); Client spawned 16-child `DoorRipperActive` in `Workspace.DoorMonsterClientRig` with `PlayerGui.DoorMonsterHUD` (`TouchMashButton`, `ApproachBanner`, `MashBarFrame`); `ForceRip()` flung door to `78.0°`, extracted driver out the left side (`Seat=nil`, `PlayerLocalX=-7.50`, `HP=100.0->65.0`, `Pen=10s`), and kept the driverless van rolling (`FwdSpd=35.93`, `Rolled=18.52` studs with `CoastRem=5.63s`); `Reset()` restored `DoorThreatActive=false`, `DoorThreatAngle=-1.0`, and `0.000` stud closed error. Zero game script errors (`get_console_output`), Studio returned to `Edit` mode. Remember to **Commit Drafts** in Team Create.


---

## 2026-11-20 — Riplatki (via Claude)
- **Focus:** Van Upgrades v1: shop GUI + 3 simple upgrades (kept minimal on request).
- **Summary of Changes:**
  - Built the `VanUpgrades` garage shop GUI (opens from the existing `ShopService` `OpenShop` remote when ShopId is `VanUpgrades`): dark panel with cash readout, 3 upgrade rows (name, description, level pips, price button, MAXED state), status line, and an X close button (`Modal = true` so the mouse is freed in first person). Auto-closes if the player walks more than 30 studs from `StartArea.VanUpgradeStation`.
  - New server `UpgradeService`: buys with team cash via `ShiftService:SetCash`, only while the shift phase is `Prep` (between shifts). Levels live in memory for the server session and are mirrored as `Upg<Id>` attributes on `Workspace.DeliveryVan`.
  - Upgrades (3 levels each): **Engine Tuning** (+8 studs/s top speed per level, costs 200/400/700), **Boost Tank** (boost drains 15% slower per level, 150/300/500), **Cargo Rack** (+4 package capacity per level, 150/300/450). All numbers live in `ReplicatedStorage.Shared.VanUpgrades`.
  - `VanController` now adds the Engine bonus to `MAX_SPEED` and scales `BOOST_DRAIN` by the Boost Tank multiplier. `ChuteService` now uses `BASE_MAX_BOXES (24) + Cargo Rack bonus` and keeps the van `MaxCargoBoxes` attribute in sync.
- **Studio Paths Modified/Created:**
  - `ReplicatedStorage.Shared.VanUpgrades` (created)
  - `ReplicatedStorage.Shared.NetDefs` (modified)
  - `ServerScriptService.Server.Services.UpgradeService` (created)
  - `ServerScriptService.Server.Core.ServiceList` (modified)
  - `ServerScriptService.Server.Services.ChuteService` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.UpgradeShopController` (created)
  - `StarterPlayer.StarterPlayerScripts.Client.Core.ControllerList` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController` (modified)
- **New Dependencies:**
  - Remotes (`NetDefs`): `UpgradeAction` (C2S: `"Buy", upgradeId` or `"Refresh"`), `UpgradeState` (S2C: `{ Levels, Cash, CanBuy, Msg }`).
  - `DeliveryVan` attributes: `UpgEngine`, `UpgBoostTank`, `UpgCargoRack` (level 0-3). Existing `MaxCargoBoxes` is now `24 + 4 * UpgCargoRack`.
  - Client `ScreenGui`: `PlayerGui.VanUpgradeShop`.
- **Next Steps / Known Issues:**
  - Verified in live Play mode: GUI builds and opens near the station, buys deducted cash correctly (`$1000 -> $500` after 3 buys), van attributes updated (`UpgEngine/UpgBoostTank/UpgCargoRack = 1`, `MaxCargoBoxes = 28`), insufficient-cash message shown, auto-close works from 147 studs away. Zero game script errors. Not visually inspected (screen capture is unreliable in Play mode) and the speed/boost effect was not driven in a real drive test.
  - Upgrade levels are NOT saved yet (reset when the server restarts); hook `UpgradeService` into `DataService` / `ProfileStore`. Cash is shared team cash.
  - Not built yet from the Planned list: tracker screen upgrade, window covers, headlights/horn, tires, rear camera/cargo lock, flashlight. `PersonalUpgrades`, `Weapons`, `Stamina`, `Gear` shops still have no UI.
  - Studio was in Play mode partway through (started by another editor); I waited for it to end by itself before editing. Remember to **Commit Drafts** in Team Create.


---

## 2026-11-20 - Riplatki (via Claude)
- **Focus:** Box carrying pose (replaces the box floating in front of the player).
- **Summary of Changes:**
  - New client `CarryAnimController`: while a player's `CarryingPackage` attribute is true, both arms swing up and forward and squeeze slightly onto the box sides (R6 shoulder `Motor6D.C0` solved relative to the saved default, with the walk-cycle `Transform` cancelled so the arms stay steady). Eases in and out, runs for every character on every client so other players see it, and is skipped while seated or dead (the van animation owns the limbs there). R15 gets a simple shoulder-only fallback.
  - `PackageService`: the carried box is now welded hugging the chest, `hrp * CFrame.new(0, 0.15, -(0.65 + box.Size.Z/2))`, instead of floating 2.5 studs out.
- **Studio Paths Modified/Created:**
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.CarryAnimController` (created)
  - `StarterPlayer.StarterPlayerScripts.Client.Core.ControllerList` (modified)
  - `ServerScriptService.Server.Services.PackageService` (modified, pickup weld offset only)
- **New Dependencies:** None (reads the existing `CarryingPackage` player attribute).
- **Next Steps / Known Issues:**
  - Verified in Play mode on an R6 character using a test box and the attribute: arms raised forward and symmetric (hands at `x = +-1.26, z = -1.55` relative to the root, box center at `z = -1.85`), then arms returned to the neutral pose after release. Not seen visually and the real pickup prompt was not exercised because of the issue below. Arms overlap the box sides slightly on the widest boxes; tune `SQUEEZE` / `RAISE` in the controller if it looks off.
  - **Blocking issue, not mine:** `ServerScriptService.Server.Services.ShopService` currently has literal `\t` characters in its source (parse error at line 23, from someone's in-progress personal-upgrades edit), so the server `Bootstrap` fails and no services start in Play mode until it is fixed. Left untouched to avoid colliding with the other editor.

---

## 2026-11-20 - Riplatki (via Claude)
- **Focus:** Fix "game loop won't start / chute won't drop boxes".
- **Root cause:** `ServerScriptService.Server.Services.ShopService` had 101 literal `\t` text sequences in its source (someone's in-progress personal-upgrades edit was saved with escaped tabs), giving a parse error at line 23. `ServiceList` requires every service, so the whole server `Bootstrap` failed: no `ShiftService`, `ChuteService`, `MapService` etc. started, so no shift and no box drops.
- **Fix:** Replaced the leading literal `\t` sequences in `ShopService` with real tabs (content otherwise unchanged; it still contains the Health/Speed `UPGRADE_CONFIGS` work in progress). Then compiled all 181 scripts in `ServerScriptService`, `ServerStorage`, `ReplicatedStorage`, `StarterPlayer`, `StarterGui` and `Workspace` with `loadstring`: 0 failures.
- **Studio Paths Modified:** `ServerScriptService.Server.Services.ShopService` (whitespace fix only).
- **New Dependencies:** None.
- **Next Steps / Known Issues:**
  - Verified in Play mode: no script errors, services running (`leaderstats`, `ProceduralMap`, `ShiftState` remote, `MaxCargoBoxes = 24` set by `ChuteService`). I did not back the van into the chute to watch boxes drop.
  - Tip for whoever edits scripts through MCP/AI tools: if a script suddenly fails with `Expected identifier ... got '\'`, the source has literal `\t`/`\n` escapes; check with `loadstring(script.Source)`.

---

## 2026-11-20 — justhyne1205 (via Antigravity)
- **Summary of Changes:**
  - **Nocturnal Horror Lighting, Silhouette Mist & Shadow-Casting Van Headlights (`Lighting`, `DeliveryVan`, `MapService`):**
    - Replaced midday lighting (`ClockTime = 11.43`, bright flat ambient, thin `0.26` `Atmosphere`) with deep midnight horror lighting (`ClockTime = 0.0`, `Brightness = 0.60`, `ExposureCompensation = 0.0`, `Ambient = (14, 19, 24)`, `OutdoorAmbient = (24, 34, 42)`, `GlobalShadows = true`) and classic dark-slate nocturnal silhouette fog (`FogColor = (32, 48, 58)`, `FogStart = 20`, `FogEnd = 290`, overcast `Sky` with no celestial orbs/stars) so towering dead trees loom as dark silhouettes out of the mist while `StartArea` stays warmly lit by its 24 `WarehouseLights`. Enforced on server startup via `MapService.applyNocturnalLighting()` so time never drifts.
    - Tuned `Lighting.HorrorGrade` (`Contrast = 0.22`, `Saturation = -0.32`, cold tint `(228, 240, 246)`) and `Lighting.Bloom` (`Intensity = 0.65`, `Size = 30`, `Threshold = 1.15`) so headlights, taillights, streetlamps, porch lamps, and monster eyes pierce the mist.
    - Upgraded `DeliveryVan.Cab.HeadlightL/R.SpotLight` to **`Shadows = true`** (`Brightness = 6.0`, `Range = 60`, `Angle = 74`, warm-white `(255, 244, 218)`) and disabled `CastShadow` on 45 front fascia/bezel/grille parts right around the lenses so the van's own grille never blocks its shadow-casting beam.
  - **Subtle Film Grain + Dark Vignette & Moderate Camera Motion (`AtmosphereCameraController` & `ControllerList`):**
    - Created `StarterPlayer.StarterPlayerScripts.Client.Controllers.AtmosphereCameraController` (registered in `ControllerList`) with a lightweight `HorrorAtmosphereOverlay` `ScreenGui` (`DisplayOrder = -5`, strictly beneath HUD/menus): 4-edge `UIGradient` soft dark vignette (subtly deepens outdoors & at high speed) + ultra-subtle faint film lines + 44 pooled 18 Hz animated film grain specks (**no** `REC` text).
    - Layered moderate, motion-sickness-safe additive camera motion (`Camera - 8` pre-strip, `Camera + 8` post-apply, zero `hum.CameraOffset` override, zero aim drift, fully compatible with `VanController`, `WindowMonsterController`, `ChaseMonsterController`, and `DoorMonsterController`):
      - **On-Foot:** Smooth figure-8 walk/sprint head-bob, subtle idle breathing sway, and light strafe roll (`≤ 1.4°`).
      - **In-Van Driving:** Speed- and road-surface-scaled rumble (`Road_DamagedRough` `1.65×`, `Road_Cracked` `1.25×`, off-road `1.45×`), gentle corner banking roll (`≤ 1.8°`), smooth speed FOV push (`+4.5°` stacking cleanly with `VanController`'s `[Q]` Boost FOV), and damped collision jolts.
    - Left player flashlight mechanics untouched as requested.
- **Studio Paths Modified/Created:**
  - `Lighting` (`ClockTime`, `Ambient`, `OutdoorAmbient`, `FogColor/Start/End`, `Sky`, `HorrorGrade`, `Bloom`; removed `Atmosphere`)
  - `Workspace.DeliveryVan.Cab.HeadlightL/R` & front fascia parts (modified)
  - `ServerScriptService.Server.Services.MapService` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.AtmosphereCameraController` (created)
  - `StarterPlayer.StarterPlayerScripts.Client.Core.ControllerList` (modified)
- **Repository Files Modified:** `GAME_DESIGN.md`, `DEVLOG.md`.
- **New Dependencies:**
  - Client `PlayerGui.HorrorAtmosphereOverlay` (`ScreenGui`, `DisplayOrder = -5`).
- **Next Steps / Known Issues:**
  - Verified live in Play mode: confirmed midnight silhouette lighting (`ClockTime = 0.0`, `Fog = 20..290`, `HeadlightL/R Shadows = true`), `HorrorAtmosphereOverlay` (`DisplayOrder = -5`, `HasREC = false`, `GrainAnimated = true`), zero idle/walk camera drift (`0.048°–0.066°`), visual `screen_capture`s of warm Factory interior readability, in-cab windshield driving view, and exterior chase view with eerie silhouetted dead trees against cold dark-slate mist, and zero console errors. Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 — justhyne1205 (via Antigravity)
- **Summary of Changes:**
  - **Slightly Brighter Midnight Lighting & 145-Stud Long-Throw Van Headlights (`Lighting`, `MapService`, `DeliveryVan`, `VanDrive`):**
    - Gently lifted midnight horror lighting (`ExposureCompensation = 0.16`, `Brightness = 0.72`, `Ambient = (20, 27, 33)`, `OutdoorAmbient = (32, 45, 55)`, `FogColor = (36, 52, 64)`, `FogStart = 32`, `FogEnd = 340`, synced in `MapService.applyNocturnalLighting()`) so first-person highway driving is clearly readable without losing the eerie misty silhouette atmosphere.
    - Upgraded `DeliveryVan.Cab.HeadlightL/R` into 3-stage ~145-stud long-throw warm-white highway beams using zero-mass `Attachment` relays (`SpotLight` primary `Shadows = true`, `HeadlightMidRelay` at `-36` studs, `HeadlightFarRelay` at `-82` studs, plus subtle volumetric `HeadlightCone` `Beam`s) and updated `VanDrive` so `[L]` cleanly toggles all 6 `SpotLight`s and 2 `Beam`s together.
  - **Advanced Tactical `[F]` Flashlight Tool — Drop-In Model Ready (`FlashlightService`, `FlashlightController`, `NetDefs`, `ServiceList`, `ControllerList`):**
    - Created `FlashlightService` + minimal placeholder `Tool` template (`ReplicatedStorage.Assets.Tools.Flashlight` with `Handle` + `LensAttachment` ready for a future custom 3D model), granting it on spawn and replicating `FlashlightOn` per player via `NetDefs.FlashlightToggle`.
    - Created `FlashlightController` with `[F]` toggle & tool-click support (works seamlessly with default Backpack CoreGui disabled), realistic dual-concentric tactical optics (`32°` shadow-casting `HotspotLight`, `70°` outer `SpillLight` halo, forward `FarSpot`, and volumetric `VolumetricCone` `Beam`), smooth spring-lagged wrist inertia + walk bob + `120`-stud raycast aim convergence, co-op remote player rendering, ProSoundEffects click SFX (`rbxassetid://9114480271`), infinite battery, and erratic monster-proximity beam flicker.
  - **True Fullscreen `EditableImage` VHS Static (`AtmosphereCameraController`):**
    - Replaced the 44 sparse floating dots with 4 pre-baked `128×128` procedural monochrome VHS static snow `EditableImage` frames (tiled fullscreen via `ImageContent` and cycled at 24 FPS with random offset jitter), 48 interlaced CRT scanlines, 2 drifting horizontal VHS tracking shimmer bars, and soft dark vignette — zero `REC`/timer/date text clutter.
    - Added dynamic monster threat scaling (`ChaseActive`, `WindowThreatSide`, `WindowApproachSide`, `DoorThreatActive`): subtle & clean during normal driving (`ImageTransparency = 0.915`), intensifying noticeably during active threats (`~0.76–0.80` with stronger tracking bars & tighter vignette).
  - **Realistic, Optimized Dynamic Rain, Muffled Van Audio & Outdoors-Only Refractive Camera Droplets (`RainController` & `ControllerList`):**
    - Created `RainController`: defaults to calm **Light Drizzle** and smoothly ramps into an **Extremely Intense Heavy Storm** whenever `DeliveryVan:GetAttribute("ChaseActive") == true` (`Rate 55 → 280`, heavier wind acceleration, ground splashes, and storm audio).
    - Uses a 4-curtain exterior perimeter rain rig (`RainFront/Back/Left/Right`) positioned strictly outside `DeliveryVan`'s hull plus `160`-stud upward roof raycasts so **zero** rain ever falls inside the van or Factory.
    - Plays balanced ProSoundEffects rain loops (`rbxassetid://9112857056` drizzle + `rbxassetid://9112853286` heavy storm) with smooth `-24 dB` high-cut `EqualizerSoundEffect` interior muffling when inside `DeliveryVan` or under the Factory roof.
    - Renders 14 pooled refractive `Enum.Material.Glass` camera lens water droplets streaking down the lens **strictly when outdoors under open sky** (instantly evaporating inside `DeliveryVan`, Factory, or under roofs).
- **Studio Paths Modified/Created:**
  - `Lighting` & `Workspace.DeliveryVan.Cab.HeadlightL/R` (modified)
  - `Workspace.DeliveryVan.VanDrive` (modified)
  - `ReplicatedStorage.Shared.NetDefs` (modified)
  - `ServerScriptService.Server.Services.MapService` (modified)
  - `ServerScriptService.Server.Services.FlashlightService` (created)
  - `ServerScriptService.Server.Core.ServiceList` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.AtmosphereCameraController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.FlashlightController` (created)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.RainController` (created)
  - `StarterPlayer.StarterPlayerScripts.Client.Core.ControllerList` (modified)
- **Repository Files Modified:** `GAME_DESIGN.md`, `DEVLOG.md`.
- **New Dependencies:**
  - Remote (`NetDefs`): `FlashlightToggle` (C2S).
  - Player Attribute: `FlashlightOn` (boolean).
  - Template Tool: `ReplicatedStorage.Assets.Tools.Flashlight` (`Handle` + `LensAttachment`).
  - Verified free ProSoundEffects Audio IDs: `rbxassetid://9112857056` (drizzle), `rbxassetid://9112853286` (heavy storm), `rbxassetid://9114480271` (flashlight click).
- **Next Steps / Known Issues:**
  - Verified live in Play mode across all scenarios: Factory interior (`VHS CalmTrans=0.915`, `HighGain=-24.0dB`, `ActiveDrops=0`), inside `DeliveryVan` (`[L]` toggles `6/2 -> 0/0 -> 6/2`, `HighGain=-24.0dB`, `ActiveDrops=0`, in-cab `screen_capture` confirming crisp 145-stud lit highway visibility through rain & mist), outdoors on foot (`HighGain=0.0dB`, `ActiveDrops=3`, `[F]` flashlight `Hotspot/Spill/Far/Cone` verified via `screen_capture`), and Chaser Heavy Storm (`VHS ThreatTrans=0.802`, `Rain Rate=280`, `StormVol=0.400`, `OutdoorsLensDrops=13`, flashlight flickering `5.78..6.33`), with zero console errors. Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create.


---

## 2026-11-20 — justhyne1205 (via Antigravity)
- **Summary of Changes:**
  - **Flashlight Artifact Removal, 3rd-Person Forward Aim & Upgraded Dual-Ring Optics (`FlashlightController`, `DeliveryVan`):**
    - Removed the untextured rectangular `VolumetricCone` `Beam` from `FlashlightController` (and untextured `HeadlightCone` `Beam`s from `DeliveryVan.Cab.HeadlightL/R`, keeping all 6 long-throw headlight `SpotLight`s intact) so no translucent flat box ever renders against walls.
    - Updated `FlashlightController` so in **3rd person** (`camDist >= 2.2`) the flashlight points strictly straight forward in front of the character (`hrp.CFrame.LookVector`) and never chases the mouse/camera, while **1st person** retains smooth spring-lagged camera aiming.
    - Sharpened the flashlight optics for a punchier, more realistic tactical beam: tight `26°` `9.0`-brightness shadow-casting `HotspotLight`, soft `68°` `1.5`-brightness cool-blue `SpillLight` halo, `30°` `3.2`-brightness `FarSpot` relay, and subtle foreground `BounceLight` (`PointLight`).
  - **Tighter Horror Fog View Distance (`Lighting`, `MapService`):**
    - Tightened nocturnal fog (`FogStart = 20`, `FogEnd = 185`, `FogColor = (28, 42, 52)`, `ExposureCompensation = 0.14`, `Brightness = 0.68`, synced in `MapService.applyNocturnalLighting()`, tightening further to `FogEnd = 152` during Chaser heavy storms) so distant trees and houses vanish into thick mist while the road ahead stays readable under the headlights.
  - **Lighter, Cleaner VHS Static (`AtmosphereCameraController`):**
    - Backed off VHS overlay opacity for clear driving visibility while preserving the VHS aesthetic (`VHSStaticSnow` `0.955` calm → `~0.875–0.919` threat, `VHSScanlines` `0.968`, `VHSTrackingBar` `0.98 → 0.90`, softer vignette `0.30–0.36`).
  - **100× Richer Rain Visuals, Louder Outdoor Audio, Clearly Audible Muffled Interior Audio & Slightly Larger Outdoors Droplets (`RainController`):**
    - Upgraded to **8 depth-layered exterior rain curtains** (`4` close crisp curtains + `4` wider outer sheets strictly outside `DeliveryVan`'s hull, `Drizzle 35..95` → `Storm 180..440`), added wind-blown `StormWindMist` sheets during Chaser storms, and boosted wet-road `SplashDrops` (`55 → 240`).
    - Upgraded rain audio to **4 official ProSoundEffects loops** (`DrizzleLoop` `9112857056`, `HeavyStormLoop` `9112853286`, `StormWashLoop` `9112856394`, plus dedicated muffled `InteriorMuffledLoop` `9112852671` at `0.58 → 1.05` inside `DeliveryVan` and `0.45 → 0.88` under roofs with `-9.5/-13.5 dB` high-cut & `+4.2/+4.5 dB` low-boost EQ) so muffled rain is unmistakably audible inside the van and Factory during both drizzle and Chaser storms (fixing the previous near-silent shelter mix).
    - Slightly enlarged the pooled refractive `Glass` lens droplets (`0.044..0.082`, pool of `15`), still strictly outdoors under open sky with zero droplets inside `DeliveryVan` or Factory.
- **Studio Paths Modified/Created:**
  - `Lighting` & `Workspace.DeliveryVan.Cab.HeadlightL/R` (modified)
  - `ServerScriptService.Server.Services.MapService` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.FlashlightController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.AtmosphereCameraController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.RainController` (modified)
- **Repository Files Modified:** `GAME_DESIGN.md`, `DEVLOG.md`.
- **New Dependencies:** Additional verified free ProSoundEffects Audio IDs: `rbxassetid://9112856394` (storm wash), `rbxassetid://9112852671` (muffled interior rain loop).
- **Next Steps / Known Issues:** Verified live in Play mode across all scenarios: Factory interior (`Fog = 20..185`, `SnowT = 0.955`, `InteriorMuffledLoop = 0.45` + `DrizzleLoop = 0.28` at `-9.5/-13.5 dB`, `0` droplets), outdoors on foot (`BEAM_BOXES = 0`, 4 upgraded optics active, 3rd-person aim verified `dotHrp = 1.000 / dotCam = 0.001`, `DrizzleLoop = 0.58`, `15` slightly larger droplets `MaxSize = 0.065`), inside `DeliveryVan` calm drizzle (`Headlights = 6/6 ON`, `HeadlightBeams = 0`, `InteriorMuffledLoop = 0.58` + `DrizzleLoop = 0.31`, `0` on-screen droplets, in-cab `screen_capture` confirming crisp road visibility and hidden far trees/houses), and inside `DeliveryVan` during Chaser Heavy Storm (`FogEnd = 152`, `SnowT = 0.919`, `8` curtains `180..440` + `StormWindMist = 32` + `SplashDrops = 240`, `InteriorMuffledLoop = 1.05` + muffled `HeavyStormLoop = 0.57` + `StormWashLoop = 0.35`, `0` on-screen droplets), with zero game script errors. Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create (`MapService`, `FlashlightController`, `AtmosphereCameraController`, `RainController`).


---

## 2026-11-20 - Riplatki (via Claude)
- **Focus:** Door-Ripper felt empty: added the driver's door-fight animations (both hands on the door, punches and kicks) and monster hit reactions.
- **Summary of Changes:**
  - **Driver (`VanAnimController`):** while `DoorThreatState` is `Threat` (and `Repelled`, for the final slam) the driver lets go of the wheel and hauls the driver door shut with BOTH hands (left on the inner handle, right just ahead of it; targets follow the swinging `DoorLowerL`). The torso braces/twists toward the door (leans further out as `DoorThreatAngle` grows, extra jolt on each yank), and the head turns to watch the monster. Every `[E]` mash fires one limb in rotation: right punch (head), left kick, left punch (chest), right kick. Remote clients see the same strikes from `DoorThreatMashSeq` changes; the local driver strikes instantly through `VanAnimController:TriggerDoorStrike()`. Works for R6 (motor solve) and R15 (IKControl) via the existing channel system.
  - **Fixed a silent bug:** `DoorMonsterController.triggerLocalMash` already called `VanAnimController:TriggerDoorStrike()` but that method never existed, and the `if VanAnimController.TriggerDoorStrike then` guard hid it. It exists now.
  - **Monster (`DoorMonsterController`):** `registerHit()` staggers the monster on every blow: head snap, torso recoil/twist, shoved back off the running board, claw grip slips, one boot skids, jaw screeches, eye light flares, small dark splatter burst (`HitSplatter` `ParticleEmitter` on `DoorRipperAudioAnchor`). Added idle breathing sway and claw scrabbling on the panel while it pries. Other players' clients react to your hits via `DoorThreatMashSeq`.
- **Studio Paths Modified/Created:**
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanAnimController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.DoorMonsterController` (modified)
- **Repository Files Modified:** `GAME_DESIGN.md` (Door-Ripper fight animations), `DEVLOG.md`, `LESSONS.md`.
- **New Dependencies:** Client-only attachments on `DeliveryVan.Chassis`: `VanAnimDoorHandL/R`, `VanAnimPunchTipL/R`, `VanAnimKickTipL/R`; `Rig.doorW` blend weight. No new remotes, tags, attributes or DataStore keys (reads existing `DoorThreat*` van attributes).
- **Next Steps / Known Issues:**
  - Verified in Play mode with a seated R6 NPC driver and a forced `doormonster threat`: lean reached about `-1.2..-2.7` studs toward the door, all four limbs swung in a repeating punch/kick rhythm (5-9 peaks in 1.8s), arm tips got within `~0.4` studs of the door grips, `HitSplatter` + monster rig present, `doormonster repel` fired the final shove, and the pose returned to neutral afterwards (`RootJoint.C0.X = 0`). Zero game script errors in the console.
  - **Not tested:** the real local-driver path (needs an actual `[E]` key press, which `execute_luau` cannot send), R15 avatars, and nothing was seen visually (`screen_capture` is unreliable in Play mode). Please play it and tune by feel: `DOOR_LEAN_R6`, `DOOR_HAND_HANDLE`, `DOOR_HAND_EDGE`, `PUNCH_*`, `KICK_*` at the top of `VanAnimController`, and the `hitFlinch` multipliers in `DoorMonsterController`.
  - R6 arms cannot bend, so when the door is pried wide open the hands reach as far as the arm allows while the torso leans out. First-person camera also shifts toward the door with the lean (up to about 2.4 studs).
  - Idea: a door-slam shake/sound for each successful blow, and a dedicated punch whoosh SFX (only verified built-in sounds are used right now). Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 - OpenCode (Big Pickle)
- **Focus:** Getting in and out of the van no longer teleports the player — added animated step-in, sit-down (rear bench) and stand-up transitions.
- **Summary of Changes:**
  - **`Workspace.DeliveryVan.VanDrive`:** Added a shared animated-seat-transition system that mirrors the proven `[C]` SwitchSeat slide. `animateIntoSeat()` welds the character's HRP to `Chassis` in chassis-local space, tweens it from the standing spot to the seated pose with a small lift/arc (0.72s cab, 0.82s rear), then destroys the weld and calls `Seat:Sit` with canonical `SeatWeld.C0/C1`. Both cab `SeatPrompt` and rear-bench `SeatPrompt` handlers now call it instead of `sitInSeat()` directly.
  - **`exitVanSeat()`:** Replaced the instant `PivotTo` hop with the same welded tween from the seat to the old door-side / cargo-aisle standing spot, then restores velocity and physics. Reuses the existing door-block, raycast and exit-token guards.
  - Character parts are temporarily moved to the `VanSeat` collision group and made `Massless` during a transition (same as the existing seat switch) so the van is not kicked by depenetration; saved values are restored afterwards.
  - **`StarterPlayer.StarterPlayerScripts.Client.Controllers.VanAnimController`:** Transitions reuse the existing `VanSwitch*` braced climb/sit choreography; a new `VanTransitionKind` attribute deepens the crouch for rear sit-down (`EnterRear`) / stand-up (`ExitRear`) and adds a lighter brace for cab entry/exit.
- **Studio Paths Modified/Created:**
  - `Workspace.DeliveryVan.VanDrive` (modified; `VanDrive` source)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanAnimController` (modified; ModuleScript source)
  - No new Instances were left in the DataModel (a temporary test remote used during verification was removed).
- **New Dependencies:** New **player attributes** (set/cleared by `VanDrive`, read by `VanAnimController`): `VanTransitionKind` (`EnterFront` / `EnterRear` / `ExitFront` / `ExitRear` / `""`), plus the pre-existing `VanSwitchActive`, `VanSwitchStart`, `VanSwitchDuration`, `VanSwitchFrom`, `VanSwitchTo`, `VanSwitchFromLocal`, `VanSwitchToLocal`, `VanSwitchHops`, `VanSeatRole` now also written for entry/exit. No new RemoteEvents, CollectionService tags, collision groups, or DataStore keys.
- **Next Steps / Known Issues:**
  - Verified in Play mode by temporarily exposing the entry handler to a test remote: rear entry showed `EnterRear` + `VanEnterWeld` active for ~0.8s before `SeatPart = PassengerSeat`; `[Space]`-equivalent `VanControl:FireServer("LeaveSeat")` showed `ExitRear` + `VanExitWeld` active for ~0.8s before `SeatPart = nil`. Console had zero game-script errors (only unrelated plugin warnings).
  - **Not visually verified:** avatar limb IK during the transition, first-person camera feel, and R15 avatars (R6 is this game's default). Tune `TRANSITION_DURATION`, `REAR_TRANSITION_DURATION`, the lift/arc constants near `animateIntoSeat()`, and the `pushDip` weights in `VanAnimController` by feel in Play mode.
  - Directly firing a `ProximityPrompt` from `execute_luau` (`InputHoldBegin`/`InputHoldEnd`) did **not** trigger the server handler; a temporary test remote was used instead and removed. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-20 - Antigravity
- **Focus:** Implemented Monster #7 — The Forest Hunter / Deep-Forest Boundary Herder (tall antlered Wendigo/Leshy).
- **Summary of Changes:**
  - **Server (`ServerStorage.Modules.Monsters.ForestHunter`, registered in `MonsterService`):** Polls on-foot players every `1.2s` and spawns a stalking encounter (`85–115` studs away, ground-raycast snapped) when `> 180` studs from both the nearest `ProceduralRoadGraph` cell and `Workspace.DeliveryVan` (and outside `StartArea`). Unobserved hunters creep `22–28` studs closer into the player's forward path after `4.5s`; direct sight/flashlight spot (`Net` `"ForestHunterSpotted"`) or `<= 32` stud proximity triggers a screech and `24` studs/s charge. Claw attacks deal `-30` HP and apply directional herding knockback toward the nearest road/van (`AntiExploitService:ExemptMovement` + `ForestHunterKnockback`). Automatically retreats and dissolves when reaching road (`<= 42`), van (`<= 35`), `StartArea`, or a seat.
  - **Client (`ForestHunterController`, registered in `ControllerList`):** Builds and drives the 46-part `Workspace.ForestHunter3D` procedural Wendigo/Leshy rig via `BulkMoveTo` (tattered moss shroud, branching antler crown, amber-to-crimson Neon eyes + `PointLight`, blood-red claws, dark mist emitter), checks unobstructed camera FOV / flashlight raycast, plays spatialized built-in audio cues, triggers crimson flash + zero-drift camera shake (`Camera.Value - 5` / `+ 5`), and applies replicated herding knockback on the client-owned `HumanoidRootPart`.
  - **Atmosphere & Admin (`FlashlightController`, `RainController`, Cmdr `forestmonster`):** Added `"ForestHunter3D"` proximity flashlight flicker, smooth rain/ambient audio ducking (`0.08x`) while `Stalk`/`Creep` is active, and Cmdr `forestmonster` (`spawn | creep | charge | attack | clear`).
- **Studio Paths Modified/Created:**
  - `ReplicatedStorage.Shared.NetDefs` (modified: added `ForestHunterSpotted`)
  - `ServerStorage.Modules.Monsters.ForestHunter` (created)
  - `ServerScriptService.Server.Services.MonsterService` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.ForestHunterController` (created)
  - `StarterPlayer.StarterPlayerScripts.Client.Core.ControllerList` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.FlashlightController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.RainController` (modified)
  - `ServerScriptService.Cmdr.Commands.forestmonster` & `forestmonsterServer` (created)
- **Repository Files Modified:** `GAME_DESIGN.md`, `DEVLOG.md`, `LESSONS.md`.
- **New Dependencies:** `ForestHunterSpotted` (`RemoteEvent` C2S); `Player` attributes `ForestHunterActive`, `ForestHunterState`, `ForestHunterPos`, `ForestHunterYaw`, `ForestHunterDist`, `ForestHunterEventSeq`, `ForestHunterLastEvent`, `ForestHunterKnockback`.
- **Next Steps / Known Issues:**
  - Verified in Studio Play mode via `CmdrFunction:InvokeServer`: `forestmonster spawn` (`active=true`, `state=Stalk`, 46-part `ForestHunter3D`, amber eyes), `creep` (`ev=Creep`, `dist=59.4`), `charge` (`state=Charge`, `ev=Screech`, eyes flared crimson), `attack` (`hp=100->70`, `peakVel=73.0`, `disp=13.6` studs), and `clear` (`activeFalse=true`, `modelGone=true`), with zero script errors in `get_console_output`.
  - **Not yet tested in live Play:** natural automatic deep-forest spawn outside the `StartArea` perimeter (tested via Cmdr overrides) and visual appearance in the viewport (`screen_capture` not run). Remember to **Commit Drafts** in Roblox Studio's **Drafts** widget if Team Create Collaborative Editing is active.

---

## 2026-11-21 - Antigravity
- **Focus:** Expanded procedural forest depth to 4 rings (`400` studs) and hardened `ForestHunter` spawn/creep ground validation so it stays in bounds.
- **Summary of Changes:**
  - **`ServerStorage.Modules.ChunkBuilder`:** Increased `FOREST_BORDER_RADIUS` from `3` (`300` studs) to `4` (`400` studs), normalized tree scale `depthT` across `350` studs (`(distFromRoad - 50) / 350`), and balanced per-chunk tree counts across Rings 1–4 (`4–6` / `8–11` / `11–15` / `12–15`, shadows disabled on Ring 4, `ForestBoundary` walls on Ring 4 outer edges).
  - **`ServerScriptService.Server.Services.MapService`:** Raised `MAX_SPAWNS_PER_TICK` from `8` to `14` and initial `ResetForShift` streaming budget from `400` to `600` to accommodate Ring 4 forest chunks.
  - **`ServerStorage.Modules.Monsters.ForestHunter`:** Added `trySnapToGroundY` and `_findValidGroundPos` sweeping angle offsets `{ 0, ±0.4, ±0.85, ±1.4, ±2.1, π }` across distance scales `{ 1.0, 0.78, 0.55 }` to reject void/off-map coordinates in `_spawnEncounter` and `_startCreep`, falling back inward toward `safetyTarget` when pinned.
- **Studio Paths Modified/Created:**
  - `ServerStorage.Modules.ChunkBuilder` (modified)
  - `ServerScriptService.Server.Services.MapService` (modified)
  - `ServerStorage.Modules.Monsters.ForestHunter` (modified)
- **Repository Files Modified:** `GAME_DESIGN.md`, `DEVLOG.md`.
- **New Dependencies:** None.
- **Next Steps / Known Issues:**
  - Verified in Studio Play mode: `Workspace.ProceduralMap` streamed `133` forest chunks including `27` Ring 4 chunks (`Forest_4_*` / `Forest_-4_*`) and `15` Ring 4 `ForestBoundary` walls; `forestmonster spawn` (`y=50.74` on `Bay_Roof_EastSlope` at spawn) and `forestmonster creep` (`y=53.79` on `Bay_Roof_EastSlope` at spawn) both hit solid geometry cleanly with zero script errors in `get_console_output`.
  - Note: Because `trySnapToGroundY` casts down from `math.max(80, fallbackY + 45)`, testing inside `StartArea` hit the factory roof (`Bay_Roof_EastSlope`), and deep-forest dead trees (`CanQuery = true`) could intercept downward casts if spawned directly above a trunk; verify deep-forest ground snapping (`GrassGround` @ `y=0.86`) out in the woods and lower cast origin or filter `GrassGround` if needed. Remember to **Commit Drafts** in Roblox Studio's **Drafts** widget if Team Create Collaborative Editing is active.

---

## 2026-11-21 - OpenCode (Big Pickle)
- **Focus:** Fixed van-seat transitions kicking/spinning the van, and removed the "hunchback" head look on seated remote players.
- **Summary of Changes:**
  - **`Workspace.DeliveryVan.VanDrive`:** Replaced the short-lived `setCharVanPhysics()` helper with a persistent per-player `vanOccupantPhysics` state. `setVanOccupantGhost()` swaps character parts to the `VanSeat` collision group **only during the welded slide** (entry, exit and `[C]` switch), then restores their original collision group while deliberately keeping `Massless = true`. A `monitorVanOccupant()` heartbeat waits until the avatar is no longer seated in a van seat and its HRP leaves the chassis-local keep-out box (`|X| < 6.5`, `-17 < Z < 17`, `-3 < Y < 16`) before restoring the original `Massless` values.
  - Root cause: the old `task.delay(0.2, restore)` returned collisions while a rear-seat player was still standing **inside** the cargo body (rear exit spot is only `~1.9` studs off the seat, local X `≈ -1.9` vs van wall `≈ ±3.25`), so walking/standing up re-enabled limb-vs-cab depenetration and shoved the physics-owned van. Keeping the avatar massless while it is in/near the van removes the impulse without disabling ground collision.
  - **`StarterPlayer.StarterPlayerScripts.Client.Controllers.VanAnimController`:** `HEAD_FORWARD_OFFSET` (`-0.45` studs) is now applied **only when `char == localPlayer.Character`**. Remote avatars keep `rig.defaultNeckC0` position (head/eye rotations for look-back and door-threat still apply), so other players no longer look hunchbacked when seated. The local first-person camera is unchanged because it reads the local head position.
- **Studio Paths Modified/Created:**
  - `Workspace.DeliveryVan.VanDrive` (modified; `VanDrive` source)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanAnimController` (modified; ModuleScript source)
  - No new Instances left in the DataModel (a temporary `TempTestEnterSeat` remote used for verification was removed).
- **New Dependencies:** None — no new attributes, remotes, collision groups, or DataStore keys. Reuses the existing `VanSeat` group and the player/van attributes from the animated-seat-entry work.
- **Next Steps / Known Issues:**
  - Verified in Play mode: entry/exit/switch all show `CollisionGroup = VanSeat` + `Massless = true` during the slide, then `CollisionGroup = Default` + `Massless = true` while seated/standing inside. After teleporting the player out of the van, `Massless` restored to `false` within `0.8s`. `VanControl:FireServer("LeaveSeat")` and `SwitchSeat:FireServer(targetSeat)` both completed with no game-script errors (only unrelated plugin warnings).
  - **Not visually verified:** whether keeping avatars Massless for the entire time they are in the van changes any other interaction (monster grabs, package pickup, collisions with other players). The keep-out box is intentionally generous; tune the `6.5 / 17 / 16` bounds in `isInsideVan()` if players report walking through van walls or staying ghosted too long. Remember to **Commit Drafts** in Team Create.

### Follow-up same date — remaining van hop during transitions
- **Focus:** User reported the van still shifted during the brief transition window and jumped noticeably when switching seats.
- **Root cause:** The welded slide loops were not just translating the character — they also animated a vertical lift and pitch/roll arc into `slideWeld.C0` / `SwitchSlideWeld.C0`. Because the weld is attached to the physics-owned chassis, that arc was transferred straight into the van. Instrumented `[C]` switch showed the chassis lift from `Y=5.76` to `~7.05` with `~5°` tilt and `AssemblyAngularVelocity` spikes up to `1.323`.
- **Fix:** In `Workspace.DeliveryVan.VanDrive`, all three slide loops (enter, exit, `[C]` switch) now interpolate **position only** with `CFrame.new(lerpedPos) * startC0.Rotation`; the `lift`, `hopSin`, `pitch`, `roll` and `rot` terms were removed. The climb/stand/sit arcs are already replicated to clients through the `VanSwitch*` attributes and remain cosmetic only.
- **Verification in Play mode:** Rear entry, rear exit, front exit, rear↔rear switch, and driver→cab switch all kept `chassis.Position.Y = 5.76` with peak `AssemblyAngularVelocity <= 0.015` (previous switch peak `1.323`). Walking a post-exit avatar against the cargo wall produced `peakAv = 0.023` and no Y movement. `loadstring(Source)` passes; temporary `TempTestEnterSeat` remote removed again; no new game-script errors.

---

## 2026-11-21 - Riplatki (via Claude)
- **Focus:** Van still got kicked/spun/flipped if the player moved while getting off the van, switching seats or standing up.
- **Summary of Changes:**
  - **Workspace.DeliveryVan.VanDrive:** Root cause: during the welded slide (enter / exit / [C] switch) the character's HRP is welded to the van, so the Humanoid's walk/jump/auto-rotate forces were applied straight to the van assembly. Added `lockOccupantControl(hum, locked)` (`PlatformStand = true` + `AutoRotate = false`, previous AutoRotate kept in the `VanPrevAutoRotate` Humanoid attribute). It is applied right after `setVanOccupantGhost(..., true)` in `animateIntoSeat`, `exitVanSeat` and the `SwitchSeat` handler, and released as soon as the slide weld is destroyed (before `sitInSeat` for entry).
- **Studio Paths Modified/Created:** `Workspace.DeliveryVan.VanDrive` (modified).
- **New Dependencies:** Humanoid attribute `VanPrevAutoRotate` (temporary). No new remotes, tags or DataStore keys.
- **Next Steps / Known Issues:**
  - `loadstring(Source)` passes. **Not Play-tested** (Studio would not leave Play mode at first; please test by holding WASD/Space while exiting, standing up from the rear bench and pressing [C]). If the van still moves, the next suspect is the moment collisions return after the slide.
  - Input is ignored for ~0.7-0.8s during each transition. Remember to **Commit Drafts** in Team Create.

---

## 2026-11-21 — justhyne1205 (via Antigravity)
- **Summary of Changes:**
  - **Slightly Softer VHS Overlay (`AtmosphereCameraController`):**
    - Nudged the VHS overlay just slightly softer while preserving the exact VHS aesthetic and monster-threat scaling (`VHSStaticSnow`: `0.966` calm → `~0.905` threat, `VHSScanlines`: `0.976`, `VHSTrackingBar`: `0.985` calm → `~0.925` threat, zero `REC`/timestamp text).
  - **Bigger Outdoors Camera Lens Droplets + Heavy Chaser Surge (`RainController`):**
    - Expanded the outdoors-only refractive `Glass` camera droplet pool from `15` → **`38`** with larger, crisper water beads (`TargetScale = 0.072..0.135`, `Transparency = 0.74`, `Reflectance = 0.25`, dynamic downward teardrop elongation as they slide).
    - Calm light drizzle maintains a moderate ~`10–14` active droplets, surging rapidly to **`32–38`** fast-spawning droplets whenever `DeliveryVan:GetAttribute("ChaseActive") == true` outdoors (still strictly **0** inside `DeliveryVan`, Factory, or under roofs).
  - **Much Louder Chaser Storm Rain Audio (`RainController`):**
    - Kept calm drizzle balanced while dramatically cranking up active Chaser storm audio: outdoors `HeavyStormLoop` up to **`2.20`** + `StormWashLoop` up to **`1.40`**; muffled inside `DeliveryVan` `InteriorMuffledLoop` up to **`1.85`** + muffled `HeavyStormLoop` (`1.25`) & `StormWashLoop` (`0.80`) with punchier **`+6.0 dB`** low-end roof-drum rumble.
  - **1st-Person Seated Van Window Rain Droplets (`RainController`):**
    - Added client-only `SurfaceGui` glossy rain droplet layers (`PlayerGui.VanWindowRain`) adorned to the interior faces of `DeliveryVan`'s 6 existing `Glass` panes (`Cab.Windshield` pool `112`, `Cab.SideGlassL/R` `76` each, `SlidingDoor.DoorWindow` `32`, `RearDoors.RearWindowL/R` `26` each) — zero new parts, welds, or physics impact, and naturally hidden behind Window Monster blinds when pulled up.
    - **Strict visibility rule:** Enabled **only** when seated in a van seat **and** in 1st person (`isSeatedFirstPerson`); immediately disabled when on foot, zoomed into 3rd person, or holding `[V]` chase cam, and zero droplets spawn when parked under the Factory roof.
    - **Sightline weighting & airflow physics:** Keeps `Cab.Windshield`'s central driving sightline clear while densely beading the windshield edges, top perimeter, and side windows; droplets trickle downward when parked, streak upward/outward on the windshield and diagonally backward on side windows as van speed climbs, and pelt heavily during Chaser storms.
- **Studio Paths Modified/Created:**
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.AtmosphereCameraController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.RainController` (modified)
- **Repository Files Modified:** `GAME_DESIGN.md`, `DEVLOG.md`.
- **New Dependencies:** Client `PlayerGui.VanWindowRain` (`Folder` of 6 client-only `SurfaceGui` layers on existing `DeliveryVan` glass panes).
- **Next Steps / Known Issues:** Verified live in Play mode across all scenarios: Factory interior (`Snow = 0.966`, `Scan = 0.976`, `Bar = 0.985`, `Lens = 0`, all 6 van windows `0` drops under Factory roof), outdoors on foot (`11–12` calm lens drops → `38` Chaser storm drops `maxS = 0.139`, `HeavyStormLoop = 2.20`, `StormWashLoop = 1.40`, verified via `Outdoor_Storm_LensDrops_v2`), in-cab 1st-person highway drizzle (`Lens = 0`, `Windshield = 59` with `center = 1 / edge+top = 45`, `SideGlassL/R = 39/41`, verified via `InCab_Final_Drizzle`), in-cab 1st-person Chaser storm (`Lens = 0`, `InteriorMuffledLoop = 1.85`, `HeavyStormLoop = 1.25`, `StormWashLoop = 0.80`, `LowGain = +6.0 dB`, `Windshield = 111`, `SideGlassL/R = 73/74`, verified via `InCab_Final_Storm`), and `[V]` 3rd-person chase cam (all 6 `VanWindowRain` `SurfaceGui`s immediately `Enabled = false`), with zero console errors. Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create (`AtmosphereCameraController`, `RainController`).


---

## 2026-11-21 — justhyne1205 (via Antigravity)
- **Summary of Changes:**
  - Modeled **15 high-detail low-poly forest & roadside horror decoration props** in Blender (`Assets/Props/ForestProps.blend`, scaled at `1 stud = 0.28 m`, grounded cleanly at `Z = 0`, single-primitive `MeshPart`-ready geometry per prop) sharing one newly baked `1024×1024` procedural weathered 4×4 texture atlas (`Assets/Props/HorrorKit_ForestPropsAtlas.png` — dark wet granite, mossy stone, pitted headstone masonry, charred ash, ridged bark, weathered timber planks, rotting heartwood rings, pale shelf fungi/rope, dead thorny bramble, heavy rusted steel, peeling faded car paint, battered oil-drum crimson, faded yellow hazard sign, road-marker reflector stripe, treaded muddied rubber, and shattered dark glass):
    - **Forest Floor & Ground Clutter:** `Prop_Boulder_Mossy` (260 tris), `Prop_RockCluster_Small` (480 tris), `Prop_FallenLog_Rotted` (392 tris, hollow splintered ends + shelf fungi), `Prop_TreeStump_Old` (478 tris, 5 buttress roots + hollow jagged top + bracket fungi), `Prop_BrambleBush_Dead` (1,300 tris, 18 arching thorny canes + dead fronds).
    - **Creepy Backwoods Landmarks:** `Prop_AbandonedWell` (3,296 tris, 4 individual cobblestone courses, missing cap slab, rotted gable roof, windlass & hanging rope), `Prop_OldGravestones` (656 tris, arched headstone, crooked stone cross, broken tablet, rusted iron fence), `Prop_WoodsTotem` (640 tris, stone cairn, lashed crossed branches, twig wreath, 3 hanging stick effigies), `Prop_AbandonedCampfire` (704 tris, 11-stone ring, charred logs, snapped tripod, tipped kettle, fallen lantern, sit log), `Prop_HuntingBlind_Rotted` (936 tris, tilted elevated wooden deer-blind on cross-braced stilts, crooked ladder with missing rung, slatted walls & sagging roof).
    - **Roadside & Yard Dressing:** `Prop_WarningSign_Rusted` (120 tris, bent post + tilted yellow diamond hazard sign), `Prop_RoadMarker_Post` (140 tris, beveled post + white/red reflector collar), `Prop_RustedBarrels` (1,228 tris, 3 ribbed dented 55-gal steel drums + oil spill & chock), `Prop_CarWreck_Rusted` (1,700 tris, slumped 70s/80s abandoned sedan with flush A/B/C pillars, popped ajar hood, 3 wheels + missing front-right wheel on exposed rusted drum), `Prop_ScrapTires_Pallet` (1,068 tris, slatted pallet, smashed crate, 3 treaded tires, cinderblock & pipe).
  - Exported all 15 single-primitive textured meshes into `Assets/Props/ForestProps.glb` (`2.2 MB`) and visually verified every prop in Blender via `look` (catching and fixing `Prop_CarWreck_Rusted`'s pillar alignment before final export).
- **Repository Files Added/Modified:**
  - `Assets/Props/ForestProps.blend` (created)
  - `Assets/Props/ForestProps.glb` (created)
  - `Assets/Props/HorrorKit_ForestPropsAtlas.png` (created)
  - `DEVLOG.md` (modified)
- **Next Steps / Known Issues:** Studio MCP `open_cloud_invoke` requires `Assets/Props` to be added to **Assistant Settings → MCP Permissions** (or a 1-click `Ctrl+M` Import 3D of `ForestProps.glb` in Roblox Studio). Once imported into Studio, scale exact `1/0.28×`, anchor, organize in `Workspace.MapModels.HorrorKit_Props`, and verify visually.


### Follow-up same date — Full 2048×2048 PBR & Hero-Detail Overhaul of All 15 Forest Props
- **Summary of Changes:**
  - Upgraded the shared texture pipeline to a **2048×2048 3-map glTF PBR Atlas** (`HorrorKit_ForestPropsAtlas.png` Albedo, `HorrorKit_ForestProps_Normal.png` Sobel tangent-space Normal Map, and `HorrorKit_ForestProps_ORM.png` packed AO + Roughness + Metallic) featuring crisp hand-crafted horror graphics: bold black `!` diamond hazard warning sign with bullet dents, carved gravestone cross & epitaph tablet, stenciled white/crimson oil-drum bands with oil drips, weathered 1970s faded teal-blue car paint with rust streaks, spiderweb-cracked auto glass, plank seams with rusted nail heads, and prismatic red/white reflector collars.
  - Massively upgraded geometry & storytelling craft across all 15 props (`Assets/Props/ForestProps.blend` & `Assets/Props/ForestProps.glb`):
    - **Row 1 (Forest Floor):** Stratified overhanging `Prop_Boulder_Mossy` (draped dead root + fern tufts), 8-rock `Prop_RockCluster_Small` (woodland mushrooms + ferns), deep hollow-tunnel `Prop_FallenLog_Rotted` (curled peeling bark + 8 shelf fungi + mushrooms), 7-root `Prop_TreeStump_Old` with an **embedded rusted logging axe** in the stump top, and dense 24-cane `Prop_BrambleBush_Dead` with thorn barbs & 16 multi-segment fern fronds.
    - **Row 2 (Haunted Landmarks):** `Prop_AbandonedWell` (layered cedar roof shakes, dangling vines, iron ratchet wheel & **hanging iron-banded wooden bucket**), `Prop_OldGravestones` (carved cross/epitaph front tablet, Celtic ring-cross, spear fence & memorial lantern), `Prop_WoodsTotem` (double twig wreath, **sculpted bleached deer-skull with branching antlers** & 5 hanging charms), `Prop_AbandonedCampfire` (3-pole tripod with rusted iron chain & **hanging cast-iron cauldron**, coffee mug & caged lantern on stump), `Prop_HuntingBlind_Rotted` (iron gusset brackets, **crooked hanging hinged window shutter**, interior bench, ladder handrail + broken dangling rung, rusted corrugated tin/plank roof & crate underneath).
    - **Row 3 (Roadside & Yard):** Flanged U-channel `Prop_WarningSign_Rusted` with painted `!` diamond sign, chipped-concrete `Prop_RoadMarker_Post` with exposed rebar stubs & recessed reflector box, `Prop_RustedBarrels` with **rotary hand-crank transfer pump**, hero `Prop_CarWreck_Rusted` (**real hollow open cabin**, dashboard, **steering wheel & seats visible inside**, spiderweb **shattered glass shards**, popped hood with V8 engine/radiator underneath, quad round headlights, hanging side mirror & missing front-right wheel propped on a **hollow dual-hole cinderblock**), and `Prop_ScrapTires_Pallet` (iron-bracket crate, **vintage red fuel jerrycan**, 4 tires & twin hollow cinderblocks).
  - Caught and fixed two visual defects via Blender `look` inspection (Blender 5.3 `colorspace_settings` buffer clear and `bmesh.ops.bevel` replaced-face UV loss) before final export (`12.5 MB` full-PBR `ForestProps.glb`).
- **Repository Files Added/Modified:**
  - `Assets/Props/ForestProps.blend`, `Assets/Props/ForestProps.glb`, `Assets/Props/HorrorKit_ForestPropsAtlas.png` (updated)
  - `Assets/Props/HorrorKit_ForestProps_Normal.png`, `Assets/Props/HorrorKit_ForestProps_ORM.png` (created)
  - `DEVLOG.md` (modified)


### Follow-up same date — Imported & Organized All 15 Full-PBR Forest Props in Roblox Studio
- **Summary of Changes:**
  - Optimized the 3 shared PBR maps to Roblox's native `1024×1024` cap and split exports into 3 fast 5-prop packs (`ForestProps_Pack1_Ground.glb`, `ForestProps_Pack2_Landmarks.glb`, `ForestProps_Pack3_Roadside.glb`, `~1 MB` each) to resolve Studio's `HTTP StatusCode: -1 Request timed out` upload error.
  - Scaled all 15 imported `MeshPart`s exact `1 / 0.28×` to match Roblox stud scale, anchored, configured collision (`CanCollide = false` on `Prop_BrambleBush_Dead`, `PreciseConvexDecomposition` on hero props `Prop_CarWreck_Rusted`, `Prop_AbandonedWell`, `Prop_OldGravestones`, `Prop_HuntingBlind_Rotted`, `Prop_FallenLog_Rotted`, `Prop_TreeStump_Old`), removed empty `Scene` wrappers, arranged them in a clean 3×5 showroom grounded at `Y = 0.20` inside `Workspace.MapModels.HorrorKit_Props`, and mirrored all 15 into `ReplicatedStorage.Assets.MapModels.HorrorKit_Props`.
  - Visually verified all 15 props in Roblox Studio (`ForestProps_Studio_DayOverview`, `ForestProps_Studio_DayHero`) with full 4-map PBR `SurfaceAppearance` rendering cleanly and midnight horror `Lighting` restored.
- **Studio Paths Created (both `Workspace.MapModels.HorrorKit_Props` & `ReplicatedStorage.Assets.MapModels.HorrorKit_Props`):**
  - `Prop_Boulder_Mossy` (`rbxassetid://114354042048909`), `Prop_RockCluster_Small` (`rbxassetid://139663193113213`), `Prop_FallenLog_Rotted` (`rbxassetid://119196068687405`), `Prop_TreeStump_Old` (`rbxassetid://112641597933513`), `Prop_BrambleBush_Dead` (`rbxassetid://80877015223738`)
  - `Prop_AbandonedWell` (`rbxassetid://90015852430222`), `Prop_OldGravestones` (`rbxassetid://84035641808634`), `Prop_WoodsTotem` (`rbxassetid://123608712818304`), `Prop_AbandonedCampfire` (`rbxassetid://77688221244201`), `Prop_HuntingBlind_Rotted` (`rbxassetid://133686692987559`)
  - `Prop_WarningSign_Rusted` (`rbxassetid://94033745012004`), `Prop_RoadMarker_Post` (`rbxassetid://107565330190286`), `Prop_RustedBarrels` (`rbxassetid://89737716135134`), `Prop_CarWreck_Rusted` (`rbxassetid://137962507090541`), `Prop_ScrapTires_Pallet` (`rbxassetid://119729021272620`)
- **Shared PBR `SurfaceAppearance` Textures (all 15 props share one single set!):**
  - `ColorMap`: `rbxassetid://72703756900554`
  - `NormalMap`: `rbxassetid://79916903243852`
  - `RoughnessMap`: `rbxassetid://128184551029044`
  - `MetalnessMap`: `rbxassetid://112128820888656`
- **Next Steps / Known Issues:** Procedural chunk placement (`ChunkBuilder`) has NOT been wired to spawn these new props yet — ready whenever requested! Remember to grant the group experience permission to use these 15 personal-account mesh assets + 4 shared PBR texture IDs in Creator Dashboard before publishing.


---

## 2026-11-21 — justhyne1205 (via Antigravity)
- **Summary of Changes:**
  - Wired all **15 new full-PBR forest & roadside horror props** into **`ServerStorage.Modules.ChunkBuilder`** (100% object-pooled with `SurfaceAppearance` preserved and `templateBaseSize` restored on reuse):
    - **On-Road Hazards (`Prop_CarWreck_Rusted` & `Prop_RustedBarrels`) on Main Roads & Sidepath Roads (`BuildRoadChunk`):** Spawns solid crashed cars (`58%`, occasionally paired with tumbled barrels off the shoulder) and barrel clusters (`42%`) on straight/cracked/damaged road cells (`14%` on main roads, `24%` on `IsBranch` sidepaths; skipped on `Gz < 2`, bridges, intersections, delivery house cells, and depot driveways). Always hugs one outer lane (`±14.5–20` studs off centerline) so `>= 28` studs of unobstructed asphalt remain for clean high-speed swerving.
    - **Roadside Shoulders, Pull-Off Bays & Yards (`BuildRoadChunk`):** Places non-collidable `Prop_WarningSign_Rusted` (facing approaching traffic), `Prop_RoadMarker_Post`, and shoulder `Prop_ScrapTires_Pallet` / `Prop_RustedBarrels`; fills `Road_PulloffBay` pockets with solid `Prop_CarWreck_Rusted` / `Prop_RustedBarrels` / `Prop_ScrapTires_Pallet`; and mixes `Prop_ScrapTires_Pallet` / `Prop_RustedBarrels` into house yards clear of `DropoffPad`s.
    - **Rare Creepy Backwoods Clearings (`BuildForestChunk`):** Spawns `Prop_AbandonedWell`, `Prop_OldGravestones`, `Prop_WoodsTotem`, `Prop_AbandonedCampfire`, and `Prop_HuntingBlind_Rotted` in Ring 1 (`22%`) and Ring 2 (`18%`) forest chunks BEFORE tree placement, reserving a `17`-stud tree-free clearing ring (zero tree clipping!) framed with 1–2 edge brambles/rock clusters and orienting hunting blinds/gravestones toward the nearest road.
    - **Forest Floor Clutter (`BuildForestChunk`):** Scatters `Prop_Boulder_Mossy`, `Prop_RockCluster_Small`, `Prop_FallenLog_Rotted`, `Prop_TreeStump_Old`, and `Prop_BrambleBush_Dead` across forest chunks (`2–4` Ring 1, `2–3` Ring 2, `1–2` Ring 3, `0–1` Ring 4, `0.88–1.18×` scale variety, `>= 9` studs from tree trunks; shadows disabled on Rings 3–4 & small clutter for smooth FPS; boulders/logs/stumps solid in deeper woods `>= 62` studs from road while brambles stay non-collidable).
- **Studio Paths Modified:**
  - `ServerStorage.Modules.ChunkBuilder` (modified)
- **Repository Files Modified:** `GAME_DESIGN.md`, `DEVLOG.md`.
- **New Dependencies:** None (uses existing `ChunkBuilder` pooling and templates in `ReplicatedStorage.Assets.MapModels.HorrorKit_Props`).
- **Next Steps / Known Issues:** Verified live in Play mode across ~30 streamed road cells and surrounding forest rings: **`15/15` new prop types** spawned cleanly (`312` new props right at spawn; across road sweep: `17` wells, `12` gravestones, `8` totems, `9` campfires, `3` hunting blinds, `14` warning signs, `34` markers, `20` pallets, `17` road/sidepath barrel clusters, `2` on-road car wrecks including sidepath `Road_-1_20` paired with barrels, and hundreds of boulders/rocks/logs/stumps/brambles), **`TreeClips = 0`** across all clearings, visually confirmed live on-road `Prop_CarWreck_Rusted` in van headlights (`Live_OnRoad_CarWreck_Headlights`), and zero console errors. Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create (`ChunkBuilder`).


### Follow-up same date — Pulled Forest & Landmark Decorations Much Closer to Roads and Sidepaths
- **Summary of Changes (`ServerStorage.Modules.ChunkBuilder`):**
  - **Immediate Roadside Verge Clutter (`BuildRoadChunk`):** Added `1–2` forest floor props per free shoulder side (`Prop_Boulder_Mossy`, `Prop_RockCluster_Small`, `Prop_FallenLog_Rotted`, `Prop_TreeStump_Old`, `Prop_BrambleBush_Dead`) right along the grassy road verge (`±36.5–47` studs off centerline, non-collidable so the van never snags), boosted roadside signs/markers/barrels/pallets, and added roadside creepy treeline landmarks (`16%` main roads, `22%` `IsBranch` sidepaths) at `±38.5–48.5` studs facing the road (`Prop_WoodsTotem`, `Prop_AbandonedCampfire`, `Prop_OldGravestones`, `Prop_AbandonedWell`, `Prop_HuntingBlind_Rotted`).
  - **Road-Edge Biased Ring 1 Clearings & Clutter (`BuildForestChunk`):** Added `sampleForestPos(halfSpan, roadPullBias)` so Ring 1 forest clearings (`36%` chance, `extraPad = 8`, `bias = 0.85`) and Ring 1 floor clutter (`4–6` per chunk, `bias = 0.72`) cluster right along the road-facing treeline (`~50–85` studs from road center inside headlight & fog range) while keeping Ring 3–4 lighter (`1` / `0–1`) for smooth FPS.
- **Verification:** Verified live in Play mode: **`153`** new props within `85` studs of the road right at spawn (`30` immediate road-verge floor props, roadside treeline landmarks, `6` signs/markers, `26` total clearings with `0` tree clips, zero console errors). Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create (`ChunkBuilder`).


### Follow-up same date — More Frequent & Slightly Brighter Road Streetlamps
- **Summary of Changes (`ServerStorage.Modules.ChunkBuilder`):**
  - Increased streetlamp spawn frequency on straight/cracked/damaged/pull-off road cells from every 2 cells (`SpineIndex % 2 == 0`, same side only) to **~78% of road cells** (guaranteed on even cells + `58%` on odd cells, alternating left/right via `(Gx + Gz) % 2` with fallback to the opposite shoulder if blocked by a pull-off bay or depot driveway), and added an outer-bend streetlamp on `Turn90` cells.
  - Gently boosted streetlamp illumination from `Range = 55, Brightness = 1.3–1.4` to **`Range = 60, Brightness = 2.10–2.25`** so warm sodium-amber pools clearly reveal the wet highway and roadside props ahead even without relying solely on van headlights.
- **Verification:** Verified live in Play mode (`10` streetlamps across `26` streamed road chunks alternating `7` left / `3` right, `Brightness = 2.10..2.25`, `Range = 60`, confirmed clear warm asphalt pools with van headlights turned OFF via `Streetlamps_NoHeadlights_View`, zero console errors). Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create (`ChunkBuilder`).


---

## 2026-11-21 — justhyne1205 (via Antigravity)
- **Summary of Changes:**
  - Built a canonical Roblox **R6 character rig & armature** in Blender (`Assets/Animations/R6_Locomotion.blend`, exact `1 stud = 0.28 m` proportions, exact Motor6D pivot positions for `HumanoidRootPart -> Torso -> Head / Left Arm / Right Arm / Left Leg / Right Leg`) and crafted two high-quality seamless looping 30 FPS R6 locomotion animations:
    - **`R6_Walk` (`32` frames / `1.07s` seamless loop):** Standard cautious horror walk — subtle `~4.8°` forward lean, double-frequency bob + hip sway, `±6.8°` opposing torso counter-twist, pendulum-lagging arms with cautious inward guard posture, and stabilized level head.
    - **`R6_Run` (`18` frames / `0.60s` seamless loop):** Panicked horror survival sprint — aggressive `~20.5°` forward lean (`+3.8°` push-off drive), high explosive bounce (`0.43`-stud arc), wide `+56°/-52°` stride, urgent fast pumping arms tucked inward across the chest, strong `±15.2°` upper-body counter-rotation, and upward counter-pitched head staring straight ahead.
  - Visually verified both full-cycle frame strips in Blender via `look` (`mode: "frames"`), exported `Assets/Animations/R6_Walk.fbx`, `Assets/Animations/R6_Run.fbx`, and `Assets/Animations/R6_Locomotion_Keyframes.json`.
  - Generated exact 1:1 30 FPS Roblox R6 `KeyframeSequence`s (`R6_Walk` and `R6_Run`, mathematically conjugated through canonical R6 `Motor6D.C0` rotations) inside **`ReplicatedStorage.Assets.Animations`** AND attached them to a ready-to-preview R6 dummy rig at **`ServerStorage.R6_AnimationRig.AnimSaves`** (verified exact forward lean `TorsoUpZ = -0.361`, level `HeadUpZ = -0.050`, opposing arm/leg drive `RFootZ = -1.20 / LFootZ = +2.27 / LHandZ = -1.02 / RHandZ = +0.54`).
- **Studio Paths Created:**
  - `ReplicatedStorage.Assets.Animations.R6_Walk` (`KeyframeSequence`, 33 keyframes, `Loop = true`)
  - `ReplicatedStorage.Assets.Animations.R6_Run` (`KeyframeSequence`, 19 keyframes, `Loop = true`)
  - `ServerStorage.R6_AnimationRig` (canonical R6 preview rig with `AnimSaves/R6_Walk` & `AnimSaves/R6_Run`)
- **Repository Files Added/Modified:**
  - `Assets/Animations/R6_Locomotion.blend`, `Assets/Animations/R6_Walk.fbx`, `Assets/Animations/R6_Run.fbx`, `Assets/Animations/R6_Locomotion_Keyframes.json` (created)
  - `DEVLOG.md` (modified)
- **Next Steps:** Ready to preview in Blender (Spacebar) or Roblox Studio Animation Editor (drag `ServerStorage.R6_AnimationRig` into Workspace briefly to publish `R6_Walk` and `R6_Run` to Roblox), then build the advanced sprinting system!

### Follow-up same date — Advanced Sprinting, Stamina Management & Exhaustion System (`justhyne1205 (via Antigravity)`)
- **Summary of Changes:**
  - Built a full server-authoritative **Advanced Sprinting, Stamina & Exhaustion System** (`SprintService` + `SprintController`), reading all shared tuning from `ReplicatedStorage.Shared.Config.Sprint`:
    - **Stamina Drain & Regeneration (`SprintService`):** Drains `18/s` while actively sprinting and moving on foot (`flatSpeed >= 4` studs/s — holding Shift while standing still or running into a wall drains zero stamina), `1.25×` drain and `0.92×` speed while carrying a delivery box (`CarryingPackage`), and `-5` stamina per jump (`Humanoid.Jumping`). Regenerates after a `1.2s` delay (`14/s` walking, `1.5×` resting bonus = `21/s` when standing still or seated inside `DeliveryVan`). Smooth `Acceleration`/`Deceleration` WalkSpeed ramping (`16` walk ↔ `24.8` sprint) that respects `PersonalUpgradeSpeed` (`ShopService`), `AdminWalkSpeed`, and `AdminFly`.
    - **Exhaustion Penalty (0 Stamina):** Hitting `0` stamina triggers `IsExhausted = true` — forces sprint off, drops WalkSpeed to `0.72×` (`~11.52` studs/s), imposes a `2.0s` winded recovery delay (`ExhaustedRegenDelay`) and `0.6×` slower initial breath recovery, and **locks sprinting until stamina recovers past `35%`** (`ExhaustionRecoverAt`), encouraging players to manage stamina instead of burning it to zero.
    - **Dark Horror Stamina Bar HUD (`PlayerGui.SprintStaminaGui`):** Positioned bottom-center cleanly above Roblox's hotbar (`UDim2.new(0.5, 0, 1, -104)`), strictly hidden while seated in `DeliveryVan` or dead, and auto-fades out smoothly after `1.5s` when full & idle. Includes 60+ FPS client smoothing, trailing drain ghost bar, dynamic states (ice-white `STAMINA`/`SPRINTING` → amber `LOW STAMINA` → pulsing crimson `EXHAUSTED — CATCH YOUR BREATH` with dark red container glow), visible `35%` recovery threshold tick mark, and horizontal HUD shake + tired breath cue when trying to sprint while exhausted.
    - **Blender R6 `R6_Walk` & `R6_Run` Co-op Animation Driver:** `SprintController` loads `ReplicatedStorage.Assets.Animations.R6_Walk` and `R6_Run` directly and blends `Motor6D.Transform` smoothly on `RunService.Stepped` for **all** on-foot R6 players (local + co-op friends, zero asset-permission 403s), adding a subtle weary forward droop while walking exhausted, yielding arms cleanly to `CarryAnimController` when carrying a package, and yielding completely to `VanAnimController` when seated.
    - **Unified Camera & Horror Audio Feel (`AtmosphereCameraController` + `SprintController`):** Smooth `+8°` sprint FOV push, `-3°` exhausted tunnel-vision FOV tighten, punchier panicked sprint head-bob, rhythmic dark-edge exhaustion vignette pulse, and official Roblox-licensed audio (`APMOfficial` `HEARTBEAT 02 96BPM` `rbxassetid://9043365842` + `ProSoundEffects` `Breath Exhale 20 (SFX)` `rbxassetid://9113586280`).
- **Studio Paths Modified/Created:**
  - `ReplicatedStorage.Shared.Config.Sprint` (modified — tuned `ExhaustedSpeedMultiplier = 0.72`, `ReplicationRate = 12`, added `ExhaustedFovDrop = -3`)
  - `ReplicatedStorage.Shared.NetDefs` (modified — added `SprintInput` C2S remote)
  - `ServerScriptService.Server.Services.SprintService` (created)
  - `ServerScriptService.Server.Core.ServiceList` (modified — registered `SprintService`)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.SprintController` (created)
  - `StarterPlayer.StarterPlayerScripts.Client.Core.ControllerList` (modified — registered `SprintController`)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.AtmosphereCameraController` (modified — integrated on-foot sprint FOV, sprint bob, and exhaustion vignette pulse)
- **New Dependencies:**
  - Remote (`NetDefs`): `SprintInput` (C2S: `holding: boolean`, `RateLimitPerSec = 16`, `MinInterval = 0.04`).
  - Replicated Player Attributes: `Stamina`, `MaxStamina`, `IsSprinting`, `IsExhausted`, `ExhaustionRecoverAt`.
  - Official Roblox Audio Assets: `rbxassetid://9043365842` (APMOfficial Heartbeat), `rbxassetid://9113586280` (ProSoundEffects Breath Exhale).
- **Verification:** Verified live in Play mode: idle+Shift drains `0` stamina; jump deducts `-5` stamina (`100 → 95`); sprinting ramps to `WalkSpeed = 24.80`, `FOV = 78.0°`, `RootJoint` leans `22.0°` (`R6_Run`), HUD displays `SPRINTING`; hitting `0` triggers `IsExhausted = true`, locks sprint (`blocked = true`), slows `WalkSpeed = 11.52`, tightens `FOV = 67.6°`, HUD switches to pulsing crimson `EXHAUSTED — CATCH YOUR BREATH` (`0% / 35%`); recovering past `35%` clears `IsExhausted = false`, restores `WalkSpeed = 16.00`, and resumes sprinting cleanly (`peakWS = 24.80`) with **zero console errors**. Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create!

### Follow-up same date — Complete Retro-Amber CRT / Automotive Terminal 2D UI Overhaul (`justhyne1205 (via Antigravity)`)
- **Summary of Changes:**
  - Created a centralized `--!strict` design system at **`ReplicatedStorage.Shared.UITheme`** matching the reference video's obsidian-and-amber retro automotive terminal aesthetic: razor-sharp `0px` rectangular frames with mitered tangerine-amber `UIStroke` borders (`RGB(255, 146, 38)`), deep obsidian backdrops (`RGB(10, 9, 8)`), crimson vertical status indicator blocks (`RGB(232, 52, 44)`), uppercase `RobotoMono` typography, `>> CHEVRON <<` brackets, solid-amber inverted primary/hover buttons with jet-black text, flat sharp rectangular progress bars & upgrade pips, and subtle ProSoundEffects mechanical click/hover audio (`rbxassetid://9114480271`).
  - Overhauled every 2D on-screen UI across the game while preserving 100% of existing behavior:
    - **`ShiftHUD` (`ShiftController`):** Top status bar with crimson status pip & amber frame, dim-amber objective strip, inverted solid-amber `>> START SHIFT #N <<` / outlined `>> READY <<` & `>> RIGHT VAN <<` buttons with hover inversion, and framed toast notification banner.
    - **`SprintStaminaGui` (`SprintController`):** Sharp flat amber stamina gauge with vertical leading slider thumb, `35%` recovery notch, trailing drain bar, crimson pulsing `>> EXHAUSTED — CATCH YOUR BREATH` alert state, and matching sharp mobile sprint button.
    - **Shop Modals (`UpgradeShopController` / `VanUpgradeShop` & `PersonalUpgradeController` / `PersonalUpgradesGui`):** Rebuilt into `SETTINGS`-style framed obsidian-and-amber terminals with crimson title status pips, solid-amber `DONE` buttons, thin horizontal amber dividers, wrapped monospace descriptions, rectangular level pips, and solid-amber inverted `BUY` buttons. Also registered `PersonalUpgradeController` cleanly in `ControllerList` (with async `task.spawn` startup).
    - **2D Seated Van HUD (`VanController` — 2D ScreenGuis ONLY):** Restyled `VanControlsHint`, `VanSeatHint`, and `VanSpeedometer` (`>> MPH <<` + flat amber/crimson `[Q] BOOST` meter) while leaving all 3D van cab buttons, switches, dashboard gauges, tracker screen, `VanDrive`, `DashController`, and `VanAnimController` 100% untouched.
    - **Monster Alert & Defense HUDs (`DoorMonsterController`, `WindowMonsterController`, `ChaseMonsterController`, `CargoMonsterController`) & `NotificationController`:** Restyled all text alert banners, pursuit meters, and defense prompts into the unified amber/crimson terminal style while leaving all full-screen horror vignette/flash overlays, circular `[E]` prompts (`PromptUIController`), `Cmdr`, and `HorrorAtmosphereOverlay` completely untouched.
- **Studio Paths Created/Modified:**
  - `ReplicatedStorage.Shared.UITheme` (created)
  - `StarterPlayer.StarterPlayerScripts.Client.Core.ControllerList` (modified — registered `PersonalUpgradeController`)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.ShiftController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.SprintController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.UpgradeShopController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.PersonalUpgradeController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanController` (modified — 2D HUD only)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.DoorMonsterController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.WindowMonsterController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.ChaseMonsterController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.CargoMonsterController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.NotificationController` (modified)
- **Verification:** Visually verified live in Play mode across 4 screenshots (`ShiftHUD` + active `SprintStaminaGui`, `VanUpgradeShop`, `PersonalUpgradesGui`, and seated `DeliveryVan` with `VanSeatHint` + `VanSpeedometer` + `ChaseBanner`), widened `VanUpgradeShop` (`640px` + wrapped descriptions) and tucked modal crimson status pips tight beside titles after visual review, re-verified cleanly with zero script errors from any UI controller, and returned Studio cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create!
- **Follow-up (Dedicated UI Hover & Click SFX):** Upgraded `ReplicatedStorage.Shared.UITheme` from the quiet clipped flashlight click to official Roblox UI audio — **`RBLX UI Hover 01 (SFX)`** (`rbxassetid://10066931761`, `0.196s`, `Volume = 0.45`, `PlaybackSpeed = 1.12`) on hover and **`RBLX UI Select 02 (SFX)`** (`rbxassetid://10066968815`, `0.488s`, `Volume = 0.50`, `PlaybackSpeed = 1.05`) on click — preloaded via `ContentProvider:PreloadAsync`, played polyphonically via `SoundService:PlayLocalSound` (zero cutoff on rapid hovering), and gated cleanly to active clickable buttons (`btn.Active and variant ~= "Disabled"`). Verified live in Play mode (`IsLoaded = true`, zero console errors).

### Follow-up same date — Advanced Retro-Amber Terminal Polish & Fluid Button/HUD Animations (`justhyne1205 (via Antigravity)`)
- **Summary of Changes:**
  - Upgraded **`ReplicatedStorage.Shared.UITheme`** into a full animation & high-detail automotive terminal UI toolkit (fast, fluid `Quint.Out` / organic sinusoidal motion — never stiff/robotic):
    - **Animated Flanking Chevrons & Smooth Hover Fill Sweeps (`UITheme.WireButton`):** Every clickable button now renders independent left/right chevron labels (`>> <<` or hero `>>> <<<`) over a smooth horizontal `FillSweep` layer (`0.18s` `Quint.Out`), smoothly breathing in/out on hero & affordable primary buttons (`Start Shift`, affordable `BUY`), gliding inward on hover with subtle `UIScale` lift (`1.025–1.03×`) and soft press dip (`0.975×`). Added `UITheme.SetButtonText` & `UITheme.SetButtonAlwaysAnimate`.
    - **Advanced Terminal Framing & Components:** Added `UITheme.AddCornerBrackets` (L-shaped amber/crimson corner reticle accents), `UITheme.AddPanelDepth` (subtle CRT vertical sheen), `UITheme.AddHeaderStrip` (signature solid-amber/crimson mini top title strips with inverted jet-black labels matching the reference video's `LEVEL 14` & `CONTRACTS` cards), `UITheme.AddCalibrationTicks` (`25% / 50% / 75%` hairline gauge ticks), softly pulsing CRT crimson status cursors (`AddStatusPip`), `UITheme.PlayModalOpen` (`0.22s` `Quint.Out` scale-in + backdrop fade), `UITheme.WireRowHover` (smooth row border/fill glow), and `UITheme.AddAlertChevrons` (pulsing `>>> <<<` monster danger chevrons).
  - Applied across every 2D HUD & modal while preserving 100% of existing logic (including teammate's new `VanUpgradesGui` lift-pad docking check, 3D `UpgradeAttachments`, stat preview deltas, and CRT scanlines/phosphor sweep — cleanly unified into the retro-amber palette!):
    - **`ShiftController` (`ShiftHUD`):** Rebuilt top bar into a solid-amber header strip (`>> GRAVEYARD EXPRESS // DISPATCH` | `SHIFT #N` + blinking red pip) above **4 compartmentalized bordered telemetry cells** (`STOPS`, `TIME`, `CARGO`, `CASH`), corner brackets, hero animated `>>> START SHIFT #N <<<` & `>> RIGHT VAN <<` buttons, and animated slide-in toasts.
    - **`SprintController` (`SprintStaminaGui`):** Upgraded to a `328×48` corner-bracketed card with solid-amber (or pulsing crimson when exhausted) mini header strip + calibrated `25%/50%/75%` gauge ticks + vertical slider thumb & `35%` recovery notch.
    - **`UpgradeShopController` (`VanUpgradesGui`) & `PersonalUpgradeController` (`PersonalUpgradesGui`):** Unified retro-amber CRT styling with corner brackets, blinking red title cursor, smooth `Quint.Out` modal open transition, hovered row amber border glow, and animated `>> <<` buttons.
    - **`VanController` (2D HUD ONLY):** Upgraded `VanSpeedometer` with solid-amber `>> TELEMETRY` header strip, blinking red pip, corner brackets, calibrated `[Q] BOOST` bar + vertical slider thumb, and corner-bracketed `VanSeatHint`.
    - **Monster Alert Banners (`ChaseMonsterController`, `DoorMonsterController`, `WindowMonsterController`, `CargoMonsterController`):** Upgraded `ChaseBanner` with solid crimson `>> CRITICAL THREAT // PURSUIT` header strip, calibrated proximity bar, corner brackets, and smoothly pulsing `>>> <<<` warning chevrons across all monster alert boxes.
  - Left 100% untouched: all 3D van buttons/dashboard/tracker screens, `VanDrive`, `DashController`, `VanAnimController`, circular `[E]` prompts (`PromptUIController`), `Cmdr`, and `HorrorAtmosphereOverlay`.
- **Verification:** Verified live in Play mode across 3 screenshots (`ShiftHUD` compartmentalized telemetry + animated `StartNextShiftBtn` `LeftArrow` range `[8..11]` confirmed moving smoothly, `VanUpgradesGui` retro-amber CRT modal with scanlines & animated buttons, and seated `DeliveryVan` with `VanSpeedometer` + `VanSeatHint` + `ChaseBanner`), fixed `TelemetryCells` `SortOrder = LayoutOrder` (`STOPS | TIME | CARGO | CASH`), confirmed **zero console errors**, and returned Studio cleanly to `Edit` mode. Remember to **Commit Drafts** in Team Create!
- **Follow-up (ChaseBanner Keybind Fix):** Updated `ChaseMonsterController` `ChaseBanner` subtitle from `"HOLD [V] LOOK BACK"` to `"HOLD [ALT] LOOK BACK  |  [Q] BOOST  |  [H] HORN FLINCH"` to match the actual look-back control, and widened `ChaseBanner` (`520 → 580px`) with `TextScaled` + `UITextSizeConstraint(10..14)` on `Title` so long distance messages never clip against the `>>> <<<` chevrons.

### Follow-up same date — Upgraded Exterior Rain (Dynamic Floor Puddles, Ground Ripples, Swirling Gusts) & True 3D Refractive Glass Van Window Droplets (`justhyne1205 (via Antigravity)`)
- **Summary of Changes (`StarterPlayer.StarterPlayerScripts.Client.Controllers.RainController`):**
  - **Dynamic Glossy Floor Puddles (`GroundPuddles`, `28` pooled reflective `Glass` discs):** Raycasts strictly onto open-sky outdoor ground/road/grass surfaces (`Normal.Y >= 0.88`, verified clear sky overhead, never inside `DeliveryVan` or under roofs). Each puddle smoothly spreads in (`FadeIn = 0.65–1.05s`), gleams with high `Reflectance = 0.84` catching streetlamps/headlights/flashlight while pulsing `2` live expanding concentric raindrop ripples across its surface, smoothly shrinks & evaporates (`FadeOut = 0.85–1.25s`), and triggers a crisp **footstep/sprint water-splash burst** (`PuddleFootSplash`) + ripple whenever the player steps across an active puddle on foot.
  - **Crisp Surface-Raycasted Ground Ripples & Crown Micro-Splashes (`GroundRipples`, `34` pool):** Replaced blurry floating `smoke_main` ground puffs with crisp flat expanding water-impact rings hitting real outdoor ground heights + arcing glistening `CrownSplashFlecks` micro-droplets.
  - **Sharper Multi-Layer Rain Needles + Live Swirling Wind Gusts + Close On-Foot Halo:** Added `OnFootCloseRain` (strictly disabled inside `DeliveryVan` or under roofs) closing the rain-free gap around on-foot players, added subtle `LowAsphaltSpray` mist skimming the wet highway, upgraded all 8 curtains to brighter moonlit silver-blue needles (`LightEmission = 0.68`, `Squash = 3.8–6.2`), and replaced static rain acceleration with live organic swirling wind gusts that intensify during Chaser storms (no thunder/lightning added, per user request).
  - **True 3D Refractive `Glass` Van Window Droplets + Wet Micro-Trails (`VanWindowDrops3D`):** Replaced the flat 2D `SurfaceGui` window circles (`VanWindowRain`) with **`202` pooled true 3D flattened `Glass` beads + `72` pooled shrinking wet-wake micro-trail beads** parented strictly under `CurrentCamera.RainSystem.VanWindowDrops3D` (zero parts/welds on `DeliveryVan`, moved in one native `workspace:BulkMoveTo` call per frame). Placed flush on the interior-facing glass surface so the 1st-person camera refracts real headlights/streetlamps/trees through each curved 3D glass bead with stick-slip sliding, 3D teardrop elongation aligned to motion, cornering inertia, driving aerodynamics, and a clear center driving sightline. Preserved camera lens droplets (`38` pool) and 4-layer ProSoundEffects rain audio untouched.
- **Verification:** Visually verified live in Play mode across 2 screenshots (`Rain_Outdoors_V2`: `9` active glossy puddles, `8` expanding ground ripples, gleaming wind-driven rain needles slashing through flashlight & taillight cones; `Rain_Van3DGlassDroplets_V2`: `120` active 3D curved refractive `Glass` window domes + `66` wet trailing micro-beads gleaming realistically across `SideGlassL` and `Windshield` with clean center driving sightline and **zero console errors**). Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** (`RainController`) in Team Create!

### Follow-up same date — Toast Text Overflow Fix & Floating Waypoint Billboards Restyle (`justhyne1205 (via Antigravity)`)
- **Summary of Changes:**
  - **Toast & Button Overflow Fixes (`ShiftController`, `UITheme`):**
    - Converted `ShiftHUD.ToastFrame` from a fixed `560×34` frame into an auto-sizing (`AutomaticSize = XY`, `UISizeConstraint` `340×36`–`780×84`) terminal card with an inner padded `ToastContent` wrapper (`PaddingLeft = 24, Right = 18, Top/Bottom = 10`) and wrapped `ToastLabel` (`AutomaticSize = XY`, `TextWrapped = true`), keeping `CornerBrackets` and the pulsing left `StatusPip` razor-flush on the outer edges while ensuring even the longest depot arrival toasts (`>> ARRIVED AT DEPOT! RESTOCK & VISIT SHOPS, THEN PRESS 'START SHIFT #3' WHEN EVERYONE IS READY!`) stay strictly inside the amber frame with balanced padding.
    - Widened `ShiftHUD.ObjectiveBar` (`610 → 652px` flush with `TopStatusBar`) with `TextScaled` + `UITextSizeConstraint(9..12)`.
    - Added `TextScaled` + `UITextSizeConstraint(9..textSize)` to `UITheme.WireButton` center labels so button text never overflows or collides with animated flanking `>> <<` / `>>> <<<` arrows.
  - **Floating World Waypoint Billboards Restyled (`ServerScriptService.Server.Services.ShiftService`):**
    - Overhauled both **`DepotBillboard`** (`NEXT FACTORY DEPOT / [ PULL IN TO RESTOCK ]`, `280×86`) and **`StopBillboard`** (`>> DELIVERY STOP #N / <ADDRESS> / [ DELIVER HERE ]`, `268×90`) from the legacy rounded green/gold `GothamBold` boxes into matching retro-amber CRT terminal cards: `4px`-inset obsidian `TerminalCard` (prevents `BillboardGui` edge clipping of borders), `2px` mitered tangerine-amber `UIStroke`, bright-amber corner reticle L-brackets, solid-amber `18px` mini header strip with crimson vertical status pip, and uppercase `RobotoMono` Bold text with `UITextSizeConstraint`.
    - Confirmed via full codebase grep that the only remaining `BillboardGui` / `GothamBold` is `PromptUIController` (circular `[E]` prompts, intentionally left untouched).
- **Studio Paths Modified:**
  - `ReplicatedStorage.Shared.UITheme` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.ShiftController` (modified)
  - `ServerScriptService.Server.Services.ShiftService` (modified)
- **Verification:** Verified live in Play mode via `screen_capture` and geometry readbacks: long depot toast rendered cleanly at `ToastFrame = (613×37)` containing `ToastLabel = (570×12)` with `19px` right margin and flush `StatusPip`/`CornerBrackets`; `DepotBillboard` & `StopBillboard` verified live with sharp unclipped `2px` amber borders, corner reticle brackets, and solid-amber header strips; **zero game script errors** in console. Studio returned cleanly to `Edit` mode. Remember to **Commit Drafts** (`UITheme`, `ShiftController`, `ShiftService`) in Team Create!


---

## 2026-11-21 — Miah (via Antigravity)
- **Summary of Changes:**
  - **Adaptive Eerie Horror Soundtrack & Ambient Soundscape (`StarterPlayer.StarterPlayerScripts.Client.Controllers.AudioController`):**
    - Replaced the stub `AudioController` with a full `--!strict` adaptive horror music & nocturnal soundscape controller using **100% official `APMOfficial` & `ProSoundEffects`** Creator Store audio pooled in `SoundService.GameMusicAudio` (preserving `AudioController:PlaySFX(soundName)` intact):
      - **Rotating Eerie Exploration Score:** Crossfades every `108s` (`10s` fade) between `Unsettling Atmosphere (b)` (`rbxassetid://9040209498`, `129.3s`) and `The Plot Thickens Underscore` (`rbxassetid://9044409538`, `105.7s`), softened & warmed during `Prep` at the Factory Depot and gently ducked inside the closed cab.
      - **Dark Ambient Bed & Ghostly Forest Wind:** Layered `Eerie Ambience 1 (SFX)` (`rbxassetid://9112775175`, `56.0s`) + reverb-washed `Eerie Wind Constant 1 (SFX)` (`rbxassetid://9114228774`, `36.0s`) that blooms outdoors on foot and in deeper woods while hushed in the depot/closed van.
      - **Adaptive Monster Tension / Pursuit Score:** Smoothly crossfades in `Stalking the Prey 60` (`rbxassetid://1835248722`, `62.0s`) and `Panic Attack Drone` (`rbxassetid://1835353529`, `53.3s`) driven by live threat telemetry across all 11 monsters (`ChaseActive`, `WindowThreatSide`/`WindowApproachSide`, `DoorThreatActive`, `PhantomSemiActive`, `CargoThreatActive`, `StareActive`/`StareStep`, `GremlinActive`, `HitchhikerActive`, `PorchThreatActive`, `ForestHunterActive`/`ForestHunterState`, `FogWardenActive`/`FogWardenLayer`) plus `< 35s` shift timer urgency. Mirrors `RainController`'s uncanny silence ducking when `ForestHunter` is in `"Stalk"`/`"Creep"`.
      - **Sparse Distant Eerie Stingers:** Pooled `4` preloaded reverb-washed stingers (`rbxassetid://9045758386`, `1837212740`, `1841714854`, `9120332419`) firing every `46–88s` during quiet active stretches outside the depot.
      - **Public API:** `PlaySFX`, `SetMusicEnabled`, `IsMusicEnabled`, `SetMusicVolume`, `GetMusicVolume`, `SetTensionOverride`, `TriggerStinger`, `GetDebugState`.
  - **3D Spatial Co-op Van Driving Soundscape (`StarterPlayer.StarterPlayerScripts.Client.Controllers.VanAudioController`):**
    - Created and registered `VanAudioController` in `ControllerList` (Phase 4 right after `DashController`). Attaches two client-only `Attachment` emitters (`VanEngineAudioEmitter` at the hood, `VanTireAudioEmitter` at the lower chassis) to `DeliveryVan.Chassis.Chassis` — **zero parts or welds added** (100% Moving Van Physics Rule safe) with `InverseTapered` 3D roll-off (`14..175` studs) so passengers and outside co-op teammates hear the van in 3D space:
      - **Ignition Startup / Shutdown & Stall Sputter:** `1987 Jeep Wrangler Engine Start` (`rbxassetid://9125345489`, smooth `1.65s` starter-catch envelope when entering `DriverSeat`) and `1988 Chevy Pickup Truck Rough Start Turn Off` (`rbxassetid://9125346576`, shutoff clunk on exit or metallic stall sputter on `VanStunned`).
      - **Layered 5-Gear Engine Idle & Driving Load:** `1934 Rolls-Royce 1 (SFX)` (`rbxassetid://9112919271`, `46.0s` vintage front-hood idle + rattle) + `1988 GMC Suburban 2 (SFX)` (`rbxassetid://9112736090`, `51.4s` heavy V8 driving hum) synced 1:1 to `DashController`'s 5-gear km/h RPM bands with brief mechanical gear-shift dips (`0.18s`) and extra low-end punch scaling with `UpgEngine` (`0..3`).
      - **Wet Asphalt vs. Off-Road Tire Roll & Drift/Brake Screech:** `Tires Constant On Bridge 1` (`rbxassetid://9112887000`, grounded speed-scaled tire hum shifting EQ warmer/crunchier off-road) + `Skids Tire Screeches On Dirt Pavement 4` (`rbxassetid://9119186275`, driven by lateral slip, `PoseDrift`, `PoseHandbrake`, and hard braking).
      - **Turbo `[Q]` Boost Roar & Blow-Off Release:** `Rumble Constant Roar Energy Building 1` (`rbxassetid://9118747824`) + `Whoosh Giant Swish By 5` (`rbxassetid://9120722836`) triggered locally (`VanController:GetBoostFovAdd()`) and remotely (`PoseBoost` drop) for co-op listeners.
      - **Dynamic Cabin Acoustic Muffler (`EqualizerSoundEffect`):** Muffles exterior highs inside the closed van (`HighGain = -10.5`, `MidGain = -2.2`, `LowGain = +2.4`) and smoothly opens up when rolling/breaking windows (`WindowRollL/R`, `WindowBrokenL/R`), leaning out, opening any van door, switching to `[V]` 3rd-person cam, or standing outside.
- **Studio Paths Modified/Created:**
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.AudioController` (modified)
  - `StarterPlayer.StarterPlayerScripts.Client.Controllers.VanAudioController` (created)
  - `StarterPlayer.StarterPlayerScripts.Client.Core.ControllerList` (modified — registered `VanAudioController`)
- **Repository Files Modified:** `GAME_DESIGN.md`, `LESSONS.md`, `DEVLOG.md`.
- **New Dependencies:**
  - Client audio containers: `SoundService.GameMusicAudio`; client-only `Attachment`s `VanEngineAudioEmitter` & `VanTireAudioEmitter` under `Workspace.DeliveryVan.Chassis.Chassis`.
  - 18 verified free official `APMOfficial` / `ProSoundEffects` Audio IDs (`9040209498`, `9044409538`, `9112775175`, `9114228774`, `1835248722`, `1835353529`, `9045758386`, `1837212740`, `1841714854`, `9120332419`, `9112919271`, `9112736090`, `9125345489`, `9125346576`, `9112887000`, `9119186275`, `9118747824`, `9120722836`).
- **Next Steps / Known Issues:**
  - Verified live in Play mode: `GameMusicAudio: 10/10 loaded | VanAudio: 8/8 loaded` (`18/18` sounds `IsLoaded == true` with valid `TimeLength > 0`, zero console errors from audio controllers); confirmed Prep vs. `ChaseActive = true` adaptive tension crossfade (`EerieTrackA 0.150 -> 0.022`, `TensionMusic 0.000 -> 0.340`, `PanicDrone 0.000 -> 0.210`), seated driving engine/tire scaling, and closed-cab vs. open-window EQ unmuffling (`HighGain -10.5 -> -4.6`). Studio returned cleanly to `Edit` mode.
  - Note for teammates: pre-existing `VanCamController` logged a `Script timeout: exhausted allowed execution time` in `populateViewportScene` (`line 189/190`) on Play boot (unrelated to audio; consider yielding periodically inside `populateViewportScene`).
  - Remember to **Commit Drafts** (`AudioController`, `VanAudioController`, `ControllerList`) in Roblox Studio Team Create!
