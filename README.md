# Ableton MCP v2

This is the active Ableton MCP v2 workspace: the upstream bschoepke/ableton-live-mcp foundation, extended with faster day-to-day Live controls, set lifecycle operations, and reliable fader automation. It is the repo to use when you want an agent that can actually operate inside Ableton, recover cleanly, and move from broad musical instructions to concrete clips, devices, routing, and mix changes.

![Ableton MCP v2 infographic](abletoninfographic.jpg)

Ever wanted to control Ableton with just your voice? Me too! I made this MCP server so I could just ask Codex to do anything in Ableton Live for me, while I was nap-trapped by my baby.
Unlike other Ableton MCPs I tried, this one can do pretty much anything that is possible via Ableton's Object model; the agent can just eval arbitrary python that runs inside Ableton. It also has some tools defined for common tasks so those work faster and more reliably. I had Codex CLI optimize this for hours with the new /goal command to prioritize low end-to-end latency, high reliability, low token usage, while maintaining full flexibility.
Things you can use it for: create MIDI clips, insert audio files, general Ableton questions (with this, your agent can see your whole live set), add tracks with different devices and effects, analyze harmony, analyze audio signals at any point in the signal chain, generate spectrograms, clip automation, setting up mastering or vocal processing chains, insert MIDI the agent finds from the web... it's very general purpose, I'm not sure what the limits are.
## How to setup
Just tell your AI agent (Codex, Claude Code, Cursor, Copilot, Gemini, etc.) to:
Set up the https://github.com/bschoepke/ableton-live-mcp MCP server for me
It should work on Mac and Windows with recent Ableton versions, but I have only tested it on Ableton Live Suite 12.3.8 on macOS Tahoe.
Back up your Live Set before using this. The MCP can edit your set directly and could corrupt it.
## How to update
git pull this repo or ask your agent to:
Update the https://github.com/bschoepke/ableton-live-mcp MCP server for me
## Provenance — source MCPs
This repo was consolidated from three Ableton MCP projects (full comparison in _Archive 2026-07 (pre-v2)/ABLETON_MCP_COMPARISON.md):
bschoepke/ableton-live-mcp — the base. All upstream code (stdio MCP server, Remote Script bridge on port 8765, generic Live object access, live_exec, browser/plugin search, clip automation, AgentAudioTap, AgentM4L, visual capture) comes from here, MIT-licensed (see LICENSE).
ahujasid/ableton-mcp — design reference for the ergonomic wrappers. live_create_midi_track, live_create_audio_track, live_rename_track, live_fire_clip, live_stop_clip, and live_set_tempo port the intent of its named tools onto the bschoepke bridge; no code was copied.
Simon-Kansara/ableton-live-mcp-server — evaluated as an AbletonOSC relay reference; nothing adopted (its local copy had a client/daemon protocol mismatch). Kept in the archive in case an OSC adapter is ever wanted.
The lifecycle tools (live_lifecycle_status, live_save_set, live_quit_ableton) and live_fade are original additions from the July 2026 handoff work; details below.
## Local additions (this fork)
This copy adds named lifecycle, convenience, and fade tools on top of upstream (server 0.2.0, Remote Script runtime lifecycle-fade-wrappers-1):
live_lifecycle_status, live_save_set, live_quit_ableton — save/quit via Live API when available, with GUI-workflow guidance returned when it isn't. Quit is scheduled so the RPC response can return first.
live_create_midi_track, live_create_audio_track, live_rename_track, live_fire_clip, live_stop_clip, live_set_tempo — ergonomic wrappers around common Live API calls.
live_fade — timed fader fade run inside Live (smoothstep default, linear optional); 0% = -inf dB, 100% = unity (0.85); target_percent > 100 requires allow_over_unity. Blocks Live's main thread for the duration; the server auto-raises the RPC timeout.
Reinstall the Remote Script and reload Live after updating (src/install_remote_script.py).
## Demos
Here are a couple examples of live sets made from scratch with Codex in just a few minutes, along with their prompts. After it makes something, you can ask for follow up changes.

https://www.youtube.com/watch?v=8dRRrIY7NI0
The chat messages I sent to Codex to make this:
in ableton, make a self reflective song, with audio vocals (via macos say) and chip tunes and 80's drum machines. should be a real edm banger
Follow up prompts:
i want midi for everything but vocals please, with ableton devices. not prerendered audio for instruments
needs some fills
and should hit way harder after "3-2-1 i become the sound"
the vocals are squished too much (read too quickly), give them a little more length
add some dynamics, the song is basically one volume. and some pumping side chain
improve dynamics of the clap, seems a bit flat and indistinguished, want it harder after the 3-2-1 drop
introduce a new element on a new track after the 3-2-1 drop, that comes in but then recedes before the final exit
doesn't seem like the new thing has any notes
the element is a bit muddy/indistinct. perhaps it needs simplification and more space, different instrument choice, i dunno

https://youtu.be/cLCHEV1jWQo
Prompt used to make this:
In Ableton, make a piano duet that tells the story of people debating the positive and negative merits of AI. The composition should be both beautiful and dynamic but surprising and fresh. Use Keyscape devices.
## Built in Agent Audio Tap Max for Live device
The MCP includes an "Agent Audio Tap" Max for Live device that enables the agent to capture audio signals at any part of the signal processing chain. This gives the agent a full feedback loop for mixing and mastering tasks: it can capture audio signals for further processing with custom python, then tweak your Ableton devices, and then repeat.

Example usage where I asked Codex to generate a spectrogram of two piano tracks I had:
<img width="3768" height="1028" alt="piano_tracks_first10_spectrograms" src="https://github.com/user-attachments/assets/6d2b6d9f-9a2c-4552-aa6c-91153de9df44" />
## Ideas
Control your external synthesizers and other hardware with the MCP
Ask it questions like "why does my mix sound muddy?" or "how do I sidechain my bass track?"
Ask it to do things like "add a chord track that fits with my melody" or "give me a basic backing track for me to noodle on my guitar with"
You can tell it use third party plugins (VSTs, audio units) like Serum and Keyscape
Tell your agent to incorporate your existing vocal samples, including asking it to trim silence and transcribe your audio samples before creatively incorporating them into your live set
Ask your agent to set up crazy user controlled DJ effects
Experiment with VJ plugins like Videosync to make music videos driven by your live set
## Maintainer notes
The older source repos and planning notes are archived in ../_Archive 2026-07 (pre-v2)/; current operating conventions live in the ableton-hermes-operator skill under skills/.
After editing src/ or Ableton_Live_MCP/bridge.py, sync the installed copy at ~/Documents/Codex/ableton-live-mcp, reinstall the Remote Script (python3 src/install_remote_script.py), and reload Live. Verify with live_ping; the runtime should report lifecycle-fade-wrappers-1 or later.