# ic4u

**Status:** Active development · **Platform:** iOS (primary) · Android (scaffold) · **Focus:** Accessible indoor navigation & object reaching for blind and low-vision users

ic4u is an **in-device** voice assistant that guides blind and low-vision users to **reach objects** and **navigate indoors**, running its spatial pipeline **fully on the phone** with ARKit. It targets [WCAG 2.1 Level AA](https://www.w3.org/TR/WCAG21/).

> **Lineage & scope.** ic4u is the in-device / publishable track derived from the ShelfScout research app. Where ShelfScout compares several server-side pipelines, ic4u keeps only the on-device path so it can generalise beyond a single mapped lab and move toward the App Store. The three server pipelines have been removed (see below).

---

## What runs on-device

| Capability | Implementation |
|---|---|
| **Reaching** | Native ARKit **Spatial Target** reaching — targets come from ARWorldMap POIs (no server bbox). `ios/ReachingModule.swift` → `startSpatialTargetReaching` |
| **Navigation** | Native ARKit route guidance — `ios/SemanticRouteNavigator.swift` |
| **Intent** | Mobile intent classification via on-device Apple Foundation Models with Groq fallback — `src/services/LLMRouter.ts`, `src/native/OnDeviceLLMModule.ts` |
| **Meta glasses** | Meta Ray-Ban camera + mic support — `src/services/WearablesCamera.ts`, `ios/WearablesCameraModule.swift` |
| **Voice speed** | Per-utterance TTS rate control — Settings → voice speed |
| **Developer mode** | Debug overlay + logging — `src/components/DebugOverlay.tsx` |

**In-Device Mode is locked on.** `SettingsContext` forces on-device ARKit reaching + navigation and ignores any stored backend-pipeline selection.

## Removed vs. ShelfScout

These three server pipelines were dropped because they are not generalisable / App-Store-publishable:

- **RTAB-Map indoor navigation** (Kasra's backend) — `RtabGuidanceService`, `RTAB_GUIDANCE_URL`
- **Melody's tracker-driven reaching** ("Standard") — `sendToSmartGuidance`, `SMART_GUIDANCE_URL`
- **Qwen VLM bbox reaching** ("Vision Box") — `DETECTION_URL`, `ACQUISITION_URL`

The service file and endpoint constants are gone; the few remaining dead branches are unreachable while In-Device Mode is locked on and are marked for cleanup.

## Remaining backend dependencies (roadmap to standalone)

ic4u is **not yet fully standalone**. Still to remove/replace before a public App Store build:

- **Orchestration webhook** (`WORKFLOW_URL`) — the intent/workflow request still POSTs to a hosted endpoint.
- **Speaches TTS/STT** (`SPEACHES_CONFIG`) — cloud speech; a native/offline path exists (`iOSTtsClient`) and should become the default.
- **Groq** intent fallback — cloud LLM; on-device Apple FM is preferred when available.

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
npm run android
```

### Test / typecheck

```bash
npx tsc --noEmit
npm test
```

---

## Notes

- **Branding:** the app icon is an empty placeholder (`ios/ic4u/Images.xcassets/AppIcon.appiconset`) — drop in the new ic4u logo (single 1024×1024). Bundle id is still the React-Native scaffold default (`org.reactjs.native.example.ic4u`) and must change before distribution.
- **Wake word:** the phrase and its phonetic dictionary (`src/hooks/useWakeWordSTT.ts`) were mechanically renamed and need re-tuning for the final product name; keep the Swift/TS phonetic lists mirrored.
- **Large asset:** `ios/model/DepthAnythingV2SmallF16.mlpackage` (~47 MB) powers on-device depth for reaching; consider Git LFS later.
