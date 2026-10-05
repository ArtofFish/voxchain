# Voxchain

### Build the vocal sound you hear in your head.

A live browser vocal-effects chain for karaoke, streaming setups, and voice experiments. Choose a ready-made sound, build an effect chain yourself, or ask a compatible browser agent to work with the same controls through WebMCP.

**[Open Voxchain](https://voxchain.arrangedgodly.com/) · [Watch the demo](https://youtu.be/chm-IvQGqzQ) · [Controls](#build-a-chain-in-advanced) · [Agent tools](#work-with-a-browser-agent) · [Run locally](#run-locally)**

![Voxchain Advanced view with its ordered effect chain and palette](docs/screenshot-advanced.png)

## Start with your voice

1. Put on headphones and keep your monitoring level low.
2. Open the live site in Chrome and press **Start**.
3. Acknowledge the first-start headphone notice, then allow microphone access when your browser requests it.
4. Select the intended microphone. Multi-input devices expose a channel selector.
5. Try a factory sound in **Simple**, or open **Advanced** to edit the individual effects.

Audio capture needs HTTPS or localhost. **Stop** ends the microphone session and disconnects output. **Bypass** sends the raw microphone to the output; it is not a mute button.

## Pick a sound in Simple

![Voxchain Simple view with the current sound and searchable preset library](docs/screenshot-simple.png)

Simple keeps the important session controls available while reducing sound design to a searchable library.

| Control | What it does |
| --- | --- |
| **Previous / Next** | Steps through sounds. |
| **Search** | Finds matches in preset names, descriptions, and tags. |
| **All / Warm / Big echo / Funny / Clean & clear** | Narrows factory sounds by a plain-language character. |
| **Preset card** | Loads that sound into the active chain. |
| **Auto Gain** | Learns an input adjustment from phrases and pauses, then holds the learned level. |
| **Recheck microphone** | Starts input calibration again. Changing microphones also resets learning. |
| **Start / Stop, mic picker, Bypass, meters** | Remain directly accessible while browsing. |

There are **33 factory presets**, including Classic Karaoke, Studio Polish, Robot Usher, Hiss Rescue, and Double Track. Simple carries its session Auto Gain adjustment between sounds. Advanced loads the preset's own chain instead of silently adding that utility.

## Build a chain in Advanced

Read the chain from left to right. A signal-order strip spells out the path, and the input/output meters stay visible while the chain scrolls.

| Gesture or control | Result |
| --- | --- |
| **Click an effect chip** | Adds it to the chain, before an existing terminal limiter. |
| **Drag a palette chip** | Places the new effect at a chosen slot. |
| **Drag a card's grip** | Reorders the chain. Escape cancels the drag. |
| **Alt + Left / Right on a focused grip** | Reorders with the keyboard. |
| **IN / BYP** | Bypasses only that effect, keeping its settings. |
| **×** | Removes the card. |
| **Chevron** | Collapses the controls without changing whether the effect is active. |
| **Bottom-right resize handle** | Changes the card width without changing the sound. |
| **Undo / Ctrl or Cmd + Z** | Uses the shared edit history, including manual and agent changes. |

The shared Undo stack retains up to 20 entries. When a later human edit conflicts with a restoration, the interface can ask for **Undo anyway** rather than silently replacing the newer work.

## Fourteen effects, plus Auto Gain

| Job | Effects and controls |
| --- | --- |
| **Set the input** | Gain; Auto Gain with learned level or manual adjustment. |
| **Control dynamics** | Compressor; Noise Gate with release control; Limiter. |
| **Shape the tone** | Three-band EQ; Distortion; Bitcrusher. |
| **Add space** | Delay; fixed plate Reverb with mix control. |
| **Add movement** | Chorus; Tremolo; Phaser. |
| **Change pitch** | Pitch Shift; experimental Autotune with key, scale, and retune controls. |

Autotune introduces a fixed 20 ms delay in addition to the rest of the capture/output path. Overall latency depends on the browser, hardware, and chain. Pitch-based effects, gates, and feedback effects still need a listening check on the actual setup.

## Save, share, and return to a sound

| Action | What is retained or shared |
| --- | --- |
| **Save As…** | A named personal preset, including each effect's bypass state. |
| **Chain autosave** | Accepted chain and layout state in this browser's local storage. |
| **Copy link** | A saved personal preset's versioned settings in the link's URL fragment. No recorded audio or microphone IDs. |
| **Add to my sounds** | Saves an incoming shared sound without automatically loading it. |
| **Name collision choices** | Rename, replace, or cancel when saving an incoming preset. |

Storage failures are surfaced in the interface. If autosave fails, current edits may be lost on reload. Copy shared links from the deployed site, since a link made on localhost points back to localhost.

Voxchain currently focuses on live processing and settings sharing. This app does not provide a production audio recorder or WAV/MP3 export workflow.

## Work with a browser agent

Voxchain exposes ten tools through an in-page WebMCP adapter. The browser must expose a compatible model-context API. The app itself includes no language model, chat service, API key, or separate MCP server process. Without WebMCP, manual controls remain usable.

An example request:

> Make my voice big and echoey, but keep the words clear.

The preset-first workflow lets an agent inspect concise preset summaries, load a close starting point, and make targeted adjustments. The app validates the requested parameters and routes accepted edits through the same chain-editing transaction layer used by manual changes.

| Tool | Purpose |
| --- | --- |
| `get_capabilities` | Read supported effects, parameters, and agent limits. |
| `get_chain` | Inspect the current ordered chain. |
| `set_chain` | Replace the chain with a validated configuration. |
| `add_node` / `remove_node` | Add or remove an effect within the agent policy. |
| `set_param` | Change a supported parameter. |
| `list_presets` / `get_preset` | Discover summaries, then inspect full preset data. |
| `load_preset` / `save_preset` | Apply or save a sound. |

Read tools can work before audio starts. Audio edits require the engine to be running. Start/Stop, microphone selection, emergency Bypass, and watchdog restoration remain human controls rather than agent tools.

The tools enforce an active terminal limiter, chain-size and gain/feedback limits, and validated parameter ranges. Rejected requests provide corrective detail. These are agent-editing constraints, not a guarantee that every manual chain or physical monitoring arrangement is safe.

For development, append `?dev` to open the Agent Harness and exercise tools directly. See the [preset-first design decision](docs/adr/0003-preset-first-agent-strategy.md) for the reasoning behind smaller, targeted tool calls.

## Monitoring and safety boundaries

- Use headphones while testing. A limiter or watchdog cannot prevent an acoustic feedback loop between speakers and an open microphone.
- Emergency Bypass is an independent, raw-microphone path. It bypasses the processed chain and is intentionally outside watchdog muting. Use **Stop** to end the session.
- The processed path includes output attenuation and protection mechanisms. Do not interpret these as a universal guarantee about sound pressure or true-peak levels.
- Background scheduling and worklet availability affect watchdog behavior. Test the actual browser, device, and venue setup before a live performance.

The [manual acceptance guide](docs/ACCEPTANCE.md) covers real microphone/PA testing, audible transitions, scheduling, and hidden-tab behavior that source tests cannot establish alone.

## How it is built

| Layer | Technology | Responsibility |
| --- | --- | --- |
| **Interface** | HTML, CSS, plain JavaScript | Simple/Advanced views, cards, presets, meters, and session controls. |
| **Audio** | Web Audio + AudioWorklets | Microphone processing, graph construction, specialized DSP, and monitoring. |
| **Selected effects** | Vendored Tone.js | Pitch Shift, Tremolo, Bitcrusher, and Phaser implementations. |
| **Shared editing** | `ChainEditing` | Accepted state, parameter updates versus graph rebuilds, rollback, persistence, and Undo. |
| **Agent integration** | In-page WebMCP registration + tool validators | Runtime discovery and bounded edits to the same chain. |
| **Persistence** | localStorage | Named presets, chain/layout state, and interface preferences. |
| **Tests** | Node.js test runner with browser stubs | Tool contracts, lifecycle races, rollback, graph reuse, presets, Auto Gain, and sharing. |

Manual and agent chain edits converge on `src/chain-editing.js`. Audio graph nodes are reused only when both their ID and type match. Parameter-only updates can avoid a structural rebuild; structural edits use the accepted-state and rollback path.

## Run locally

The application can run from a static server without a frontend build. Python provides a convenient local server:

```bash
git clone https://github.com/ArtofFish/voxchain.git
cd voxchain
python3 -m http.server 8000
```

On Windows, use the corresponding installed Python command, such as `py -m http.server 8000`. Open `http://localhost:8000`, then follow the headphone and microphone steps above. The repository also contains `start.bat` and `start.command` launchers.

Install Node.js to run tests or the static build; CI uses Node 22. These two scripts use Node built-ins and do not require `npm install`.

| Command | Purpose |
| --- | --- |
| `npm test` | Runs the repository's Node test runner. |
| `npm run build` | Creates the static deployment output in `dist/`. |

The test runner has no separate test-package dependency and CI uses Node 22. Deployment tooling is separate from the simple local static-server path. Chrome is recommended; this README does not claim comprehensive cross-browser or physical-device verification.

<details>
<summary><strong>Source guide</strong></summary>

| Area | File |
| --- | --- |
| Session controls | [`src/main.js`](src/main.js) |
| Advanced chain interface | [`src/canvas.js`](src/canvas.js) |
| Simple sound browser | [`src/simple-view.js`](src/simple-view.js) |
| Shared edit transactions | [`src/chain-editing.js`](src/chain-editing.js) |
| Audio graph / raw Bypass | [`src/audio-graph.js`](src/audio-graph.js), [`src/audio-bypass.js`](src/audio-bypass.js) |
| Agent registration / policy | [`src/mcp-server.js`](src/mcp-server.js), [`src/mcp-tools.js`](src/mcp-tools.js) |
| Preset sharing | [`src/preset-link.js`](src/preset-link.js) |
| Tests | [`tests/run.js`](tests/run.js) |

</details>
