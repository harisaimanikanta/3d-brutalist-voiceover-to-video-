# Lighting & Materials

Lighting is storytelling. Global mood comes from **presets** blended continuously across shots
(`lighting/LightingDirector.tsx`, data in `lighting/lighting-presets.json`); local motivation comes from
**practicals** placed in the world (slot lights, hanging lamps, spot pools, emissive screens, glowing liquids).

## Emotion → preset

| Emotion / beat | Preset | Recipe |
|---|---|---|
| Mystery, opening | `mystery` | deep red backlight + rim, heavy fog 0.045, minimal fill |
| Discovery | `discovery` | toxic-green focused key, red rim, moderate fog |
| Rejection, bureaucracy | `rejection` | cold grey-white top key, hard shadows, desaturated tint |
| Experiment, work | `experiment` / `low-key-lab` | warm focused key on work surfaces, deep falloff |
| Scientific explanation | `explanation` | readable even key, green rim for subjects, light fog |
| Triumph, reward | `triumph` / `golden-award` | warm gold key + top spot, golden haze |
| Final punchline | `punchline` | return to crimson and black, single strong rim |
| Corridors / journeys | `volumetric-corridor`, `industrial-red` | side shafts through haze |
| Portraits / orbits | `high-contrast-character`, `crimson-rim` | hard 3/4 key + red rim, no fill |
| Microscopic world | `microscopic-green` | green key, red tissue rim, dense medium |

Set `shot.lighting`; consecutive identical presets merge; changes cross-fade over `lighting.blendFrames`
(default 36) centred on the boundary. Put lighting changes at transitions (inside an occlusion window, at a
doorway threshold, behind a particle veil) so they feel motivated.

## Physical coherence

- Global key/rim/fill are directional lights with fixed world directions per preset; the key's shadow frustum
  follows the camera target (`LightingDirector focus`).
- Every bright area should have a visible or implied source: add `SlotLight`, `HangingLamp`, `SpotPool`,
  `DeskLamp`, `LightShaft` (fake volumetric), emissive materials.
- Keep 1–3 shadow-casting lights per zone. Mounting/unmounting zones changes light counts (shader recompiles —
  render-time only; fine).
- Fog density is the depth tool: denser = mystery/claustrophobia; lighter = clarity.

## Materials (always from presets: `getMaterial(id, overrides?)`)

`concrete-rough`, `concrete-dark`, `concrete-red`, `metal-painted`, `metal-brushed`, `metal-dark`, `glass`,
`glass-tinted-red`, `bio-translucent`, `mucus`, `flesh-wall`, `bacteria-toxic`, `matte-black`, `red-dark`,
`red-lacquer`, `emissive-red`, `emissive-toxic`, `emissive-gold`, `emissive-white`, `gold-metal`,
`fabric-labcoat`, `fabric-suit-dark`, `fabric-suit-burgundy`, `skin-stylized(-aged)`, `hair-dark/grey`, `paper`.

- Concrete is always textured (procedural board-form map + roughness) and flat-shaded.
- Emissives use `toneMapped: false` so they glow against fog.
- Glass = physical material with clearcoat + low opacity (no transmission pass: too slow in software WebGL).
- Material transformations: `blendedMaterial(a, b, t)` (cached steps) or `MaterialMorphSlab`.
- New material? Add a preset to `materials.ts` (library update + version bump), never inline in scenes.

## Exposure & grade

Each preset carries `exposure` and a `grade {tint, tintOpacity, vignette, grain}` used by `effects/Grade.tsx`
(soft-light tint, vignette, deterministic grain). `exposure` is applied by `PostFX` before tone mapping
(v3; before that it was silently ignored by the ACES pass). `PostFX.lift` (default 0.012) keeps shadow detail
so dark sets read as dark, not as black voids.

**Exposure target:** mean luma 55–140 on every proof-of-concept still (the stills step prints it and flags
< 45 as TOO DARK). Moody ≠ black: a dark frame still shows the set's shapes and a clearly lit subject.
If a still is too dark: 1) check the subject light reaches the subject (camera `target` on the subject),
2) add practicals/spot pools, 3) raise `exposure` / `hemi` in a `shot.lighting` override. Never flatten with
a big ambient light.

## Subject light (v3) — the frame must have a focus

`LightingDirector` takes `cameraPosition` and mounts a soft, neutral, camera-relative spot aimed at the camera's
look target (`preset.subject {color, intensity, angle?, side?}`; defaults #ffe9dc, 2.2). It sits between camera
and subject, lifted and offset to one side (3/4 modelling, not flat flash), with no distance falloff, so the
subject is always the best-lit, true-coloured thing in frame even inside an all-red set. This is the main fix
for "red-on-red, nothing to look at" frames. Consequences:
- Always aim the camera `target` at what the beat is about (a face, a hand, the object), not at empty space.
- `SubjectRig` now adds on top of it: use key 25–40, rim 40–65 (v2 used 40–70 and blew out white hair and
  shirts). Lower `subject.intensity` (1.5–1.9) for punchline/mystery silhouettes; 0 disables it for a shot.

## v2 lighting system (what actually renders)

| Layer | Component | Notes |
|---|---|---|
| Mood (global) | `LightingDirector` + `lighting-presets.json` | key/fill/rim directional lights, hemisphere, fog, exposure; cross-fades across shots |
| Reflections | IBL in `LightingDirector` (PMREM `RoomEnvironment`) | `preset.environment.intensity` (0.28–0.5); metals, glass, wet tissue, clearcoat need it |
| Practicals | `FluorescentFixture` (RectAreaLight, optional flicker), `CageLamp`, `HangingLamp`, `DeskLamp`, `SpotPool`, `Monitor`/`ScreenGlow`, emissive strips | every bright area must have one |
| Subject (auto) | camera-relative subject light in `LightingDirector` (`preset.subject`) | always on; follows the camera target |
| Subject | `SubjectRig` (key spot with shadows, dim fill, strong rim) | one per hero character, placed in the character's facing frame; key 25–40, rim 40–65 |
| Atmosphere | `LightShaft`, `HazeVolume`, `DustMotes` (soft bokeh sprites) | haze sells depth; keep particles subtle |
| Post | `PostFX`: N8AO, depth of field on the subject, bloom, ACES, grade, vignette, grain | preset `post` defaults ← `shot.post` overrides |

Rules: no flat ambient fill; contrast the subject against the background (rim + darker background);
emissives bloom only above the HDR threshold 1.2 (so lit skin, white hair and lab coats don't glow); ACES keeps crimson deep (AgX
desaturates saturated reds to salmon — use `toneMapping: 'agx'` only for pastel looks). Keep ≤3 shadow-casting
lights per active set; `render.shadowMapSize` 1024 for previews, 2048 for finals.
