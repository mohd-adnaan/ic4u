# ic4u

A voice-driven assistant that helps blind and low-vision users **reach specific objects** and **navigate indoor spaces**, built around [WCAG 2.1 Level AA](https://www.w3.org/TR/WCAG21/). The reaching and navigation pipeline runs entirely on the phone via ARKit — no server round-trip for spatial tracking itself.

**Status:** active development. **Platforms:** iOS (primary, functional) · Android (React Native scaffold only — no native reaching/navigation ported yet).

## Lineage

ic4u is the in-device, App-Store-publishable track pulled out of the ShelfScout research app. ShelfScout compared several server-side reaching pipelines against different backends; ic4u keeps only the on-device path, since anything depending on a hosted server can't generalize past a single mapped lab or ship publicly. The three server pipelines were removed — see below.

---

## What runs on-device

| Capability | Implementation |
| --- | --- |
| **Reaching** | Native ARKit Spatial Target reaching — targets come from ARWorldMap POIs, no server bounding box. `ios/ReachingModule.swift` → `startSpatialTargetReaching` |
| **Navigation** | Native ARKit route guidance — `ios/SemanticRouteNavigator.swift` |
| **Intent classification** | On-device via Apple Foundation Models, with Groq as cloud fallback — `src/services/LLMRouter.ts`, `src/native/OnDeviceLLMModule.ts` |
| **Meta Ray-Ban glasses** | Camera + mic streaming over MWDAT (Meta Wearables Device Access Toolkit) — `src/services/WearablesCamera.ts`, `ios/WearablesCameraModule.swift` |
| **Voice speed** | Per-utterance TTS rate control — Settings → voice speed |
| **Developer mode** | Debug overlay + session log export — `src/components/DebugOverlay.tsx` |

In-device mode is locked on: `SettingsContext.resolveReachingPipeline` always forces the on-device ARKit path and ignores any stored backend-pipeline selection.

## What was removed, and why

Three server pipelines from ShelfScout were dropped as not generalizable / not App-Store-publishable:

- **RTAB-Map indoor navigation** (Kasra's backend) — `RtabGuidanceService`, `RTAB_GUIDANCE_URL`
- **Tracker-driven "Standard" reaching** (Melody's pipeline) — `sendToSmartGuidance`, `SMART_GUIDANCE_URL`
- **Qwen VLM bounding-box reaching** ("Vision Box") — `DETECTION_URL`, `ACQUISITION_URL`

The endpoint constants are gone entirely. A couple of stub functions (e.g. `sendToSmartGuidance` in `WorkflowService.ts`) remain only so existing imports keep compiling — they throw immediately if ever called, and are marked for cleanup.

## Not yet standalone

ic4u still depends on a few hosted endpoints before it can be a fully offline, public build:

- **Orchestration webhook** (`WORKFLOW_URL`) — intent/workflow requests still POST to a lab-hosted endpoint.
- **Speaches TTS/STT** (`SPEACHES_CONFIG`) — cloud speech; a native/offline path exists (`iOSTtsClient`) and should become the default.
- **Groq** — cloud LLM fallback for intent classification when on-device Apple Foundation Models are unavailable.

---

## Setup

```bash
git clone git@github.com:mohd-adnaan/ic4u.git
cd ic4u

# JS deps
npm install

# API keys — copy the examples and fill in your own (gitignored, never committed)
cp src/config/groq.secrets.example.ts   src/config/groq.secrets.ts
cp src/config/openai.secrets.example.ts src/config/openai.secrets.ts

# iOS pods
cd ios && bundle install && bundle exec pod install && cd ..
```

### Run

```bash
npm start                 # Metro
npm run ios               # or open ios/ic4u.xcworkspace in Xcode and build
npm run android           # scaffold only — no native reaching/navigation
```

### Test / typecheck

```bash
npx tsc --noEmit
npm test
```

---

## Known gaps

- **Branding** — the app icon is an empty placeholder (`ios/ic4u/Images.xcassets/AppIcon.appiconset`); bundle id is still the React Native scaffold default (`org.reactjs.native.example.ic4u`) and must change before distribution.
- **Wake word** — the phrase and its phonetic dictionary (`src/hooks/useWakeWordSTT.ts`) were mechanically renamed to "ic4u" and need re-tuning for the final product name; keep the Swift and TS phonetic lists in sync.
- **Large asset** — `ios/model/DepthAnythingV2SmallF16.mlpackage` (~48 MB) powers on-device depth for reaching; worth moving to Git LFS.
