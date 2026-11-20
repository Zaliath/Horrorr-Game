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
