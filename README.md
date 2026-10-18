# Milkyplayer

Port of Milkyplayer 1.05.01 for Amiga. MiniGL support. Software rendering version built, but needs optimizations. Supports AHI, AmiGUS and experimental Gravis Ultrasound emulation outputting sound over HDMI, running on a free Emu68 RPi4 core.

Developed using ChatGPT/Codex.

## Development status

All functionality described in this README is implemented, but needs testing before a public release.

## Overview

Milkyplayer brings MilkyTracker’s editor and MilkyPlay playback engine to AmigaOS 3, with separate MiniGL and software rendering builds.

Audio options include standard AHI playback, direct AmiGUS support through `amigus.library`, and experimental Gravis Ultrasound emulation running on an unused Raspberry Pi 4 core alongside Emu68.

The editor provides pattern, instrument and sample editing. MilkyPlay handles module sequencing and tracker effects, while the selected audio backend handles software mixing, hardware sample playback or emulated Gravis playback.

## Requirements

Runs on classic AmigaOS 3.x. **RTG required.**

The MiniGL version requires a working MiniGL installation. A separate software rendering build is available but needs further optimization.

The experimental Gravis Ultrasound emulation requires PiStorm with a Raspberry Pi 4 and currently needs a **custom-built Emu68 kernel**.

## Motivation and community

The project began with a wish to bring a recent version of MilkyTracker to AmigaOS 3 and make use of the power PiStorm brings to the Amiga. That grew into the idea of using a Raspberry Pi 4 core unused by Emu68 for dedicated multichannel audio processing.

During research, I discovered the AmiGUS sound card and wanted to support it too. Direct integration through `amigus.library` allows the project to support its sample playback hardware. Testing on physical AmiGUS cards will depend on contributions from people who own the hardware.

The motivation comes from a love of the Amiga and its tracker community, along with fond memories of Gravis Ultrasound and FastTracker. The ambition is to bring that music-making experience to the Amiga while contributing useful work to the MilkyTracker, PiStorm, Emu68 and AmiGUS communities.

Milkyplayer is an independent project, developed with respect for these projects and their contributors.

## Goals

1. Bring MilkyTracker’s editing and playback capabilities to AmigaOS 3.
2. Provide separate MiniGL and software rendering versions.
3. Support AHI and direct AmiGUS playback.
4. Develop experimental Gravis Ultrasound emulation on a free Raspberry Pi 4 core alongside Emu68, outputting sound over HDMI.
5. Preserve tracker timing and musical behavior, including pitch, loops, panning, envelopes and retriggers.
6. Retain software audio playback and offline rendering.
7. Keep the implementation maintainable and documented for community collaboration.

## Technical design

### Application and playback responsibilities

MilkyTracker provides editing and user interaction. MilkyPlay interprets module data and calculates tracker effects, then passes playback operations to the selected backend.

| Component | Responsibility |
| --- | --- |
| Editor | Pattern, instrument and sample editing; playback controls and audition |
| MilkyPlay | Module sequencing, tracker effects, envelopes and voice commands |
| AmiGUS backend | Sample management and hardware voice control through `amigus.library` |
| Gravis backend | Translate playback operations into commands for the emulated GF1 audio engine |
| Raspberry Pi audio service | Execute emulation and audio processing alongside Emu68 |
| AHI output path | Output software-mixed audio through supported Amiga audio hardware |

SDL remains part of the port for display and input. Software audio playback remains available through AHI and for offline rendering.

### Display versions

The project provides two separate rendering builds:

- **MiniGL version:** presents the tracker interface through MiniGL.
- **Software rendering version:** presents the interface without a MiniGL dependency. This version has been built but needs further optimization.

The versions have separate executables and configuration files, allowing them to be tested independently.

### AmiGUS integration

The AmiGUS backend uses `amigus.library` for the playback operations MilkyPlay needs:

- Upload, replace and release samples.
- Start and stop voices.
- Set pitch, volume and panning.
- Set sample offsets and loop boundaries.
- Coordinate voice changes with tracker timing.
- Allocate and release playback resources.

MilkyPlay retains sequencing and effect processing, while AmiGUS handles sample playback through its hardware voices. Integration must respect the card’s voice capacity, sample formats and playback capabilities.

Physical hardware testing remains an important part of validating this support.

### Experimental Gravis Ultrasound emulation

The Gravis backend targets the GF1 sample playback engine, with separate Classic and MAX driver selections.

The emulation runs on a free Raspberry Pi 4 core alongside Emu68, outputting sound over HDMI. It currently requires a custom-built Emu68 kernel. Its playback design covers:

- Multichannel sample playback.
- Sample memory and voice registers.
- Pitch, volume and panning.
- Sample offsets, looping and playback direction.
- Volume ramps and retrigger behavior.
- GF1 rendering rates and conversion to the output rate.

Gravis-specific playback changes are kept within the Gravis path. Other audio drivers retain normal MilkyTracker playback behavior.

Matching the sound and timing of original FastTracker II and Gravis hardware remains a development goal; the emulation is experimental.

### Timing and communication

The Amiga application transfers sample data and timed playback commands to the Raspberry Pi service through shared memory.

The communication design covers:

- Data layout, versioning, alignment and byte order.
- Command queue ownership and synchronization.
- Cache handling and memory barriers.
- Sample ownership and acknowledgement before memory is reused.
- Handling of underruns, queue overflow, shutdown and restart.

Related voice changes must be applied together, with note retriggers distinguished from pitch and volume updates. Sample replacement must remain safe during playback and editing.

## Testers wanted

Looking for AmiGUS owners to test hardware support, and musicians to test Milkyplayer’s editing and playback functionality.

Interested in helping? **PM me on Discord: Axlv500 on the PiStorm server.**

## Development

Developed using ChatGPT/Codex, with testing on Amiga hardware guiding fixes and further development.

## Reference projects

- [MilkyTracker](https://github.com/milkytracker/MilkyTracker)
- [Emu68](https://github.com/michalsc/Emu68)
- [AmiGUS public documentation and sources](https://github.com/necronomfive/AmiGUS-pub)
- [HippoPlayer](https://github.com/koobo/HippoPlayer)
