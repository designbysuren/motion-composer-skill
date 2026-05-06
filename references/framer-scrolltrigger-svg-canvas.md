# Framer Motion, ScrollTrigger, SVG & Canvas Reference

## Framer Motion — Core Patterns

```tsx
import { motion, AnimatePresence, useScroll, useTransform, useSpring } from 'framer-motion';

// Basic animate
<motion.div animate={{ x: 100, opacity: 1 }} transition={{ duration: 0.5, ease: "easeOut" }} />

// from initial to animate
<motion.div initial={{ opacity: 0, y: 50 }} animate={{ opacity: 1, y: 0 }} />

// Variants pattern (preferred for sequences)
const containerVariants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: { staggerChildren: 0.1, delayChildren: 0.2 }
  }
};

const itemVariants = {
  hidden: { opacity: 0, y: 20 },
  visible: { opacity: 1, y: 0, transition: { type: "spring", stiffness: 300, damping: 20 } }
};

<motion.ul variants={containerVariants} initial="hidden" animate="visible">
  {items.map(item => (
    <motion.li key={item} variants={itemVariants}>{item}</motion.li>
  ))}
</motion.ul>
```

## AnimatePresence (exit animations)

```tsx
<AnimatePresence mode="wait">
  {isVisible && (
    <motion.div
      key="modal"
      initial={{ opacity: 0, scale: 0.9 }}
      animate={{ opacity: 1, scale: 1 }}
      exit={{ opacity: 0, scale: 0.9 }}
      transition={{ type: "spring", stiffness: 400, damping: 25 }}
    />
  )}
</AnimatePresence>
```

## Scroll-linked animations

```tsx
const { scrollYProgress } = useScroll({ target: ref });
const scale = useTransform(scrollYProgress, [0, 1], [1, 1.5]);
const opacity = useTransform(scrollYProgress, [0, 0.5, 1], [1, 1, 0]);
const smoothScale = useSpring(scale, { stiffness: 100, damping: 30 });

<motion.div style={{ scale: smoothScale, opacity }} />
```

## Layout Animations (magic: items rearrange smoothly)

```tsx
<motion.div layout layoutId="shared-element">
  {/* Animates when layout changes */}
</motion.div>

// Shared element transition between pages
// Page A:
<motion.img layoutId="hero-image" src="..." />
// Page B (same layoutId = transition):
<motion.img layoutId="hero-image" src="..." />
```

## Gesture Animations

```tsx
<motion.button
  whileHover={{ scale: 1.05, boxShadow: "0 10px 30px rgba(0,0,0,0.2)" }}
  whileTap={{ scale: 0.95 }}
  drag
  dragConstraints={{ left: -100, right: 100, top: -100, bottom: 100 }}
  dragElastic={0.1}
/>
```

---

## GSAP ScrollTrigger Reference

### Basic Setup

```javascript
gsap.registerPlugin(ScrollTrigger);

gsap.to(".element", {
  scrollTrigger: {
    trigger: ".element",
    start: "top 80%",    // [element edge] [viewport edge]
    end: "bottom 20%",
    scrub: false,        // true = linked to scroll, number = lag seconds
    pin: false,
    markers: false,      // debug - set true during development
    toggleClass: "active",
    onEnter: () => {},
    onLeave: () => {},
    onEnterBack: () => {},
    onLeaveBack: () => {},
  },
  y: -100,
  opacity: 1,
});
```

### Pinned Scroll Sequence (AE-style scrubbing)

```javascript
const tl = gsap.timeline({
  scrollTrigger: {
    trigger: ".sequence-section",
    pin: true,
    scrub: 1,
    start: "top top",
    end: "+=3000",
    anticipatePin: 1,
  }
});

tl.from(".text-1", { xPercent: -100, opacity: 0, duration: 1 })
  .from(".text-2", { yPercent: 100, opacity: 0, duration: 1 }, "+=0.5")
  .to(".background", { scale: 1.2, duration: 2 }, "<")
  .to(".text-1", { opacity: 0, xPercent: 100, duration: 0.5 });
```

### Batch (performance-optimized reveal)

```javascript
ScrollTrigger.batch(".card", {
  onEnter: (elements) => {
    gsap.from(elements, {
      opacity: 0, y: 60, stagger: 0.15, duration: 0.8, ease: "power3.out",
    });
  },
  start: "top 85%",
  once: true,
});
```

### Horizontal Scroll

```javascript
const sections = gsap.utils.toArray(".panel");

gsap.to(sections, {
  xPercent: -100 * (sections.length - 1),
  ease: "none",
  scrollTrigger: {
    trigger: ".container",
    pin: true,
    scrub: 1,
    snap: 1 / (sections.length - 1),
    end: () => "+=" + document.querySelector(".container").offsetWidth,
  }
});
```

---

## SVG Animation Reference

### CSS Stroke Draw

```css
.path {
  stroke-dasharray: var(--path-length, 1000);
  stroke-dashoffset: var(--path-length, 1000);
  animation: draw 2s cubic-bezier(0.4, 0, 0.2, 1) forwards;
}

@keyframes draw {
  to { stroke-dashoffset: 0; }
}
```

```javascript
// Measure path length dynamically
const paths = document.querySelectorAll('path, polyline, circle, rect');
paths.forEach(p => {
  const len = p.getTotalLength();
  p.style.setProperty('--path-length', len);
});
```

### GSAP DrawSVG

```javascript
gsap.from(".path", { drawSVG: "0%", duration: 2, ease: "power2.inOut" });
gsap.to(".path", { drawSVG: "50% 80%", duration: 1 }); // draw a segment
```

### SVG Morphing

```javascript
// Shapes must have same number of points for smooth morph
gsap.to("#shape-a", {
  morphSVG: "#shape-b",
  duration: 1.5,
  ease: "power2.inOut",
  yoyo: true,
  repeat: -1,
});
```

### CSS SVG Filters (blur, glow, distortion)

```html
<svg style="position:absolute;width:0;height:0">
  <defs>
    <filter id="glow">
      <feGaussianBlur stdDeviation="4" result="blur"/>
      <feMerge><feMergeNode in="blur"/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
    <filter id="distort">
      <feTurbulence type="fractalNoise" baseFrequency="0.015" numOctaves="2"/>
      <feDisplacementMap in="SourceGraphic" scale="30"/>
    </filter>
  </defs>
</svg>
<div style="filter: url(#glow)">Glowing element</div>
```

---

## Canvas & WebGL Reference

### Canvas 2D Animation Loop

```javascript
const canvas = document.querySelector('canvas');
const ctx = canvas.getContext('2d');
let frame = 0;

function resize() {
  canvas.width = window.innerWidth * devicePixelRatio;
  canvas.height = window.innerHeight * devicePixelRatio;
  ctx.scale(devicePixelRatio, devicePixelRatio);
}
window.addEventListener('resize', resize);
resize();

function tick() {
  requestAnimationFrame(tick);
  frame++;
  
  // Clear (or use trail effect with low-alpha fill)
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  
  // Trail effect
  ctx.fillStyle = 'rgba(0, 0, 0, 0.05)';
  ctx.fillRect(0, 0, canvas.width, canvas.height);
  
  draw(frame);
}
tick();
```

### Three.js Scene Template

```javascript
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, innerWidth/innerHeight, 0.1, 1000);
camera.position.z = 5;

const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
renderer.setPixelRatio(devicePixelRatio);
renderer.setSize(innerWidth, innerHeight);
document.body.appendChild(renderer.domElement);

// Geometry
const geo = new THREE.TorusKnotGeometry(1, 0.3, 100, 16);
const mat = new THREE.MeshStandardMaterial({ color: 0x6C63FF, metalness: 0.8, roughness: 0.2 });
const mesh = new THREE.Mesh(geo, mat);
scene.add(mesh);

// Lighting
scene.add(new THREE.AmbientLight(0xffffff, 0.3));
const point = new THREE.PointLight(0xffffff, 2);
point.position.set(5, 5, 5);
scene.add(point);

// Resize
window.addEventListener('resize', () => {
  camera.aspect = innerWidth / innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(innerWidth, innerHeight);
});

// Animate
(function tick() {
  requestAnimationFrame(tick);
  mesh.rotation.x += 0.005;
  mesh.rotation.y += 0.01;
  renderer.render(scene, camera);
})();
```

### p5.js Quick Template

```javascript
new p5((p) => {
  p.setup = () => {
    p.createCanvas(p.windowWidth, p.windowHeight);
    p.colorMode(p.HSB, 360, 100, 100, 100);
  };
  
  p.draw = () => {
    p.background(0, 0, 0, 10); // trail
    p.translate(p.width/2, p.height/2);
    
    for (let i = 0; i < 100; i++) {
      const angle = p.frameCount * 0.02 + i * p.TWO_PI / 100;
      const r = 200 + p.sin(p.frameCount * 0.03 + i) * 50;
      const x = p.cos(angle) * r;
      const y = p.sin(angle) * r;
      
      p.fill(p.frameCount % 360, 80, 100);
      p.noStroke();
      p.circle(x, y, 5);
    }
  };
});
```

### Particle System (Canvas 2D)

```javascript
class Particle {
  constructor(x, y) {
    this.x = x; this.y = y;
    this.vx = (Math.random() - 0.5) * 2;
    this.vy = (Math.random() - 0.5) * 2;
    this.life = 1;
    this.decay = 0.02;
    this.hue = Math.random() * 360;
  }
  update() {
    this.x += this.vx;
    this.y += this.vy;
    this.life -= this.decay;
    this.vy += 0.05; // gravity
  }
  draw(ctx) {
    ctx.globalAlpha = this.life;
    ctx.fillStyle = `hsl(${this.hue}, 80%, 60%)`;
    ctx.beginPath();
    ctx.arc(this.x, this.y, 3, 0, Math.PI * 2);
    ctx.fill();
  }
  isDead() { return this.life <= 0; }
}

const particles = [];
canvas.addEventListener('mousemove', (e) => {
  for (let i = 0; i < 5; i++) {
    particles.push(new Particle(e.clientX, e.clientY));
  }
});

function tick() {
  requestAnimationFrame(tick);
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  particles.forEach(p => { p.update(); p.draw(ctx); });
  particles.filter(p => !p.isDead());
}
tick();
```
