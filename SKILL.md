---
name: motion-composer
description: Create professional motion graphics, animations, and video compositions using code. Use this skill whenever the user asks for: animations, motion graphics, video intros/outros, animated logos, scroll animations, timeline-based sequences, GSAP animations, Remotion video compositions, Motion Canvas scenes, After Effects-style keyframe animations, Framer Motion transitions, particle systems, SVG path animations, kinetic typography, animated data visualizations, product demo videos, App Store preview animations, or any time-based visual storytelling. Trigger on keywords like "animate", "motion", "video", "sequence", "timeline", "transition", "keyframe", "scroll animation", "intro", "outro", "reveal", "loop", or when the user wants something to move. This skill encodes expert-level knowledge of animation principles, timing systems, easing curves, and multi-framework composition patterns. Always use this skill for animation requests — even simple ones — as it dramatically improves output quality.
---

# Motion Composer

A skill for creating production-grade motion graphics, animations, and video compositions using code. This covers everything from micro-interactions to full Remotion video productions — the equivalent of having After Effects, GSAP Studio, and Motion Canvas in one skill.

---

## Decision Tree: Which Output Mode?

Before writing a single line, choose the right tool for the job:

```
User wants...

├── A looping web animation / hero section / micro-interaction
│   └── → MODE A: GSAP + HTML/CSS (or Framer Motion if React)

├── SVG path drawing, morphing, or icon animation
│   └── → MODE B: SVG Animation (GSAP DrawSVG / CSS stroke)

├── A programmatic video / MP4 export / data-driven video
│   └── → MODE C: Remotion Composition

├── An imperative scene with full timeline control (closest to AE)
│   └── → MODE D: Motion Canvas

├── Scroll-triggered storytelling / parallax / pinned sequences
│   └── → MODE E: GSAP ScrollTrigger

├── Particle systems / canvas physics / generative motion
│   └── → MODE F: Canvas / WebGL (Three.js / p5.js)

└── AI-generated video (Veo, Runway, Sora)
    └── → MODE G: API Composition (see references/ai-video-apis.md)
```

When ambiguous, default to MODE A for web and MODE C for "video".

---

## Core Animation Philosophy

Before coding, internalize these principles. Every frame is a decision.

### The 12 Animation Principles (applied to code)

1. **Squash & Stretch** — Scale transforms on impact/bounce. `scaleX(1.2) scaleY(0.8)`
2. **Anticipation** — Small reverse motion before the main action. Wind-up before launch.
3. **Staging** — One focal point per moment. Don't animate everything simultaneously.
4. **Straight Ahead / Pose-to-Pose** — For code: define keyframes (poses), let easing fill between.
5. **Follow Through** — Elements overshoot and settle. Use spring physics or custom bezier.
6. **Slow In / Slow Out** — Never use linear. Always use easing. Default: `power2.inOut`.
7. **Arcs** — Natural movement follows curves. Use motionPath for organic paths.
8. **Secondary Action** — Subtle supporting animation (shadow, reflection) adds realism.
9. **Timing** — Everything is about duration relationships. Fast=snappy, Slow=heavy/cinematic.
10. **Exaggeration** — Push it 20% further than feels right. Then pull back 10%.
11. **Solid Drawing** — In code: consistent transform origins, no jitter, clean positioning.
12. **Appeal** — Every animation should have a personality. Name it before you code it.

### Timing Reference Table

| Feel | Duration | Easing |
|------|----------|--------|
| Instant / snappy | 100–200ms | power3.out |
| UI / interactive | 200–350ms | power2.inOut |
| Page transition | 400–600ms | expo.out |
| Cinematic / dramatic | 800ms–1.5s | power1.inOut |
| Ambient / loop | 2s–8s | sine.inOut |
| Slow reveal | 1–2s | power2.out |

### Easing Cheat Sheet

```
Entrances:    power3.out, expo.out, back.out(1.7)
Exits:        power3.in, expo.in
Bouncy:       elastic.out(1, 0.3), back.out(2)
Smooth loop:  sine.inOut, power1.inOut
Spring feel:  Use GSAP springEasing or CSS spring()
Physics:      CustomEase.create("bounce", "M0,0 C0.14,...")
```

---

## MODE A: GSAP + HTML/CSS

Read: `references/gsap.md` for full API patterns.

### Quick Pattern — Staggered Reveal

```javascript
gsap.from(".card", {
  y: 60,
  opacity: 0,
  duration: 0.6,
  stagger: 0.12,
  ease: "power3.out",
  delay: 0.2
});
```

### Quick Pattern — Timeline Sequence

```javascript
const tl = gsap.timeline({ defaults: { ease: "power2.out" } });
tl.from(".hero-title", { y: -40, opacity: 0, duration: 0.8 })
  .from(".hero-sub", { y: 20, opacity: 0, duration: 0.6 }, "-=0.4")
  .from(".cta-btn", { scale: 0.8, opacity: 0, duration: 0.5 }, "-=0.3")
  .to(".hero-line", { scaleX: 1, duration: 1.2, ease: "expo.inOut" }, 0.2);
```

### Quick Pattern — Infinite Loop

```javascript
gsap.to(".floating-element", {
  y: -20,
  duration: 2,
  ease: "sine.inOut",
  yoyo: true,
  repeat: -1
});
```

### Critical GSAP Rules

- Always set `will-change: transform` on animated elements
- Use `gsap.set()` for initial states, not CSS (avoids flash)
- Prefer transform properties (`x`, `y`, `scale`, `rotation`) over layout props
- Use `gsap.context()` for React component cleanup
- Never animate `width`/`height` — use `scaleX`/`scaleY` instead
- Use `force3D: true` for hardware acceleration on critical animations

---

## MODE B: SVG Animation

Read: `references/framer-scrolltrigger-svg-canvas.md` for path morphing and draw patterns.

### Quick Pattern — Path Draw

```javascript
// GSAP DrawSVG
gsap.from(".path", {
  drawSVG: "0%",
  duration: 2,
  ease: "power2.inOut"
});

// CSS-only approach
.path {
  stroke-dasharray: 1000;
  stroke-dashoffset: 1000;
  animation: draw 2s ease forwards;
}
@keyframes draw {
  to { stroke-dashoffset: 0; }
}
```

---

## MODE C: Remotion Composition

Read: `references/remotion.md` for full composition patterns.

Remotion = React components that receive `frame` and `fps` as props. Everything is a function of time.

### Quick Pattern — Composition with interpolate

```tsx
import { useCurrentFrame, useVideoConfig, interpolate, spring } from 'remotion';

export const MyScene: React.FC = () => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  const scale = spring({
    frame: frame - 10,
    fps,
    config: { damping: 12, stiffness: 180, mass: 0.8 },
  });

  return (
    <div style={{ opacity, transform: `scale(${scale})` }}>
      Hello Motion
    </div>
  );
};
```

### Remotion Timing Math

```
frames = seconds × fps
// 30fps: 1s=30f, 2s=60f, 5s=150f, 10s=300f
// 60fps: 1s=60f, 2s=120f, 5s=300f
```

### Critical Remotion Rules

- Every frame must be deterministic — no `Math.random()` without seeded RNG
- Use `<Sequence>` to stagger scenes in time
- Use `<Series>` for back-to-back scenes
- Use `<Audio>` and `<Video>` for media — they auto-sync to frame time
- Use `staticFile()` for local assets

---

## MODE D: Motion Canvas

Read: `references/motion-canvas.md` for imperative scene patterns.

Motion Canvas is the closest to After Effects in code — imperative, generator-based, frame-perfect.

### Quick Pattern — Generator Scene

```typescript
export default makeScene2D(function* (view) {
  const box = createRef<Rect>();
  view.add(<Rect ref={box} width={200} height={200} fill="#6C63FF" opacity={0} />);

  yield* box().opacity(1, 0.5, easeInOutCubic);
  yield* box().rotation(360, 1.2);
  yield* all(
    box().scale(1.5, 0.6),
    box().fill('#FF6B6B', 0.6),
  );
  yield* waitFor(0.5);
  yield* box().opacity(0, 0.4);
});
```

---

## MODE E: GSAP ScrollTrigger

Read: `references/framer-scrolltrigger-svg-canvas.md` for advanced scroll patterns.

### Quick Pattern — Pinned Scroll Sequence

```javascript
const tl = gsap.timeline({
  scrollTrigger: {
    trigger: ".section",
    pin: true,
    scrub: 1,
    start: "top top",
    end: "+=2000",
  }
});
tl.from(".element-1", { x: -200, opacity: 0 })
  .from(".element-2", { y: 100, opacity: 0 }, "-=0.5");
```

---

## MODE F: Canvas / WebGL / Particles

Read: `references/framer-scrolltrigger-svg-canvas.md` for Three.js and p5.js patterns.

---

## MODE G: AI Video Generation APIs

Read: `references/ai-video-apis.md` for Veo, Runway, Sora prompt engineering.

### Prompt Engineering for Video AI

- Motion first: Lead with camera movement and subject motion before aesthetics
- Shot language: Use cinematography terms (dolly, pan, rack focus, handheld)
- Duration: Specify [5s], [10s] in prompt where supported

```
Example Veo 3 prompt:
"Slow dolly push into a glowing neon sign that reads 'NEXUS' [5s],
cinematic, rain-slicked street reflection, shallow DOF,
volumetric fog, 4K, photorealistic"
```

---

## Output Quality Checklist

Before delivering any animation output, verify:

- [ ] **Easing** — No linear unless intentional. Every tween has a meaningful curve.
- [ ] **Staging** — Not everything animates at once. Clear sequence or hierarchy.
- [ ] **Performance** — Only transform and opacity animated (not layout props).
- [ ] **Loop quality** — If looping, seamless start/end (no jump).
- [ ] **Responsive** — Uses %, vw/vh, or dynamic sizing, not hardcoded px.
- [ ] **Accessibility** — `prefers-reduced-motion` media query respected.
- [ ] **Initial state** — Elements set to their start state before animation runs (no flash).
- [ ] **Cleanup** — In React: animations killed on component unmount.

### Reduced Motion Pattern (always include)

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

```javascript
// GSAP version
if (!window.matchMedia("(prefers-reduced-motion: reduce)").matches) {
  // run animations
}
```

---

## Signature Patterns Library

### Kinetic Typography

```javascript
const chars = new SplitText(".headline", { type: "chars" });
gsap.from(chars.chars, {
  y: "100%",
  opacity: 0,
  rotationX: -90,
  stagger: 0.03,
  duration: 0.7,
  ease: "back.out(1.7)",
  transformOrigin: "50% 50% -20px"
});
```

### Magnetic Button

```javascript
btn.addEventListener('mousemove', (e) => {
  const { left, top, width, height } = btn.getBoundingClientRect();
  const x = (e.clientX - left - width / 2) * 0.3;
  const y = (e.clientY - top - height / 2) * 0.3;
  gsap.to(btn, { x, y, duration: 0.4, ease: "power2.out" });
});
btn.addEventListener('mouseleave', () => {
  gsap.to(btn, { x: 0, y: 0, duration: 0.7, ease: "elastic.out(1, 0.3)" });
});
```

### Counter Animation

```javascript
gsap.to(counter, {
  innerText: 2847,
  duration: 2,
  ease: "power2.out",
  snap: { innerText: 1 },
  onUpdate() { this.targets()[0].innerText = Math.round(this.targets()[0].innerText); }
});
```

### Page Transition Curtain

```javascript
const tl = gsap.timeline();
tl.to(".curtain", { scaleY: 1, duration: 0.5, ease: "power3.inOut", transformOrigin: "bottom" })
  .call(() => { /* navigate */ })
  .to(".curtain", { scaleY: 0, duration: 0.5, ease: "power3.inOut", transformOrigin: "top" });
```

### Scroll Reveal (Intersection Observer + GSAP)

```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      gsap.from(entry.target, {
        y: 50, opacity: 0, duration: 0.8, ease: "power3.out"
      });
      observer.unobserve(entry.target);
    }
  });
}, { threshold: 0.15 });
document.querySelectorAll('.reveal').forEach(el => observer.observe(el));
```

---

## When Building for Specific Products

### For Remotion video exports (product demos, App Store previews)

- Default: 1920×1080 at 30fps for web; 1080×1920 at 60fps for mobile/vertical
- App Store previews: max 30s, must show actual UI in first 3 seconds
- Always add `<AbsoluteFill style={{ background }}>` as scene wrapper

### For GSAP in web artifacts

- Load via CDN: `https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js`
- ScrollTrigger: `https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js`
- Register plugins: `gsap.registerPlugin(ScrollTrigger)`

### For Framer Motion (React artifacts)

- Import: `import { motion, AnimatePresence } from 'framer-motion'`
- Variants pattern for complex sequences (see `references/framer-scrolltrigger-svg-canvas.md`)

---

## Reference Files

Load these when you need deeper patterns for a specific mode:

- `references/gsap.md` — Full GSAP API, plugins, CustomEase, MotionPath, React integration
- `references/remotion.md` — Remotion project setup, Audio, Video, Sequence, Series, Player
- `references/motion-canvas.md` — Motion Canvas scene patterns, signals, camera, shaders
- `references/framer-scrolltrigger-svg-canvas.md` — Framer Motion, ScrollTrigger, SVG, Three.js, p5.js
- `references/ai-video-apis.md` — Veo 3, Runway Gen-4, Sora — prompt engineering + API calls
