# GitHub Copilot Instructions

This file provides guidance to GitHub Copilot when working with code in this repository.

## Overview

This is a Remotion video project designed to make it easier to create videos programmatically with GitHub Copilot. The project uses React, TypeScript, and Tailwind CSS.

## Package Manager

**Always use `pnpm` for all commands.** Do not use `npm` or `npx`.

## Common Commands

```bash
# Install dependencies
pnpm i

# Start development preview
pnpm run dev

# Lint code
pnpm run lint

# Render a composition (by ID)
pnpm exec remotion render Example1-Landscape

# Render still image
pnpm exec remotion still Example1-Landscape

# Upgrade Remotion
pnpm run upgrade
```

## Architecture

### Project Structure

```
src/
├── components/              # Reusable components (black/white theme, className override)
│   ├── TitleSlide.tsx       # Full-screen title
│   ├── ContentSlide.tsx     # Header + body text
│   ├── CodeSlide.tsx        # Code with title
│   ├── DiagramSlide.tsx     # Mermaid/D2 diagrams
│   ├── VideoSlide.tsx       # Video playback
│   ├── BRollVideo.tsx       # B-roll with zoom
│   ├── ZoomableVideo.tsx    # Video with zoom segments
│   ├── Screenshot.tsx       # Scrolling screenshot
│   ├── Logo.tsx             # Logo overlay (animated)
│   ├── Caption.tsx          # Subtitle/caption overlay
│   ├── AsciiPlayer.tsx      # Terminal recording playback
│   ├── Code.tsx             # Syntax-highlighted code
│   ├── Diagram.tsx          # Diagram renderer
│   └── Music.tsx            # Background music with fade
├── compositions/
│   ├── example1/            # Reference: basic slideshow
│   └── example2/            # Reference: multi-feature demo
├── utils/
│   ├── createComposition.tsx  # Helper to create compositions
│   └── segmentTranscript.ts   # Parse transcripts for timing
├── config.ts                # Timing utilities: secondsToFrames(), framesToSeconds()
├── presets.ts               # VIDEO_PRESETS (aspect ratios, 60fps)
├── content.ts               # Sample content for component previews
└── Root.tsx                 # Composition registry
```

### Quick Start

1. Ask Copilot: "Create a new composition called my-video" — see [New Composition](#new-composition) workflow below
2. Edit `src/compositions/my-video/content.ts` — change the text
3. Edit `src/compositions/my-video/config.ts` — adjust timing (in seconds)
4. Run `pnpm exec remotion render MyVideo` — render your video

### Key Concepts

**Components** have black/white defaults with `className` prop for theming:

```tsx
// Default black background, white text
<TitleSlide title="Hello" />

// Custom theme via Tailwind classes
<TitleSlide title="Hello" className="bg-blue-900 text-yellow-300" />
```

**Transitions** use Remotion's built-in `<TransitionSeries>`, not component props:

```tsx
// GOOD: Use Remotion's TransitionSeries for fades
<TransitionSeries>
  <TransitionSeries.Sequence durationInFrames={180}>
    <TitleSlide title="Hello" />
  </TransitionSeries.Sequence>
  <TransitionSeries.Transition
    presentation={fade()}
    timing={linearTiming({ durationInFrames: 30 })}
  />
  <TransitionSeries.Sequence durationInFrames={300}>
    <ContentSlide header="Main" content="..." />
  </TransitionSeries.Sequence>
</TransitionSeries>
```

**Timing** uses `<Sequence>` for positioning, not component props:

```tsx
import { secondsToFrames } from "../../config";

// GOOD: Position with Sequence (secondsToFrames defaults to 60fps)
<Sequence from={secondsToFrames(5.2)} durationInFrames={secondsToFrames(2.7)}>
  <Logo src="logo.svg" />
</Sequence>
```

### Timing Utilities

All utilities default to **60fps** and can be used anywhere (components, config files, etc.):

```tsx
import { secondsToFrames, framesToSeconds } from "../../config";

// Convert seconds to frames (defaults to 60fps)
secondsToFrames(2.5)        // => 150 frames
secondsToFrames(2.5, 30)    // => 75 frames (custom fps)

// Convert frames to seconds
framesToSeconds(150)        // => 2.5 seconds
framesToSeconds(150, 30)    // => 5 seconds (custom fps)
```

Use these for transcript-based timing:

```tsx
// In config.ts - define segment timing from transcript
export const SEGMENTS = {
  intro: { start: 0, end: 3.2 },
  feature: { start: 3.2, end: 8.5 },
};

// In Composition.tsx
<Sequence from={secondsToFrames(SEGMENTS.intro.start)}>
  <IntroSegment />
</Sequence>
```

### Composition Pattern

Use the `createComposition` helper:

```typescript
import { createComposition } from "../../utils/createComposition";

const MyVideoComposition: React.FC = () => {
  // ... video content
};

export const MyVideo = createComposition({
  name: "MyVideo",
  component: MyVideoComposition,
  durationInSeconds: 10,
  preset: "Landscape-1080p",
});
```

### Root.tsx Structure

- `Examples` folder with reference compositions
- `Components` folder with previews for each component
- Ask Copilot to scaffold new compositions using the [New Composition](#new-composition) workflow

### Video Presets

All videos run at **60fps**. Available in `src/presets.ts`:

- `Landscape-720p`: 1280x720 @ 60fps
- `Landscape-1080p`: 1920x1080 @ 60fps
- `Square-1080p`: 1080x1080 @ 60fps
- `Portrait-1080p`: 1080x1920 @ 60fps

### Styling

- Use Tailwind CSS classes
- Default theme: `bg-black text-white`
- Override via `className` prop on most components
- Code uses `github-dark` theme by default
- AsciiPlayer uses `nord` theme by default

### Preloading Assets

For seamless playback in the Studio, prefetch audio/video assets. Use `staticFile()` for local files in `public/`:

```tsx
import { prefetch, staticFile } from "remotion";

// Prefetch at module level (outside component)
prefetch(staticFile("audio/music.mp3"));
prefetch(staticFile("audio/voiceover.mp3"));

const MyComposition: React.FC = () => {
  // <Audio> automatically uses prefetched blob URL
  return <Audio src={staticFile("audio/music.mp3")} />;
};
```

### Remotion Context

Reference `#file:context/remotion.md` for detailed Remotion patterns and APIs.
Reference `#file:context/remotion-video.md` for details around embedding videos.
Reference `#file:context/remotion-audio.md` for details around embedding audio.

Use the `remotion-documentation` MCP tool for specific questions.

## MCP Tools

### Playwright

- Visit websites and take screenshots
- Debug compositions in browser
- Capture reference images

### ElevenLabs

- **Text-to-speech**: Generate voiceovers
- **Voice library**: Search and use voices
- **Sound effects**: Generate from text descriptions
- **Music generation**: Create background music

### Replicate

- **Veo 3.1**: Generate videos with audio
- **Nano Banana Pro**: Generate/edit images

## Component Reference

| Component     | Props                                                                                | Notes                 |
| ------------- | ------------------------------------------------------------------------------------ | --------------------- |
| TitleSlide    | `title`, `className?`                                                                | Full-screen title     |
| ContentSlide  | `header`, `content`, `className?`                                                    | Header + body         |
| CodeSlide     | `title?`, `code`, `language`, `highlightLines?`, `animatedHighlights?`, `className?` | Code with title       |
| DiagramSlide  | `title?`, `type`, `diagram`, `theme?`, `sketch?`, `className?`                       | Mermaid/D2            |
| VideoSlide    | `filename`, `startTime?`                                                             | Video playback        |
| BRollVideo    | `filename`, `startTime?`, `endTime?`, `zoomStart?`, `zoomEnd?`, `playbackRate?`      | B-roll with zoom      |
| ZoomableVideo | `src`, `zoomSegments`                                                                | Multiple zoom regions |
| Screenshot    | `src`, `scrollSpeed?`, `scrollDelaySeconds?`                                         | Scrolling screenshot  |
| Logo          | `src`, `alt?`, `position?`, `size?`                                                  | Animated logo overlay |
| Caption       | `transcript`, `className?`                                                           | Subtitle overlay      |
| AsciiPlayer   | `mode`, `castFile`, `playbackSpeed?`, `startTime?`, `theme?`                         | Terminal recording    |
| Code          | `code`, `language`, `highlightLines?`, `animatedHighlights?`, `theme?`               | Syntax highlighting   |
| Diagram       | `type`, `diagram`, `theme?`, `sketch?`                                               | Render diagrams       |
| Music         | `src`, `volume?`, `fadeInSeconds?`, `fadeOutSeconds?`, `loop?`                       | Background audio      |

## Workflow Examples

### Basic Video Creation

1. Ask Copilot: "Create a new composition called my-video"
2. Edit `src/compositions/my-video/content.ts` with your text
3. Edit `src/compositions/my-video/config.ts` for timing
4. Add/modify segments as needed
5. Render: `pnpm exec remotion render MyVideo`

### AI-Generated Assets

1. Ask Copilot: "Generate an image of a futuristic city" — see [Generate Image](#generate-image) workflow
2. Ask Copilot: "Generate a video of a camera flying through clouds" — see [Generate Video](#generate-video) workflow
3. Use ElevenLabs MCP tools for voiceovers
4. Combine assets in Remotion compositions

### Timed Content from Transcripts

1. Ask Copilot: "Transcribe public/video.mp4" — see [Transcribe](#transcribe) workflow
2. Parse `results.channels[0].alternatives[0].words`
3. Create segments timed to the transcript
4. Use `<Sequence from={...}>` for precise positioning

---

- Don't run the dev server unless explicitly asked

---

## Workflow Recipes

These are detailed task descriptions for common operations. Invoke them by describing what you want in Copilot Chat (e.g. "generate a video of X", "take a screenshot of Y", "create a new composition called Z").

---

### New Composition

**Trigger:** User asks to create a new composition, e.g. "create a new composition called my-video"

Create a new Remotion video composition with the given name.

#### Steps

1. **Validate the name**:
   - Convert to lowercase for folder name (e.g., "My Video" → "my-video")
   - Use PascalCase for component names (e.g., "my-video" → "MyVideo")

2. **Create the folder structure**:
   ```
   src/compositions/{folder-name}/
   ├── Composition.tsx
   ├── config.ts
   ├── content.ts
   └── segments/
       ├── TitleSegment.tsx
       └── ContentSegment.tsx
   ```

3. **Generate files** using these templates:

   **config.ts**
   ```typescript
   // Segment durations in seconds
   export const TITLE_DURATION_SECONDS = 3;
   export const CONTENT_DURATION_SECONDS = 5;
   ```

   **content.ts**
   ```typescript
   export const CONTENT = {
     title: "{Composition Name}",
     contentHeader: "Your Header",
     contentBody: "Your content goes here.",
   };
   ```

   **segments/TitleSegment.tsx**
   ```typescript
   import { TitleSlide } from "../../../components/TitleSlide";
   import { CONTENT } from "../content";

   export const TitleSegment: React.FC = () => {
     return <TitleSlide title={CONTENT.title} />;
   };
   ```

   **segments/ContentSegment.tsx**
   ```typescript
   import { ContentSlide } from "../../../components/ContentSlide";
   import { CONTENT } from "../content";

   export const ContentSegment: React.FC = () => {
     return (
       <ContentSlide header={CONTENT.contentHeader} content={CONTENT.contentBody} />
     );
   };
   ```

   **Composition.tsx**
   ```typescript
   import { Series, useVideoConfig } from "remotion";
   import { TitleSegment } from "./segments/TitleSegment";
   import { ContentSegment } from "./segments/ContentSegment";
   import { getDurationInFrames } from "../../config";
   import { TITLE_DURATION_SECONDS, CONTENT_DURATION_SECONDS } from "./config";
   import { createComposition } from "../../utils/createComposition";

   const {ComponentName}Composition: React.FC = () => {
     const { fps } = useVideoConfig();
     const titleDuration = getDurationInFrames(TITLE_DURATION_SECONDS, fps);
     const contentDuration = getDurationInFrames(CONTENT_DURATION_SECONDS, fps);

     return (
       <Series>
         <Series.Sequence durationInFrames={titleDuration}>
           <TitleSegment />
         </Series.Sequence>
         <Series.Sequence durationInFrames={contentDuration}>
           <ContentSegment />
         </Series.Sequence>
       </Series>
     );
   };

   const TOTAL_DURATION_SECONDS = TITLE_DURATION_SECONDS + CONTENT_DURATION_SECONDS;

   export const {ComponentName} = createComposition({
     name: "{ComponentName}",
     component: {ComponentName}Composition,
     durationInSeconds: TOTAL_DURATION_SECONDS,
     preset: "Landscape-1080p",
   });
   ```

4. **Update Root.tsx**:
   - Add import: `import { {ComponentName} } from "./compositions/{folder-name}/Composition";`
   - Add inside the Compositions folder with its own Folder wrapper:
     ```tsx
     <Folder name="{ComponentName}">
       <{ComponentName} />
     </Folder>
     ```

5. **Report completion**:
   - List all created files
   - Show how to render: `pnpm exec remotion render {ComponentName}`
   - Remind user to customize content.ts and config.ts

---

### Transcribe

**Trigger:** User asks to transcribe a video or audio file, e.g. "transcribe public/interview.mp4"

Transcribe audio from a video or audio file using the Deepgram API.

> **Requires:** `DEEPGRAM_API_KEY` environment variable. For advanced settings (diarization, language detection, etc.), see `#file:context/deepgram.md`.

#### Steps

1. **For video files** (mp4, mov, avi, mkv, etc.):
   - Extract audio to MP3 using ffmpeg: `ffmpeg -i input.mp4 -vn -acodec libmp3lame -q:a 2 output.mp3`
   - Use the original filename stem (e.g., video.mp4 → video.mp3)
   - Transcribe the extracted audio file

2. **For audio files** (mp3, wav, etc.):
   - Transcribe directly

3. **Default transcription settings**:
   - Use `nova-3` model (most accurate)
   - Enable `smart_format=true` for formatting
   - Enable `punctuate=true` for punctuation
   - Enable `filler_words=true` to keep "uh", "um", etc.
   - Enable `paragraphs=true` for structure

4. **API call**:
   ```bash
   curl -X POST "https://api.deepgram.com/v1/listen?model=nova-3&smart_format=true&punctuate=true&filler_words=true&paragraphs=true" \
     -H "Authorization: Token $DEEPGRAM_API_KEY" \
     -H "Content-Type: audio/mpeg" \
     --data-binary @audio_file.mp3
   ```

5. **Save output**:
   - Save JSON response to `{original_filename_stem}_transcript.json`
   - Example: `interview.mp4` → `interview_transcript.json`
   - Report transcript location and show a snippet of the text

---

### Generate Image

**Trigger:** User asks to generate an image, e.g. "generate an image of a futuristic city at sunset"

Generate an image using Nano Banana Pro via the Replicate MCP server.

> **Requires:** Replicate MCP server connected and a funded Replicate account.

The model supports:
- Text-to-image generation
- Image-to-image editing
- 16:9, 1:1, 9:16, 4:3, 3:4 aspect ratios
- 2K, 4K, 8K resolutions
- PNG or JPEG output

#### Steps

1. **Create the prediction**:
   - Use model: `google/nano-banana-pro`
   - Version ID: `944891d151f5463d9e6eca5a6942f04053e664853dca30c21864021b046fea1d`
   - Default parameters:
     - `aspect_ratio`: "16:9"
     - `resolution`: "2K"
     - `output_format`: "png"
     - `safety_filter_level`: "block_only_high"
   - Use `Prefer: wait` to wait for completion

2. **Call the Replicate API** via the `replicate` MCP server:
   ```
   create_predictions with:
   - version: google/nano-banana-pro:944891d151f5463d9e6eca5a6942f04053e664853dca30c21864021b046fea1d
   - input: {"prompt": "{user_prompt}", "aspect_ratio": "16:9", "resolution": "2K", "output_format": "png", "safety_filter_level": "block_only_high"}
   - Prefer: wait
   - jq_filter: {id, status, output, error}
   ```

3. **Display results**:
   - Show the image URL
   - Show the prediction ID
   - Summarize key features of the generated image

4. **Optionally download**:
   - Ask the user if they want to download the image to the `public/` directory
   - If yes, ask for a filename (suggest a descriptive name based on the prompt)
   - Download using curl: `curl -o public/{filename}.png {image_url}`
   - Confirm the download location

#### Notes

- The prompt should be descriptive and include style, lighting, and composition details
- For talking head videos, specify "centered frame, medium close-up"
- For reference images, mention "photorealistic" or specific photography styles
- Images are saved to Replicate's CDN and accessible via HTTPS URLs

---

### Generate Video

**Trigger:** User asks to generate a video, e.g. "generate a video of a camera flying through clouds"

Generate a video using Veo 3.1 Fast via the Replicate MCP server.

> **Requires:** Replicate MCP server connected and a funded Replicate account. Video generation takes 90–120 seconds.

**Model options:**
- **veo-3.1-fast** (default): Faster and cheaper, great for most use cases
- **veo-3.1**: Higher fidelity, use when user specifically requests "full model" or "high quality"

Veo 3.1 features:
- Text-to-video with context-aware audio
- Starting frame support (reference image)
- 16:9, 9:16, 1:1 aspect ratios
- 1080p resolution
- 4, 6, or 8 second durations

#### Steps

1. **Determine which model to use**:
   - Check if prompt contains keywords: "full model", "high quality", "high fidelity", "veo 3.1"
   - If yes: use `google/veo-3.1` (full model)
   - If no: use `google/veo-3.1-fast` (default)
   - Inform user which model is being used

2. **Parse arguments and analyze prompt**:
   - Ask if they want to use a starting frame image (optional)
   - Ask for aspect ratio: 16:9, 9:16, or 1:1 (default: 16:9)

3. **Intelligently determine duration** (4, 6, or 8 seconds):
   - **4 seconds**: Short phrases (1–5 words), quick actions, simple gestures
   - **6 seconds**: Medium phrases (6–12 words), standard dialogue, most talking head content
   - **8 seconds**: Long phrases (13+ words), complex sentences, detailed actions
   - Consider natural speaking pace: ~2 words per second
   - Inform user of duration choice and reasoning

4. **Create the prediction WITHOUT waiting** (avoids timeout):
   - Do NOT include `Prefer: wait`
   - Call the `replicate` MCP server `create_predictions` with:
     ```
     - version: google/veo-3.1-fast (or google/veo-3.1 if requested)
     - input: {
         "prompt": "{user_prompt}",
         "aspect_ratio": "16:9",
         "duration": {calculated_duration},
         "resolution": "1080p",
         "generate_audio": true,
         "image": "{starting_frame_url}" (if provided)
       }
     - jq_filter: {id, status, created_at}
     ```

5. **Poll for completion**:
   - Inform user that video generation started (show prediction ID, model, duration)
   - Explain it typically takes 90–120 seconds
   - Wait 30 seconds initially, then check status every 15 seconds using `get_predictions`
   - Show progress updates: "Still processing... (45s elapsed)"
   - Continue until status is "succeeded", "failed", or "canceled"

6. **Display results when complete**:
   - Show the video URL (mp4 file)
   - Show prediction ID and total generation time
   - Summarize video details (duration, resolution, has audio, model used)

7. **Optionally download**:
   - Ask if they want to download to `public/` directory
   - Suggest a descriptive filename
   - Download using curl: `curl -o public/{filename}.mp4 {video_url}`
   - Confirm the download location

#### Notes

- **veo-3.1-fast** is the default and recommended for most use cases
- Only use **veo-3.1** (full model) when explicitly requested
- Always include details about movements, camera angles, and audio in the prompt
- Veo generates context-aware audio automatically when `generate_audio: true`
- Starting frame images help maintain consistency in appearance and style

---

### Screenshot

**Trigger:** User asks to take a screenshot of a URL, e.g. "take a screenshot of https://example.com"

Take full-page screenshots at 1280x720 viewport using the Playwright MCP server.

> **Requires:** Playwright MCP server connected.

Screenshots are automatically saved to `public/screenshots/` for use in the project.

#### Steps

1. **Parse arguments**:
   - URL to navigate to (required)
   - Special instructions (optional) — e.g., "scroll to bottom", "click login button", "wait 5 seconds"

2. **Navigate and resize**:
   - Navigate to the URL using the `playwright` MCP `browser_navigate` tool
   - Resize viewport to 1280x720 using `browser_resize`

3. **Follow special instructions (if provided)**:
   - Execute them using appropriate Playwright tools
   - Examples: clicking elements, scrolling, waiting, typing text

4. **Take screenshots**:
   - Viewport screenshot: `browser_take_screenshot` with filename `public/screenshots/viewport-{timestamp}.png`
   - Full page screenshot: `browser_take_screenshot` with `fullPage: true` and filename `public/screenshots/fullpage-{timestamp}.png`

5. **Move screenshots to correct location**:
   - Playwright saves to `.playwright-mcp/public/screenshots/` by default
   - Run: `mkdir -p public/screenshots && mv .playwright-mcp/public/screenshots/*.png public/screenshots/`

6. **Confirm completion**:
   - List the screenshots that were saved with their full paths

#### Notes

- Default viewport size is 1280x720
- Both viewport and full page screenshots are captured
- Screenshots are saved with timestamps to avoid overwriting
- Full page screenshots include the entire scrollable page height
