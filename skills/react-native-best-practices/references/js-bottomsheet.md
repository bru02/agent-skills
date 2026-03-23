---
title: Bottom Sheet
impact: CRITICAL
tags: bottom-sheet, gorhom, re-renders, shared-values, gestures, context, scrollable, modal, keyboard
---

# Skill: Bottom Sheet Best Practices

Optimize `@gorhom/bottom-sheet` for smooth 60 FPS by keeping gesture/scroll-driven state on the UI thread.

## Quick Pattern

**Incorrect (bridges to JS every frame — full subtree re-render):**

```jsx
const handleAnimate = useCallback((fromIndex, toIndex) => {
  setIsExpanded(toIndex > 0); // re-renders entire tree
}, []);

<BottomSheet onAnimate={handleAnimate}>
  <ExpensiveContent isExpanded={isExpanded} />
</BottomSheet>
```

**Correct (stays on UI thread — zero re-renders):**

```jsx
const animatedIndex = useSharedValue(0);

const overlayStyle = useAnimatedStyle(() => ({
  opacity: withTiming(animatedIndex.value > 0 ? 0.5 : 0),
}));

<BottomSheet animatedIndex={animatedIndex}>
  <ExpensiveContent />
</BottomSheet>
<Animated.View style={[styles.overlay, overlayStyle]} />
```

## When to Use

- Implementing or optimizing a bottom sheet with `@gorhom/bottom-sheet`
- Bottom sheet gestures cause jank or dropped frames
- Scroll inside bottom sheet triggers excessive re-renders
- Context provider wrapping bottom sheet re-renders the entire subtree
- Visual-only state (shadow, opacity, footer visibility) managed with `useState`
- Need to choose between `BottomSheet` and `BottomSheetModal`
- Scrollable content inside bottom sheet doesn't coordinate with gestures
- Keyboard doesn't interact properly with the sheet

## Prerequisites

- `@gorhom/bottom-sheet` v4+ (v5 recommended)
- `react-native-reanimated` v3+
- `react-native-gesture-handler` v2+

```bash
npm install @gorhom/bottom-sheet react-native-reanimated react-native-gesture-handler
```

> **Note**: In v5, `enableDynamicSizing` defaults to `true`. If you use static `snapPoints`, set `enableDynamicSizing={false}` explicitly to avoid unexpected behavior.

## Problem Description

Gesture and scroll callbacks bridge UI→JS via `runOnJS`, call `setState`, update context providers, and re-render the entire bottom sheet subtree every frame. At 60 FPS each frame has a 16.6ms budget — a single `setState` in a gesture handler can blow through that, causing visible jank and dropped frames.

## Step-by-Step Instructions

### 1. Identify Visual-Only vs React State

| Use SharedValue | Use useState |
|----------------|-------------|
| Visual changes (opacity, shadow, transform) | Conditional rendering (mount/unmount) |
| High-frequency updates (gestures, scroll) | Side effects (analytics, navigation) |
| Values read inside `useAnimatedStyle` | JS logic branching |

### 2. Convert Gesture-Driven State to SharedValue

Remove the `runOnJS` bridge — set `.value` directly and consume via `useAnimatedStyle`.

**Before:**

```jsx
const [shadowOpacity, setShadowOpacity] = useState(0);

const handleAnimate = useCallback((fromIndex, toIndex) => {
  setShadowOpacity(toIndex > 0 ? 0.3 : 0);
}, []);

<BottomSheet onAnimate={handleAnimate}>
  <View style={{ shadowOpacity }}>
    <HeavyContent />
  </View>
</BottomSheet>
```

**After:**

```jsx
const animatedIndex = useSharedValue(0);

const shadowStyle = useAnimatedStyle(() => ({
  shadowOpacity: withTiming(animatedIndex.value > 0 ? 0.3 : 0),
}));

<BottomSheet animatedIndex={animatedIndex}>
  <Animated.View style={shadowStyle}>
    <HeavyContent />
  </Animated.View>
</BottomSheet>
```

### 3. Add Bail-Out Guards for Scroll-Driven State

Only update shared values when they actually change to avoid unnecessary style recalculations.

```jsx
const animatedPosition = useSharedValue(0);

const headerStyle = useAnimatedStyle(() => {
  const show = animatedPosition.value < 200;
  // Reanimated already optimizes this, but for derived values:
  return {
    opacity: withTiming(show ? 1 : 0),
    transform: [{ translateY: withTiming(show ? 0 : -50) }],
  };
});

<BottomSheet animatedPosition={animatedPosition}>
  <Animated.View style={headerStyle}>
    <Header />
  </Animated.View>
</BottomSheet>
```

### 4. Replace Conditional Rendering with Animated Visibility

Always mount the element. Use `useAnimatedStyle` for opacity/translateY and disable interaction when hidden.

**Before:**

```jsx
const [showFooter, setShowFooter] = useState(false);

// re-mounts footer on every toggle
{showFooter && <Footer />}
```

**After:**

```jsx
const animatedIndex = useSharedValue(0);
const [isInteractive, setIsInteractive] = useState(false);

const footerStyle = useAnimatedStyle(() => ({
  opacity: withTiming(animatedIndex.value >= 1 ? 1 : 0),
  transform: [{ translateY: withTiming(animatedIndex.value >= 1 ? 0 : 50) }],
}));

useAnimatedReaction(
  () => animatedIndex.value >= 1,
  (visible, prev) => {
    if (visible !== prev) {
      runOnJS(setIsInteractive)(visible);
    }
  }
);

<Animated.View
  style={footerStyle}
  pointerEvents={isInteractive ? 'auto' : 'none'}
  accessibilityElementsHidden={!isInteractive}
  importantForAccessibility={isInteractive ? 'auto' : 'no-hide-descendants'}
>
  <Footer />
</Animated.View>
```

### 5. Sync Minimal React State via `useAnimatedReaction`

When you need a boolean for `pointerEvents` or accessibility, bridge from shared value to state — but only re-render the wrapper, not the full tree.

```jsx
const WrapperOnly = ({ animatedIndex, children }) => {
  const [active, setActive] = useState(false);

  useAnimatedReaction(
    () => animatedIndex.value > 0,
    (curr, prev) => {
      if (curr !== prev) runOnJS(setActive)(curr);
    }
  );

  return (
    <View pointerEvents={active ? 'auto' : 'none'}>
      {children}
    </View>
  );
};
```

### 6. Use Library-Provided Components and Props

**Scrollables** — always use these instead of React Native built-ins inside a bottom sheet:

```jsx
import {
  BottomSheetScrollView,
  BottomSheetFlatList,
  BottomSheetSectionList,
} from '@gorhom/bottom-sheet';

// For FlashList:
import { BottomSheetFlashList } from '@gorhom/bottom-sheet';

<BottomSheet snapPoints={snapPoints} enableDynamicSizing={false}>
  <BottomSheetFlatList
    data={data}
    keyExtractor={(item) => item.id}
    renderItem={renderItem}
  />
</BottomSheet>
```

**Key props:**

| Prop | Purpose |
|------|---------|
| `containerHeight` | Provide to skip extra measurement re-render on mount |
| `enableDynamicSizing={false}` | Required with static `snapPoints` in v5 |
| `animatedIndex` | SharedValue for continuous index tracking on UI thread |
| `animatedPosition` | SharedValue for continuous position tracking on UI thread |
| `onChange` | Fires on snap **completion** only (discrete) — use for analytics/side effects |
| `onAnimate` | Fires **once** before animation starts — good for pre-animation logic |

### 7. BottomSheetModal Setup

```jsx
import {
  BottomSheetModal,
  BottomSheetModalProvider,
} from '@gorhom/bottom-sheet';

const App = () => (
  <BottomSheetModalProvider>
    <BottomSheetModal
      ref={modalRef}
      snapPoints={snapPoints}
      enableDynamicSizing={false}
      enableDismissOnClose={true}
      stackBehavior="push" // 'push' | 'switch' | 'replace'
    >
      <Content />
    </BottomSheetModal>
  </BottomSheetModalProvider>
);
```

**iOS layering fix** — use `FullWindowOverlay` to render above native navigation:

```jsx
import { FullWindowOverlay } from 'react-native-screens';

<BottomSheetModal
  containerComponent={(props) => <FullWindowOverlay>{props.children}</FullWindowOverlay>}
>
```

### 8. Keyboard Handling

```jsx
<BottomSheet
  snapPoints={snapPoints}
  enableDynamicSizing={false}
  keyboardBehavior="interactive"    // 'extend' | 'fillParent' | 'interactive'
  keyboardBlurBehavior="restore"    // reset sheet position when keyboard dismisses
  enableBlurKeyboardOnGesture={true} // dismiss keyboard on drag
>
  <BottomSheetTextInput
    placeholder="Type here..."
    style={styles.input}
  />
</BottomSheet>
```

| `keyboardBehavior` | Effect |
|--------------------|--------|
| `extend` | Sheet grows to accommodate keyboard |
| `fillParent` | Sheet fills parent when keyboard appears |
| `interactive` | Sheet follows keyboard position interactively |

> Always use `BottomSheetTextInput` instead of React Native's `TextInput` inside a bottom sheet.

## Derived Animations with `animatedPosition`

Use the `animatedPosition` shared value for smooth derived UI that stays on the UI thread:

```jsx
const animatedPosition = useSharedValue(0);

const backdropStyle = useAnimatedStyle(() => ({
  opacity: interpolate(
    animatedPosition.value,
    [0, 300],
    [0.5, 0],
    Extrapolation.CLAMP
  ),
}));

<BottomSheet animatedPosition={animatedPosition} snapPoints={snapPoints}>
  <Content />
</BottomSheet>
<Animated.View style={[StyleSheet.absoluteFill, backdropStyle]} pointerEvents="none" />
```

## Common Pitfalls

- **Using `onChange` for continuous position tracking** — it fires on snap completion only (discrete). Use `animatedPosition` or `animatedIndex` shared values instead.
- **Forgetting `pointerEvents='none'` on always-mounted hidden elements** — invisible elements still capture touches.
- **Missing accessibility attributes on hidden elements** — add `accessibilityElementsHidden` and `importantForAccessibility='no-hide-descendants'`.
- **Bundling independent state values in one context** — split context or use atomic state to prevent cascading re-renders.
- **Using `enableDynamicSizing` with static snap points in v5** — v5 defaults to `true`, which conflicts with explicit `snapPoints`. Set `enableDynamicSizing={false}`.
- **Using React Native `ScrollView`/`FlatList` inside bottom sheet** — gestures won't coordinate. Use `BottomSheetScrollView`, `BottomSheetFlatList`, etc.
- **Using React Native touchables on Android** — use `BottomSheetTouchableOpacity` and similar from the library.
- **Not providing `containerHeight`** — causes an extra re-render on mount for measurement.
- **Using regular `TextInput` instead of `BottomSheetTextInput`** — keyboard handling won't work properly.

## Related Skills

- [js-animations-reanimated.md](./js-animations-reanimated.md) — SharedValue and useAnimatedStyle fundamentals
- [js-atomic-state.md](./js-atomic-state.md) — Context splitting and atomic state patterns
- [js-profile-react.md](./js-profile-react.md) — Profiling to measure re-render reduction
- [js-measure-fps.md](./js-measure-fps.md) — Verify FPS improvement after optimization
