# Evidence and technical details

Recorded on October 27, 2026. These notes cover what I’ve tested and changed with help from GPT-6.1 Sol. The patches target the source version in SETUP.md.

## Yoshi graphics

The local game builds started at nxnative commit c484b13cdbccd89a43733abf28d1185a43ad4917. GPT-6.1 Sol applied selected upstream changes during my testing from these commits:

- [1c562de](https://github.com/seedds/nxnative/commit/1c562de): Yoshi profile, texture uploads, and layout fixes.
- [810f999](https://github.com/seedds/nxnative/commit/810f999): GPU-copy readbacks.
- [f24e6b8](https://github.com/seedds/nxnative/commit/f24e6b8): small sparse textures.
- [d6e6c57](https://github.com/seedds/nxnative/commit/d6e6c57): texture tracking and safe snapshot retirement.

The graphics fixes belong to upstream. The patches here do not include those upstream graphics changes. Yoshi reached Rail-Yard Run with normal scenery in the local app test. Graphics tests passed for texture layout, sparse layout, readback, texture tracking, and NVN behavior.

## Audio buffers and release events

Yoshi’s SDK 4.4.1 buffer stores its data pointer at byte 8, capacity at byte 16, and used size at byte 24. The old host treated them as bytes 0, 8, and 16. The audio patch corrects that structure in both AudioOut and the in-process renderer. A test writes the fields as raw bytes so a matching mistake in a C initializer cannot hide the bug.

The game can collect one released buffer at a time. Its event must remain ready while more released buffers wait. The patch signals readiness again when that is needed.

A timed Yoshi 1.0.1 app test logged 26,000 audio callbacks with one underrun and about 0.7 ms of zero-padding reported by the audio callback. This is one observed run, not a guarantee. The patch tests also check release order across 100,000 buffers.

## Stadia timing

The controller accepted direct HID output report 5 over Bluetooth. Forty zero-strength reports took 2.515 seconds in one measurement. Sending every game update through that slower output path can build a backlog. This measurement does not establish the exact vibration-update rate in every game.

The patch keeps at most one waiting state per player. It checks the route generation before output and drops queued states from an older route. The current write can still take time. Output is conventional two-motor rumble, with the two game motors' low/high band power averaged into separate output strengths.

The regression test queues 400 changes behind a blocked worker and ends with zero. It requires one zero output after the worker resumes. A test program used the real controllers for pulse and stop output. It also called the software’s suspend/resume and route-removal functions. It did not test a physical disconnect or a real window-focus change. Cleanup finished in 0.368 seconds on Stadia and 0.208 seconds on Xbox, including deliberate test waits. These figures are not measurements of game-to-motor latency.

## Xbox helper failure

Core Haptics reported available motor locations, but pattern creation repeatedly failed with NSCocoaErrorDomain 4097. Direct Microsoft Xbox Bluetooth report 3 worked. I confirmed that the Xbox test produced a short pulse followed by silence. The patch prefers that output when it can open the matching Bluetooth device. It leaves trigger motors off.

The packet formats are documented in SDL's [Stadia driver](https://github.com/libsdl-org/SDL/blob/main/src/joystick/hidapi/SDL_hidapi_stadia.c) and [Xbox driver](https://github.com/libsdl-org/SDL/blob/main/src/joystick/hidapi/SDL_hidapi_xboxone.c).

Upstream later added [plain-player and other controller fixes](https://github.com/seedds/nxnative/commit/2e07a88). The Bluetooth patch does not apply cleanly to that later code. Review it before porting these changes. The audio patch applied cleanly to the later 1963fab snapshot during preparation, but only the fixed d6e6c57 snapshot is used for the reproduction tests here.

## Mario Kart two-player pauses

The host assigned Xbox to player 1 and Stadia to player 2. I confirmed that both players could race on Mario Kart 4.0.1, before the final rumble patch was installed. Earlier reports described frozen animation and input with music continuing. One reported stall recovered by itself.

Thread samples showed frame waits and graphics work. Logs recorded focus changes and short loading pauses. One frame took 174 ms while loading 19 pipelines and uploading textures. This does not explain every reported pause.

[An Eden fix](https://git.eden-emu.dev/eden-emu/eden/pulls/4027) described a similar Mario Kart symptom caused by missing focus notifications. The captures from my tests did not establish that same cause. No focus workaround was applied, and these notes do not establish a fix for every pause.

## What remains open

- Longer gameplay checks for rumble timing after the latest install.
- More than two controllers, duplicate controller names, and physical reconnect behavior.
- Real window-focus transitions, physical disconnects, and forced app termination with active rumble.
- Xbox USB vibration.
- The cause of every intermittent Mario Kart pause.
- Broad regression testing across other games. No upstream pull request was opened.
