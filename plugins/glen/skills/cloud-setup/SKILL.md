---
name: cloud-setup
description: Set up glen to record Cursor or Grok cloud agents (Cursor's Cloud Agents, Grok's dashboard Cloud Agents) that work on this repo. Use when the user asks to "set up glen for cloud agents", "record my Cursor cloud agents", "make glen work in Grok's cloud", or says cloud agent sessions don't show up in glen.
---

Cloud agents run only the hooks committed to the repo, never plugin hooks, so
glen's hooks have to live in the repo itself.

1. From the repo's root, run `glen install --agent cursor --cloud` via Bash.
   It writes `.cursor/hooks.json` (keeping any hooks already there) and
   `.cursor/glen-hook.sh`, and prints the remaining steps.
2. Show the user the diff. Commit and push the two files only if they say so.
3. Tell the user the one step only they can do: open Connect an agent in the
   glen dashboard, pick "Cursor & Grok cloud agents", and add the
   `GLEN_ORG_ID`, `GLEN_BASE_URL` and `GLEN_API_KEY` it shows as Secrets on
   the repo's environment at grok.com/dashboard/cloud-agents.
   Never ask them to paste the key into this chat.
4. Tell them what to expect: hooks apply to cloud agents started after the
   push; the first hook run installs glen on the VM, so it takes about a
   second; and a session's first turn starts before the hooks run, so glen
   records from the second turn on.

Teammates who open the repo in Cursor locally are unaffected: without
`GLEN_API_KEY` the hook script exits without doing anything.
