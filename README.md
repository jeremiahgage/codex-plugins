# Codex Plugins

Public Codex plugin marketplace for small workflow skills.

It currently packages one plugin, `codex-plugins`, with these skills:

- `$ultradev`: staged implementation through serial worker agents, followed by verification.
- `$ultrareview`: parallel multi-lane review of a focus area or current changes.
- `$ultrasimplify`: behavior-preserving simplification through parallel cleanup review agents, approved fixes, and verification.

These skills are experimental and intended for local Codex use.

## Layout

- `.agents/plugins/marketplace.json`: marketplace manifest for Codex.
- `plugins/codex-plugins/.codex-plugin/plugin.json`: plugin manifest.
- `plugins/codex-plugins/skills/`: packaged Codex skills.

## Remote Install

Add this repository as a remote marketplace, then install the plugin:

```bash
codex plugin marketplace add https://github.com/jeremiahgage/codex-plugins.git
codex plugin add codex-plugins@codex-plugins
```

Start a new Codex thread after installing so the packaged skills are loaded.

## Local Install

Clone this repository, then add the repository root as a local marketplace:

```bash
git clone https://github.com/jeremiahgage/codex-plugins.git
cd codex-plugins
codex plugin marketplace add "$(pwd)"
codex plugin add codex-plugins@codex-plugins
```

Start a new Codex thread after installing so the packaged skills are loaded.

## Usage

These skills are meant to be used with subagents, which must be invoked manually. Here are some example prompts:

```text
Use subagents and $ultradev [PLAN].
```

```text
Use subagents and $ultrareview [PLAN].
```

```text
Use subagents and $ultrasimplify [FOCUS].
```

## License

MIT
