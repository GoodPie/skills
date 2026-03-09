# Skills

A collection of [Claude Code skills](https://docs.anthropic.com/en/docs/claude-code/skills) — reusable knowledge packages that provide domain expertise in conversations.

## Available Skills

| Skill | Description |
|-------|-------------|
| [tamagui-expo](./tamagui-expo/) | Expert guide for building Tamagui-styled React Native components in Expo projects. Covers `styled()`, tokens, themes, variants, babel plugin setup, and design system translation. |

## Skill Structure

```
skill-name/
  SKILL.md          # Main skill definition (YAML frontmatter + markdown body)
  evals.json        # Test prompts and expected outputs
  references/       # Supporting reference docs loaded conditionally
```
