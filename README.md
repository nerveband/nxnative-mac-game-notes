# nxnative Mac game notes

Hey, this is stuff I’m exploring with nxnative on Mac. I’ll update these notes as I fix things and learn more.

So far, I’ve been testing Yoshi's Crafted World, Mario Kart 8 Deluxe, and Super Mario Bros. Wonder, with help from GPT-6.1 Sol on the code and checks.

Start with [the setup guide](SETUP.md), or [give the notes to your coding agent](SETUP.md#use-this-repo-with-a-coding-agent). This repo contains text and source patches. Bring your own game files and keys.

## What I tested

Tests ran on an Apple silicon Mac with macOS 27.0.1 and an Audient EVO8 audio interface.

| Game version | Observed in the game | Still unverified |
| --- | --- | --- |
| Yoshi's Crafted World 1.0.1 | Opening scenes, Rail-Yard Run gameplay, repaired scenery, autosave log entries, and a timed audio test | Longer play and gameplay timing with the final rumble patch |
| Mario Kart 8 Deluxe 4.0.1 with Booster Course Pass DLC installed | Boot, menus, and two-player racing with Xbox and Stadia over Bluetooth. I confirmed both players could race | The cause of intermittent pauses, DLC course coverage, and longer races with the final rumble patch |
| Super Mario Bros. Wonder 1.0.0 | Launch and controller detection after the shared host updates | Longer play, multiplayer, and gameplay timing with the final rumble patch |

These gameplay observations came before the final rumble patch. That patch passed separate software and controller-output tests, then was installed in all three apps. I have not yet confirmed its timing during gameplay.

## Fixes and findings

| Problem | Change or finding | Evidence |
| --- | --- | --- |
| Yoshi’s black scenery and striped textures | Applied graphics fixes from nxnative upstream | Scenery rendered normally in Rail-Yard Run |
| Yoshi’s crackling audio | Corrected audio-buffer fields and release-event handling | Automated tests passed. One timed game run reported one underrun across 26,000 audio callbacks |
| Stadia effects played late or kept going | Keep the latest waiting vibration state instead of replaying old effects | Software test passed. Direct controller tests accepted pulse and stop commands |
| Xbox input worked but rumble did not | Added direct Bluetooth rumble output after Apple’s helper failed | I felt the test pulse and confirmed it stopped |
| Mario Kart paused during two-player setup | Captured logs and thread samples | It recovered and both players raced. No fix for every pause is claimed |
| Mac mute key left EVO8 audio audible | The tested output exposed volume controls but no standard macOS mute control | Audient documents holding the relevant output-volume button to mute |

The vibration patch also keeps low- and high-frequency strengths separate. It provides ordinary rumble, which cannot copy the Switch’s HD Rumble exactly. Read [the measurements and limits](TECHNICAL.md) before treating a result as proven on your setup.

## Controllers and saves

Connect controllers before opening the game. Button positions follow Nintendo’s layout: right is A, bottom is B, top is X, and left is Y. Xbox’s printed letters differ from those prompts.

The direct output paths were tested with one Stadia and one Microsoft Xbox Bluetooth controller. USB vibration, more than two players, duplicate controller names, and physical reconnect behavior remain unverified. The patch matches each output device by its reported name.

Yoshi recorded successful autosave commits. Let saving finish before quitting, and back up saves before testing another build. Saves live outside the app in the user’s Library folder.

For EVO8 mute, use [Audient’s output-button instructions](https://support.audient.com/hc/en-us/articles/360050421191-How-to-Mute-or-Unmute-an-Output). I did not change audio drivers or the Mac’s global mute behavior.

## Credits and source versions

[nxnative and its authors](https://github.com/seedds/nxnative) provide the game host, conversion tools, and upstream Yoshi graphics fixes. The local audio and Bluetooth patches in this repo came from my work with GPT-6.1 Sol. SDL documents the controller protocols linked in [the technical notes](TECHNICAL.md).

The app builds used nxnative c484b13 plus selected upstream graphics fixes and local patches. The published patches were also applied and unit-tested on d6e6c57. That check did not include a full game build or playthrough on that exact snapshot. Newer nxnative code has other controller fixes, so check it before applying these patches. No upstream pull request has been submitted.

## Share a useful bug report

Include the game version, macOS version, controller model, Bluetooth or USB, and steps to repeat the problem. Say whether animation, input, and music stopped together, and whether the pause recovered.

Remove account names, controller nicknames, home-folder paths, serial numbers, and tokens from log excerpts. Keep game files, keys, saves, and firmware private.
