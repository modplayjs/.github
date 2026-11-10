# modplayjs

[![npm](https://img.shields.io/npm/v/@modplayjs/core?label=npm&logo=npm)](https://www.npmjs.com/package/@modplayjs/core)
[![CI](https://img.shields.io/github/actions/workflow/status/modplayjs/modplayjs/ci.yml?label=CI)](https://github.com/modplayjs/modplayjs/actions/workflows/ci.yml)
[![license](https://img.shields.io/badge/license-BSD--3--Clause-blue)](https://github.com/modplayjs/modplayjs/blob/main/LICENSE)

A tracker and chiptune player that runs in the browser, on Android and on
Meta Quest 3.

It parses classic module files and renders them with the timing and mixing of
the players they came from — libxmp, OpenMPT and cRSID are ported, not
approximated. The same engine powers the library, the phone app and the
headset app.

## Formats

| Group | Formats |
|---|---|
| Tracker | MOD (19 tracker variants and ProWizard-packed files), S3M, XM, IT, MTM, STM, 669, Soundtracker ST, Ice Tracker, Asylum (AMF), DIGI Booster, MED/OctaMED, MO3 |
| Chip | SID (PSID/RSID) — a full cRSID port: 6510 CPU, SID chip, CIA/VIC |
| Audio files | WAV, MP3, OGG Vorbis, decoded as streams |

Loaders take bytes. There is no URL loader and nothing in the library fetches
over the network — the caller decides how a file arrives.

## Install

```sh
npm install @modplayjs/core @modplayjs/fmt-mod @modplayjs/dsp-softmixer @modplayjs/out-webaudio
```

Every package is published to npmjs.com and mirrored to
[GitHub Packages](https://github.com/orgs/modplayjs/packages). GitHub Packages
needs a token with `read:packages` and the scope redirect:

```
//npm.pkg.github.com/:_authToken=YOUR_GH_TOKEN
@modplayjs:registry=https://npm.pkg.github.com
```

## Use

```ts
import { CorePlayer } from '@modplayjs/core';
import { plugin as modPlugin } from '@modplayjs/fmt-mod';
import { createSoftMixerPlugin } from '@modplayjs/dsp-softmixer';
import { WebAudioOutput } from '@modplayjs/out-webaudio';

const core = new CorePlayer();
core.registries.registerFormat(modPlugin);
core.registries.registerDsp(createSoftMixerPlugin());

await core.loadModule(bytes);          // Uint8Array of the module file
core.setDsp('softmixer');
core.setSampleRate(deviceSampleRate);
core.startPlayer();

const out = new WebAudioOutput();
await out.start(core, workletUrl);     // one user gesture

out.pause();
await out.resume();
out.stop();
```

A format plugin, a mixer plugin and an output plugin are registered on the
core; adding a format means adding one package, not editing the player. The
core also exposes libxmp's interactive surface — `startSmix`, `playNote`,
`stopNote`, `setChannelMute`, `setChannelVol` — which the demo and both apps
use for audition and the channel strip.

## Apps

| App | What it is |
|---|---|
| **Android** | Native shell around the same engine in V8: SAF folder trees indexed in Room, favourites and recents, background playback with a media session, live pattern view. Android 15 (API 35)+, arm64-v8a |
| **modplay XR** | Quest 3 / 3S, Horizon OS v207+. Meta Spatial SDK panels on an arc, one audio object per tracker channel, Interaction SDK hands and controllers, MRUK room bounds for the ring and reverb |
| **Playground** | The browser player with transport, seek, channel mutes, instrument audition and a realtime pattern view: [playground.modplayjs.org](https://playground.modplayjs.org) |

The marketing site and the API reference live at
[modplayjs.org](https://modplayjs.org): one page per area — core and timing,
formats, mixers, outputs, streamed audio — with a row per package, its entry
point and the contract it implements.

## Verification

Everything is measured against the C reference players, not argued about:

- **0 load failures** across the **4,863** reference-parsed files of the
  5,534-file corpus; tracker loaders produce byte-identical `ModuleData` to
  libxmp.
- **Mixer-state parity: 110 passed / 0 failed** against libxmp's own per-frame
  voice dumps.
- **Render parity: correlation 1.0000** (bit-identical) on in-tune MOD/XM
  fixtures; **SID is sample-exact** against a cRSID oracle on all 48 pack
  tunes.
- **OpenMPT cross-check:** on 14 of 15 three-way files the output sits as
  close to OpenMPT as libxmp's.

The corpus also holds 546 files of formats that have no parser here, so the
claim is "0 failures in everything a reference parser also reads", not
"everything loads".

## Repositories

| Repository | Contents |
|---|---|
| [modplayjs](https://github.com/modplayjs/modplayjs) | Library, apps, verification harnesses |
| [website](https://github.com/modplayjs/website) | modplayjs.org (Astro, deployed on Cloudflare Workers) |
| [.github](https://github.com/modplayjs/.github) | This organisation profile |

The main repository is a Turborepo workspace: `packages/` holds the 27
published packages, `apps/` the Android and XR shells plus the demo, `tools/`
the reference builds and parity harnesses, and `docs/` the plugin APIs and the
remaining-parity notes.

## Development

```sh
npm install
npx turbo run build
npx turbo run typecheck
npx turbo run dev --filter=@modplayjs/demo
```

## License

BSD-3-Clause — see [LICENSE](https://github.com/modplayjs/modplayjs/blob/main/LICENSE).
Ported third-party material keeps its original licence: OpenMPT
(BSD-3-Clause), libxmp (MIT), Paula-Tracker (MIT), cRSID 1.58 (WTFPL-style,
Hermit/Mihály Horváth), stb_vorbis (public domain). Details in
[NOTICE](https://github.com/modplayjs/modplayjs/blob/main/NOTICE).

Every line of the engine and of the Android app was written by an AI
assistant; review, debugging, verification and testing by Bitti09.
