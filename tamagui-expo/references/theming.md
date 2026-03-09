# Theming in Tamagui (Native)

## Table of Contents
1. [Tokens vs Themes](#tokens-vs-themes)
2. [The 12-Step Color Scale](#12-step-color-scale)
3. [Creating Tokens](#creating-tokens)
4. [Creating Themes](#creating-themes)
5. [Sub-Themes](#sub-themes)
6. [Dark Mode](#dark-mode)
7. [Theme Provider Setup](#theme-provider)
8. [useTheme Hook](#use-theme)
9. [Dynamic Theming Patterns](#dynamic-theming)

---

## Tokens vs Themes

This distinction is fundamental:

**Tokens** = global constants. Same everywhere. Never change at runtime.
- Spacing: `$space.4` = 16px always
- Static colors: `$color.brand` = '#2E6B4F' always
- Sizes: `$size.md` = 44 always
- Radii: `$radius.3` = 12 always

**Themes** = contextual variables. Change based on `<Theme>` wrapper.
- `$background` = '#F8F5F0' in light, '#000000' in dark
- `$color` = '#1A1815' in light, '#E8E4DE' in dark
- `$brand` = '#2E6B4F' in light, '#4EA87A' in dark

**Rule of thumb:** If a value changes between light/dark mode (or any other theme), it's a theme value. If it's the same everywhere, it's a token.

---

## The 12-Step Color Scale

Tamagui's default themes (and the Radix convention they're based on) organize each color into a 12-step scale with well-defined semantic roles. When you see theme values like `$color1` through `$color12`, this is what they mean:

| Steps | Role | Example usage |
|-------|------|---------------|
| 1–2 | **Backgrounds** | App background, subtle surface fills |
| 3–4 | **Interactive surfaces** | Card backgrounds, hover/press fills |
| 5–6 | **Borders & separators** | Input borders, dividers, subtle outlines |
| 7–8 | **Interaction states** | Hover borders, active outlines, focus rings |
| 9–10 | **Solid fills** | Primary button backgrounds, badges, indicators |
| 11–12 | **Text** | Secondary text (11), primary/high-contrast text (12) |

This convention is useful because it gives every color a consistent visual hierarchy. A `$blue5` is always a border-weight blue, a `$blue9` is always a solid fill blue, regardless of light/dark mode — the actual hex values change between themes, but the *role* stays the same.

**When to use this:** If you're working with Tamagui's built-in color sub-themes (blue, red, green, etc. from `defaultConfig`), reference the step numbers to pick the right shade for the job. If you're defining a fully custom palette, you don't have to follow the 12-step convention, but it's a solid mental model for organizing color roles — especially if you want your custom themes to interop cleanly with Tamagui's built-in components.

---

## Creating Tokens

```ts
import { createTokens } from 'tamagui'

export const tokens = createTokens({
  color: {
    // Static colors that don't change with theme
    white: '#FFFFFF',
    black: '#000000',
    trueBlack: '#000000',

    // Brand colors (static — theme variants go in themes)
    brandGreen: '#2E6B4F',
    brandGreenLight: '#2D7A5A',
    brandSubtle: '#E8F3ED',

    // Semantic (static base values)
    successBase: '#2E7D4F',
    warningBase: '#9A6A15',
    errorBase: '#C43D3D',
    infoBase: '#2E6B8A',

    // Special purpose
    rare: '#8B5CF6',
    endemic: '#D97706',
    mapPin: '#E04E3A',
  },

  space: {
    0: 0,
    1: 4,
    2: 8,
    3: 12,
    4: 16,
    5: 20,
    6: 24,
    8: 32,
    10: 40,
    12: 48,
    16: 64,
    true: 16,   // default
    '-1': -4,
    '-2': -8,
    '-3': -12,
    '-4': -16,
  },

  size: {
    0: 0,
    sm: 36,
    md: 44,
    lg: 52,
    true: 44,
    // Icon sizes
    iconSm: 14,
    iconMd: 20,
    iconLg: 24,
    // Touch targets
    touchMin: 44,
    touchPreferred: 48,
    // Component heights
    inputHeight: 48,
    chipHeight: 28,
    listItemMin: 56,
    avatar: 40,
    fab: 56,
  },

  radius: {
    0: 0,
    1: 4,
    2: 8,
    3: 12,
    4: 14,   // pill for chips
    5: 16,   // bottom sheet
    round: 9999,
    true: 8,
  },

  zIndex: {
    0: 0,
    map: 0,
    nav: 10,
    mapControls: 15,
    fab: 20,
    sheet: 30,
    connectivity: 40,
    system: 50,
  },
})
```

---

## Creating Themes

Themes define contextual values that change with `<Theme>` wrappers:

```ts
export const themes = {
  light: {
    // Backgrounds
    background: '#F8F5F0',
    surface: '#FFFFFF',
    surfaceRaised: '#F0EBE4',

    // Text
    color: '#1A1815',            // primary
    colorSecondary: '#3D3832',   // secondary
    colorTertiary: '#6B6358',    // placeholders, tertiary
    colorMuted: '#857D74',       // borders meeting contrast

    // Brand (can shift between themes)
    brand: '#2E6B4F',
    brandLight: '#2D7A5A',
    brandSubtle: '#E8F3ED',

    // Borders
    border: '#E0DAD2',
    borderSubtle: '#F0EBE4',

    // Semantic
    success: '#2E7D4F',
    warning: '#9A6A15',
    error: '#C43D3D',
    info: '#2E6B8A',

    // Shadows (native)
    shadowColor: 'rgba(0,0,0,0.08)',
    shadowColorStrong: 'rgba(0,0,0,0.15)',
  },

  dark: {
    background: '#000000',        // true black for OLED
    surface: '#1A1815',           // warm dark surface
    surfaceRaised: '#262220',

    color: '#E8E4DE',
    colorSecondary: '#A69E93',
    colorTertiary: '#6B6358',
    colorMuted: '#3D3832',

    brand: '#4EA87A',             // lighter green for dark bg
    brandLight: '#5FBD8F',
    brandSubtle: '#1A2E23',

    border: '#3D3832',
    borderSubtle: '#262220',

    // Desaturated semantics for dark mode (10-15% less vivid)
    success: '#3A8C5E',
    warning: '#B8842A',
    error: '#D45A5A',
    info: '#4A8BA8',

    shadowColor: 'rgba(0,0,0,0.3)',
    shadowColorStrong: 'rgba(0,0,0,0.5)',
  },
}
```

---

## Sub-Themes

Sub-themes customize specific components without changing the global theme. Name them `{parentTheme}_{ComponentName}`:

```ts
export const themes = {
  light: { /* ... */ },
  dark: { /* ... */ },

  // Button sub-themes
  light_Button: {
    background: '#2E6B4F',      // brand fill
    color: '#FFFFFF',           // white text on brand
    backgroundPress: '#245A41', // darker on press
  },
  dark_Button: {
    background: '#4EA87A',
    color: '#1A1815',
    backgroundPress: '#3D8A65',
  },

  // Secondary button
  light_ButtonSecondary: {
    background: 'transparent',
    color: '#2E6B4F',
    borderColor: '#2E6B4F',
  },
  dark_ButtonSecondary: {
    background: 'transparent',
    color: '#4EA87A',
    borderColor: '#4EA87A',
  },

  // Destructive button
  light_ButtonDestructive: {
    background: '#C43D3D',
    color: '#FFFFFF',
  },
  dark_ButtonDestructive: {
    background: '#D45A5A',
    color: '#1A1815',
  },

  // Card sub-theme
  light_Card: {
    background: '#FFFFFF',
    borderColor: '#E0DAD2',
  },
  dark_Card: {
    background: '#1A1815',
    borderColor: '#3D3832',
  },
}
```

The component opts into sub-theme lookup via `name`:

```tsx
const Button = styled(View, {
  name: 'Button',            // ← matches light_Button / dark_Button
  backgroundColor: '$background',
  // ...
})
```

When rendered inside `<Theme name="dark">`, Tamagui automatically looks up `dark_Button` for the `$background` value.

---

## Dark Mode

### Provider setup

```tsx
import { useColorScheme } from 'react-native'
import { TamaguiProvider, Theme } from 'tamagui'
import config from './tamagui.config'

export default function RootLayout() {
  const colorScheme = useColorScheme()

  return (
    <TamaguiProvider config={config}>
      <Theme name={colorScheme === 'dark' ? 'dark' : 'light'}>
        {/* App content */}
      </Theme>
    </TamaguiProvider>
  )
}
```

### Fast scheme change (iOS)

For v5, enable `fastSchemeChange` in settings for faster dark/light transitions on iOS:

```ts
settings: {
  fastSchemeChange: true,
}
```

This uses `DynamicColorIOS` under the hood, avoiding full re-renders on scheme change. Only works if `defaultTheme` matches the system scheme.

### Design rules for dark mode

Based on common design system patterns for OLED-first dark modes:

1. **True black (`#000000`) for background only.** Cards and content use warm dark surface colors. This prevents the "floating text in void" feeling while keeping OLED benefits.

2. **Borders replace shadows in dark mode.** Use 1px borders instead of elevation shadows — shadows are invisible against dark backgrounds.

3. **Desaturate semantic colors 10-15%.** Vivid greens and reds cause afterimage effects on dark backgrounds in low light conditions.

4. **Status indicators (rare, endemic) stay vivid.** They need to pop even in low light.

---

## useTheme Hook

Access the current theme values in component logic:

```tsx
import { useTheme } from 'tamagui'

function SightingCard() {
  const theme = useTheme()

  // theme.brand.val gives the resolved hex value
  const pinColor = theme.brand.val

  return (
    <MapPin color={pinColor} />
  )
}
```

`theme.{tokenName}` returns a `Variable` object. Use `.val` to get the actual value, or `.get()` for a reactive version that updates with theme changes.

---

## Dynamic Theming Patterns

### Nested themes

Themes can nest — each `<Theme>` wrapper overrides its parent:

```tsx
<Theme name="light">
  {/* Everything here uses light theme */}
  <Card>
    <Theme name="dark">
      {/* This card's content uses dark theme */}
      <Text color="$color">Dark text</Text>
    </Theme>
  </Card>
</Theme>
```

### Conditional theme based on data

```tsx
function SpeciesBadge({ rarity }: { rarity: 'common' | 'rare' | 'endemic' }) {
  // Use different sub-themes based on rarity
  const themeName = rarity === 'rare' ? 'purple' : rarity === 'endemic' ? 'orange' : undefined

  return (
    <Theme name={themeName}>
      <Badge>
        <BadgeText>{rarity}</BadgeText>
      </Badge>
    </Theme>
  )
}
```

### Theme-aware conditional logic

```tsx
import { useThemeName } from 'tamagui'

function Card() {
  const themeName = useThemeName()
  const isDark = themeName === 'dark'

  return (
    <View
      // In dark mode, use border instead of shadow
      borderWidth={isDark ? 1 : 0}
      borderColor="$border"
      elevation={isDark ? 0 : 2}
    />
  )
}
```
