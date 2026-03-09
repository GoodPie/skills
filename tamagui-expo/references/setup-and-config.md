# Tamagui Setup & Configuration for Expo

## Table of Contents
1. [Installation](#installation)
2. [Config File Structure](#config-file-structure)
3. [Babel Plugin (Compiler)](#babel-plugin)
4. [Metro Plugin](#metro-plugin)
5. [TypeScript Setup](#typescript-setup)
6. [v5 Config Specifics](#v5-config)

---

## Installation

```bash
# Core packages
npm install tamagui @tamagui/config

# Animation driver (pick one)
npm install @tamagui/animations-react-native
# OR if reanimated is already in the project:
npm install @tamagui/animations-reanimated

# Babel plugin (optional but recommended for production)
npm install -D @tamagui/babel-plugin
```

Note: `tamagui` re-exports everything from `@tamagui/core` plus the full UI kit. If you only want the style engine without pre-built components, use `@tamagui/core` directly.

---

## Config File Structure

Create `tamagui.config.ts` at the project root:

```tsx
import { defaultConfig } from '@tamagui/config/v5-reanimated'
// Use v5-rn if not using reanimated:
// import { defaultConfig } from '@tamagui/config/v5-rn'
import { createTamagui } from 'tamagui'

// Import your custom tokens/themes if overriding defaults
// import { tokens } from './theme/tokens'
// import { themes } from './theme/themes'
// import { fonts } from './theme/fonts'
// import { animations } from './theme/animations'

const config = createTamagui({
  ...defaultConfig,
  // Override with your own:
  // tokens,
  // themes,
  // fonts,
  // animations,
  settings: {
    // For native-only apps, disable SSR for a small perf gain
    disableSSR: true,
    // Use RN-compatible flex behavior (default in v5)
    styleCompat: 'react-native',
  },
})

export default config
export type AppConfig = typeof config

declare module 'tamagui' {
  interface TamaguiCustomConfig extends AppConfig {}
}
```

### What `defaultConfig` from v5 gives you

- **Tokens:** size, space, radius, zIndex, color scales
- **Themes:** light, dark, plus 10 color sub-themes (blue, red, green, orange, pink, purple, teal, yellow, gray, neutral)
- **Shorthands:** Tailwind-style abbreviations (p, m, px, py, bg, etc.)
- **Media queries:** xs through xxxl breakpoints, plus height-based and capability queries (touch, hoverable)
- **Animations:** timing presets (0ms-500ms) and spring presets (quick, bouncy, lazy)

### v5 breaking changes from v4
- Media query names changed: `2xl`/`2xs` → `xxl`/`xxs`/`xxxs`; max queries use kebab-case (`max-xxl`)
- `styleCompat: 'react-native'` is now the default (uses `flexBasis: 0` instead of legacy `auto`)
- `defaultPosition` is no longer set (reverts to CSS `static` on web — irrelevant for native-only)
- Animations are not bundled — import from the specific driver entry point

---

## Babel Plugin

The babel plugin enables the optimizing compiler on native. It analyzes your styled components at build time and hoists static styles.

In `babel.config.js`:

```js
module.exports = function (api) {
  api.cache(true)
  return {
    presets: ['babel-preset-expo'],
    plugins: [
      [
        '@tamagui/babel-plugin',
        {
          components: ['tamagui'],
          config: './tamagui.config.ts',
          logTimings: true,
          // Keep extraction off in dev for faster iteration
          disableExtraction: process.env.NODE_ENV === 'development',
        },
      ],
      // If using reanimated, this MUST come last
      'react-native-reanimated/plugin',
    ],
  }
}
```

**Important:** The `components` array tells the compiler which packages contain Tamagui components to optimize. If you have your own design system package, add it here: `components: ['tamagui', '@myapp/ui']`.

---

## Metro Plugin

The Metro plugin is primarily for web CSS extraction. For native-only Expo projects, you typically **do not need** `@tamagui/metro-plugin`. The babel plugin handles native optimization.

If you do need it (e.g., for Expo web support later):

```ts
// metro.config.js
const { getDefaultConfig } = require('expo/metro-config')
const { withTamagui } = require('@tamagui/metro-plugin')

const config = getDefaultConfig(__dirname, { isCSSEnabled: true })

module.exports = withTamagui(config, {
  components: ['tamagui'],
  config: './tamagui.config.ts',
  outputCSS: './tamagui-web.css',
})
```

---

## TypeScript Setup

The `declare module` block in `tamagui.config.ts` is critical — it enables typed token/theme autocomplete across your entire app. Without it, you'll get string types instead of specific token names.

Ensure your `tsconfig.json` includes the config file:

```json
{
  "compilerOptions": {
    "strict": true,
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["**/*.ts", "**/*.tsx", "tamagui.config.ts"]
}
```

---

## v5 Config

### Custom tokens

Override the default tokens by spreading and replacing:

```ts
import { defaultConfig } from '@tamagui/config/v5-rn'
import { createTamagui, createTokens } from 'tamagui'

const tokens = createTokens({
  ...defaultConfig.tokens,
  space: {
    ...defaultConfig.tokens.space,
    // Add your design system's spacing scale
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
    true: 16, // default spacing value for the compiler
  },
  color: {
    ...defaultConfig.tokens.color,
    brand: '#2E6B4F',
    brandLight: '#2D7A5A',
    brandSubtle: '#E8F3ED',
    // ... your palette
  },
})
```

### Custom themes

```ts
const themes = {
  light: {
    background: '#F8F5F0',
    surface: '#FFFFFF',
    surfaceRaised: '#F0EBE4',
    color: '#1A1815',        // primary text
    colorSecondary: '#3D3832',
    brand: '#2E6B4F',
    border: '#E0DAD2',
    // ... map all your design tokens
  },
  dark: {
    background: '#000000',
    surface: '#1A1815',
    surfaceRaised: '#262220',
    color: '#E8E4DE',
    colorSecondary: '#A69E93',
    brand: '#4EA87A',
    border: '#3D3832',
  },
}
```

### Custom fonts

```ts
import { createFont } from 'tamagui'

const interFont = createFont({
  family: 'Inter',
  size: {
    1: 11,   // overline
    2: 12,   // caption
    3: 14,   // body-sm
    4: 16,   // body (true/default)
    5: 18,   // body-lg
    6: 20,   // heading
    7: 24,   // heading-lg
    8: 32,   // display
    true: 16,
  },
  lineHeight: {
    1: 16,
    2: 16,
    3: 20,
    4: 24,
    5: 28,
    6: 28,
    7: 32,
    8: 40,
    true: 24,
  },
  weight: {
    4: '400',
    5: '500',
    6: '600',
    7: '700',
    true: '400',
  },
  letterSpacing: {
    4: 0,
    true: 0,
  },
})
```

### Animation configuration (native)

```ts
import { createAnimations } from '@tamagui/animations-react-native'

const animations = createAnimations({
  micro: {
    type: 'timing',
    duration: 200,
  },
  transition: {
    type: 'timing',
    duration: 300,
  },
  sheet: {
    type: 'spring',
    damping: 25,
    mass: 1.2,
    stiffness: 250,
  },
  bounce: {
    type: 'spring',
    damping: 10,
    mass: 0.9,
    stiffness: 200,
  },
})
```

### The `true` token

Every token category (`size`, `space`, `radius`, etc.) should have a `true` key. This tells the compiler what default value to use when a component doesn't specify a size. For example, `<Button />` without a size prop uses the `true` size token.

```ts
size: {
  sm: 36,
  md: 44,
  lg: 52,
  true: 44, // default = md
}
```
