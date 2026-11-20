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




