---
name: audioanalysis
description: Expert knowledge in audio engineering, loudness standards, streaming delivery specs, Dolby Atmos, and audio measurement. Use this skill whenever working with LUFS, dBTP, LRA, phase correlation, BS.1770-4, EBU R128, ADM BWF files, Dolby Atmos binaural render modes, or any audio measurement and analysis topic. Also use when validating audio specs, explaining technical audio concepts to non-engineers, or building tools that process audio data.
---

# Audio Analysis & Engineering Skill

## Loudness Standards

### Key measurements
| Metric | Full name | Typical range | What it means |
|--------|-----------|---------------|---------------|
| LUFS   | Loudness Units Full Scale (integrated) | -23 to -6 | Average perceived loudness over entire track |
| LRA    | Loudness Range | 1–20 LU | Dynamic variation, high = dynamic, low = compressed |
| TP     | True Peak (dBTP) | -6 to 0 | Absolute highest sample at 4× oversampling |
| φ      | Phase correlation | -1 to +1 | Stereo relationship: +1=mono, 0=wide, negative=risk |

### Algorithm: BS.1770-4 (ITU)
- K-weighting filter applied before measurement
- Gating: blocks silence/near-silence from skewing average
- True peak: 4× oversampling to catch inter-sample peaks
- Momentary: 400ms window | Short-term: 3s window | Integrated: full program

## Streaming Platform Targets

Targets change. Always verify against each platform's current official spec before delivery.

| Platform         | Target LUFS | TP max  | Notes |
|-----------------|-------------|---------|-------|
| Spotify          | -14 ±1      | -1.0    | Loud tracks are turned down |
| Apple Music      | -16 ±1      | -1.0    | Normalizes both ways |
| YouTube Music    | -14 ±1      | -1.0    | |
| Tidal            | -14 ±1      | -1.0    | MQA/lossless support |
| Amazon Music     | -14 ±1      | -2.0    | |
| Deezer           | -15 ±1      | -1.0    | |
| Netflix (stereo) | -27 ±2      | -2.0    | Dialogue-anchored, check current spec |
| Dolby Atmos      | -18 ±2      | -1.0    | Common music target, varies by platform |
| EBU R128         | -23 ±1      | -1.0    | European broadcast |
| CD / Vinyl       | -9 ±3       | 0.0     | High loudness acceptable |
| Club Mix / DJ    | -8 ±2       | -0.3    | PA system competitive loudness |

## Dolby Atmos Specifics

### ADM BWF file structure
- Container: Broadcast WAV (BWF) with ADM metadata chunk
- ADM = Audio Definition Model (ITU-R BS.2076)
- Contains: bed channels + audio objects with 3D positioning
- Binaural render mode embedded in ADM XML chunk

### Binaural Render Modes
```
Mid        , Renders with moderate binaural processing
Near       , Close/intimate binaural rendering  
Far        , Spacious/distant binaural rendering
Not Indicated, Renderer decides (default)
```
Parsing without Dolby SDK: read BWF `axml` chunk, parse XML namespace `urn:ebu:metadata-schema:ebuCore_2014`

### Atmos delivery specs
- File: ADM BWF (.wav with embedded ADM)  
- Loudness target: -18 LUFS integrated (rendered binaural output)
- True peak: ≤ -1.0 dBTP
- Bed: 7.1.4 (L/R/C/LFE/Ls/Rs/Ltf/Rtf/Ltr/Rtr + Ltm/Rtm optional)

## Phase Correlation Interpretation
```
+0.8 to +1.0  → Excellent mono compatibility
+0.4 to +0.8  → Good stereo, safe
 0.0 to +0.4  → Wide stereo, check mono
-0.2 to  0.0  → Risk zone, test in mono
< -0.2        → Phase inversion likely, cancellation in mono
```

## Verdict System (6 levels)
For any given check, compare measured value against platform spec:

```
PASS   , Within tolerance (e.g., ±1 LU of target)
CLOSE  , Just outside tolerance (e.g., 1–2 LU off)
ALMOST , Moderately off (2–3 LU off)
WARN   , Outside spec but considered acceptable for delivery context
FAR    , Significantly outside spec
FAIL   , Critical failure, must fix before delivery
```

Use WARN specifically when: value is outside spec but platform behavior is benign
(e.g., Spotify will just normalize it down, not catastrophic)

## Common Delivery Errors & Solutions

| Error | Cause | Fix |
|-------|-------|-----|
| TP > -1.0 | No true peak limiting | Insert TP limiter at -1.0 dBTP, re-export |
| Too quiet (streaming) | Under-compressed mix | Match platform target in mastering chain |
| Phase issues (φ < 0) | Mismatched mics, plugin phase | Mid/Side check, flip phase on suspect track |
| 16-bit instead of 24 | Wrong export setting | Re-export from DAW at 24-bit |
| Wrong sample rate | Down-converted file | Re-export at 48kHz or 96kHz |
| Atmos loudness off | Object/bed balance | Re-render after adjusting object levels |

## Explaining Technical Concepts to Non-Engineers

**LUFS** → "How loud your track feels on average. Like asking: if someone listened for the whole song, how loud did it feel overall?"

**True Peak** → "The absolute loudest single moment, like a spike. Streaming apps get nervous if this is too high because they compress the file, which can cause crackling."

**LRA** → "How much your track breathes. A song that's always the same level = low LRA. A symphony with quiet and loud parts = high LRA."

**Phase** → "Whether your left and right channels are 'in agreement.' When they fight each other, the track sounds thin or disappears on mono speakers."
