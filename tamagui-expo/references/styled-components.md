# Styled Components in Tamagui

## Table of Contents
1. [Basic styled()](#basic-styled)
2. [Variants](#variants)
3. [Spread Variants](#spread-variants)
4. [.styleable() for HOCs](#styleable)
5. [accept for Custom Props](#accept)
6. [Pseudo Styles](#pseudo-styles)
7. [Platform-Specific Styles](#platform-styles)
8. [Composition Patterns](#composition-patterns) (incl. `createStyledContext`)
9. [Performance Considerations](#performance)

---

## Basic styled()

```tsx
import { styled, View, Text } from 'tamagui'

// Extend View with default styles
export const Card = styled(View, {
  name: 'Card',                    // enables sub-theme lookup
  backgroundColor: '$surface',
  borderRadius: '$4',              // uses radius token
  padding: '$4',
  borderWidth: 1,
  borderColor: '$border',
})

// Extend Text
export const Title = styled(Text, {
  name: 'Title',
  color: '$color',
  fontSize: '$6',
  fontWeight: '600',
})
```

The `name` property is important — it enables:
- Sub-theme resolution (`light_Card`, `dark_Card`)
- DevTools identification
- Compiler optimization hints

---

## Variants

Variants define conditional style branches. They're the Tamagui equivalent of props that change appearance.

```tsx
export const Button = styled(View, {
  name: 'Button',
  backgroundColor: '$brand',
  borderRadius: '$3',
  paddingHorizontal: '$4',
  alignItems: 'center',
  justifyContent: 'center',

  variants: {
    // Boolean variant
    disabled: {
      true: {
        opacity: 0.4,
        pointerEvents: 'none',
      },
    },

    // Enum variant
    variant: {
      primary: {
        backgroundColor: '$brand',
      },
      secondary: {
        backgroundColor: 'transparent',
        borderWidth: 1,
        borderColor: '$brand',
      },
      destructive: {
        backgroundColor: '$error',
      },
      tertiary: {
        backgroundColor: 'transparent',
      },
    },

    // Size variant
    size: {
      sm: {
        height: 36,
        paddingHorizontal: '$3',
      },
      md: {
        height: 44,
        paddingHorizontal: '$4',
      },
      lg: {
        height: 52,
        paddingHorizontal: '$6',
      },
    },

    // Full-width variant
    fullWidth: {
      true: {
        width: '100%',
      },
    },
  } as const,

  // Default variant values
  defaultVariants: {
    variant: 'primary',
    size: 'md',
  },
})
```

Usage:
```tsx
<Button variant="secondary" size="lg" fullWidth />
<Button variant="destructive" disabled />
```

**Always add `as const`** to the variants object for proper TypeScript inference.

---

## Spread Variants

Spread variants map token scales to component styles dynamically. They're prefixed with `...`:

```tsx
export const Square = styled(View, {
  variants: {
    size: {
      // `...size` maps to your size tokens
      '...size': (val, { tokens }) => ({
        width: tokens.size[val] ?? val,
        height: tokens.size[val] ?? val,
      }),
    },

    elevation: {
      // `...space` maps to space tokens for shadow offset
      '...space': (val, { tokens }) => ({
        shadowOffset: { width: 0, height: tokens.space[val] ?? val },
        shadowOpacity: 0.08,
        shadowRadius: (tokens.space[val] ?? val) * 2,
      }),
    },
  } as const,
})
```

Available spread prefixes: `...size`, `...space`, `...color`, `...radius`, `...zIndex`, `...fontSize`, `...fontWeight`, `...lineHeight`.

---

## .styleable()

When you wrap a `styled()` component in a functional component (for logic, refs, hooks), the outer component loses the ability to be wrapped with `styled()` again. `.styleable()` fixes this:

```tsx
const StyledInput = styled(TextInput, {
  name: 'Input',
  borderWidth: 1,
  borderColor: '$border',
  borderRadius: '$2',
  padding: '$3',
  fontSize: '$4',
  color: '$color',

  focusStyle: {
    borderColor: '$brand',
    borderWidth: 2,
  },

  variants: {
    error: {
      true: {
        borderColor: '$error',
      },
    },
  } as const,
})

// Wrap with .styleable() so it can be further styled
export const Input = StyledInput.styleable<{ label?: string }>(
  ({ label, ...props }, ref) => {
    return (
      <View>
        {label && <Label>{label}</Label>}
        <StyledInput ref={ref} {...props} />
      </View>
    )
  }
)

// Now this works:
const BigInput = styled(Input, {
  fontSize: '$6',
})
```

The key: pass ALL Tamagui style props through to the inner styled component. Don't destructure and drop them.

---

## accept

The `accept` configuration tells Tamagui to resolve token/theme values for custom (non-style) props:

```tsx
import Svg, { Path } from 'react-native-svg'

const StyledIcon = styled(Svg, {
  accept: {
    fill: 'color',      // resolves $brand → #2E6B4F
    stroke: 'color',
  },
})

// Usage:
<StyledIcon fill="$brand" stroke="$border" width={24} height={24}>
  <Path d="..." />
</StyledIcon>
```

`accept` values can be:
- `'color'` — resolves from color tokens and themes
- `'size'` — resolves from size tokens
- `'space'` — resolves from space tokens
- `'style'` — resolves a full style object (useful for `contentContainerStyle` etc.)

---

## Pseudo Styles

Tamagui supports interaction pseudo-states that work on native:

```tsx
const Pressable = styled(View, {
  backgroundColor: '$surface',

  // When pressed/touched
  pressStyle: {
    backgroundColor: '$surfaceRaised',
    scale: 0.97,
  },

  // When focused (accessibility)
  focusStyle: {
    borderColor: '$brand',
    borderWidth: 2,
  },

  // Hover only works on web and devices with mouse/trackpad
  hoverStyle: {
    backgroundColor: '$surfaceRaised',
  },

  // Enter/exit animations
  enterStyle: {
    opacity: 0,
    y: 10,
  },
  exitStyle: {
    opacity: 0,
    y: -10,
  },
})
```

`pressStyle` is the most important on native. It replaces the need for `Pressable` or `TouchableOpacity` in many cases.

**Note:** Tamagui does NOT support `Pressable`, `TouchableOpacity`, or `TouchableHighlight` as base components for `styled()`. Use `View` with `pressStyle` instead, or use Tamagui's built-in `Button` component from the UI kit.

---

## Platform-Specific Styles

Use `$platform-` prefix for iOS/Android differences:

```tsx
const Card = styled(View, {
  backgroundColor: '$surface',
  borderRadius: '$3',

  // Android elevation
  elevation: 2,

  // iOS shadow (overrides elevation visual on iOS)
  '$platform-ios': {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.08,
    shadowRadius: 3,
    elevation: undefined, // clear Android elevation on iOS
  },
})
```

This is cleaner than `Platform.select()` because the compiler can still optimize these branches.

---

## Composition Patterns

### Compound components with `createStyledContext`

When building compound components (a parent with coordinated children like `Card` with `Card.Title`, `Card.Description`), use `createStyledContext` to share variant values across parts. This is the idiomatic Tamagui approach — it lets a parent's `size` (or any variant) automatically cascade to children without manual prop drilling.

```tsx
import { styled, View, Text, Image, createStyledContext } from 'tamagui'
import { withStaticProperties } from '@tamagui/helpers'

// 1. Create a typed context with default values for shared variants
const CardContext = createStyledContext({
  size: 'md' as 'sm' | 'md' | 'lg',
})

// 2. Each part references the same context
const CardFrame = styled(View, {
  name: 'Card',
  context: CardContext,
  backgroundColor: '$surface',
  borderRadius: '$3',
  borderWidth: 1,
  borderColor: '$border',

  variants: {
    size: {
      sm: { padding: '$2', gap: '$1' },
      md: { padding: '$4', gap: '$2' },
      lg: { padding: '$6', gap: '$3' },
    },
  } as const,

  defaultVariants: {
    size: 'md',
  },
})

const CardImage = styled(Image, {
  name: 'CardImage',
  context: CardContext,
  borderTopLeftRadius: '$3',
  borderTopRightRadius: '$3',
  width: '100%',

  variants: {
    size: {
      sm: { aspectRatio: 2 },
      md: { aspectRatio: 16 / 9 },
      lg: { aspectRatio: 4 / 3 },
    },
  } as const,
})

const CardTitle = styled(Text, {
  name: 'CardTitle',
  context: CardContext,
  fontWeight: '600',
  color: '$color',

  variants: {
    size: {
      sm: { fontSize: '$4' },
      md: { fontSize: '$6' },
      lg: { fontSize: '$7' },
    },
  } as const,
})

const CardDescription = styled(Text, {
  name: 'CardDescription',
  context: CardContext,
  color: '$colorSecondary',

  variants: {
    size: {
      sm: { fontSize: '$2' },
      md: { fontSize: '$3' },
      lg: { fontSize: '$4' },
    },
  } as const,
})

// 3. Compose with withStaticProperties
export const Card = withStaticProperties(CardFrame, {
  Image: CardImage,
  Title: CardTitle,
  Description: CardDescription,
})
```

Usage — setting `size` on the parent cascades to all children automatically:

```tsx
<Card size="lg">
  <Card.Image source={birdPhoto} />
  <Card.Title>Laughing Kookaburra</Card.Title>
  <Card.Description>Dacelo novaeguineae</Card.Description>
</Card>
```

**Key points:**
- Every part that should inherit shared variants needs `context: CardContext`
- The variant names must match across parts (all use `size` here)
- `withStaticProperties` is a Tamagui helper that attaches sub-components while preserving types — use it instead of `Object.assign`
- You can share multiple variants through a single context (e.g., `size` and `variant`)

### Simple compound components (no shared state)

If children don't need to respond to parent variants — they're just namespaced for API ergonomics — you can skip the context and use a plain `Object.assign`:

```tsx
const CardFrame = styled(View, {
  name: 'Card',
  backgroundColor: '$surface',
  borderRadius: '$3',
  padding: '$4',
  borderWidth: 1,
  borderColor: '$border',
})

const CardTitle = styled(Text, {
  name: 'CardTitle',
  fontSize: '$6',
  fontWeight: '600',
  color: '$color',
})

export const Card = Object.assign(CardFrame, {
  Title: CardTitle,
})
```

Use `createStyledContext` when variants need to cascade; use `Object.assign` when they don't.

### Extending existing styled components

```tsx
const BaseText = styled(Text, {
  color: '$color',
  fontFamily: '$body',
})

// Extend with more specific styles
const Heading = styled(BaseText, {
  fontSize: '$6',
  fontWeight: '600',
})

const Caption = styled(BaseText, {
  fontSize: '$2',
  fontWeight: '500',
  color: '$colorSecondary',
})
```

---

## Performance

1. **Static styles are free.** Styles defined directly in `styled()` are extracted at compile time. They cost nothing at runtime.

2. **Variants are cheap.** The compiler can often pre-compute variant branches. Keep variant values simple (literal objects, not functions) when possible.

3. **Spread variants have a runtime cost.** The function runs at render time to compute styles. Use them sparingly for truly dynamic sizing.

4. **`animation` prop adds overhead.** Only use `animation` on components that actually animate. Don't add it "just in case".

5. **`enterStyle`/`exitStyle` need an animation driver.** They don't work without one, and they add the animation overhead even if the component never mounts/unmounts dynamically.

6. **Measure before optimizing.** Tamagui's compiler output is usually fast enough. Profile with Flashlight or React DevTools before micro-optimizing.

### Compiler output verification

When `logTimings: true` is set in the babel plugin, you'll see lines like:

```
app/components/card.tsx  12ms · 3 optimized · 2 flattened
```

"Optimized" means styles were hoisted. "Flattened" means the component tree was simplified (e.g., a `<View>` wrapping a `<View>` was collapsed into one). Both are good signals.
