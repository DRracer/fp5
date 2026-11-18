# Voice calls: the microphone is silent

Fairphone 5, Nura (postmarketOS) edge, kernel 7.3.0 (sc7280-mainline), PulseAudio, the UCM files of
sc7280-mainline/alsa-ucm-conf. November 2026.

## Symptom

Calls connect, and the Fairphone plays the far end through the earpiece, but the far end hears nothing. Recording
the microphone during the call (`parec` on the Voice Call source, or `arecord -D hw:0,1`) gives **exact digital
zeros**. The modem, the DSP voice session and all their commands report success.

It reproduces without a call: switch the card to the Voice Call profile
(`pactl set-card-profile alsa_card.platform-sound 'Voice Call'`, or `callaudiocli -m 1`) and record. Outside the
Voice Call profile, recording works (with the 7.3 fix from [PR #50](https://github.com/sc7280-mainline/linux/pull/50)
in place; without it every capture on 7.3 gives zeros, for an unrelated reason).

## Cause

Two things combine:

1. **wcd938x applies its SoundWire port switches only when the capture stream is set up.** `ADC1 Switch`
   (`wcd938x_set_swr_port()`) only edits the port's channel mask in the driver. The mask is read once, in
   `wcd938x_sdw_hw_params()`, which passes it to `sdw_stream_add_slave()`. The SoundWire core does not allow
   adding ports to a stream that is already prepared or enabled. So a switch turned on while the stream runs has
   no effect until the stream is torn down and set up again.

   Shown without any voice session, in the HiFi profile with raw `arecord -D hw:0,1`: switch on before the
   recording starts gives about −33 dBFS of room noise; switch off at the start and on one second later gives
   exact zeros for the whole recording.

2. **On the profile switch, the capture starts before the switch is turned on.** Traced with kprobes on
   `wcd938x_set_swr_port()` (control name and value), `wcd938x_sdw_hw_params()` and `q6afe_port_start()`:
   - the HiFi Mic1 `DisableSequence` turns `ADC1 Switch` off;
   - PulseAudio opens its capture on the new profile, so the back end `TX_CODEC_DMA_TX_3` (SoundWire TX)
     gets `hw_params` and starts, without the ADC1 port;
   - about 150 ms later the VoiceCall Mic1 `EnableSequence` turns `ADC1 Switch` on, too late.

   The voice call's hostless capture stream (VoiceMMode1) then joins the running back end and keeps it alive for
   the whole call, so the stream is never set up again.

This also explains a workaround that had worked only sometimes: restarting the voice helper streams when the call
becomes active. It rebuilds the stream only if PulseAudio's source happens to be suspended at that moment.

## Workaround (in use)

Turn `ADC1 Switch` on in the VoiceCall **verb** `EnableSequence`, which runs before any stream of the new verb
opens: [`alsa-ucm/0001`](../alsa-ucm/). With it, the profile switch gives about −33 dBFS on the microphone
every time (5 of 5: three profile switches, two `callaudiocli -m 1`). Calls in daily use work in both
directions; so far the older helper-restart workaround (below) was still enabled during those calls, so a call
with the UCM change alone is still to be confirmed.

The same race happens when the call ends (Voice Call → HiFi), but there nothing holds the stream: PulseAudio
suspends its idle source after a few seconds and the next recording starts clean. A recording started within
those seconds would still be silent.

## A proper fix

The workaround fixes the order in one UCM file. Options for a real fix:

- **UCM:** set the codec port switches at verb level for every verb, and mute the microphone through the TX
  decimator volume or mute instead of toggling `ADC1 Switch` per device.
- **Kernel:** let wcd93xx apply port changes to a running stream. This needs SoundWire core support for changing
  the ports of an enabled stream (a bank switch with the new port set), and the same change in wcd937x, wcd938x
  and wcd939x, which share the pattern. That is an upstream design question.
- **Diagnostics:** a warning from wcd938x when a port switch is turned on after its stream was set up would have
  made this visible immediately instead of failing silently. A patch for that exists locally and is not
  published yet.

## Still open

- No echo cancellation: the voice session runs with TX topology `NONE`. The DSP offers an echo and noise
  suppression topology (`VoiceMMode1 TX Topology`); not tried yet.
