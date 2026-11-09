---
name: jarvis-launcher-scenario-start
description: Start or resume a Jarvis scenario (a demo, a presales or integration environment) with jarvis-launcher - pick the working folder, reach the launcher (directly, or through the bridge on the PC when your commands cannot reach Docker, as in Cowork or in the Codex sandbox - started with the bridge_start tool or by the user), then follow the agent-workflow guide. Use whenever the user asks to build, prepare, resume or change a Jarvis demo, scenario or environment, for presales or integration work.
---

# Start a Jarvis scenario

The Jarvis environment and its tools run on the user's PC through `jarvis-launcher`. This skill gets you connected to it; the rest of the work is in the guide `agent-workflow`. Talk to the user in their language. Always write the command as `jarvis-launcher` (never a personal alias); in the steps you give the user, write `npx jarvis-launcher …`, which works without installing anything.

**Connect first, design after.** Do not propose a data model before you have read the guide `agent-workflow` and the skill `scenario-design` from the launcher (step 4): they hold the Jarvis modelling rules. Until then you may ask about the brief, but present no entities or fields. Connecting (the folder, the bridge, `status`) writes no demo files, so it can come first; tell the user that the model proposal follows right after.

## 1. The working folder

Ask the user whether to create a **new folder** or to use an existing **empty folder**: one folder per demo. In Cowork the folder must be **connected** (ask the user to grant access to it); in the Codex app it is the local project folder (a cloud task cannot reach the PC). A folder that holds other files is not a working folder: ask for another one.

Never the user's **home folder** itself, the disk root, or Desktop, Documents, Downloads, OneDrive: the launcher refuses these protected folders. Suggest a new sub-folder instead, for example `<home>/jarvis/<demo name>`.

## 2. Direct or bridge

Decide once, at the start, from what you have:

1. **You have the `bridge_start` tool** (MCP server `jarvis-launcher`, as in Cowork or in the Codex app): **bridge mode**, step 3, whenever `docker version` does not work in your shell — a VM without Docker (Cowork) or a sandbox that blocks Docker or the network (Codex, Claude Code with sandboxing). Do not ask the user to widen the sandbox: the bridge runs the launcher on the PC for you. If you are not sure whether your shell reaches Docker, use the bridge.
2. **You can run commands on the PC itself** (`docker version` and `jarvis-launcher --version` both work, as in Claude Code in a PC terminal): **direct mode**. Run `jarvis-launcher <command> --json` in the working folder and go to step 4.
3. **Neither**: bridge mode started by the user (step 3, "Ask the user"), or, for an agent with a sandbox, the permission below.

**Agents with a sandbox and no `bridge_start` tool.** The launcher needs the Docker engine (on Windows its pipe `//./pipe/docker_engine`) and the network (image registry, Scenario Library). Inside a sandbox `doctor` may report the Docker engine "not reachable" or "permission denied" while Docker Desktop is running, and `start` may fail to reach the registry. That is the sandbox, not the launcher:
- prefer the bridge started by the user (step 3); otherwise ask for the narrowest permission that works: running `jarvis-launcher` commands outside the sandbox (or with Docker and network access), not turning the sandbox or the approvals off altogether;
- mention once that `npx jarvis-launcher mcp install` (guided: it asks for the app and the demo folder) lets you start the bridge yourself next time;
- when you report a failure, say which command, the error, whether it ran inside the sandbox, and the result after the permission.

## 3. Bridge mode

The bridge is a small program on the PC, running in the working folder: it executes your requests there.

**Is it running?** If `.jarvis/bridge/jl.mjs` exists in the working folder, run `node .jarvis/bridge/jl.mjs status`: exit code 0 means the bridge answers; exit code 4 with `BRIDGE_DOWN` (or no `jl.mjs` at all) means it is not running. Do not judge the heartbeat by comparing its time with your clock: the VM clock may differ from the PC's. Do not work around the client by reading `outbox/` yourself: if it misbehaves, tell the user.

**Start it yourself** when you have the `bridge_start` tool (MCP server `jarvis-launcher`, installed once per PC with `npx jarvis-launcher mcp install`, for Claude Desktop and/or Codex):

1. **Ask first** and wait for the yes. If your app has a tool to ask the user a question with choices (a clickable question, such as `AskUserQuestion` in Cowork or the user-input / choice prompt of the Codex app), use it, with two options: "Yes, start the bridge" and "No"; otherwise ask in one line. The question: "May I start the Jarvis launcher bridge on your PC? A window will open; leave it open while we work, closing it stops the bridge." You may ask in the same question to confirm the working folder. The bridge runs programs on the user's PC: it needs their explicit agreement, given in this conversation (once per session is enough). The approval the app may also show when you call `bridge_start` is in addition to this question, not instead of it.
2. Write a random token in the file `.jarvis/bridge-request` of the working folder (create `.jarvis/` if missing), for example with `node -e "console.log(require('crypto').randomBytes(12).toString('hex'))" > .jarvis/bridge-request`. The PC sees the folder at another path: the token is how it finds it.
3. Call `bridge_start` with `marker` = that token.
4. Run `node .jarvis/bridge/jl.mjs status` and continue when it answers.

Keep the token for `bridge_status` and, only if the user asks, `bridge_stop`.
- **No folder holds the token.** The folder is outside the folders the PC searches. Ask the user to run `npx jarvis-launcher mcp install` in a terminal and give the parent folder of the working folder as the base folder (the command asks; it also says when to close and reopen the app), or to start the bridge by hand as below.
- **The call is denied by a permission or safety check.** Do not retry and do not look for another way to start programs on the PC. Tell the user that the check stopped it, and that they can allow the `jarvis-launcher` tools in the app's settings (Claude Desktop, Codex) or start the bridge by hand as below.

**Ask the user** when you do not have the tool or it fails. Mention once that `npx jarvis-launcher mcp install` (guided, once per PC) lets you start it yourself next time. The steps (translated, with the real folder name):

> To let me work on Jarvis from here, please start the launcher bridge on your PC:
>
> 1. Open the folder **`<folder name>`** in File Explorer.
> 2. Right-click an empty area inside the folder and choose **Open in Terminal**.
> 3. Type `npx jarvis-launcher bridge` and press Enter.
> 4. Leave that window open while we work (Ctrl+C stops the bridge). Tell me when it shows "Leave this window open".

If the user reports a problem:

| The terminal says | What to tell the user |
|---|---|
| `'npx' is not recognized…` / `command not found` | Node.js is missing: install Node.js 20 LTS or later from nodejs.org, open a new terminal, then step 3 again. If they run the launcher from a copy of its source: `node <launcher folder>\dist\cli.js bridge`. |
| `docker` errors, "Docker engine not reachable" | Start Docker Desktop and wait until it is running, then step 3 again. |
| "holds other files" | The folder is not empty: choose another folder (step 1). |
| "cannot be a Jarvis environment folder" | It is a protected folder (the home folder, the disk root, Desktop, Documents…): create a new sub-folder for the demo, connect it, and start the bridge there (step 1). |
| "A bridge is already running" | It is already running in another window: nothing to do. |
| No *Open in Terminal* in the menu | Open any terminal, `cd "<full path of the folder>"`, then `npx jarvis-launcher bridge`. On macOS: right-click the folder → Services → New Terminal at Folder. |

When the user says it is running, run `node .jarvis/bridge/jl.mjs status` again: continue only when it answers.

**Using it.** Run `node .jarvis/bridge/jl.mjs <command> [args…]` (after `project init` also `node project/tools/jl.mjs …`). Do not pass the folder, `--json` or `--yes`: the bridge sets them. The output is the usual JSON envelope plus a `bridge` block; the exit code says what to do:

| Exit | Meaning | What to do |
|---|---|---|
| 0 | ok | go on |
| 1 | error | read `error` and `bridge.log` |
| 3 | still running (`start`, `reset`, `run --all`…) | `node .jarvis/bridge/jl.mjs wait <id>` |
| 4 | needs the user | a confirmation (add `--confirm` only after the user said yes to that operation), registry credentials (the user runs `jarvis-launcher registry-login` in another terminal), or `BRIDGE_DOWN` (ask to start the bridge again, as above) |

`delete` is not available through the bridge: the user runs it on the PC.

## 4. Hand over to the workflow guide

1. `status` (`jarvis-launcher status --json`, or `node .jarvis/bridge/jl.mjs status`): `folderKind` `empty` → you will create the environment with `start`; `scenario` → resume it; `not-a-scenario` → stop, tell the user why (`folderProblem`: a protected folder such as the home folder, or other files) and ask for another folder. Never use `--any-folder` on your own.
2. **Jarvis version.** The scenarios, guides and skills are built for the Jarvis version in `minJarvisVersion` (in `status`) or later. If `jarvisVersionSupported` is `false`, tell the user that this environment runs an older Jarvis on which the guides and skills may not work, and ask to update it: `update --tag latest` keeps the data (through the bridge it needs the user's yes, then `--confirm`). The Scenario Library refuses its items on an older Jarvis: if the user does not want to update, suggest a new environment in an empty folder instead. A new environment (`start` in an empty folder) gets the latest version and needs nothing.
3. When you point the user to Jarvis, always give the automatic-login link `urls.devLoginAdmin` from `status` (the plain web address only shows a login page).
4. Read the guide and follow it from its rules on: `library show agent-workflow` (in bridge mode `node .jarvis/bridge/jl.mjs library show agent-workflow`). Before proposing the model, read `library show scenario-design` too.

## Modelling rules you must never break

These hold even before you can read `scenario-design`:
- **People are Jarvis users**, never a "Person" work item type or dataset.
  - Whoever owns an item is its **assignee**.
  - Others involved are collaborators or user fields.
  - Roles and offices are workspace roles or groups.
  - Datasets are only for parties without a login (customers, suppliers).
- **Native fields first.** Every work item has number/id, title, status with its life cycle, assignee, collaborators, start and due date, description. Do not add custom "Code", "Status", "Owner" or "Deadline" fields.
