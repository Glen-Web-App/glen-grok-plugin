# glen: Grok plugin

Shared team memory for coding agents. Glen automatically captures what you build so your
whole team's agents share the same institutional knowledge. Recall is on demand: run
`glen search "<topic>"` to pull relevant team memory into the session.

This plugin is for xAI's Grok Build CLI (`grok`).

## What it does

When you use Grok, the glen plugin fires these hooks:

- **SessionStart**: starts glen's background self-check (`glen doctor --auto`) and
  remembers the session for this folder. Grok ignores this hook's output, so nothing
  is injected here.
- **UserPromptSubmit**: captures your prompt (plus workspace context: repo,
  branch, agent name) to your glen org before each turn.
- **Stop**: captures the completed turn (user prompt + assistant response) to your
  glen org after each turn. The extra Stop Grok fires when a session ends is ignored.
- **PostToolUse** (shell commands): after a `git commit`, links the commit to the
  session in glen.

Grok ignores the output of its SessionStart and UserPromptSubmit hooks, so glen
cannot inject team memory automatically. Recall is on demand: run
`glen search "<topic>"` in the shell and treat the output as team memory. A plugin
can't ship Grok rules, so `glen install --agent grok` writes `~/.grok/rules/glen.md`
(under `$GROK_HOME` if you set it), which nudges the agent to search whenever you
reference past team work. `glen doctor --auto` keeps that file current and
`glen uninstall` removes it. Capture (recording turns) is automatic on every turn.

`glen off` records nothing. In incognito (`glen incognito`) nothing is recorded to
team memory. Glen reads only the hook input and git metadata for the current repo.

### Grok also runs your Claude Code plugin's hooks

Grok runs the hooks of installed Claude Code plugins and of `~/.claude/settings.json`,
so if you also use glen's Claude Code plugin, Grok runs glen's hooks twice for every
event. Glen records each turn once, labelled `grok`: whichever hook line runs it, a
hook started by Grok is a Grok turn, and both copies send the same turn key, so the
second is dropped. If you only have the Claude Code plugin, your Grok turns are
still recorded, as `grok`.

## Skills

The plugin ships skills the agent invokes on demand, including:

- **search**: search the team's shared glen memory for a specific fact,
  decision, or past discussion.
- **controls**: turn glen on/off, go off the record (incognito), go silent,
  toggle skill suggestions, or switch which organization's memory is active,
  only when you explicitly ask.
- **setup**: set up, fix, or update glen on this machine. If glen is ever
  broken (not connected, no org selected, hooks missing), just ask the agent to
  "set up glen" and it repairs whatever `glen doctor` reports.

## Install

```sh
npm install -g @tryglen/cli
glen install --agent grok
```

`glen install` signs you in, then runs:

```sh
grok plugin marketplace add Glen-Web-App/glen-grok-plugin
grok plugin install glen --trust
```

`--trust` is what lets Grok run the plugin's hooks and skills; Grok asks for it
because hooks run commands on your machine. Start a new Grok session afterwards:
Grok loads plugins and rules when a session starts.

After login your active organization is saved locally. Switch orgs at any time with
`glen org switch`.

## What data is sent

On every `UserPromptSubmit` hook, glen sends to your glen org:

- The current user prompt
- Workspace metadata: repo name, branch, commit hash, remote URL
- Agent name (`grok`) and the Grok session id

On every `Stop` hook, glen captures the completed turn (prompt + response).

**Nothing is recorded while incognito is on.** `glen search` still works, but nothing
is written to team memory. Admin analytics still count your prompts as numbers only.
Toggle with `glen incognito` / `glen on`. `glen off` records nothing.

The glen CLI also reports content-free error codes about itself (error code, command and flag names, CLI, OS and agent
versions; never prompts, paths, flag values or content), incognito included. Turn this off with
`glen telemetry disable` or `DO_NOT_TRACK=1`.

## Updating

Grok updates installed plugins when a session starts, unless your organization turned
that off. `grok plugin update glen` updates it by hand.

The glen CLI itself checks for updates hourly in the background and upgrades
automatically when installed via npm global. To update everything manually:

```sh
glen update
```

`glen update` updates the CLI **and** any installed glen plugins in one go.

## Troubleshooting

**Check session status (org, mode):**

```sh
glen status
```

**Full diagnostics:**

```sh
glen doctor
```

**Broken setup?** Ask the agent to "set up glen"; the bundled setup skill
runs `glen doctor` and fixes whatever it reports.

**Turns aren't recorded:** check that the plugin is installed and trusted with
`grok plugin list`. A plugin installed without `--trust` loads no hooks; reinstall
with `grok plugin install glen --trust`, then start a new session.

**Stale update lock** (if `glen update` hangs): remove the lock file and retry:

```sh
rm -rf ~/.glen/update.lock
glen update
```

**No active organization error:** run `glen org switch` to select an org, or
`glen org list` to see your memberships. If the list itself fails with an auth error,
run `glen login` to reconnect.
