# Agent Lessons Learned (Self-Improvement Loop)

> **Every new agent MUST read this file before starting work** so the next run never repeats a mistake.

## How This Self-Improvement Loop Works
1. **Before you start:** Read every active rule below along with `GAME_DESIGN.md` and `DEVLOG.md`.
2. **Every run (or every few runs):** Review the saved logs of recent runs — `DEVLOG.md`, your agent's saved run logs/transcripts when accessible (e.g., Antigravity's `.system_generated/logs/transcript.jsonl`, Claude's session logs), and any corrections the team gave you during the session.
3. **Record new lessons:** For every failure, preventable tool error, wasted-token/time detour, or user correction, add a short rule in the format **`Rule`** — *Reason*.
4. **Fix wrong or outdated rules:** If a rule turns out to be wrong, overly broad, or fixed in the codebase, **update or delete it** immediately. Keep entries concise and deduplicated.

---

## Roblox Studio & MCP
- **Check Studio mode (`get_studio_state`) before running `execute_luau`:** `Edit` datamodel fails in Play mode, and `Server`/`Client` datamodels fail in Edit mode. — *Reason:* Calling the wrong datamodel wasted 4 consecutive MCP error round-trips during Play-mode testing.
- **Always put explicit timeouts on yields inside `execute_luau` (especially on `Client`), and don't queue slow calls in parallel with `get_console_output`:** Never write unbounded `WaitForChild()` or `CharacterAdded:Wait()` in test scripts. — *Reason:* An unbounded client yield stalled the Roblox Studio MCP queue for 3 full minutes (`context deadline exceeded`) twice in one session, blocking parallel `get_console_output` calls behind it.
- **Inspect remotes in Edit mode before flagging unknown Play-mode remotes:** `ServerScriptService.AntiCheat` intentionally renames `ReplicatedStorage.Remotes.DevRemote` to a random string at runtime (e.g., `VYUCVTYJIDVADSUTPCV`). — *Reason:* Avoids wasting turns investigating or breaking the anti-cheat remote.
- **Avoid calling `WaitForChild` on `StarterGui.Cmdr` children without a replication guard:** Wait for `StarterGui.Cmdr` / `PlayerGui` replication safely inside `CmdrController`. — *Reason:* Unchecked `CmdrClient` initialization yielded indefinitely on `Window` and broke client test execution.

## Game Architecture & Live Place Gotchas
- **Check existing frameworks (`Loader`, `Cmdr`, `ProfileStore`, `AntiCheat`) before designing a feature:** Build admin commands inside `Cmdr`, and build game systems as `Loader`-managed Services/Controllers (`Loader.Get(...)`) instead of standalone `Script`/`LocalScript` instances. — *Reason:* The user had to clarify using `Cmdr` for admin commands and later asked to refactor standalone van/shop/chute scripts into modular Services and Controllers.
- **Exempt new movement/vehicle mechanics in `AntiExploitService`:** Any boost, high-speed driving (van boost `140` > `120` cap), server-set `WalkSpeed`, or admin flight (`AdminFly`) must be explicitly exempted. — *Reason:* Otherwise `AntiExploitService` falsely flags or rubber-bands legitimate gameplay and admin testing.
- **Account for `HorrorKit_Houses` porch collision bounds:** `HorrorKit_Houses` meshes (`Default` CollisionFidelity) block players ~`4.2` studs in front of porches. — *Reason:* Delivery `ProximityPrompt`s need generous range (`MaxActivationDistance >= 14`, `RequiresLineOfSight = false`) so front-of-porch drop-offs work reliably.

## Blender MCP & 3D Asset Pipeline
- **Pass individual field names to `Blender.get_scene_info`:** Use `location`, `size`, `ground`, `details`, `weights`, `topology`, `settings` — never category headings like `placement` or `contents`. — *Reason:* Passing category headings fails with `Error: unknown fields`.
- **Bypass `Blender.get_addon_status` if it raises `No module named 'blender_mcp.config'`:** Query Blender version and scene state directly via `execute_blender_code`. — *Reason:* Addon version mismatch causes `get_addon_status` to fail even while all modeling/export tools work normally.
- **Solo the target collection before calling `look` when multiple assets share `(0, 0, 0)`, and purge orphan datablocks before exporting:** Run `bpy.ops.outliner.orphans_purge(...)` after rebuilding a mesh or armature. — *Reason:* Prevents overlapping renders at origin and keeps `.001` suffixes out of exported rig/mesh datablock names.

## Git, Docs & Team Workflow
- **Never run a bare `git push` that can hang waiting on interactive HTTPS prompts:** Run `GIT_TERMINAL_PROMPT=0 git push` so it fails fast when credentials aren't cached, then give the user the commit hash to push via GitHub Desktop. — *Reason:* Non-interactive terminals without macOS Keychain GitHub tokens hang or fail on username prompts.
- **On Windows without `git` on `PATH`, use GitHub Desktop's bundled `git.exe`:** Located at `%LOCALAPPDATA%\GitHubDesktop\app-*\resources\app\git\cmd\git.exe` (version folder updates automatically with GitHub Desktop). — *Reason:* Saved teammates using Claude on Windows from failing git sync steps.
- **Use exact repo filenames (`GAME_DESIGN.md`, `DEVLOG.md`, `LESSONS.md`):** Verify filenames before referencing them. — *Reason:* `GAME DESIGN.md` previously had a space mismatch against the instruction docs and caused lookup confusion.
