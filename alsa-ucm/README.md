# UCM changes for sc7280-mainline/alsa-ucm-conf

Base: [sc7280-mainline/alsa-ucm-conf](https://github.com/sc7280-mainline/alsa-ucm-conf) `main` at 56047f6174be,
`ucm2/Fairphone/fp5/`. Licence: BSD-3-Clause, as alsa-ucm-conf.

| # | Patch | Status |
|---|---|---|
| 0001 | ucm2: Fairphone/fp5: enable ADC1 in the VoiceCall verb | local workaround, not sent. It works around an ordering problem rather than fixing it; see [`docs/call-audio.md`](../docs/call-audio.md) for the proper fix. |

## Applying it on the phone

The installed file comes from the distribution package (on Nura: `alsa-ucm-conf-qcom-sc7280`), so a package
upgrade overwrites a local edit. Apply it, then restart the sound server **outside a call**:

```sh
cd /usr/share/alsa/ucm2/Fairphone/fp5
sudo cp VoiceCall.conf VoiceCall.conf.orig
# -p4 strips a/ucm2/Fairphone/fp5/ from the paths in the patch
sudo patch -p4 < /path/to/0001-ucm2-Fairphone-fp5-enable-ADC1-in-the-VoiceCall-verb.patch
```

BusyBox has no `patch`; there, add the one `cset "name='ADC1 Switch' 1"` line at the end of the
`SectionVerb` `EnableSequence` by hand.

Do not restart PulseAudio while a call is active or while the Voice Call profile's helper streams run: the sound
card is busy then, PulseAudio skips it, and you get no sound until the helpers are stopped and the card is
loaded again.
