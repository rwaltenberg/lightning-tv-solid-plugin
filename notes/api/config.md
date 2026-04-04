# Config API Reference

> Full Config interface with all fields, types, and defaults.

**Source**: `core-rendering-nodes.md` | **Severity**: informational

## Detail

### Runtime Config Object

```ts
export const Config: Config;

export interface Config {
  debug: boolean;
  // default: false
  // Enables general debug logging.

  focusDebug: boolean;
  // default: false
  // Enables focus state debug logging.

  domRendererEnabled: boolean;
  // default: false
  // Enables the DOM renderer backend at runtime.

  keyDebug: boolean;
  // default: false
  // Enables key event debug logging.

  animationSettings?: AnimationSettings;
  // default: { duration: 250, easing: 'ease-in-out' }
  // Global default animation settings for transitions.

  animationsEnabled: boolean;
  // default: true
  // Master switch for all transition animations.

  fontSettings: Partial<TextProps>;
  // default: { fontFamily: 'Ubuntu', fontSize: 100 }
  // Default font properties applied to all text nodes.

  rendererOptions?: Partial<RendererMainSettings> | DomRendererMainSettings;
  // default: undefined
  // Options passed to the renderer at init time.

  setActiveElement: (elm: ElementNode) => void;
  // Callback invoked when focus changes. Wired to activeElement signal setter.

  focusStateKey: DollarString;
  // default: '$focus'
  // The state key added to the focused element's states.

  lockStyles?: boolean;
  // default: true
  // When true, style setter is a no-op after first assignment.

  fontWeightAlias?: Record<string, number | string>;
  // default: undefined
  // Maps font weight names (e.g., 'bold') to numeric values.

  throttleInput?: number;
  // default: undefined
  // Global input throttle in ms.

  taskDelay?: number;
  // default: 50 (used in processTasks)
  // Delay in ms between scheduled tasks.

  convertToShader: (node: ElementNode, v: StyleEffects) => IRendererShader;
  // Converts effect config to a renderer shader. Default implementation:
  // builds type string ('rounded', 'roundedWithBorder', etc.) and calls renderer.createShader.

  stateOrder?: DollarString[];
  // default: []
  // Global state priority order. Later entries have higher specificity.
}
```

### Compile-Time Globals

```ts
declare global {
  var LIGHTNING_DOM_RENDERING: boolean | undefined;
  // Set in Vite define config. Switches to DOM renderer.

  var LIGHTNING_DISABLE_SHADERS: boolean | undefined;
  // Set in Vite define config. Disables shader effects.
}
```

### Derived Constants (from config.ts)

```ts
export const isDev: boolean;
// true when import.meta.env.DEV is truthy

export const DOM_RENDERING: boolean;
// true when LIGHTNING_DOM_RENDERING === true

export const SHADERS_ENABLED: boolean;
// true unless LIGHTNING_DISABLE_SHADERS === true
```

### Default convertToShader Implementation

The default `convertToShader` builds the shader type string from the effects present:

```ts
function convertToShader(_node: ElementNode, v: StyleEffects): IRendererShader {
  let type = 'rounded';
  if (v.border) type += 'WithBorder';
  if (v.shadow) type += 'WithShadow';
  return renderer.createShader(type, v);
}
```

Produces one of: `'rounded'`, `'roundedWithBorder'`, `'roundedWithShadow'`, `'roundedWithBorderWithShadow'`.

## Gotchas

- Config must be set **before** calling `createRenderer` or `startLightningRenderer` -- some fields (like `rendererOptions`) are consumed at init time and cannot be changed after.
- `lockStyles` defaults to `true` -- explicitly set to `false` if you need to reassign styles (not recommended).
- `fontSettings` defaults to `{ fontFamily: 'Ubuntu', fontSize: 100 }` -- always override with your app's font settings.
- `stateOrder = []` means state merge order follows insertion order into the `states` array.

## Related Notes

- [constraints/style-locked-once.md] -- lockStyles behavior in detail
- [reactivity/state-order-specificity.md] -- stateOrder specificity rules
- [reactivity/task-scheduling.md] -- taskDelay usage
- [core/dual-renderer-architecture.md] -- domRendererEnabled and compile-time globals
