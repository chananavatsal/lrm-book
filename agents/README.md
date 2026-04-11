# Agents

Tooling for AI assistants (Claude Code, Cursor, etc.) to help with writing the book. These are **optional** - the book can be written without them. They exist to automate the tedious parts of Manning style compliance so humans focus on the creative work.

## What's here

### `book/` - The `/book` skill for Claude Code

A Claude Code skill that provides commands for Manning style linting, structure checking, and content review. Use it to catch Manning style violations before they reach Erik (our development editor).

**Primary commands:**
- `/book lint <file>` - Find em dashes, marketing words, meta-language, passive voice, long sentences
- `/book structure <file>` - Verify required Manning Ch 1 sections (opening bullets, mental model, summary)
- `/book code-check <file>` - Validate code listings (line length, tabs, annotation format)
- `/book caption-check <file>` - Verify figure captions describe action, not just labels
- `/book review <file>` - 3-perspective review (Erik, tech reviewer, MQR target reader)
- `/book voice-check <file>` - Check you/we consistency and tone
- `/book mqr-check <file>` - Verify terms are defined for the MQR
- `/book stylebook` - Print locked style decisions

**How to use it (Claude Code users):**

```bash
# One-time setup: symlink or copy into your Claude skills folder
mkdir -p ~/.claude/skills
ln -s "$(pwd)/agents/book" ~/.claude/skills/book

# Then in any Claude Code session:
/book lint chapter_1/draft.md
```

**If you're not using Claude Code:** the SKILL.md file is still useful as a plain reference document. It codifies Manning's style rules, the book's terminology decisions, and the content review criteria. Treat it as documentation.

## Relationship to existing files

These agents complement (not replace) files already in the repo:

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Book context and audience (for AI assistants) |
| `writing_instructions/writing_instructions.md` | Manning formatting rules (fonts, styles, doc setup) |
| `agents/book/SKILL.md` | Content linting rules (em dashes, marketing words, meta-language) |

`writing_instructions.md` covers **how to format** the Google Doc. `agents/book/SKILL.md` covers **what to write** inside it. Together they cover the full Manning compliance surface.

## Contributing

If you add or modify an agent:
1. Keep it portable (no hardcoded author paths)
2. Document it here in the README
3. Note whether it's Claude Code specific or tool-agnostic
4. Commit alongside a changelog entry
