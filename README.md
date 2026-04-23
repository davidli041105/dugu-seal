# 獨孤印 · The Dugu Seal

An interactive, physically-based 3D reconstruction and stamping simulator for the **Dugu Seal (獨孤信多面體煤精印)** — a 26-faceted personal seal belonging to the 6th-century Western Wei general **Dugu Xin (獨孤信)**, one of the most unusual artifacts in the history of Chinese sigillography.

### ▸ [**Try the live demo →**](https://davidli041105.github.io/dugu-seal/)

No download, no install. Opens in any modern browser.

![Dugu Seal showing carved inscriptions on multiple faces](screenshots/overall_screenshot.png)

---

## About the Artifact

Dugu Xin (獨孤信, 503–557) was a Xianbei aristocrat and one of the Eight Pillars of State under the Western Wei. His personal seal — unearthed in Shaanxi in 1981 and now held at the Shaanxi History Museum — is unique in Chinese history: rather than a single inscription, it combines **fourteen different carved seal faces** on a single polyhedral body, each used for a different purpose (imperial memorials, military orders, personal correspondence, and so on).

The polyhedron itself is a **rhombicuboctahedron**: an Archimedean solid with 26 faces (18 squares and 8 equilateral triangles), 48 edges of equal length, and 24 vertices. It is believed to be one of the oldest surviving rhombicuboctahedra in any culture.

This project was built as the final project for the **3D Computer Graphics** course (Spring 2026) at Xi'an Jiaotong University, fulfilling the brief:

- Geometric modeling of the 26-hedron
- Photorealistic rendering with a sense of historical weathering
- Interactive free design of seal contents
- Free-form viewing and observation of the seal

---

## Running it

### Option 1 — The live demo (easiest)

Just open **[davidli041105.github.io/dugu-seal](https://davidli041105.github.io/dugu-seal/)** in your browser.

### Option 2 — Run locally

Clone the repo and serve it over localhost. The viewer loads three.js via ES module imports from a CDN, so some browsers block direct `file://` opens due to CORS rules.

```bash
git clone https://github.com/davidli041105/dugu-seal.git
cd dugu-seal
python3 -m http.server 8000

# then open http://localhost:8000/ in your browser
```

Any static server works — `npx serve .`, `php -S localhost:8000`, etc.

An internet connection is required the first time you open the page (to fetch three.js from the CDN). The browser will cache it for offline use afterward.

---

## Navigation

The app has two modes, selectable from the pill-shaped toggle at the top of the screen.

### Inspect Mode

The default scene. The seal sits on a dark pedestal under museum spotlights, ready to be examined.

**Rotate & zoom**
- **Drag** anywhere on the canvas to orbit
- **Scroll** to zoom
- **View preset buttons** (bottom left) — Isometric / Top / Front / Bottom — snap the camera to fixed angles
- **Auto-rotate** — lets the seal spin slowly on its own

**Editing inscriptions**
- **Click any face** of the seal. The seal rotates so the clicked face is perfectly centered, and the inscription editor opens on the right
- **Type in the text box** to carve your own inscription into that face (supports Chinese, max 8 characters)
- **Style** — choose between:
  - **Recessed (Yinwen 陰文)** — characters carved *into* the surface, appearing sunken with cinnabar ink pooled in the grooves (traditional Chinese seal style)
  - **Raised (Yangwen 陽文)** — characters standing *out* from the surface, appearing raised above the lacquer
- **Size** — adjust character size
- **Ink opacity** — adjust the intensity of cinnabar pigment
- **Clear** — wipe the current face
- **Deselect** — close the editor

**Load Historical Preset** (big orange button) stamps 14 of the real Dugu Seal's documented inscriptions onto their historically-accurate faces:

| Face | Inscription | Purpose |
|------|-------------|---------|
| Top | 臣信上疏 | Memorial to the emperor |
| Bottom | 臣信上表 | Formal petition |
| Band 0 | 大司馬印 | Grand Marshal's seal |
| Band 2 | 大都督印 | Grand Commander's seal |
| Band 4 | 柱國之印 | Pillar of State's seal |
| Band 6 | 刺史之印 | Regional Governor's seal |
| Band 1 | 令 | Order |
| Band 3 | 密 | Confidential |
| Band 5 | 信白牋 | Personal letter |
| Band 7 | 信啟事 | Personal report |
| Upper 0 | 獨孤信白書 | Dugu Xin's hand-written |
| Upper 2 | 耶敕 | Edict |
| Lower 0 | 臣信啟事 | Minister's report |
| Lower 2 | 臣信上章 | Minister's formal submission |

**Material tab** — switches the editor panel to material controls:
- **Edge wear** — how much the lacquer has worn through at facet edges
- **Patina** — age-related color mottling and oxidation
- **Sheen** — polish level of the lacquer
- **Ink bleed** — residual cinnabar ink absorbed into the surface
- **Exposure** — overall scene brightness

### Stamp Mode

A separate scene where the seal descends onto rice paper, presses, and leaves its stamp — just as Dugu Xin would have used it fifteen centuries ago.

**What you see**
- The seal hovers above a sheet of weathered rice paper
- A **live preview of the bottom face** appears bottom-left, showing exactly what will be stamped (mirrored so the paper result reads correctly — the true historical physics of carved seals)
- A large **印 · Stamp** button sits at the bottom center

**How to stamp**
- **Press and hold** the Stamp button to charge ink pressure. A charge meter appears center-screen and fills up
- The seal bobs more aggressively as the charge builds
- **Release** to stamp — the seal descends, presses the paper, transfers ink, and lifts off
- Harder presses (longer holds) produce bolder, more saturated stamps with more ink bleed
- Over-charging (past 80%) triggers an "overdrive" state with visual shake — be careful
- Each stamp lands at a slightly different random position and rotation, simulating real hand stamping

**Other controls**
- **Clear Paper** — wipes all prior stamps for a fresh sheet
- Switch back to **Inspect** at any time to re-edit inscriptions before stamping

**Mirroring:** When you enter Stamp Mode, the seal's bottom face is automatically redrawn with mirror-carved characters (as real seals are carved). When the seal presses the paper, that mirroring combines with the stamp transfer to produce a correctly-oriented inscription on paper. In Inspect Mode, all faces display normally for readability.

---

## Technical Overview

This is a pure-browser WebGL application using [three.js](https://threejs.org/). All rendering, material shading, and geometry generation run client-side.

### Geometry

The rhombicuboctahedron is generated procedurally from its canonical definition: 24 vertices at every permutation of (±1, ±1, ±(1+√2)). Each of the 26 faces is built as an independent polygon with its own vertex set (no shared vertices across faces), giving each face its own flat normal, UV space, and face ID.

Each face's UV [0,1]² maps to one cell of a **texture atlas** (6×5 grid) with a padding gutter around each cell to prevent texture bleed between neighbors.

### Material

The lacquered-wood appearance is generated by a custom fragment shader that hooks into three.js's `MeshStandardMaterial` via `onBeforeCompile`. It injects:

- **Multi-octave fBm noise** in world space for natural color variation, oxidation patches, and wood-grain streaks beneath worn areas
- **Edge darkening** simulating the AO that collects in concave edges of a 1,500-year-old lacquer piece
- **Occasional wear patches** where the lacquer has flaked off to reveal lighter wood
- **Cinnabar ink residue** tinted subtly across mid-face areas
- **Per-face color drift** so different faces show slightly different aging

### Engraved Characters

The inscription effect is done purely in-shader:

1. **Canvas atlas** — each face's inscription is drawn onto a transparent cell of a shared 2304×1920 canvas
2. **Depth-field sampling** — the fragment shader reads the inscription alpha at each pixel with a 9-tap blur kernel, treating it as a local height field
3. **Normal perturbation** — screen-space derivatives (`dFdx`/`dFdy`) of the height field are combined with world-position derivatives to build a true tangent-space normal bend. The surface normal physically tilts into the grooves (Yinwen) or out of them (Yangwen), so three.js's PBR lighting correctly shades the stroke interiors
4. **Pigment & AO** — stroke interiors darken (groove AO) and pick up cinnabar red; stroke rims get a subtle bright lacquer highlight
5. **Roughness shift** — carved grooves are slightly rougher than the polished lacquer around them

### Lighting

- **Procedural HDR environment** baked via three.js's `PMREMGenerator` — a dark museum interior with four warm lamp hotspots for image-based reflections
- **Three-spotlight setup** (key / fill / rim) with shadow-casting key light
- **Tone mapping** — ACES Filmic with tunable exposure
- **Post-processing** — UnrealBloom pass + custom vignette / film grain / warm tint pass

### Stamping

The paper is a `PlaneGeometry` textured by a `CanvasTexture`. When the seal descends and contacts the paper, the animator:
1. Reads the inner region of the bottom face's atlas cell
2. Horizontally mirrors it (cancelling the mirror-carved seal → correctly oriented paper result)
3. Per-pixel replaces the color with cinnabar RGB modulated by:
   - Multi-scale procedural noise (ink density variation)
   - Radial pressure falloff from the center (simulating uneven contact)
   - Press-charge level (affects saturation and bleed)
4. Composites the result onto the paper canvas at a slightly randomized position & rotation

The descend/press/lift animation has four distinct phases with tuned easing curves, and the charge meter uses a responsive spring system so the UI feels physical.

---

## Project Structure

```
.
├── index.html                                  # entry point served at the live demo URL
├── dugu-seal-viewer.html                       # standalone viewer (same app, direct-open filename)
├── dugu_seal_deck_final_print_chinese.pdf      # presentation deck (Chinese)
├── dugu_seal_deck_final_print_english.pdf      # presentation deck (English)
├── screenshots/                                # rendered stills of the project
└── README.md                                   # this file
```

Both HTML files are the same application; `index.html` exists so that GitHub Pages serves the project at the repo root.

---

## Presentation Decks

Print-ready slide decks covering the project's technical design — geometry, material & lighting, engraved-character depth effect, and the stamping simulation — are in the repo root:

- 🇨🇳 [`dugu_seal_deck_final_print_chinese.pdf`](dugu_seal_deck_final_print_chinese.pdf) — 中文版本
- 🇬🇧 [`dugu_seal_deck_final_print_english.pdf`](dugu_seal_deck_final_print_english.pdf) — English version

---

## Customising

Want to tweak the look? Everything interesting is exposed as either UI sliders or near the top of the `<script type="module">` block in `index.html`:

- `PARAMS` — geometry proportions
- `_SEAL_SCALE` — overall seal size
- `sealUniforms` — material parameters (wear, patina, sheen, ink bleed)
- `HISTORICAL_INSCRIPTIONS` — the preset text map (edit to reflect new historical research)
- Camera positions for each view preset and for Stamp Mode
- Light intensities, colors, and positions

For debugging, the running app exposes `window.__seal` with handles to the geometry, mesh, inscription atlas, and materials.

---

## Browser Support

Tested on recent versions of Chrome, Firefox, Safari, and Edge. Requires **WebGL 2** support (ubiquitous on modern hardware). A discrete GPU produces significantly cleaner derivatives for the engraved normal effect than software rasterization; the project should still render on integrated graphics, just with somewhat grainier groove edges.

---

## Credits & References

- **Artifact**: 獨孤信多面體煤精印, Shaanxi History Museum (陝西歷史博物館)
- **Inscriptions**: Transcribed from published photographs of the original seal and Tang-dynasty records of Dugu Xin's offices
- **Geometry**: Rhombicuboctahedron (小斜方截半立方體 · Archimedean solid)
- **Rendering library**: [three.js](https://threejs.org/) r160
- **Chinese font stack**: FangSong / STFangsong / KaiTi / STKaiti / Source Han Serif SC, with browser fallbacks
- **Course**: 3D Computer Graphics · Xi'an Jiaotong University · Spring 2026

---

## License

This project is released for educational and academic use. The geometry, shaders, and UI code are free to adapt; the cultural artifact and its inscriptions belong to the public historical record.

If you're remixing this for your own course, a citation to this repository is appreciated but not required.

---

*六朝風煙，印信猶存。 — The winds of the Six Dynasties have passed, but the seal endures.*
