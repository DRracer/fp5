# Kernel patches for sc7280-mainline 7.3

Base: tag `v7.3.0-sc7280` of [sc7280-mainline/linux](https://github.com/sc7280-mainline/linux).
Each patch has the licence of the file it changes (GPL-2.0 here).

The earlier USB 2.0 dock fix ([PR #40](https://github.com/sc7280-mainline/linux/pull/40)) is merged in
sc7280-mainline and already part of its 7.3 kernels, so it is not repeated here.

| # | Patch | Status |
|---|---|---|
| 0001 | ASoC: codecs: lpass-tx-macro: keep macro and dcodec votes while bound | open, [PR #50](https://github.com/sc7280-mainline/linux/pull/50) |

## 0001: microphone capture on 7.3

Since `b9b23e72abef` ("ASoC: codecs: lpass-tx-macro: switch to PM clock framework for runtime PM") runtime
suspend of the TX macro also drops the macro and dcodec hardware votes, which 7.2 held from probe to remove.
After the first autosuspend the capture path returns only zeros. The patch holds both votes while the driver is
bound.

Tested on the Fairphone 5 with a speaker → microphone loopback: zeros without the patch; −8.5 dBFS with it,
both on first use and after the macro had runtime-suspended and resumed. Details and the full test matrix are in
the PR.

This patch is needed for any microphone capture on 7.3. It is not enough for voice calls: there the microphone
stays silent for a second, unrelated reason, see [`docs/call-audio.md`](../../docs/call-audio.md).
