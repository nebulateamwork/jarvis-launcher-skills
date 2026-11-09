# Jarvis launcher skills

Skills for AI agents that build Jarvis demos and scenarios with `jarvis-launcher`. The agent uses them to connect to the launcher on your PC; everything else (guides, examples, templates) it reads from the launcher's Scenario Library.

> This repository is generated from the Jarvis Scenario Library: do not edit it by hand, changes are overwritten at the next publication.

## Install

You need `jarvis-launcher` on your PC and its MCP server registered in the app: in a terminal run `jarvis-launcher` and choose **Connect an AI app**.

| App | How |
|---|---|
| **Claude Desktop (Cowork)** | Customize → Plugins → **Add marketplace** → `nebulateamwork/jarvis-launcher-skills`; install the plugin **jarvis-launcher**; turn on **Sync automatically** on the marketplace, so new versions arrive by themselves |
| **Claude Code** | `/plugin marketplace add nebulateamwork/jarvis-launcher-skills`, then `/plugin install jarvis-launcher@jarvis-launcher-skills` |
| **Codex** | nothing to do: `jarvis-launcher mcp install --client codex` installs the skill itself |

## Contents

The plugin **jarvis-launcher** is the repository itself (`.claude-plugin/plugin.json`); its skills are in `skills/`:

| Skill | Version | What it does |
|---|---|---|
| `jarvis-launcher-scenario-start` | 1.1.2 | Start a Jarvis scenario |
