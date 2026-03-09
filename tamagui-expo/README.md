# tamagui-expo

Expert guide for building Tamagui-styled React Native components in Expo projects.

## Install

```bash
npx skills add goodpie/skills/tamagui-expo
```

## When It Triggers

This skill activates when working with Tamagui in an Expo or React Native context:

- Creating styled components, variants, or design system primitives
- Configuring tokens, themes, or the optimizing compiler
- Translating a design spec into Tamagui code
- Dark/light theme switching, animations on native, performance tuning

Keywords: `tamagui`, `styled component`, `design tokens`, `theme`, `createTamagui`, `variants`

## What It Covers

- **Core mental model** — tokens (global constants) vs themes (contextual variables), `styled()`, and the optimizing compiler
- **Setup & config** — `createTamagui()`, babel plugin, TypeScript `declare module`, animation driver selection (RN Animated vs Reanimated)
- **Styled components** — `styled()`, variants, spread variants, `.styleable()`, `accept`, pseudo styles (`pressStyle`, `focusStyle`), platform-specific styles
- **Theming** — `createTokens()`, light/dark themes, sub-themes (`light_Button`, `dark_Button`), `useTheme()`, nested themes, `fastSchemeChange`
- **Styling rules** — token references over raw values, variants over ternaries, compound component patterns, composition via extension
- **Performance** — babel plugin for production, the `true` token requirement, when spread variants cost runtime, compiler output verification
- **Design spec translation** — step-by-step process to map a design system doc into tokens, themes, fonts, and `styled()` components

## Targets

- Tamagui v2+ with `@tamagui/config/v5` preset
- Expo SDK 54+ managed workflow
- iOS and Android (native-only, no web)

## File Structure

```
SKILL.md                          # Main skill prompt
evals.json                        # 3 eval scenarios (setup, Button, SightingCard)
references/
  setup-and-config.md             # Installation, config, babel plugin, TypeScript, v5 specifics
  styled-components.md            # styled(), variants, .styleable(), accept, pseudo styles, composition
  theming.md                      # Tokens vs themes, createTokens, sub-themes, dark mode, useTheme
```

Reference files are loaded conditionally — the skill only reads what's relevant to the current task.
