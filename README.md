# Brutalist 3D Documentary Director: a Claude Code skill

This is a Claude Code skill that turns a **voiceover** into a directed, fully animated **3D documentary**. Optionally it can take a storyboard, a script and reference images as well.

The film is shot as one continuous camera move through one connected world. The default style is dark brutalist architecture lit in crimson. The pipeline uses Remotion, React Three Fiber, Three.js and FFmpeg, and draws on a reusable, versioned library of procedural 3D assets.

## Install

```bash
# all projects
cp -r brutalist-3d-documentary-director ~/.claude/skills/
# or one project
cp -r brutalist-3d-documentary-director <your-project>/.claude/skills/
```

**Requirements:** Node ≥ 22.18, Python ≥ 3.10, FFmpeg, and headless Chromium (Remotion downloads one if none is found). Optional: numpy, plus a speech-recognition backend (faster-whisper, openai-whisper, whisper.cpp or vosk).

First run on a machine:
```bash
python3 ~/.claude/skills/brutalist-3d-documentary-director/scripts/asset-library.py init
bash    ~/.claude/skills/brutalist-3d-documentary-director/scripts/selftest.sh
```

## Use

In Claude Code, ask for it in plain language, for example *"Make a cinematic 3D documentary from vo.mp3 using storyboard.png (9:16)"*, or run `/brutalist-3d-documentary-director`.

See [`brutalist-3d-documentary-director/README.md`](brutalist-3d-documentary-director/README.md) and [`SKILL.md`](brutalist-3d-documentary-director/SKILL.md) for the full workflow. A complete worked production is in `brutalist-3d-documentary-director/examples/the-patient-was-him/`.

## What's inside

- **`SKILL.md`:** the 12-phase production workflow, the visual quality standard and the proof-of-concept quality gate.
- **`references/`:** 13 guides covering style, story, storyboard analysis, cinematography, camera, transitions, lighting, animation, the asset library, audio, the technical stack, QC and the asset-quality checklist.
- **`templates/`:** a Remotion + React Three Fiber project template, plus shot-plan, config, scene and camera-path templates.
- **`scripts/`:**
  - audio inspection, transcription and script alignment, timeline generation and sound design;
  - the asset-library manager, storyboard splitter and project scaffold;
  - four validators, the render driver and a self-test.
- **`assets/`:** the seed library of 123 components and modules plus 113 presets: sets, characters, animals, props, biology, PBR materials, lighting, the post-processing chain, 3D lettering and the camera engine.

Fonts: Oswald (SIL Open Font License 1.1), in `assets/textures/fonts/`.
