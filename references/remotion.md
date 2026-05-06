# Remotion Reference — Full Composition Patterns

## Setup

```bash
npx create-video@latest my-video
# or add to existing:
npm install remotion @remotion/player @remotion/cli
```

## Core Hooks

```tsx
import {
  useCurrentFrame,      // current frame number (0-indexed)
  useVideoConfig,       // { fps, durationInFrames, width, height, id }
  interpolate,          // map frame ranges to value ranges
  spring,               // physics-based spring animation
  Easing,               // built-in easing functions
} from 'remotion';
```

## interpolate — The Core Tool

```tsx
const frame = useCurrentFrame();

// Basic: frame 0→30 maps to opacity 0→1
const opacity = interpolate(frame, [0, 30], [0, 1]);

// With extrapolation (always clamp unless intentional)
const y = interpolate(frame, [20, 50], [100, 0], {
  extrapolateLeft: 'clamp',
  extrapolateRight: 'clamp',
  easing: Easing.out(Easing.cubic),
});

// Multiple keyframes
const x = interpolate(frame, [0, 30, 60, 90], [0, 100, 50, 200]);
```

## spring — For Bouncy/Natural Motion

```tsx
const scale = spring({
  frame,
  fps,
  config: {
    damping: 12,      // higher = less bounce (80 = no bounce)
    stiffness: 180,   // higher = faster
    mass: 0.8,        // higher = slower/heavier
  },
  from: 0,
  to: 1,
  delay: 15,          // delay in frames
});
```

### Spring Presets

```javascript
// Snappy UI
{ damping: 20, stiffness: 300, mass: 0.5 }

// Gentle
{ damping: 15, stiffness: 120, mass: 1 }

// Bouncy
{ damping: 8, stiffness: 200, mass: 0.8 }

// No bounce (overdamped)
{ damping: 80, stiffness: 200, mass: 1 }
```

## Layout Components

```tsx
import { AbsoluteFill, Sequence, Series } from 'remotion';

// Full canvas positioning
<AbsoluteFill style={{ background: '#000', display: 'flex', justifyContent: 'center' }}>
  <div>Content</div>
</AbsoluteFill>

// Sequence: show component at specific frame time
<Sequence from={30} durationInFrames={60} layout="none">
  <MyComponent />
</Sequence>

// Series: components play back-to-back
<Series>
  <Series.Sequence durationInFrames={60}><Intro /></Series.Sequence>
  <Series.Sequence durationInFrames={90}><Main /></Series.Sequence>
  <Series.Sequence durationInFrames={40}><Outro /></Series.Sequence>
</Series>
```

## Media

```tsx
import { Audio, Video, Img, OffthreadVideo, staticFile } from 'remotion';

<Audio src={staticFile("bgmusic.mp3")} volume={0.5} startFrom={30} />
<Video src={staticFile("clip.mp4")} startFrom={0} endAt={90} />
<Img src={staticFile("logo.png")} style={{ width: 200 }} />

// For videos that need frame-perfect accuracy
<OffthreadVideo src={staticFile("clip.mp4")} />
```

## Full Scene Template

```tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate, spring, Easing } from 'remotion';

interface SceneProps {
  title: string;
  accent: string;
}

export const MyScene: React.FC<SceneProps> = ({ title, accent }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  // Entrance (frames 0–30)
  const titleY = interpolate(frame, [0, 30], [60, 0], {
    extrapolateRight: 'clamp',
    easing: Easing.out(Easing.cubic),
  });

  const titleOpacity = interpolate(frame, [0, 20], [0, 1], { extrapolateRight: 'clamp' });

  // Exit (last 20 frames)
  const exitOpacity = interpolate(
    frame,
    [durationInFrames - 20, durationInFrames],
    [1, 0],
    { extrapolateLeft: 'clamp' }
  );

  const finalOpacity = Math.min(titleOpacity, exitOpacity);

  // Spring scale
  const scale = spring({ frame, fps, config: { damping: 15, stiffness: 150 } });

  return (
    <AbsoluteFill style={{ background: '#0a0a0a', fontFamily: 'Inter, sans-serif' }}>
      <AbsoluteFill style={{
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center',
        alignItems: 'center',
        opacity: finalOpacity,
        transform: `translateY(${titleY}px) scale(${scale})`,
      }}>
        <h1 style={{ color: accent, fontSize: 80, margin: 0 }}>{title}</h1>
      </AbsoluteFill>
    </AbsoluteFill>
  );
};
```

## Root.tsx — Register Compositions

```tsx
import { Composition } from 'remotion';
import { MyScene } from './compositions/MyScene';

export const RemotionRoot: React.FC = () => (
  <>
    <Composition
      id="MyScene"
      component={MyScene}
      durationInFrames={150}  // 5 seconds at 30fps
      fps={30}
      width={1920}
      height={1080}
      defaultProps={{ title: "Hello", accent: "#6C63FF" }}
    />
  </>
);
```

## Kinetic Text — Per-Character Animation

```tsx
const KineticText: React.FC<{ text: string }> = ({ text }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <div style={{ display: 'flex', overflow: 'hidden' }}>
      {text.split('').map((char, i) => {
        const delay = i * 2;
        const charFrame = Math.max(0, frame - delay);

        const y = interpolate(charFrame, [0, 20], [80, 0], {
          extrapolateRight: 'clamp',
          easing: Easing.out(Easing.back(1.7)),
        });

        const opacity = interpolate(charFrame, [0, 15], [0, 1], { extrapolateRight: 'clamp' });

        return (
          <span key={i} style={{ display: 'inline-block', transform: `translateY(${y}px)`, opacity }}>
            {char === ' ' ? '\u00A0' : char}
          </span>
        );
      })}
    </div>
  );
};
```

## Data-Driven Video Pattern

```tsx
const data = [
  { id: 'product1', name: 'Nexus', color: '#6C63FF' },
  { id: 'product2', name: 'Luné', color: '#FF6B9D' },
];

export const RemotionRoot = () => (
  <>
    {data.map(item => (
      <Composition
        key={item.id}
        id={item.id}
        component={ProductCard}
        durationInFrames={150}
        fps={30}
        width={1080}
        height={1080}
        defaultProps={item}
      />
    ))}
  </>
);
```

## Timing Reference

```
frames = seconds × fps
// 30fps: 1s=30f, 2s=60f, 5s=150f, 10s=300f, 30s=900f
// 60fps: 1s=60f, 2s=120f, 5s=300f
```

## Critical Rules

- Every frame must be **deterministic** — no `Math.random()` without seeded RNG
- Use `<Sequence>` to stagger scenes in time
- Use `<Series>` for back-to-back scenes
- Use `<Audio>` and `<Video>` for media — they auto-sync to frame time
- Use `staticFile()` for local assets
- Always wrap content in `<AbsoluteFill>`

## Render Commands

```bash
npx remotion render src/index.ts MyScene out/video.mp4
npx remotion render src/index.ts MyScene out/video.mp4 --codec=h264 --crf=18
npx remotion still src/index.ts MyScene --frame=30 out/thumbnail.png
npx remotion studio  # open browser preview
```

## Output Specifications

- Product demo (web): 1920×1080 at 30fps
- App Store preview (mobile): 1080×1920 at 60fps, max 30s
- Square (social): 1080×1080 at 30fps
- App Store: must show actual UI in first 3 seconds

## @remotion/player — Embed in React App

```tsx
import { Player } from '@remotion/player';

<Player
  component={MyScene}
  durationInFrames={150}
  fps={30}
  compositionWidth={1920}
  compositionHeight={1080}
  style={{ width: '100%' }}
  controls
  autoPlay
  loop
  inputProps={{ title: "Hello", accent: "#6C63FF" }}
/>
```
