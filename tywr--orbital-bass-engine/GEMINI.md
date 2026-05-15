## orbital-bass-engine

> Bass guitar audio plugin built with JUCE and C++. Runs as a standalone app (macOS/Windows), with plugin format support planned (AU, VST3).

# Orbital Bass Engine

Bass guitar audio plugin built with JUCE and C++. Runs as a standalone app (macOS/Windows), with plugin format support planned (AU, VST3).

## Build

```bash
cmake -S . -B builds/local/release -DCMAKE_BUILD_TYPE=Release
cmake --build builds/local/release --config Release
```

The standalone app is output to `builds/local/release/src/orbital-bass-engine_artefacts/Release/Standalone/`.

Shortcut targets are available via `make run`, `make build-release`, `make build-debug`, etc.

## Project Structure

```
src/
├── plugin_audio_processor.{h,cpp}   # Main audio processor, DSP chain, state persistence
├── plugin_editor.{h,cpp}            # Top-level editor, wires GUI to processor
├── parameters.h                     # All audio parameter definitions (createParameterLayout)
├── preset_manager.{h,cpp}           # Save/load/apply individual presets (XML files)
├── session_manager.{h,cpp}          # Manages root folder, collections, and preset slots
├── gui/
│   ├── header.{h,cpp}               # Top bar: meters, gain knobs, tuner, preset controls
│   ├── panels.{h,cpp}               # Main panel layout (compressor, amp, EQ, chorus, IR)
│   ├── preset_bar.{h,cpp}           # 5 clickable preset slots
│   ├── preset_icon_buttons.{h,cpp}  # Icon buttons: folder, +, save, reload
│   ├── session_name_display.{h,cpp} # Collection selector dropdown (ComboBox)
│   ├── colours.h                    # Colour palette (ColourCodes, GuiColours namespaces)
│   ├── dimensions.h                 # GUI dimension constants
│   ├── components/                  # Reusable components (LabeledKnob, SolidTooltip)
│   ├── compressor/                  # Compressor panel (knobs, meter, component)
│   ├── amp/                         # Amp panel with multiple designs (Helios, Borealis, Nebula)
│   ├── eq/                          # 5-band EQ panel
│   ├── chorus/                      # Chorus panel
│   ├── ir/                          # Impulse response / cabinet sim panel
│   ├── synth/                       # Synth voices panel
│   ├── looks/                       # Custom LookAndFeel classes
│   ├── tuner.{h,cpp}                # Chromatic tuner overlay
│   └── meter.{h,cpp}                # Level meter
├── dsp/
│   ├── compressor.{h,cpp}           # Compressor DSP
│   ├── eq.{h,cpp}                   # Parametric EQ DSP
│   ├── ir.{h,cpp}                   # Convolution-based IR loader
│   ├── chorus.{h,cpp}               # Chorus effect DSP
│   ├── pitch_detector.{h,cpp}       # Pitch detection for tuner
│   ├── overdrives/                   # Overdrive circuits (Helios, Borealis)
│   ├── synth_voices/                 # Sub-octave synth voices
│   ├── circuits/                     # Circuit models (CMOS, JFET, diodes, triodes, BJT)
│   ├── filters/                      # Drive filter
│   └── maths/                        # Lookup tables, omega function
└── assets/                           # Binary impulse response data
```

## Architecture

### Audio Processing Chain

Signal flow in `processBlock`: mono summing → input gain → (tuner) → compressor → overdrive + amp master → EQ → stereo copy → chorus → IR convolution → output gain → startup fade.

Each DSP module has a bypass parameter. The processor uses `juce::AudioProcessorValueTreeState` for all parameters.

### Preset & Collection System

- **Root folder**: A user-chosen directory containing collection subdirectories.
- **Collection**: A subdirectory within the root folder. Each collection holds up to 5 presets (`preset_1.xml` through `preset_5.xml`).
- **Preset**: An XML file containing the full `AudioProcessorValueTreeState` serialized via `ValueTree::createXml()`, with a `presetName` attribute.

`SessionManager` manages the root folder, enumerates collections, loads/switches collections, and tracks the 5 preset slots. `PresetManager` handles individual preset file I/O and applying state.

Persistence uses both `juce::PropertiesFile` (for standalone: `rootFolderPath`, `lastCollectionName`, `lastPresetIndex`) and the APVTS state tree properties (`root_folder_path`, `session_folder_path`) for DAW host recall.

### GUI

Fixed size 1080x720. The `Header` contains input/output meters and gain knobs, a tuner toggle, preset icon buttons (folder, new collection, save, reload), a collection selector dropdown, and 5 preset slots. The `Panels` component holds the DSP module panels.

All custom drawing uses JUCE `Graphics` — no external image assets for the UI. The colour palette is defined in `ColourCodes` namespace (Nord-inspired theme). Custom `LookAndFeel` classes are in `gui/looks/`.

### Impulse Responses

Cabinet IRs are embedded as binary data (`assets/ImpulseResponseBinary.{h,cpp}`), generated from WAV files in `impulses/` using JUCE's BinaryBuilder. The mapping is in `assets/ImpulseResponseBinaryMapping.h`. Available cabs: B15, SVT810, EBS410, XL410, PPC212, TC410.

## Conventions

- C++ with JUCE framework idioms. No STL containers for audio-thread code.
- GUI components follow JUCE's `Component` pattern with `paint()` and `resized()`.
- Parameters are defined in `parameters.h` and accessed via `AudioProcessorValueTreeState`.
- Listener pattern (`SessionManager::Listener`) for GUI updates on state changes.
- DSP modules implement `prepare(ProcessSpec)` and `process(ProcessContextReplacing)`.
- Font used: "Index" at 13pt for UI text.

## CI/CD

GitHub Actions workflow in `.github/workflows/release.yml`:
- Triggers on `v*` tag push or manual dispatch.
- Builds macOS and Windows in parallel.
- macOS uses `ditto` for packaging (preserves `.app` bundle structure).
- On tag push, creates a GitHub Release with both platform zips attached.

Tag and push to release: `git tag v1.0.0 && git push <remote> v1.0.0`.

---
> Source: [tywr/orbital-bass-engine](https://github.com/tywr/orbital-bass-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-13 -->
