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
- **Summary of Changes:** Overhauled the Skinwalker Chase Monster into a **Colossal Broken-Legged Crawler**:
  - **`ReplicatedStorage.Assets.Monsters.Monster6_Skinwalker` (rebuilt, 168 parts, `ColossalCrawlerRig = true`):** ~38×28×29-stud crawler (bigger than the van) with a giant bleached deer skull, towering asymmetrical antlers, unhinging `Jaw` + crimson `GulletCavern`/neon `GulletCoreGlow` throat, 24-stud 2-bone arms (`UpperArm` 11.5 + `Forearm` 12.5, rest-aligned along -Z for zero-drift `CFrame.lookAt` IK), new articulated `FingersL/R` bones (4 curlable talons each + thumb talon), and fractured hind legs rest-aligned along +Z for ragdoll solving.
  - **`ServerStorage.Modules.Monsters.ChaseMonster`:** New `ChaseSpawnSide` attribute; spawns `42` studs inside the woods on a random side and crawls onto the road over a `2.3s` emergence; hand-over-hand lurching haul speed (`0.78–1.22×` surge, same average); grab distance `4.5` studs; `Ramming` is now a `1.25s` van grab (van yanked backward + upward toward the maw, stunned) followed by a wrench-free forward jolt and rebound to `52` studs; bigger on-foot reach (`15.5`).
  - **`StarterPlayer.StarterPlayerScripts.Client.Controllers.ChaseMonsterController` (rewritten):**
    - Forest emergence: woods roar + tremor, topples the 3 nearest map trees toward the road and shoves flanking trees apart (non-destructive client-side tilt that restores afterward; spawns temporary fallen trees if the area is open).
    - Environment anchor scanner over `ProceduralMap`, `StartAreaForest`, `StartArea`, `Factory`: classifies **House / LampPost / Tree / Guardrail** (fallback: road curb), solves 2-bone analytical arm IK to the prop surface, curls fingers −82° around it on plant, and bends trees/poles/guardrails back as it hauls past.
    - Verlet ragdoll broken legs (gravity, road clamp, asphalt friction, fishtail chatter, twisted joint angles).
    - Devour climax: rears up, jaw unhinges 82°, both hands clamp the van's sides (verified hands at van-local `(±5.0, 4.3, 4.6)`).
  - **`chasemonster` Cmdr:** added `devour` / `eat` / `grab` aliases for the grab climax.
- **Studio Paths Modified/Created:** `ReplicatedStorage.Assets.Monsters.Monster6_Skinwalker` (rebuilt), `ServerStorage.Modules.Monsters.ChaseMonster`, `StarterPlayer.StarterPlayerScripts.Client.Controllers.ChaseMonsterController`, `ServerScriptService.Cmdr.Commands.chasemonster`, `ServerScriptService.Cmdr.Commands.chasemonsterServer`.
- **Repository Files Modified:** `DEVLOG.md`, `LESSONS.md`.
- **Next Steps / Known Issues:** Verified in Play mode: `State=Spawning Side=L Lat=-19.1`, `ToppledTreesCount=4`, `Ext=38.6×27.8×28.7`, live grabs `House:Bay_Wall_East / Road:RoadCurb` and `House:Window_East_Upper_02`, `LegDragVerified=true`, `cm devour` → `State=Ramming, GrabKind=Van, JawOpenDeg=82`, rear doors popped open, zero script errors. Play-mode `screen_capture` only rendered the UI layer, so the look still needs a visual pass in-game (`cm spawn`, `cm close`, `cm devour`). Fallback sounds are placeholders. Remember to **Commit Drafts** in Team Create.
