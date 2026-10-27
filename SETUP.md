# Try the patches

Use a separate copy of nxnative. Keep your working game apps and saves backed up.

## Use this repo with a coding agent

I developed these fixes with GPT-6.1 Sol. You can point your own coding agent at this repo to investigate a similar bug or implement a feature. Give it your versions, hardware, and a task. Copy this prompt and replace the final line:

> Read https://github.com/nerveband/nxnative-mac-game-notes, including README.md, SETUP.md, TECHNICAL.md, and the relevant patches. Use the findings and test limits in this repo to investigate my setup or implement the feature I request. Inspect my source version, game version, and hardware first. Check whether upstream already fixes the issue. Adapt only what my setup needs and preserve my other work. Reproduce the problem where possible, run focused tests, and check the result in the game. Keep game files, keys, saves, and personal details private. Report what you verified and what remains unknown. These notes do not prove that every Mario Kart pause is fixed.
>
> My task: [describe the bug or feature, game version, macOS version, controller model, and Bluetooth or USB connection].

The agent needs your own local resources to build a game. The [coverage table](README.md#what-i-tested) separates gameplay observations from software and motor-output tests.

## 1. Get the source

Install Git, Python 3, Xcode, and its command-line tools first. Then run:

    git clone https://github.com/nerveband/nxnative-mac-game-notes.git
    git clone https://github.com/seedds/nxnative.git nxnative-test
    cd nxnative-test
    git checkout --detach d6e6c575ef719c1b96de812dd7bbf700ccf39a94

This source version includes the upstream Yoshi graphics fixes. GPT-6.1 Sol applied both patches to this snapshot and ran the audio and vibration unit tests. I did not test full game builds on this exact snapshot. My installed apps used the older c484b13 base with selected upstream graphics fixes.

Do not apply these patches to a newer version without checking the changes first. Newer nxnative versions already change the controller code.

## 2. Check and apply

Run these commands from the nxnative-test folder:

    git apply --check ../nxnative-mac-game-notes/patches/audio-buffer-and-release-events.patch
    git apply --check ../nxnative-mac-game-notes/patches/bluetooth-rumble.patch
    git apply ../nxnative-mac-game-notes/patches/audio-buffer-and-release-events.patch
    git apply ../nxnative-mac-game-notes/patches/bluetooth-rumble.patch
    git diff --check

If a check fails, stop and check the source version. Do not force a patch into place.

## 3. Run the small tests

These tests use fake game buffers and fake vibration engines. They do not need a game dump. The game ID below chooses a build profile.

    make -C dm/host GAME=0100152000022000 game_profile
    make -C dm/host/tests GAME=0100152000022000 build/t_audio build/t_audio_spsc build/t_haptics_apple
    DM_AUDIO=0 ./dm/host/tests/build/t_audio
    DM_AUDIO=0 ./dm/host/tests/build/t_audio_spsc
    ./dm/host/tests/build/t_haptics_apple

All three should finish without a failed assertion. Some vibration test lines show simulated errors on purpose. Read the final pass messages and check that each command exits with status 0.

The vibration tests block the output queue, send 400 changing values, then send stop. They check that the output gets the latest stop rather than the old effects. They also check separate motor strengths and the stop sent when the software receives a focus-loss flag. That does not test a real window losing focus or a controller being unplugged.

## 4. Build your own app

Follow the [build instructions for the fixed nxnative version](https://github.com/seedds/nxnative/blob/d6e6c575ef719c1b96de812dd7bbf700ccf39a94/README.md). Install its listed dependencies and Metal tools first. Keep one game's base dump, update, and any DLC in its own folder.

The build command has this form. Replace both example paths with your own paths:

    python3 scripts/build_ipa_from_nsp.py "/path/to/game-folder" -k "/path/to/prod.keys" --platform mac-app

The tool chooses the game's profile and builds a Mac app. Read its output for the app's location. Use the game versions listed in the profile, or check what fixes an unlisted update needs. My Mario Kart 4.0.1 racing test does not establish that this snapshot’s build tools support that update without extra work.

Build each game on its own. A host built for Yoshi must not be copied into Mario Kart or Wonder. Reuse cached shaders only if they match the host's shader-chain version.

Move the finished app to your Applications folder. Pair the controllers and open it. Test that input, short vibration effects, and stop commands work before a long session.

## 5. Undo these changes

Your original checkout and apps are unchanged if you followed the separate-folder steps. To undo only these source edits, run this from nxnative-test:

    git apply -R ../nxnative-mac-game-notes/patches/bluetooth-rumble.patch
    git apply -R ../nxnative-mac-game-notes/patches/audio-buffer-and-release-events.patch

Check and preserve any other work before replacing a built app. Keep your save backup until the original app loads it correctly.
