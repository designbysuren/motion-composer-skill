# AI Video Generation APIs — Veo, Runway, Sora

## Prompt Engineering Principles

### The MACS Framework for AI Video Prompts

- Motion: What moves? How? Camera + subject simultaneously.
- Aesthetic: Visual style, lighting, color grade, lens
- Context: Scene, environment, time of day, atmosphere
- Specs: Duration, aspect ratio, quality level

### Prompt Template

[CAMERA MOVEMENT] of [SUBJECT] [ACTION], [ENVIRONMENT],
[LIGHTING], [VISUAL STYLE], [MOOD/ATMOSPHERE],
[QUALITY TAGS], [DURATION]

### Camera Vocabulary

Movement: slow dolly in, dolly out, pan left/right, tilt up/down,
          crane shot, aerial descent, orbit (360 deg), handheld,
          steadicam, whip pan, rack focus, pull focus, zoom in

Distance: extreme close-up (ECU), close-up (CU), medium shot (MS),
          wide shot (WS), extreme wide shot (EWS), bird's eye, worm's eye

---

## Google Veo 3 (via Vertex AI)

### API Setup

```python
import google.generativeai as genai
import time

client = genai.Client(api_key="YOUR_KEY")

operation = client.models.generate_video(
    model="veo-3.0-generate-preview",
    prompt="Slow dolly push into a glowing neon sign reading 'NEXUS'...",
    config={
        "duration_seconds": 8,
        "aspect_ratio": "16:9",  # or "9:16" for vertical
        "resolution": "1080p",
        "number_of_videos": 1,
    }
)

# Poll for completion
while not operation.done:
    time.sleep(5)
    operation = operation.refresh()

video_url = operation.result.generated_videos[0].video.uri
```

### Veo 3 Prompt Examples

Product Launch:
```
Slow cinematic dolly push into a sleek smartphone floating in void,
metallic surface catching volumetric light rays, deep space background
with subtle particle field, ultra-sharp product photography aesthetic,
color grade: desaturated with cyan-gold split tone, [8s]
```

App Preview (Mobile):
```
Screen recording style animation of a mobile app interface,
clean hands scrolling through a beautifully designed app,
bright studio lighting on white background, shallow DOF on phone,
modern tech commercial aesthetic, vertical 9:16 format, [15s]
```

Brand Identity:
```
Abstract logo reveal: geometric shapes assembling from particles
into a clean wordmark, dark background, metallic sheen,
cinematic lens flare at assembly completion, [5s]
```

Kinetic Typography:
```
3D text "NEXUS" assembling letter by letter from shattered glass pieces,
dark background, dramatic rim lighting in electric blue,
slow motion particle debris, photorealistic, ultra-HD [8s]
```

---

## Runway Gen-4 API

### API Setup

```python
import requests
import time

RUNWAY_API_KEY = "your_key_here"

headers = {
    "Authorization": f"Bearer {RUNWAY_API_KEY}",
    "Content-Type": "application/json",
    "X-Runway-Version": "2024-11-06"
}

# Text-to-video
response = requests.post(
    "https://api.runwayml.com/v1/image_to_video",
    headers=headers,
    json={
        "model": "gen4_turbo",
        "promptText": "Your prompt here",
        "ratio": "1280:768",  # or "768:1280" for vertical
        "duration": 10,       # 5 or 10 seconds
        "seed": 42,           # for reproducibility
    }
)

task_id = response.json()["id"]

# Poll for completion
while True:
    status = requests.get(
        f"https://api.runwayml.com/v1/tasks/{task_id}",
        headers=headers
    )
    data = status.json()
    if data["status"] == "SUCCEEDED":
        video_url = data["output"][0]
        break
    elif data["status"] == "FAILED":
        raise Exception(f"Generation failed: {data.get('error')}")
    time.sleep(3)
```

### Runway Prompt Patterns

Negative Prompts (always include for quality):
```python
"negativePrompt": "low quality, blurry, distorted, watermark, text overlay, duplicate, mutation, deformed, ugly, bad anatomy"
```

Motion Control:
- Camera: [pan left] [slow zoom] [static]
- Subject motion: [person walks forward] [logo rotates] [particles explode outward]

---

## OpenAI Sora API

### API Setup

```python
from openai import OpenAI

client = OpenAI(api_key="YOUR_KEY")

response = client.video.generations.create(
    model="sora-1.0-turbo",
    prompt="Your prompt here",
    size="1920x1080",    # or "1080x1920", "1080x1080"
    duration=10,          # 5, 10, 15, 20 seconds
    n=1,
)

video_url = response.data[0].url
```

---

## Prompt Library — Ready to Use

SaaS Product Demo:
```
Overhead flat-lay of a MacBook Pro displaying a clean SaaS dashboard,
hands typing smoothly, coffee cup beside it, warm morning light,
shot on large format, slight film grain, teal and white color palette,
professional commercial aesthetic [10s]
```

Mobile App Showcase:
```
Close-up of iPhone in hand, thumb scrolling through a beautifully
designed app with smooth animations, bokeh background of modern office,
natural window lighting from left, color grade: warm and inviting,
feels like an Apple commercial [8s]
```

Abstract Brand Reveal:
```
Aerial slow rotation above abstract geometric landscape, shapes emerging
from fog, warm golden hour light breaking through, cinematic scope,
Terrence Malick aesthetic, 4K, [10s]
```

Looping Background (for web use):
```
Abstract fluid simulation, deep navy and electric teal colors,
seamlessly looping, slow organic movement, no hard cuts,
suitable for use as website hero background [8s loop]
```

---

## Choosing the Right Model

| Need | Best Model |
|------|-----------|
| Photorealistic humans | Veo 3 |
| Fast iteration / turnaround | Runway Gen-4 Turbo |
| Long duration (20s+) | Sora |
| Text rendering in video | Veo 3 |
| Creative/artistic | Runway |
| Image-to-video | Runway or Veo 3 |
| Precise motion control | Veo 3 |

---

## Pro Tips

### Prompt Engineering Best Practices

1. Lead with camera movement: "Slow dolly push into..." before describing the subject
2. Be specific with duration: Add [8s] or [10s] to control clip length
3. Name cinematographers/directors: "Terrence Malick aesthetic", "Roger Deakins lighting"
4. Color grade explicitly: "teal and orange color grade", "desaturated with warm highlights"
5. Specify lens: "shallow DOF", "anamorphic lens flare", "wide angle"
6. Use film terms: "dolly", "rack focus", "crane", "handheld"

### Common Mistakes to Avoid

- Starting with style instead of motion
- Too many subjects in one prompt
- Forgetting to specify aspect ratio for vertical content
- Not using negative prompts with Runway
- Prompting for fast cuts (these are single clips)

### For Brand/Product Videos

Always include:
- Specific brand color palette (hex or color name)
- Lighting reference (natural, studio, golden hour, etc.)
- Camera proximity (close-up for products, wide for lifestyle)
- Mood/energy (professional, energetic, calm, luxury)
