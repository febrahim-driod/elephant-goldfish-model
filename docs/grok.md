<!--
Copyright 2026 Google LLC.
SPDX-License-Identifier: Apache-2.0
-->

# Grok integration

Grok Build loads the same `skills/`, `agents/`, and `resources/` tree as the
other harnesses. Named reviewer spawn with this repository's snake_case
persona filenames is not verified. See [Named reviewers](#named-reviewers).

The skills route Codex and Claude sessions to their guides, but not Grok
sessions. A Grok session doesn't read this file unless you point it here.

## Install

Start a new session after installation. Grok trusts a plugin under
`~/.grok/plugins/` automatically. Other install paths need `--trust`.

```sh
grok plugin marketplace add google/elephant-goldfish-model
grok plugin install egm --trust
```

A local checkout installs the same way:

```sh
grok plugin install /absolute/path/to/elephant-goldfish-model --trust
```

Grok copies the plugin into `~/.grok/installed-plugins/` at install time.
Reinstall after you change the checkout. `grok inspect` lists each loaded
agent with its source path; confirm the path points at the current copy.
`grok plugin validate` checks the manifest. It does not prove that
`spawn_subagent` can open a reviewer.

## Manifest

Grok 1.0.77 reads the root `plugin.json` first. When that file exists, Grok
ignores `.grok-plugin/plugin.json`. An `agents` list in the manifest it reads
replaces the default `agents/` scan; a bare directory string is not
accepted. Do not add an `agents` list to the root `plugin.json` for Grok:
other hosts share that manifest and scan `agents/` by default.
`.grok-plugin/marketplace.json` is only the marketplace index.

## Named reviewers

The subagent tool is `spawn_subagent`. Plugin agents appear in its
`subagent_type` enum as `egm:<frontmatter-name>`, for example
`egm:egm-design`. An unknown type fails the call with the list of valid
types.

On Grok 1.0.70, `egm:egm-design` resolved only when the file was named
`agents/egm-design.md`. This repository names it `agents/design_critic.md`.
Whether Grok 1.0.77 resolves the type by frontmatter `name` instead is not
verified. If a named spawn fails as an unknown type, spawn a general-purpose
child and tell it to read the persona file first.

Do not pass `general-purpose`, `explore`, or `plan` as `subagent_type`. Those
built-ins are real, and the model is not allowed to select them by name. Omit
`subagent_type` when a general-purpose child is required.

Grok does not reliably nest subagents. Direct-dispatch the reviewers. Do not
depend on `egm-synthesizer` to spawn them. If this session has no subagent
tool, stop. A review in the parent context is not a Goldfish review.

Pass absolute paths to the persona, the standard, and the target. Grok expands
`${CLAUDE_PLUGIN_ROOT}` in a plugin agent body. The canonical personas do not
use that variable.

## Reviewer isolation

Grok restricts a plugin agent through its frontmatter: `capabilityMode`,
`permissionMode`, `agentsMd`, `discoverSkills`, and `inheritSkills`. The
canonical personas in `agents/` set none of these, so Grok does not enforce a
read-only reviewer. A prompt that asks the reviewer to stay read-only is not
the control. If a spawned reviewer can create or edit a file, stop the review
and report that.

## Artifact directory

If the user's prompt or an instructions file has this line, write briefs,
drafts, and reports there:

```text
EGM artifact directory: .dev/docs
```

A relative path is relative to the working directory. Create the folder if
it does not exist. Design docs and other project files do not move.
