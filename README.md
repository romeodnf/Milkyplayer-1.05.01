# RPiGUS — Multichannel Audio for Amiga

## Overview

RPiGUS is a native Raspberry Pi audio engine for PiStorm/Emu68, designed to provide a playback interface compatible with **amigus.library**. The aim is for MilkyPlay to use RPiGUS or physical AmiGUS hardware through the same application interface.

The engine runs on Raspberry Pi 4 core 3 alongside Emu68, providing 32-channel sample playback and stereo output through the Pi’s audio connector.

As part of the project, the latest MilkyTracker is being ported to **AmigaOS 3**. MilkyTracker provides the editor, while MilkyPlay handles module sequencing and tracker effects and delegates sample playback to the audio engine. AHI support remains available for other Amiga audio hardware.

## Motivation and community

The project began with a wish to bring the latest MilkyTracker to AmigaOS 3 and make use of the power PiStorm brings to the Amiga. That grew into the idea of running a dedicated 32-channel audio engine on a Raspberry Pi 4 core unused by Emu68.

During research, I discovered the AmiGUS sound card and wanted to support it too. Since I do not own an AmiGUS card, implementing an amigus.library-compatible interface in RPiGUS offered a practical development path. MilkyPlay could then use one interface for both implementations, with physical AmiGUS support as a shared goal.

The motivation comes from a love of the Amiga and its tracker community, along with fond memories of Gravis UltraSound and FastTracker. The ambition is to bring that kind of music-making experience to the Amiga while contributing useful work to the MilkyTracker, PiStorm, Emu68 and AmiGUS communities.

RPiGUS is an independent project, developed with respect for these projects and their contributors.

## Goals

1. Build a 32-channel audio engine on Raspberry Pi 4 core 3, with stereo output through the Pi audio connector.
2. Provide an amigus.library-compatible playback interface usable by MilkyPlay with RPiGUS or physical AmiGUS hardware.
3. Port the latest MilkyTracker to AmigaOS 3 and integrate that playback interface into MilkyPlay.
4. Preserve tracker timing and musical behavior, including pitch, loops, panning, envelopes and retriggers.
5. Retain AHI support, software playback and offline rendering.
6. Keep the implementation maintainable and documented for community collaboration.

## Technical design

### Application and playback responsibilities

MilkyTracker remains responsible for editing and user interaction. MilkyPlay interprets module data and calculates tracker effects, then sends sample and voice operations through the compatible playback interface.

| Component | Responsibility |
| --- | --- |
| MilkyTracker | Pattern, instrument and sample editing; playback controls and audition |
| MilkyPlay | Sequencing, tracker effects, envelopes and voice commands |
| amigus.library-compatible interface | Sample management, voice control and playback resources |
| RPiGUS Amiga-side implementation | Transfer samples and commands to the ARM engine |
| RPiGUS ARM engine | Sample resampling, volume control, mixing and stereo output |
| Physical AmiGUS hardware | Sample playback through amigus.library |
| AHI output path | Playback through other supported Amiga audio hardware |

SDL remains part of the MilkyTracker port for display and input. Software audio remains available for fallback and offline rendering.

### Library compatibility

The common interface covers the playback operations MilkyPlay needs:

- Upload, replace and release samples.
- Start and stop voices.
- Set pitch, volume and panning.
- Set sample offsets and loop boundaries.
- Coordinate voice changes with tracker timing.
- Allocate and release playback resources.

Compatibility work focuses on the amigus.library behavior needed by these operations. Differences in voice capacity, sample formats and other capabilities must be handled by the integration.

RPiGUS uses shared memory internally to communicate with the ARM engine. This transport stays behind the application-facing interface.

### Audio engine

The ARM service maintains voice state and continuously generates stereo audio. Its initial feature targets are:

- 32 independent mono sample voices.
- Signed 8-bit and 16-bit samples.
- Variable pitch and independent left/right gain.
- One-shot playback, forward loops and prepared ping-pong loops.
- Nearest-neighbor and linear interpolation.
- Wide internal mixing precision with controlled output clipping.
- Gain ramps for smoother transitions.

Tracker envelopes remain in MilkyPlay initially. Song playback and instrument audition share the available voice resources; additional RPiGUS preview voices may be considered after performance testing.

Pitch conversion preserves musical playback rates independently of each implementation’s internal clock and output rate.

### Timing and communication

RPiGUS schedules commands against its audio frame clock to keep playback timing independent of application scheduling. Related voice changes are applied together, with note retriggers distinguished from pitch and volume updates.

The shared-memory protocol defines:

- Data layout, versioning, alignment and byte order.
- Command queue ownership and synchronization.
- Cache handling and memory barriers.
- Sample ownership and acknowledgement before memory is reused.
- Handling of underruns, queue overflow, shutdown and restart.

Sample replacement must remain safe during playback and editing.

## Development milestones

| Milestone | Deliverable |
| --- | --- |
| Native audio output | Stable stereo output from the core-3 service through the Pi4 audio connector |
| Sample playback | Sample upload and timed control of playback, pitch, volume and loops |
| Multichannel engine | 32-voice playback with measured processing cost and timing |
| Library compatibility | An amigus.library-compatible interface backed by RPiGUS |
| MilkyTracker integration | AmigaOS 3 port and MilkyPlay integration, retaining software playback |
| Physical AmiGUS support | The same MilkyPlay integration verified on an AmiGUS card |
| AHI support | Reliable playback and device/mode selection on supported hardware |
| Editing and polish | Instrument audition, safe sample editing, module compatibility, documentation and packaging |

## Reference projects

- [MilkyTracker](https://github.com/milkytracker/MilkyTracker)
- [Emu68](https://github.com/michalsc/Emu68)
- [AmiGUS public documentation and sources](https://github.com/necronomfive/AmiGUS-pub)
- [HippoPlayer](https://github.com/koobo/HippoPlayer)
