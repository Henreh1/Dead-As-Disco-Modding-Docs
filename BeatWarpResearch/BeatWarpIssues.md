# BeatWarp timing issues

**A gameplay log confirms abnormal playback rates.** The
connection to the reported slide-kick delay still needs a matched recording.

## What players reported

After the November 9 Fall Beats hotfix, one player reported that slide kicks
felt slower on their setup, even though the initial Fall Beats build had felt
smoother to them. Brain Jar's hotfix notes confirm a change to attack-chain
timing. They give no move-specific details for the slide kick. [Fall Beats patch notes](https://steamcommunity.com/app/3404260/allnews/?l=english)

Brain Jar describes BeatWarping as adjusting the speed of marked animation
sections so impacts line up with musical cues. An attack can start moving
when the input is accepted, then take a different amount of time to connect.
That gives us two intervals to measure in footage, from press to first
movement and from first movement to impact. [Brain Jar's BeatWarping interview](https://80.lv/articles/interview-how-dead-as-disco-synchronizes-combat-to-player-uploaded-music)

## Log evidence

The excerpts below come from a player-supplied `Pagoda.log` captured on
November 16, 2026. The game reported build `CL-35298`, DX12 / SM6, a Ryzen 9
9900X and a GeForce RTX 4080 SUPER. The warning bursts occurred while playing
the **Sugar and Spice** challenge (`PS_ED_SugarandSpice_130`). All times below
are copied from the log.

### Repeated low playback rates

The log contains **706 abnormal-rate warning lines** across four challenge
attempts with warnings. Grouping consecutive warnings separated by at most
100 ms gives **46 bursts**, including **34** lasting roughly **130–150 ms**
from first to last warning. The count includes repeated updates and duplicate
entries, including the two identical lines at the start of this example
(lines 5229–5247, intermediate updates omitted).

```text
[2026.11.16-11.39.28:478][469]LogPagodaBeatWarping: Warning: [BeatWarpPlayer] Set the rate to 0.060118. That's abnormal - consider changing the beat warp settings of this target.
[2026.11.16-11.39.28:478][469]LogPagodaBeatWarping: Warning: [BeatWarpPlayer] Set the rate to 0.060118. That's abnormal - consider changing the beat warp settings of this target.
[2026.11.16-11.39.28:487][470]LogPagodaBeatWarping: Warning: [BeatWarpPlayer] Set the rate to 0.061413. That's abnormal - consider changing the beat warp settings of this target.
[…]
[2026.11.16-11.39.28:619][486]LogPagodaBeatWarping: Warning: [BeatWarpPlayer] Set the rate to 0.099214. That's abnormal - consider changing the beat warp settings of this target.
```

If these values are the applied animation multipliers, `0.06×` to `0.10×`
is about **6–10% of normal playback speed**. The log gives no animation or
actor name for these bursts. The **141 ms** example is the span between
warnings. We still need footage to measure button-to-kick delay.

Consecutive warning frames in the bursts were usually about **8 ms apart**.
That makes a single long frame stall during the burst a poor fit for the
observed pattern. We have no frame-time trace for the full run.

### Failed perfect-dodge montage transition

At `11.50.32:456`, the drumstick perfect-dodge montage failed to jump to its
`Montage_PerfectDodge` section. **18 ms later**, BeatWarping logged a rate of
**13.38×**. The rate warning leaves the target unnamed, so these entries establish
the timing of the two events and leave their relationship open (lines 10531
and 10537, intervening entries omitted).

```text
[2026.11.16-11.50.32:456][783]LogAnimMontage: Warning: JumpToSectionName Montage_PerfectDodge bEndOfSection: 0 failed for Montage AM_Player_Dodge_Perfect_CCW_Drumstick_Start
[2026.11.16-11.50.32:474][785]LogPagodaBeatWarping: Warning: [BeatWarpPlayer] Set the rate to 13.379266. That's abnormal - consider changing the beat warp settings of this target.
```

## Video evidence

Pending a clip with the input display, game audio and the matching log.
Start with one short **Sugar and Spice** clip on the affected setup. Record
one standing attack and a few slide kicks, including an attempt that feels
late or fails to come out. Keep the usual settings for the first clip.

For each clip, record the build, controller or keyboard and mouse, gameplay
FPS cap and recording FPS. Add a link with the timestamps for **slide start,
attack input, first visible kick movement and impact**. Note where the impact
lands relative to the audible beat and include the nearest BeatWarp warnings
from the same run.

## What to check next

- **The press appears, then Charlie stays in the slide.** Check when the
  slide-to-kick transition becomes eligible and what happens to a press that
  arrives earlier.
- **The kick starts promptly and then slows down.** Check which animation
  BeatWarping is adjusting, the requested and applied rates, and the target
  beat or song marker.
- **The perfect-dodge sequence repeats.** Check whether
  `Montage_PerfectDodge` is a valid section for the active drumstick montage
  and which animation receives the following rate change.
