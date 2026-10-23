## Prompt per session

`Read https://github.com/svanas/delphiskills/blob/main/SKILL.md and follow it before reviewing, writing, refactoring or compiling Delphi code.`

Works in Claude Code, OpenAI Codex, or any other agent.

The skills support Delphi 11 and newer. One of the sources is about Delphi 5 but got annotated in 2025 with information about the latest version.

## Installation (optional)

Keep this repository in a location outside your Delphi project repositories. Copy **all directories inside `skills/`**, preserving their names and sibling relationships, into the appropriate personal skill directory, or symlink each skill directory there. Do not overwrite an existing same-name skill without reviewing it.

| Agent | Personal directory | Entry file |
| --- | --- | --- |
| Claude Code | `~/.claude/skills/` | `delphi/SKILL.md` |
| OpenAI Codex | `~/.agents/skills/` | `delphi/SKILL.md` |

These locations were checked against [Claude Code's skill documentation](https://code.claude.com/docs/en/skills) and [OpenAI's skill documentation](https://learn.chatgpt.com/docs/build-skills) on 2026-10-23

Use `/delphi` in Claude Code or `$delphi` in Codex where native skill invocation is available. Individual topics can also be invoked by their directory names. No project `AGENTS.md`, `CLAUDE.md`, or checked-in skill files are required. Copying only the entry folder leaves its topic links unresolved; install the complete collection.

## Contents

Start with [SKILL.md](SKILL.md). See [SOURCES.md](SOURCES.md) for the book editions, page conventions, coverage, and corrections.

The material is original engineering guidance synthesized from the four supplied books and selected primary documentation. Book PDFs, extracted text, and book example projects are intentionally absent. Examples included here are newly written and are labeled with their validation status.

## Sources

- *Object Pascal Handbook*, Delphi 11 edition (2021)
- *Expert Delphi*, Marco Cantù and Paweł Głowacki (2024)
- *Mastering Delphi 5*, Marco Cantù, 2025 annotated edition (released February 17, 2025)
- *Code Faster in Delphi*, Alister Christie (2020)
- The official Embarcadero documentation here: https://docwiki.embarcadero.com/RADStudio/
