# BeatWarp timing issues

**Abnormal BeatWarp rates coincide with brief holds during attacks in the
Neko Neko footage.** The rate warnings leave the affected animation unnamed.
The cause of the reported slide-kick delay remains open.

## What players reported

After the November 9 Fall Beats hotfix, one player reported that slide kicks
felt slower on their setup, even though the initial Fall Beats build had felt
smoother to them. Brain Jar's hotfix notes confirm a change to attack-chain
timing. They give no move-specific details for the slide kick. [Fall Beats patch notes](https://steamcommunity.com/app/3404260/allnews/?l=english)

Brain Jar describes BeatWarping as adjusting the speed of marked animation
sections so impacts line up with musical cues. An attack can start moving
when the input is accepted, then take a different amount of time to connect. [Brain Jar's BeatWarping interview](https://80.lv/articles/interview-how-dead-as-disco-synchronizes-combat-to-player-uploaded-music)

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
is about **6–10% of normal playback speed**. The **141 ms** example measures
the span between warnings.

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

### Neko Neko Let's Go

A 17.55-second Medal clip shows **Neko Neko Let's Go** on `CL-35298`. Around
**0:09 into the clip**, the in-game song timer reads **1:26**. Charlie is
already moving, then his pose changes very little for about three tenths of
a second while an airborne enemy keeps moving. The next strike then starts.
This is the moment the player described as feeling like Charlie got stuck in
quicksand.

The player reported that **Sugar and Spice** felt fine despite the warnings
in its log, and that the slowdown on **Neko Neko Let's Go** was noticeable
even during normal punches. The Medal clip has no input overlay or matching
log.

### November 17 recording with a matching log

`NekoNeko.mp4` is an 88-second gameplay recording on `CL-35298` with a
controller overlay. The log starts at `17:35:48.667` with song asset
`PS_ED_NekoMiku_194`. Wave indices `1`, `2` and `3` match the HUD's enemy
count jumps from zero to **5**, **8** and **7** at `0:22.78`, `0:35.18`
and `0:57.45` in the video. These matches put the log's start about
`0:00.65` into the recording.

The player identified the first mosher in wave 2 (`WaveIndex 1`) as a spot
where the slowdown felt noticeable. The reported warnings line up with
**about `0:31.3` in the video**. Charlie's kick is already extended while
the airborne mosher and effects keep moving. The controller overlay shows
the left trigger already held. The warning sequence spans **17 ms** across
three consecutive logged frames (lines 82–85, duplicate omitted below).
The projectile warnings quoted with it occur about **2 seconds earlier**.

```text
[2026.11.17-17.36.19:341][356]LogPagodaBeatWarping: Warning: [BeatWarpPlayer] Set the rate to 0.008245. That's abnormal - consider changing the beat warp settings of this target.
[2026.11.17-17.36.19:349][357]LogPagodaBeatWarping: Warning: [BeatWarpPlayer] Set the rate to 0.031870. That's abnormal - consider changing the beat warp settings of this target.
[2026.11.17-17.36.19:358][358]LogPagodaBeatWarping: Warning: [BeatWarpPlayer] Set the rate to 0.043469. That's abnormal - consider changing the beat warp settings of this target.
```

- **Around `0:42.9` in the recording**, Charlie briefly holds his arm out
  after a hit while the airborne enemy keeps rotating. In that window,
  BeatWarp logged rates from `0.001144` to `0.078429` over **49 ms**
  (lines 161–168).
- **Around `0:48` in the recording**, the player reported another odd moment.
  Charlie briefly holds the Drummerang follow-through while its trail keeps
  moving. At about `0:48.16` (`17:36:36.176` in the log), two projectile
  warnings report `ArcToTarget` with no gravity (lines 177–178). The supplied
  log has no BeatWarp rate warning between about `0:43` and `0:54` in the
  video. The cause of this pause is still open.
- **Around `0:38.1` and `1:00.1`**, the HUD awards **Perfect Dodge** even
  though the aligned log shows `Montage_PerfectDodge` section jumps failing
  for the counterclockwise drumstick dodge (lines 118 and 194). This matches
  the montage warning seen on **Sugar and Spice** and shows that the dodge
  can still score when that animation transition fails.

```text
[2026.11.17-17.36.30:920][739]LogPagodaBeatWarping: Warning: [BeatWarpPlayer] Set the rate to 0.001144. That's abnormal - consider changing the beat warp settings of this target.
[2026.11.17-17.36.30:969][745]LogPagodaBeatWarping: Warning: [BeatWarpPlayer] Set the rate to 0.078429. That's abnormal - consider changing the beat warp settings of this target.
```

The excerpt contains **74 abnormal-rate warning lines**, including repeated
entries, and **three failed perfect-dodge montage jumps**.
