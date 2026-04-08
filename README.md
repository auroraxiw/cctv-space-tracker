# CCTV Space Tracker

A single-file, browser-based application for analysing CCTV footage. It detects and tracks people frame-by-frame, maps their positions onto a building floor plan, and generates spatial density heatmaps, per-person dwell-time statistics, and zone usage data — all exportable as CSV.

**No server. No installation. No data leaves your browser. Just open `index.html`.**

---

## Live demo

https://auroraxiw.github.io/cctv-space-tracker

*(Settings → Pages → Branch: `main` → Folder: `/` → Save)*

---

## Features

| | Feature | Detail |
|---|---|---|
| 🎥 | **In-browser detection** | COCO-SSD via TensorFlow.js — no GPU, no Python |
| 👤 | **Multi-person tracking** | IoU tracker keeps consistent IDs across frames |
| 🔍 | **Pre-scan labelling** | Video is scanned at 16× first; every person who appears gets a thumbnail to label before analysis begins |
| 🏷️ | **Manual ID assignment** | Click any bounding box mid-video to reassign AA01–AA07 or a custom string |
| 🗺️ | **Floor plan overlay** | Upload your own PNG/JPG; toggle visibility independently |
| 📐 | **Homography calibration** | 4–12 point pairs per camera, IDW interpolation — handles fish-eye lenses |
| 📍 | **Camera position editor** | Place C1–C7 markers anywhere on the floor plan |
| 🔷 | **Custom zones** | Draw named polygons on the floor plan; track occupied seconds per zone |
| 🌡️ | **Density heatmap** | True Float32 accumulation buffer; custom gradient, opacity, radius |
| ⏱️ | **Real video-time seconds** | Dwell times based on actual `video.currentTime` delta — accurate at any playback speed |
| 📊 | **CSV export** | Per-session and full-run export with timestamp, person ID, camera, zone, floor coordinates |
| ♻️ | **CSV replay** | Reload exported CSVs to replay heatmaps and zone stats without re-running detection |
| 🗓️ | **14-day study metadata** | Every CSV row carries date, day number, and session label |

---

## Recommended workflow

```
Upload video → confirm camera + timestamp
       ↓
Upload floor plan image
       ↓
Set camera positions (Cam pos)
       ↓
Calibrate homography (Calibrate) — 4–12 point pairs
       ↓
Draw zones on floor plan (Draw zone)
       ↓
Click ▶ Start analysis
  → video scans at 16× collecting every person
  → thumbnail grid appears — assign AA01–AA07 to each person
  → click Start analysis to begin recording from frame 0
       ↓
Export CSV when done
```

---

## Controls reference

### Camera panel (left)
| Control | Function |
|---|---|
| **Label** button | Enter click-to-label mode; click a bounding box to assign an ID |
| Bounding boxes | Colour-coded per person; show confidence % |

### Floor plan panel (centre)
| Button | Function |
|---|---|
| **Heat** | Toggle heatmap layer |
| **Tracks** | Toggle movement trails |
| **Live** | Toggle live position dots |
| **BG** | Toggle floor plan image |
| **Zones** | Show / hide drawn zones |
| **Draw zone** | Enter polygon draw mode |
| **Heat opts** | Adjust opacity, radius, gradient colours |
| **Calibrate** | Open homography calibration dialog |
| **Cam pos** | Edit camera marker positions on the floor plan |

### Control bar (bottom)
| Control | Function |
|---|---|
| ▶ / ⏸ | Play / pause video |
| Timeline | Click to seek; clears tracker state |
| Speed | 0.25× – 16× playback |
| **▶ Start analysis** | Trigger pre-scan → label → record cycle |

---

## Homography calibration

Click **Calibrate** → click 4–12 matching points on the video frame (left) and floor plan (right).

The system uses **Inverse Distance Weighting** interpolation, which works well for fish-eye / wide-angle lenses without needing a full projective transform.

| Button | Action |
|---|---|
| Apply | Save calibration for this camera |
| Undo last | Remove the most recent pair |
| Undo cam pt | Remove only the last camera-side point |
| Undo floor pt | Remove only the last floor-side point |
| Clear all | Start over |
| Delete saved | Remove the previously saved calibration |

A green dot on the floor plan indicates a calibrated camera.

---

## CSV output format

```
frame, datetime, date, day, session, person_id, camera, floor_x, floor_y, space, confidence
1234, 2026-02-20 08:20:34, 2026-02-20, 1, Morning 08–10, AA03, C1, 0.4120, 0.3870, Atrium, 0.91
```

`floor_x` / `floor_y` are normalised [0, 1] coordinates on the floor plan image.  
Multiply by your floor plan's real-world dimensions to get metres.

---

## Camera IDs

| ID | Default zone |
|---|---|
| C1 | Atrium |
| C2 | Junction |
| C3 | Kitchen |
| C4 | Operations |
| C5 | Mech Workshop |
| C6 | Elec Workshop |
| C7 | Bio Lab |

All positions are editable via **Cam pos**.

---

## Technical notes

- **Model**: COCO-SSD `lite_mobilenet_v2` via TensorFlow.js (~3 MB, cached after first load)
- **Tracking**: IoU greedy matching; track dropped after 45 frames without detection
- **Heatmap**: Float32 density grid → normalise → map through custom gradient stops
- **Time accuracy**: Dwell seconds use `video.currentTime` delta — correct at all playback speeds
- **Privacy**: All processing is local; no video, images, or coordinates are sent anywhere

---

## Browser support

| Browser | Notes |
|---|---|
| Chrome 90+ | ✓ Recommended |
| Edge 90+ | ✓ |
| Firefox 88+ | ✓ |
| Safari 15+ | ✓ (WebGL may be slower) |

---

## Project structure

```
cctv-space-tracker/
├── index.html      ← entire application (HTML + CSS + JS, ~1 400 lines)
├── README.md
├── LICENSE
└── .gitignore
```

---

## Possible extensions

- [ ] YOLOv8 + Python offline batch processor for 14-day video archives  
- [ ] Multi-camera simultaneous view on one floor plan  
- [ ] ReID model for cross-camera person identity  
- [ ] Fish-eye undistortion pre-pass (`cv2.undistort`)  

---

## License

MIT
