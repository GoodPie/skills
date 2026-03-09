---
name: tamagui-expo
description: >-
  Expert guide for building Tamagui-styled React Native components in Expo projects.
  Use this skill whenever the user is working with Tamagui in an Expo or React Native context —
  creating styled components, configuring tokens/themes, setting up the optimizing compiler,
  building design system primitives, or translating a design spec into Tamagui code.
  Trigger on: "tamagui", "styled component", "design tokens", "theme", "createTamagui",
  "variants", even if the user just says "style this component" or "make a button" in a
  project that uses Tamagui. Also trigger when the user asks about Tamagui performance,
  animations on native, or dark/light theme switching in React Native.
---

# Tamagui for Expo / React Native

You are an expert in Tamagui v2+ with the `@tamagui/config/v5` preset, targeting **Expo SDK 54+ managed workflow** on iOS and Android. No web output — all guidance is native-only.

## When to read reference files

- **Setting up Tamagui from scratch or modifying config** → read `references/setup-and-config.md`
- **Creating styled components, variants, or design system primitives** → read `references/styled-components.md`
- **Working with themes, tokens, or dark mode** → read `references/theming.md`

If the project has a `docs/DESIGN_SYSTEM.md`, read it before writing any component code — it contains the source of truth for colors, spacing, typography, and component specs that your Tamagui tokens and themes must match.

---

## Core Mental Model

Tamagui sits between you and React Native's `View`/`Text`. It gives you:

1. **Tokens** — named constants (like CSS variables) for spacing, color, size, radius. They never change at runtime. Reference them with `$` prefix: `margin="$4"`.
2. **Themes** — like tokens, but they *do* change (light/dark, sub-themes per component). Reference with `$` prefix on color-related props: `color="$color"`, `backgroundColor="$background"`.
3. **`styled()`** — wraps any RN component to accept Tamagui props, variants, and compiler optimization.
4. **Optimizing compiler** — at build time, flattens component trees and hoists static styles. On native this happens via `@tamagui/babel-plugin`. It's optional but worth enabling for production.

The key insight: tokens are *global constants*, themes are *contextual variables*. A `$space.4` token is always 16px everywhere. A `$background` theme value changes depending on which `<Theme>` ancestor wraps the component.

---

## Project Structure Convention

For an Expo project, organize Tamagui artifacts like this:

```
tamagui.config.ts          # createTamagui() — the single source of truth
theme/
  tokens.ts                # createTokens() — spacing, size, radius, color, zIndex
  themes.ts                # light, dark, and sub-themes
  fonts.ts                 # createFont() for Inter + JetBrains Mono
  animations.ts            # createAnimations() with chosen driver
components/
  ui/                      # Design system primitives (Button, Card, Input, etc.)
    button.tsx
    card.tsx
    ...
```

Keep `tamagui.config.ts` lean — it imports from the `theme/` directory. This makes tokens and themes independently testable and readable.

---

## Choosing an Animation Driver (Native)

Since this is native-only, pick one:

| Driver | Package | Best for |
|--------|---------|----------|
| React Native Animated | `@tamagui/config/v5-rn` | Simple, zero extra deps |
| Reanimated | `@tamagui/config/v5-reanimated` | Best native perf, spring physics |

If the project already has `react-native-reanimated` installed (check `package.json`), use the reanimated driver. Otherwise default to `v5-rn` to avoid adding a native dependency.

Import the default config from the chosen driver entry point:

```ts
import { defaultConfig } from '@tamagui/config/v5-reanimated'
// or
import { defaultConfig } from '@tamagui/config/v5-rn'
```

---

## Styling Rules

1. **Use `$` token references, not raw values.** Write `padding="$4"` not `padding={16}`. This keeps everything tied to the design system and lets the compiler optimize.

2. **Prefer `styled()` over inline style objects.** Inline `style={{}}` props bypass the compiler entirely. If you're repeating a style pattern, extract it into a `styled()` component.

3. **Variants over ternaries.** Instead of `backgroundColor={isActive ? '$brand' : '$surface'}`, define a variant:
   ```tsx
   const Item = styled(View, {
     backgroundColor: '$surface',
     variants: {
       active: {
         true: { backgroundColor: '$brandSubtle' },
       },
     } as const,
   })
   ```
   Variants are compiler-friendly and self-documenting.

4. **Use `.styleable()` when wrapping styled components.** If you create a functional component that renders a Tamagui `styled()` component inside, call `.styleable()` on the inner component so that variants and theme merging work correctly when someone later wraps *your* component with `styled()`.

5. **`accept` for non-style props that need token resolution.** If a component has a prop like `iconColor` that should resolve `$brand` to an actual hex, use:
   ```tsx
   const MyIcon = styled(SomeIcon, {
     accept: { iconColor: 'color' },
   })
   ```

6. **Pseudo styles for interaction states.** Use `pressStyle`, `hoverStyle`, `focusStyle` instead of managing state yourself:
   ```tsx
   const Button = styled(View, {
     backgroundColor: '$brand',
     pressStyle: { backgroundColor: '$brandDark', scale: 0.97 },
   })
   ```

7. **Parent-based styling with `$platform-` prefix.** For iOS vs Android differences:
   ```tsx
   const Card = styled(View, {
     elevation: 2,
     '$platform-ios': {
       shadowColor: '#000',
       shadowOffset: { width: 0, height: 1 },
       shadowOpacity: 0.08,
       shadowRadius: 3,
     },
   })
   ```

---

## Theme Sub-Themes Pattern

Sub-themes let you customize any component's theme tokens without global changes. Name them `{parentTheme}_{ComponentName}`:

```ts
themes: {
  light: { background: '#F8F5F0', color: '#1A1815', brand: '#2E6B4F' },
  dark:  { background: '#000000', color: '#E8E4DE', brand: '#4EA87A' },

  // Sub-themes for Button in each mode
  light_Button: { background: '#2E6B4F', color: '#FFFFFF' },
  dark_Button:  { background: '#4EA87A', color: '#1A1815' },
}
```

Then `<Theme name="dark"><Button />` automatically picks up `dark_Button` theme values on the `<Button>` if it uses `theme: 'Button'` in its styled definition.

---

## Performance on Native

- **Enable the babel plugin for production builds.** Add `@tamagui/babel-plugin` to `babel.config.js` with `disableExtraction: process.env.NODE_ENV === 'development'`. This gives you fast dev iteration while optimizing production.
- **The `true` token matters.** Always define a `true` key in your `size` and `space` token objects — it's the default the compiler uses when no size is specified.
- **Avoid `disableOptimization` unless debugging.** Adding this prop to a component prevents the compiler from flattening it.
- **String render props optimize best.** `styled(View, { render: 'div' })` is a web pattern — on native, just use the default component rendering.

---

## Common Patterns

### Responsive to device — media queries on native
Tamagui media queries work on native too (evaluated at runtime via `Dimensions`):

```ts
media: {
  short: { maxHeight: 700 },
  tall: { minHeight: 900 },
}
```

```tsx
<View padding="$4" $tall={{ padding: '$6' }} />
```

### Group container queries
Use `group` for parent-size-aware children. On native, this requires the parent to have explicit `width`/`height` or uses `onLayout`:

```tsx
<View group="card" width={300}>
  <Text $group-card-sm={{ fontSize: '$caption' }}>
    Adapts to card width
  </Text>
</View>
```

### Dark mode switching
Wrap your app in `<Theme name={colorScheme}>` where `colorScheme` comes from `useColorScheme()`. Tamagui handles the rest — all `$background`, `$color`, `$brand` references resolve to the active theme.

```tsx
import { useColorScheme } from 'react-native'
import { TamaguiProvider, Theme } from 'tamagui'

export default function App() {
  const scheme = useColorScheme()
  return (
    <TamaguiProvider config={config}>
      <Theme name={scheme ?? 'light'}>
        {/* app content */}
      </Theme>
    </TamaguiProvider>
  )
}
```

---

## Translating a Design Spec into Tamagui

When the project has a design system document, follow this process:

1. **Map the color palette to tokens and themes.** Static colors (that don't change between light/dark) go in `tokens.color`. Colors that flip between modes go in `themes`.
2. **Map spacing to `tokens.space`.** Use the spec's spacing scale names as token keys.
3. **Map typography to `createFont()`.** Define each weight and size from the type scale.
4. **Map border radii to `tokens.radius`.**
5. **Map component specs to `styled()` components with variants.** Button sizes → size variant. Button types (primary/secondary/destructive) → variant variant.
6. **Map motion specs to animation configs.** Durations and easings become named animations.
7. **Map dark mode palette to theme overrides.**

The goal is: every magic number in the design spec becomes a named token, and every visual state becomes a variant. Raw hex codes and pixel values should not appear in component files.
