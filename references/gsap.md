# GSAP Reference — Full API Patterns

## Plugin Registration

```javascript
gsap.registerPlugin(ScrollTrigger, SplitText, MorphSVG, DrawSVG, MotionPathPlugin, CustomEase, Flip);
```

## Core Methods

### gsap.to / from / fromTo

```javascript
gsap.to(target, { x: 100, duration: 1, ease: "power2.out" });
gsap.from(target, { opacity: 0, y: 50, duration: 0.8 });
gsap.fromTo(target, { x: -200 }, { x: 0, duration: 1 });
gsap.set(target, { opacity: 0 }); // instant, no animation
```

### Timeline

```javascript
const tl = gsap.timeline({
  defaults: { ease: "power2.out", duration: 0.6 },
  paused: true,
  repeat: -1,
  yoyo: true,
  repeatDelay: 1,
  onComplete: () => {},
  onUpdate: () => {},
});

// Position parameter
tl.to(a, {})            // after previous
  .to(b, {}, "+=0.2")  // 0.2s after previous ends
  .to(c, {}, "-=0.3")  // 0.3s before previous ends
  .to(d, {}, "<")       // same start as previous
  .to(e, {}, "<0.1")   // 0.1s after previous starts
  .to(f, {}, 2)         // absolute: 2s into timeline
  .to(g, {}, "label")   // at a named label
  .addLabel("myLabel", 1.5);
```

### Stagger

```javascript
gsap.from(".items", {
  opacity: 0, y: 30,
  stagger: {
    amount: 0.8,
    from: "start",   // "start" | "end" | "center" | "random" | index
    ease: "power1",
    grid: "auto",
    axis: "y",
  }
});
```

## Properties Reference

### Transform (GPU accelerated — always prefer these)

```javascript
{ x: 100 }          // translateX in px
{ y: -50 }          // translateY in px
{ xPercent: -50 }   // translateX in %
{ yPercent: -50 }   // translateY in %
{ scale: 1.5 }      // uniform scale
{ scaleX: 1.2 }     // x scale only
{ scaleY: 0.8 }     // y scale only
{ rotation: 360 }   // degrees
{ rotationX: 45 }   // 3D rotation
{ rotationY: 45 }   // 3D rotation
{ skewX: 10 }       // skew
{ transformOrigin: "center bottom" }
```

### Appearance

```javascript
{ opacity: 0 }
{ autoAlpha: 0 }    // opacity + visibility (avoids ghost clicks)
{ color: "#fff" }
{ backgroundColor: "#000" }
{ borderRadius: "50%" }
```

### Special

```javascript
{ attr: { cx: 50, cy: 100 } }  // SVG attributes
{ css: { filter: "blur(10px)" } }
```

## CustomEase

```javascript
CustomEase.create("myEase", "M0,0 C0.126,0.382 0.282,0.674 0.44,0.822 0.632,1.002 0.818,1.001 1,1");
gsap.to(el, { x: 200, ease: "myEase" });
```

## MotionPath

```javascript
gsap.to(el, {
  motionPath: {
    path: "#svgPath",
    align: "#svgPath",
    alignOrigin: [0.5, 0.5],
    autoRotate: true,
  },
  duration: 3,
  ease: "none",
  repeat: -1,
});
```

## Flip Plugin (layout animation)

```javascript
const state = Flip.getState(".cards");
container.appendChild(newCard);
Flip.from(state, {
  duration: 0.6,
  ease: "power2.inOut",
  stagger: 0.05,
  absolute: true,
});
```

## Utility Methods

```javascript
gsap.utils.clamp(0, 100, value);
gsap.utils.mapRange(0, 1, 0, 100, 0.5);  // → 50
gsap.utils.normalize(0, 100, 50);         // → 0.5
gsap.utils.interpolate("#ff0000", "#0000ff", 0.5);
gsap.utils.toArray(".items");
gsap.utils.shuffle(array);
gsap.utils.random(0, 100);
gsap.utils.snap(5, 14);  // → 15 (snaps to nearest 5)
```

## React Integration

```javascript
import { useGSAP } from "@gsap/react";
import { useRef } from "react";

function Component() {
  const container = useRef(null);

  useGSAP(() => {
    gsap.from(".box", { opacity: 0, y: 50 });
  }, { scope: container });

  return <div ref={container}><div className="box" /></div>;
}
```

## SplitText (Kinetic Typography)

```javascript
const split = new SplitText(".headline", { type: "chars,words,lines" });

gsap.from(split.chars, {
  y: "100%",
  opacity: 0,
  rotationX: -90,
  stagger: 0.03,
  duration: 0.7,
  ease: "back.out(1.7)",
  transformOrigin: "50% 50% -20px"
});
```

## Common Patterns

### Magnetic Button

```javascript
const btn = document.querySelector('.magnetic-btn');
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
const obj = { val: 0 };
gsap.to(obj, {
  val: 2847,
  duration: 2,
  ease: "power2.out",
  onUpdate() {
    document.querySelector('.counter').innerText = Math.round(obj.val).toLocaleString();
  }
});
```

### Page Transition Curtain

```javascript
const tl = gsap.timeline();
tl.to(".curtain", { scaleY: 1, duration: 0.5, ease: "power3.inOut", transformOrigin: "bottom" })
  .call(() => { router.push('/next-page'); })
  .to(".curtain", { scaleY: 0, duration: 0.5, ease: "power3.inOut", transformOrigin: "top" });
```

### Infinite Marquee

```javascript
const items = gsap.utils.toArray('.marquee-item');
const totalWidth = items.reduce((w, el) => w + el.offsetWidth, 0);
gsap.to('.marquee-track', {
  x: -totalWidth,
  duration: 20,
  ease: "none",
  repeat: -1,
  modifiers: {
    x: gsap.utils.unitize(x => parseFloat(x) % totalWidth)
  }
});
```

## CDN Imports for Artifacts

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/TextPlugin.min.js"></script>
<!-- Premium (Club GSAP): SplitText, MorphSVGPlugin, DrawSVGPlugin, MotionPathPlugin -->
```

## Performance Best Practices

- Animate only `transform` and `opacity` — compositor-only properties
- Use `gsap.set()` to set initial states (prevents flash of unstyled content)
- Use `will-change: transform` on heavily animated elements
- Kill animations on component unmount: `return () => ctx.revert()` in React
- For lists of 50+ items, combine with virtualization
