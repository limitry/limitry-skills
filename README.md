# Limitry agent skill

Teaches an AI agent how to use Limitry: the workspace, every operation, approvals and errors. It is generated from Limitry's API after each release, so it always matches.

## Install

Any agent (Claude Code, Codex, Cursor, GitHub Copilot and others):

```bash
npx skills add limitry/limitry-skills
```

With the GitHub CLI:

```bash
gh skill install limitry/limitry-skills limitry
```

Claude Code, as a plugin with Limitry's MCP server:

```text
/plugin marketplace add limitry/limitry-skills
/plugin install limitry@limitry-skills
```

In Claude or ChatGPT, download [the skill](https://limitry.com/docs/skill/limitry.zip) and add it in the Skills section of the settings. More: [Build with AI agents](https://limitry.com/docs/agents/).

This repository is written by Limitry's release process; changes made here are replaced. Report a problem in an issue.
