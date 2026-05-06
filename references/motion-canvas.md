# Motion Canvas Reference — Imperative AE-Style Animation

Motion Canvas is the closest code equivalent to After Effects — fully imperative, generator-based, with a real timeline editor in the browser.

## Setup

```bash
npm create motion-canvas@latest my-animation
cd my-animation && npm install && npm run serve
# Opens browser with live timeline editor
```

## Core Concepts

### Generator Scenes — Sequential by Default

```typescript
// yield* = wait for this to complete before moving on
// yield = fire and continue immediately (parallel)

export default makeScene2D(function* (view) {
  const rect = createRef<Rect>();
  
  view.add(<Rect ref={rect} width={200} height={200} fill="#6C63FF" />);
  
  // Sequential
  yield* rect().opacity(0).opacity(1, 0.5);
  yield* rect().position.y(-100, 1, easeInOutCubic);
  yield* waitFor(0.3);
  yield* rect().scale(2, 0.8, easeOutBack);
  
  // Parallel with all()
  yield* all(
    rect().fill('#FF6B6B', 0.5),
    rect().rotation(360, 1),
  );
  
  // Chain with chain()
  yield* chain(
    rect().scale(0.5, 0.3),
    rect().scale(1.5, 0.3),
    rect().scale(1, 0.3),
  );
});
```

## Components

```tsx
// Rect
<Rect width={300} height={200} fill="#fff" stroke="#000" lineWidth={2} radius={8} />

// Circle
<Circle size={100} fill="#6C63FF" />

// Text
<Txt fontSize={48} fontFamily="'Inter', sans-serif" fill="#fff" text="Hello" />

// Layout (Row/Column)
<Layout direction="column" gap={20} alignItems="center">
  <Rect width={100} height={100} fill="red" />
  <Rect width={100} height={100} fill="blue" />
</Layout>

// Line
<Line points={[[-200, 0], [200, 0]]} stroke="#fff" lineWidth={3} />

// Bezier
<CubicBezier p0={[-200, 0]} p1={[-100, -200]} p2={[100, 200]} p3={[200, 0]}
  stroke="#fff" lineWidth={4} />

// Image
<Img src="/logo.png" width={200} />

// Video
<Video src="/clip.mp4" width={1920} height={1080} />
```

## Signals — Reactive Values

```typescript
import { createSignal } from '@motion-canvas/core';

const progress = createSignal(0);

// Use in component
<Rect width={() => progress() * 400} fill="#6C63FF" />

// Animate the signal
yield* progress(1, 2); // 0→1 over 2 seconds
```

## Easing Functions

```typescript
import { 
  easeInOutCubic, easeOutBack, easeInOutElastic,
  easeOutBounce, easeInOutSine, linear,
  createEaseInOut, map, remap,
} from '@motion-canvas/core';
```

## Camera / View Control

```typescript
const camera = createRef<Camera>();

view.add(<Camera ref={camera}>
  {/* All scene content as children */}
  <Rect ... />
</Camera>);

// Zoom
yield* camera().zoom(2, 1);

// Pan
yield* camera().position([200, -100], 1, easeInOutCubic);

// Rotate
yield* camera().rotation(45, 1);

// Reset
yield* camera().reset(1);
```

## Shader / Effects

```typescript
import { blur, brightness, contrast, grayscale } from '@motion-canvas/2d';

// Apply effects to node
rect().filters([blur(10), brightness(1.2)]);

yield* rect().filters([blur(0), brightness(1)], 1);
```

## Full Scene Template — Logo Reveal

```tsx
import { makeScene2D, Rect, Txt, Layout } from '@motion-canvas/2d';
import { createRef, all, chain, waitFor, easeOutBack, easeInOutCubic } from '@motion-canvas/core';

export default makeScene2D(function* (view) {
  view.fill('#0A0A0A');
  
  const logo = createRef<Rect>();
  const title = createRef<Txt>();
  const subtitle = createRef<Txt>();
  const line = createRef<Rect>();
  
  view.add(
    <Layout direction="column" gap={24} alignItems="center">
      <Rect ref={logo} size={80} radius={16} fill="#6C63FF" opacity={0} />
      <Rect ref={line} width={0} height={2} fill="#6C63FF" />
      <Txt ref={title} fontSize={56} fontWeight={700} fill="#fff" opacity={0} text="NEXUS" />
      <Txt ref={subtitle} fontSize={18} fill="#888" opacity={0} text="API Intelligence Platform" />
    </Layout>
  );
  
  // Logo drops in
  yield* logo().opacity(1, 0.3);
  yield* logo().scale(0).scale(1, 0.6, easeOutBack);
  
  yield* waitFor(0.2);
  
  // Line expands
  yield* line().width(300, 0.8, easeInOutCubic);
  
  // Text reveals
  yield* all(
    title().opacity(1, 0.5),
    title().position.y(20).position.y(0, 0.5, easeInOutCubic),
  );
  
  yield* subtitle().opacity(1, 0.4);
  
  yield* waitFor(1);
  
  // Exit: everything fades
  yield* all(
    logo().opacity(0, 0.4),
    title().opacity(0, 0.4),
    subtitle().opacity(0, 0.4),
    line().width(0, 0.4),
  );
});
```

## Parallel vs Sequential

```typescript
// Sequential (one after another)
yield* a().opacity(1, 0.5);  // waits for this
yield* b().opacity(1, 0.5);  // then does this

// Parallel (both at once)
yield* all(
  a().opacity(1, 0.5),
  b().opacity(1, 0.5),
);

// Staggered start
yield* all(
  a().opacity(1, 0.5),
  sequence(0.1, b().opacity(1, 0.5), c().opacity(1, 0.5)),
);
```

## Render Output

```bash
npx motion-canvas render src/project.ts

# Outputs to out/ as PNG sequence
# Convert to video:
ffmpeg -r 60 -i out/scene/frame%07d.png -c:v libx264 -crf 18 -pix_fmt yuv420p output.mp4
```

## Key Differences from Remotion

| Feature | Motion Canvas | Remotion |
|---------|--------------|---------|
| Style | Imperative (AE-like) | Declarative (React) |
| Syntax | Generator functions (yield*) | React components |
| Timeline | Built-in visual editor | Browser preview |
| Output | PNG sequence → ffmpeg | Direct MP4 |
| Best for | Complex scenes, AE exports | Data-driven, programmatic |
