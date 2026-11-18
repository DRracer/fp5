# fp5: notes and patches for mainline Linux on the Fairphone 5

> Not affiliated with Fairphone, postmarketOS or Nura. Hobby work, no warranty.

Kernel patches, configuration changes and write-ups from using a Fairphone 5 as a Linux field workstation
on mainline Linux: Nura (postmarketOS) edge with the
[sc7280-mainline](https://github.com/sc7280-mainline/linux) kernel 7.3.

This is an umbrella repository: patches live here until they are merged upstream, or as a reference once
they are. More (battery charge limit, dock power, desktop tweaks) will follow.

## What is here

| Item | Status | Where |
|---|---|---|
| USB 2.0 devices on a USB-C dock enumerate at high speed (they failed with `SET_ADDRESS` -62) | merged in sc7280-mainline ([PR #40](https://github.com/sc7280-mainline/linux/pull/40)) and included in its 7.3 kernels; nothing to apply | — |
| Microphone capture on 7.3 returns only zeros (regression from the TX macro's switch to the PM clock framework) | open, [PR #50](https://github.com/sc7280-mainline/linux/pull/50); the PR is authoritative, the copy here may lag behind it | [`kernel/sc7280-7.3/0001`](kernel/sc7280-7.3/) |
| Microphone silent during voice calls (wcd938x port switch set after the capture stream starts) | workaround in the UCM configuration; the proper fix is open | [`alsa-ucm/`](alsa-ucm/), [`docs/call-audio.md`](docs/call-audio.md) |

With these, voice calls on 7.3 work in both directions in daily use: the far end hears the Fairphone's
microphone. Known gap: there is no echo cancellation yet (the far end can hear itself).

## Layout

| Path | What |
|---|---|
| [`kernel/sc7280-7.3/`](kernel/sc7280-7.3/) | kernel patches for sc7280-mainline 7.3, with an index |
| [`alsa-ucm/`](alsa-ucm/) | changes to the UCM files of [sc7280-mainline/alsa-ucm-conf](https://github.com/sc7280-mainline/alsa-ucm-conf) |
| [`docs/`](docs/) | write-ups: cause, evidence, workaround, what a proper fix needs |
| [`LICENSES.md`](LICENSES.md) | licences per directory |

## Credits

Written by D.R.racer `<drracer@seznam.cz>` with the help of an AI assistant (Claude). Every change was reviewed
and tested on the hardware, and the patches say so with an `Assisted-by:` trailer.
