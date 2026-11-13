# AR Jewellery Virtual Try-On (AR DEMO)

Real-time AR virtual try-on for jewellery in the browser. The webcam feed is tracked with MediaPipe (face, hand, pose) and jewellery images are overlaid on the right body part: earrings, necklaces, nose rings, finger rings, bangles and bracelets.

Everything runs client-side in the browser.

---

## Features

- Live camera try-on with face, hand and pose tracking (MediaPipe)
- Jewellery catalog: bangles, bracelets, earrings, finger rings, necklaces, nose rings, sets
- GLB 3D model viewer (Draco compressed)
- Background removal for uploaded jewellery images
- Catalog driven by a single JSON file, so adding items needs no code change

## Tech stack

- Next.js (App Router) + React + TypeScript
- Tailwind CSS
- MediaPipe Tasks Vision (face / hand / pose landmarkers)
- Three.js (GLB viewer)
- ONNX Runtime Web / Hugging Face Transformers (background removal)

## Folder structure

| Folder | Purpose |
|---|---|
| `src/` | App source: pages, components, hooks, lib |
| `public/catalog/` | Catalog images and `catalog.json` |
| `public/jewellery/` | Sample jewellery images |
| `public/models/` | MediaPipe model files |
| `public/wasm/` | WASM runtimes (MediaPipe vision, Draco decoder) |
| `scripts/` | Helper scripts for catalog / asset preparation |
| `docs/` | Project documentation |
| `test_assets/` | Sample images for testing |
| `uploads/` | Sample uploaded assets |
| `.backup_pre_fix/` | Old backup files, safe to ignore |

## Prerequisites

- Node.js 18.18 or newer (20 LTS recommended)
- npm
- A webcam
- A modern browser (Chrome or Edge recommended)

## Setup

```bash
git clone <your-repo-url>
cd "AR DEMO"
npm install
npm run dev
```

Open http://localhost:3000 and allow camera access.

## Reinstalling removed files

These are not in the repo. They are regenerated automatically:

| Folder / File | How to restore |
|---|---|
| `node_modules/` | `npm install` |
| `.next/` | `npm run dev` or `npm run build` |
| `tsconfig.tsbuildinfo` | created on the next build |
| `next-env.d.ts` | created by `npm run dev` |
| `scripts/__pycache__/` | created when the Python scripts run |

## Models and WASM files

### `public/models/`

| File | Used for |
|---|---|
| `face_landmarker.task` | Face tracking (earrings, nose ring, necklace anchor) |
| `hand_landmarker.task` | Hand tracking (rings, bangles, bracelets) |
| `pose_landmarker_lite.task` | Body pose tracking (necklace placement) |
| `selfie_multiclass_256x256.tflite` | Not used by the app right now (can be removed) |

Download if missing:

```
https://storage.googleapis.com/mediapipe-models/face_landmarker/face_landmarker/float16/1/face_landmarker.task
https://storage.googleapis.com/mediapipe-models/hand_landmarker/hand_landmarker/float16/1/hand_landmarker.task
https://storage.googleapis.com/mediapipe-models/pose_landmarker/pose_landmarker_lite/float16/1/pose_landmarker_lite.task
```

### `public/wasm/`

| File | Used for |
|---|---|
| `vision_wasm_internal.js` / `.wasm` | MediaPipe runtime (SIMD) |
| `vision_wasm_nosimd_internal.js` / `.wasm` | MediaPipe runtime fallback (no SIMD) |
| `draco/` | Draco decoder for the GLB viewer |
| `vision_wasm_module_internal.*` | Not used by the current MediaPipe version |

The MediaPipe WASM files come from `node_modules/@mediapipe/tasks-vision/wasm`. If they are missing, run `npm install` and copy them from there into `public/wasm/`.

## Catalog

The catalog is defined in `public/catalog/catalog.json`. Images live in `public/catalog/<category>/`.

Categories: `bangles`, `bracelet`, `earrings`, `finger_ring`, `ring`, `necklace`, `nose_ring`, `sets`.

To add a new item:

1. Put the transparent PNG in the matching folder, e.g. `public/catalog/earrings/gold_earrings_17.png`
2. Add an entry for it in `public/catalog/catalog.json`, copying the format of an existing entry in the same category
3. Reload the app

Tip: use PNGs with a transparent background, around 1000 px wide.

## Scripts

Helper scripts in `scripts/` are for preparing catalog and asset data. Check the top of each file for what it does and how to run it. Python scripts need Python 3.

## Useful commands

```bash
npm run dev      # start dev server
npm run build    # production build
npm run start    # run the production build
npm run lint     # lint check
```

## Browser requirements

- Camera permission must be allowed
- Must run on `localhost` or HTTPS, since browsers block camera access on plain HTTP
- Desktop Chrome or Edge gives the best performance

## Known issues

- Large `public/` folder (about 126 MB), mostly catalog PNGs and WASM files. Converting images to WebP would shrink it.
- `bangles/` and `bracelet/`, and `finger_ring/` and `ring/`, hold identical image copies.
- Some unused files remain (e.g. `ErrorBoundary.tsx`, `ImageUploader.tsx`, `JewelleryPicker.tsx`, the unused selfie model).
- Tracking accuracy drops in low light or with fast movement.

## Troubleshooting

| Problem | Fix |
|---|---|
| Camera does not start | Allow camera permission; use `localhost` or HTTPS; close other apps using the camera |
| Models fail to load (404) | Check the files exist in `public/models/` and `public/wasm/` |
| `Cannot find module` | Run `npm install` |
| Port 3000 already in use | Run `npm run dev -- -p 3001` |
| Build or type errors after a fresh clone | Delete `.next/` and run `npm run dev` again |
| Slow or laggy tracking | Use Chrome, close other tabs, make sure the GPU is enabled |
