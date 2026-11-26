
# Project & Team Workflow Rules (Roblox Team Create + MCP)

You are collaborating with a team of developers using both **Google Antigravity (`GEMINI.md`)** and **Claude (`CLAUDE.md`)** on a Roblox horror delivery game. Follow this exact workflow on every session:

## 1. Start of Session (Sync & Read Context First)
- **Pull Latest Changes:** Before reading files or answering, run `git pull` in the workspace terminal so you have the latest updates from teammates.
- **Read Lessons Learned Before Starting:** Always read `LESSONS.md` first so you never repeat a past failure, wasted-token loop, or correction from previous runs.
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
- **Pixel UI Asset Reuse & Placeholder Tracking Rule:** All new or modified 2D UIs must reuse existing pixel art assets from `ReplicatedStorage.Assets.PixelUI.AssetCatalog` (and Studio `Prefabs` / `StarterGui.PixelGame_*` or `PixelLobby_*` templates) using `ResampleMode = Enum.ResamplerMode.Pixelated`, `AssetCatalog.Fonts.Header` (`Arcade`) / `AssetCatalog.Fonts.Body` (`Code`), `AssetCatalog.Colors`, and `UITheme` pixel helpers (`WirePixelButton`, `WirePixelRowHover`, `SetPixelPips`, `RegisterBlinkPip`, `PulsePop`, `PlayPixelModalOpen`, `AttachPhosphorSweep`). Do not create new smooth/rounded procedural GUIs or duplicate art uploads. If a feature needs custom pixel art not yet in `AssetCatalog.Images`, build a clean retro-CRT placeholder using existing catalog frames/icons (or flat pixel-aligned frames) and **immediately log it in `UI_PLACEHOLDERS.md`** with its Studio path, controller, temporary fallback, and desired art for the next UI overhaul.

## 3D Modeling & Roblox Asset Standards (Horror)
Mandatory rules for all Blender (`bpy`) / 3D modeling tasks:
1. **Roblox Scale & Transform Standards:**
   - Use exact Roblox proportions (`1 stud = 0.28 meters`; standard player reference height = `1.65m`).
   - Set object origin to bottom-center at `(0, 0, 0)`, facing `-Z` forward.
   - Apply all transforms (Location, Rotation, Scale) before exporting.
2. **Topology, Polycount & Export Pipeline:**
   - Keep every individual mesh strictly under `8,000–10,000` triangles to prevent Roblox import failures.
   - Recalculate all normals outside and delete loose/stray vertices before export.
   - Apply Smart UV Project (island margin `0.02`) and use named Principled BSDF materials with flat base colors/roughness (no complex procedural shader nodes that fail to bake to FBX).
   - Export `.fbx` with `scale_units='FBX_SCALE_NONE'`, scale `1.0`, and Leaf Bones disabled.
3. **Phased "Blockout-to-Detail" Execution:**
   - Never generate complex organic shapes in a single unverified pass.
   - Construct scripts in 4 phases: (1) Base primitive bounding box & proportion check, (2) Primary silhouette extrusions/tapers using `bmesh`, (3) Secondary horror details (damage, wear, jagged edges), and (4) Geometry cleanup, tri-count verification, and FBX export.
4. **Horror Aesthetic & Geometric Rules:**
   - Avoid vague adjectives ("scary", "creepy"); use measurable geometry cues (e.g., elongated limb ratios, `5–15` degree crooked offsets, sag depths).
   - Break strict mirror symmetry by offsetting left/right angles and wear by `5–12` degrees for an unsettling, organic silhouette.
   - Check existing scene/workspace reference bounds (doorways, player rigs) before sizing new assets so models fit without clipping.

## 3. End of Session (Log & Push Changes)
- **Remind to Commit Drafts:** Remind the user to commit any open script drafts in Roblox Studio's **Drafts** widget if Team Create Collaborative Editing is active.
- **Keep `GAME_DESIGN.md` Up to Date:** Whenever a new gameplay idea, mechanic, monster, upgrade, or system is proposed that is not yet in `GAME_DESIGN.md`, add or update it in `GAME_DESIGN.md` so the design doc stays the single source of truth.
- **Keep `UI_PLACEHOLDERS.md` Up to Date:** Whenever a placeholder UI element is added or replaced with final pixel art, add or remove its entry in `UI_PLACEHOLDERS.md`.
- **Sync AI Instruction Files When Needed:** If a workflow convention, architectural rule, or cross-agent guideline changes in a way teammates' AIs should follow, update `GEMINI.md`, `CLAUDE.md`, and `README.md` together (preserving developer-specific notes like attribution lines). If no rule changes are needed, just continue updating `DEVLOG.md`.
- **Self-Improvement Loop (`LESSONS.md`):** Every run or so, audit the saved logs of recent runs (`DEVLOG.md` plus your agent's saved run transcripts/logs when accessible) to find every place an agent failed, wasted tokens/time, or was corrected by the team. Add each new lesson to `LESSONS.md` as a short **rule + reason**. If an existing rule turns out wrong or outdated, fix or remove it—the next run should never repeat a mistake.
- **Update `DEVLOG.md`:** Before finishing your response, **ALWAYS append a concise entry to the BOTTOM of `DEVLOG.md`** (oldest at top, newest at bottom — **NEVER prepend at the top**) containing:
  - **Date & Developer/Agent** (e.g., Antigravity or Claude). **When Claude makes changes on behalf of this user, log them as `Riplatki (via Claude)`** and put `(Riplatki)` in the commit message (e.g., `Update devlog (Riplatki)`), so teammates and their AIs know who made the change.
  - **Summary of Changes** (what feature or fix was built)
  - **Studio Paths Modified/Created** (exact hierarchy paths in Studio)
  - **New Dependencies** (any new RemoteEvents, tags, attributes, or DataStore keys)
  - **Next Steps / Known Issues**
- **Sync to GitHub:** Run `git add . && git commit -m "Update devlog" && git push` so teammates receive your updates immediately. If a push fails due to a teammate pushing at the same time, run `git pull --rebase`, resolve any `DEVLOG.md` merge cleanly, and push again.
