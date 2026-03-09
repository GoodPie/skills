# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

This is a collection of **Claude Code skills** — reusable knowledge packages that get loaded into Claude Code conversations to provide domain expertise. Each skill lives in its own directory.

## Skill Structure

Each skill directory follows this structure:

```
skill-name/
  SKILL.md          # Main skill definition (YAML frontmatter + markdown body)
  evals.json        # Test prompts and expected outputs for validating the skill
  references/       # Supporting reference docs the skill can selectively read
    topic-a.md
    topic-b.md
```

### SKILL.md Format

- **Frontmatter** (`---` delimited YAML): `name`, `description` (used for trigger matching)
- **Body**: The full skill prompt — instructions, patterns, code examples, and decision trees
- Skills reference files in `references/` with conditional read instructions (e.g., "if setting up from scratch, read `references/setup-and-config.md`")

### evals.json Format

Array of eval scenarios with `id`, `prompt`, `expected_output`, and `files` (context files to include).

## Current Skills

- **tamagui-expo** — Tamagui v2+ styling for Expo/React Native (native-only). Covers `styled()`, tokens, themes, variants, babel plugin setup, and design system translation.

## Authoring Guidelines

- Skills should be self-contained experts — include enough context that Claude can act without external docs
- Use `references/` for detailed content that only needs loading conditionally, keeping SKILL.md focused
- The `description` field in frontmatter drives when the skill triggers — make trigger conditions explicit
- Include a `true` token pattern note for any design system skill (Tamagui compiler requirement)
- Evals should cover setup, component creation, and integration scenarios
