<div align="center">

![motion-composer](motion-composer.gif)

# motion-composer-skill

**A Claude skill for production-grade motion graphics, animations, and video compositions using code.**

[![License: MIT](https://img.shields.io/badge/License-MIT-orange.svg)](LICENSE)
[![Claude Skill](https://img.shields.io/badge/Claude-Skill-FF6B00?logo=anthropic)](https://github.com/designbysuren/motion-composer-skill)
[![Tools](https://img.shields.io/badge/Tools-GSAP%20%7C%20Remotion%20%7C%20Motion%20Canvas%20%7C%20ScrollTrigger%20%7C%20Veo%203-black)](SKILL.md)

</div>

---

## What This Skill Does

Encodes expert-level animation knowledge so Claude can:

- Choose the right framework for every animation request (GSAP, Remotion, Motion Canvas, Framer Motion, or AI video)
- Apply the 12 Animation Principles translated into code patterns
- Generate production-ready animations with correct easing, timing, and performance
- Build Remotion video compositions for product demos and App Store previews
- Write ScrollTrigger scroll sequences, SVG path animations, and Three.js scenes
- Compose AI video prompts for Veo 3, Runway Gen-4, and Sora

---

## Install

**Claude Code (recommended):**

```bash
git clone https://github.com/designbysuren/motion-composer-skill ~/.claude/skills/motion-composer-skill
```

Claude will automatically detect and use the skill when relevant.

**Manual:** Copy `SKILL.md` into any `.claude/skills/` directory in your project.

---

## Usage

Once installed, reference the skill in any animation prompt:

```
Build a scroll-triggered reveal animation for a landing page hero section.
Use the motion-composer skill.
```

```
Create a Remotion composition for a 30-second product demo video at 1920x1080.
Use the motion-composer skill.
```

```
Write a Veo 3 prompt and API call to generate a cinematic product launch video.
Use the motion-composer skill.
```

---

## Decision Tree

| User wants... | Tool |
|---|---|
| Looping web animation / micro-interaction | GSAP + HTML/CSS |
| SVG path drawing or morphing | GSAP DrawSVG / CSS stroke |
| Programmatic video / MP4 export | Remotion |
| After Effects-style scene (imperative) | Motion Canvas |
| Scroll-triggered storytelling / parallax | GSAP ScrollTrigger |
| Particle systems / generative motion | Canvas / Three.js / p5.js |
| AI-generated video | Veo 3 / Runway / Sora API |

---

## Timing Reference

| Feel | Duration | Easing |
|---|---|---|
| Snappy | 100-200ms | power3.out |
| UI / interactive | 200-350ms | power2.inOut |
| Page transition | 400-600ms | expo.out |
| Cinematic | 800ms-1.5s | power1.inOut |
| Ambient loop | 2s-8s | sine.inOut |

---

## Repo Structure

```
motion-composer-skill/
├── SKILL.md                          <- the brain Claude reads
├── references/
│   ├── gsap.md                       <- Full GSAP API, plugins, patterns
│   ├── remotion.md                   <- Remotion compositions, hooks, Player
│   ├── motion-canvas.md              <- Motion Canvas scenes, signals, camera
│   ├── framer-scrolltrigger-svg-canvas.md  <- Framer, ScrollTrigger, SVG, Three.js, p5
│   └── ai-video-apis.md              <- Veo 3, Runway Gen-4, Sora API calls
├── motion-composer.gif               <- animated banner
├── README.md
└── LICENSE
```

---

## Using with Other Agents

**Cursor** — add to `.cursorrules`:
```
For any animation, motion graphics, or video composition requests,
apply the motion-composer skill. Reference SKILL.md for the decision
tree and patterns.
```

**Codex CLI / Opencode** — add to `AGENTS.md`:
```markdown
## Animation & Motion
For animations, motion graphics, videos, GSAP, Remotion, or ScrollTrigger
requests, apply the rules in SKILL.md (motion-composer skill).
```

**ChatGPT / API** — paste `SKILL.md` contents into the system prompt.

---

<div align="center">
<sub>Built with love by <a href="https://github.com/designbysuren">designbysuren</a> · MIT License</sub>
</div>
