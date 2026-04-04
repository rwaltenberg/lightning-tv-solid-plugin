# ElementNode API Reference

> Full API reference for ElementNode: constructor, all properties, and all methods.

**Source**: `core-rendering-nodes.md` | **Severity**: informational

## Detail

### Constructor

```ts
class ElementNode extends Object {
  constructor(name: string);
  // name === 'text' --> _type = 'textNode'
  // anything else   --> _type = 'element'
}
```

### Animatable Properties (proxy to this.lng via _sendToLightningAnimatable)

Defined via `Object.defineProperty` on the prototype:

```ts
x: number;
y: number;
w: number;          // width (alias: width)
h: number;          // height (alias: height)
color: number;      // 0xRRGGBBAA
colorTop: number;
colorRight: number;
colorLeft: number;
colorBottom: number;
colorTl: number;
colorTr: number;
colorBl: number;
colorBr: number;
alpha: number;
scale: number;
scaleX: number;
scaleY: number;
rotation: number;
mount: number;
mountX: number;
mountY: number;
pivot: number;
pivotX: number;
pivotY: number;
zIndex: number;
fontSize: number;
lineHeight: number;
```

### Non-Animating Properties (direct proxy to this.lng[key])

```ts
autosize: boolean;
clipping: boolean;
contain: 'width' | 'both' | 'none';
data: Record<string, unknown>;
text: string;
textAlign: 'left' | 'center' | 'right';
texture: any;
textureOptions: any;
fontStretch: string;
fontStyle: string;
imageType: string;
letterSpacing: number;
maxHeight: number;
maxLines: number;
maxWidth: number;
offsetY: number;
overflowSuffix: string;
rtt: boolean;
scrollable: boolean;
scrollY: number;
src: string;
srcHeight: number;
srcWidth: number;
srcX: number;
srcY: number;
strictBounds: boolean;
textBaseline: string;
textOverflow: string;
verticalAlign: string;
wordBreak: boolean;
wordWrap: boolean;
group: string;
destroyed: boolean;
preventCleanup: boolean;
```

### Layout Properties

```ts
display?: 'flex' | 'block';
flexDirection?: 'row' | 'column' | 'row-reverse' | 'column-reverse';
flexGrow?: number;
flexShrink?: number;
flexBasis?: number | string;
flexWrap?: 'nowrap' | 'wrap' | 'wrap-reverse';
flexItem?: boolean;
flexOrder?: number;
flexBoundary?: 'contain' | 'fixed';
flexCrossBoundary?: 'fixed';
gap?: number;
rowGap?: number;
columnGap?: number;
justifyContent?: 'flexStart' | 'flexEnd' | 'center' | 'spaceBetween' | 'spaceAround' | 'spaceEvenly';
alignItems?: 'flexStart' | 'flexEnd' | 'center';
alignSelf?: 'flexStart' | 'flexEnd' | 'center';
padding?: number | [number, number] | [number, number, number] | [number, number, number, number];
paddingTop?: number;
paddingRight?: number;
paddingBottom?: number;
paddingLeft?: number;
margin?: number | [number, number] | [number, number, number] | [number, number, number, number];
marginTop?: number;
marginRight?: number;
marginBottom?: number;
marginLeft?: number;
```

### Positioning Shorthands

```ts
right?: number;     // Sets x = parentWidth - right, mountX = 1
bottom?: number;    // Sets y = parentHeight - bottom, mountY = 1
center?: boolean;   // Sets both centerX and centerY
centerX?: boolean;  // Sets x += parentWidth/2, mountX = 0.5
centerY?: boolean;  // Sets y += parentHeight/2, mountY = 0.5
```

### Lifecycle Callbacks

```ts
onCreate?: (this: ElementNode, el: ElementNode) => void;
onDestroy?: (this: ElementNode, el: ElementNode) => Promise<any> | void;
onRender?: (this: ElementNode, el: ElementNode) => void;
onRemove?: (this: ElementNode, el: ElementNode) => void;
onLayout?: (this: ElementNode, target: ElementNode) => void;
onEvent?: OnEvent;
```

### Shader/Effects Accessors

```ts
border: BorderStyle;
borderBottom: BorderStyle;
borderTop: BorderStyle;
borderLeft: BorderStyle;
borderRight: BorderStyle;
shadow: ShadowProps;
rounded: BorderRadius;          // number | number[]
borderRadius: BorderRadius;     // alias for rounded
linearGradient: LinearGradientProps;
radialGradient: RadialGradientProps;
effects: StyleEffects;          // getter/setter on class body
```

### Key Methods

```ts
// Tree manipulation
insertChild(node: ElementNode | ElementText | TextNode, beforeNode?: ...): void;
removeChild(node: ElementNode | ElementText | TextNode): void;
getChildById(id: string): ElementNode | ElementText | undefined;
searchChildrenById(id: string): ElementNode | undefined;  // deep search

// Rendering
render(topNode?: boolean): void;
destroy(): void;

// Animation
animate(
  props: Partial<INodeAnimateProps<CoreShaderNode>>,
  animationSettings?: AnimationSettings
): IAnimationController;

chain(
  props: Partial<INodeAnimateProps<CoreShaderNode>>,
  animationSettings?: AnimationSettings
): ElementNode;  // returns this for chaining

start(): Promise<void>;  // executes the animation queue

// Focus
setFocus(): void;

// State
get states(): States;
set states(states: NodeStates): void;

// Events (bubble up the tree)
emit(event: string, ...args: any[]): boolean;

// Layout
requiresLayout(): boolean;  // true when display === 'flex' or onLayout is set
updateLayout(): void;
```

## Related Notes

- [core/element-node-proxy.md] -- proxy pattern that backs all properties
- [core/node-lifecycle.md] -- full lifecycle using these methods
- [constraints/animate-before-render.md] -- animate/chain/start require rendered === true
- [constraints/chain-while-running.md] -- chain() resets queue if already running
- [api/states-class.md] -- States class returned by the states getter
