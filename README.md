# Horrorr-Game
# Project & Team Workflow Rules (Roblox Team Create + MCP)

You are collaborating with a team of developers using both **Google Antigravity (`GEMINI.md`)** and **Claude (`CLAUDE.md`)** on a Roblox horror delivery game. Follow this exact workflow on every session:

## 1. Start of Session (Sync & Read Context First)
- **Pull Latest Changes:** Before reading files or answering, run `git pull` in the workspace terminal so you have the latest updates from teammates.
- **Read Single Source of Truth:** Always read `GAME_DESIGN.md` and `DEVLOG.md` in the root of this repository before planning or writing any code.
- **Inspect Live Studio via MCP:** Because this project uses Roblox Studio Team Create without Rojo, always use the **Roblox Studio MCP** to inspect the live `Workspace`, `ServerScriptService`, `ServerStorage`, and `ReplicatedStorage` hierarchy before creating or modifying scripts.

## 2. Core Technical & Architectural Rules
- **Luau Style & Service/Controller Architecture:** Use `--!strict` at the top of new scripts and build features as modular `ModuleScript`s managed by `ReplicatedStorage.Shared.Loader`:
  - Put server systems in `ServerScriptService.Server.Services` and register them in `ServerScriptService.Server.Core.ServiceList` (e.g., `MapService`, `ChuteService`, `ShopService`, `ShiftService`, `PackageService`, `MonsterService`).
  - Put client systems in `StarterPlayer.StarterPlayerScripts.Client.Controllers` and register them in `StarterPlayer.StarterPlayerScripts.Client.Core.ControllerList` (e.g., `VanController`, `ShiftController`, `CmdrController`, `WindowMonsterController`).
  - Expose clean public methods and `Signal`s so other modules can call them via `Loader.Get("ServiceOrControllerName")` (for example, monster controllers calling `Loader.Get("VanController")`), and put shared/helper modules in `ServerStorage.Modules` or `ReplicatedStorage.Shared`.
  - Only create standalone `Script` or `LocalScript` instances when literally required (such as bootstrap entry points or model-embedded scripts).
- **Avoid Duplicate Systems:** Check `DEVLOG.md` and live Studio folders for existing `RemoteEvent`s, `CollectionService` tags, and DataStore keys before creating new ones.
- **Moving Van Physics Rule:** Never anchor parts attached to the delivery van. Any player-placed upgrades (cameras, monitor screens, props) must be welded to the van's primary body using `WeldConstraint`s and saved as relative `CFrame` offsets (`vanRoot.CFrame:ToObjectSpace(...)`).
- **Procedural Map Rule:** Keep infinite road/house generation memory-safe by despawning or pooling chunks behind the van.

## 3. End of Session (Log & Push Changes)
- **Remind to Commit Drafts:** Remind the user to commit any open script drafts in Roblox Studio's **Drafts** widget if Team Create Collaborative Editing is active.
- **Keep `GAME_DESIGN.md` Up to Date:** Whenever a new gameplay idea, mechanic, monster, upgrade, or system is proposed that is not yet in `GAME_DESIGN.md`, add or update it in `GAME_DESIGN.md` so the design doc stays the single source of truth.
- **Sync AI Instruction Files When Needed:** If a workflow convention, architectural rule, or cross-agent guideline changes in a way teammates' AIs should follow, update `GEMINI.md`, `CLAUDE.md`, and `README.md` together (preserving developer-specific notes like attribution lines). If no rule changes are needed, just continue updating `DEVLOG.md`.
- **Update `DEVLOG.md`:** Before finishing your response, append a concise entry to `DEVLOG.md` containing:
  - **Date & Developer/Agent** (e.g., Antigravity or Claude)
  - **Summary of Changes** (what feature or fix was built)
  - **Studio Paths Modified/Created** (exact hierarchy paths in Studio)
  - **New Dependencies** (any new RemoteEvents, tags, attributes, or DataStore keys)
  - **Next Steps / Known Issues**
- **Sync to GitHub:** Run `git add . && git commit -m "Update devlog" && git push` so teammates receive your updates immediately. If a push fails due to a teammate pushing at the same time, run `git pull --rebase`, resolve any `DEVLOG.md` merge cleanly, and push again.

