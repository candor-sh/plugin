# Candor

Run AI-moderated user studies and human evaluations from your coding agent: create studies, recruit real participants, and get prioritized findings back.

## Install

**Devin**

```
devin plugins install candor-sh/plugin -y
```

**Claude Code**

```
claude plugin marketplace add candor-sh/plugin
claude plugin install candor@candor-sh
```

**Codex / ChatGPT desktop**

```
codex plugin marketplace add candor-sh/plugin
codex plugin add candor@candor-sh
```

**Antigravity**

```
agy plugin install https://github.com/candor-sh/plugin
```

Or let the Candor installer do it: `curl -fsSL https://candor.sh | bash`

## Setup

The plugin uses the Candor CLI, which installs automatically on first use (or run `npm install -g @candor.sh/cli`). Authenticate with `candor init` (browser login), or provide an API key from https://candor.sh/dashboard/api-keys as the `CANDOR_API_KEY` environment variable.

---

This repository is generated from Candor's main repo; open issues at https://candor.sh/contact.
