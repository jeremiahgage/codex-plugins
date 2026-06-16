# Codex Plugins

This repository is a local Codex plugin marketplace. It currently packages one plugin,
`codex-plugins`, with these skills:

- `$dev`: staged implementation through serial worker agents, followed by verification.
- `$ultrareview`: parallel multi-lane review of a focus area or current changes.

## Layout

- `.agents/plugins/marketplace.json`: marketplace manifest for Codex.
- `plugins/codex-plugins/.codex-plugin/plugin.json`: plugin manifest.
- `plugins/codex-plugins/skills/`: packaged Codex skills.

## Local Install

From any shell, add this repository as a local marketplace and install the plugin:

```bash
codex plugin marketplace add /Users/gage/Developer/Projects/codex-plugins
codex plugin add codex-plugins@codex-plugins
```

Start a new Codex thread after installing so the packaged skills are loaded.
