# EasyEffects Profiles

This folder contains two EasyEffects profiles for the Legion Pro 7/7i Gen 10:

- `echo_canceling.json`: microphone input profile that eliminates echo on the headphone jack
- `loudness.json`: speaker output profile that makes the speakers slightly louder, closer to Windows defaults

## Installation

Start by installing EasyEffects. The flatpak version is recommended as it includes all necessary plugins and is guaranteed to be up to date, as it's packaged by the main developer (e.g. the Fedora package tends to lag behind the official release). Then:

- For `echo_canceling.json`: go to **Input > Presets > Import**
- For `loudness.json`: go to **Output > Presets > Import**

While EasyEffects is running you will see devices called "Easy Effects Sink" and "Easy Effects Source" appear in your sound settings; *do not select them*. EasyEffects is designed to automatically hijack the default devices instead.

## The echo canceling profile

### The problem

When using a headset via jack, electric crosstalk causes the DAC (audio output) signal to bleed into the ADC (microphone input) nodes on the Realtek codec. The result is that other people on calls can faintly hear whatever audio you are playing, even with the headset mic physically muted. This is a hardware limitation, not a bug in the Linux driver.

Windows solves this not by configuring the hardware differently, but through a proprietary software pipeline developed by Elevoc that performs real-time signal subtraction. You can confirm this yourself: in Windows audio settings, find the external microphone and disable the "device default optimizations" toggle, and the echo will come back immediately, exactly as on Linux.

### How the profile works

The profile chains four stages that together approximate what the Windows driver is (likely) doing:

**Stage 1: WebRTC echo canceling (AEC).** WebRTC is the same open-source echo canceling engine by Google that the Windows driver uses internally. It subtracts the output signal from the mic stream in real time. The echo is reduced to inaudibility in most practical scenarios after this stage, but some white-noise-like residual artifacts remain.

**Stage 2: RNNoise noise reduction.** The residual artifacts from AEC look like white noise in the frequency domain, i.e. spectrally flat and much weaker than any real speech signal. RNNoise uses a neural network to distinguish speech from noise and removes the residual. After this stage the echo is completely silenced. The key parameter is the voice activation threshold (default 30%): too low and some noise leaks through, too high and the algorithm clips consonants. The release time (default 60 ms) controls how quickly the gate closes between words.

**Stage 3: Equalizer.** The noise reduction inevitably catches some speech-like frequency components in the crossfire, making words sound somewhat muffled. A gentle boost at 2.5 kHz and 5 kHz (the intelligibility range) compensates for this.

**Stage 4: Upward compressor.** When music is playing the AEC subtracts some energy from the mic signal, making the processed voice quieter than an unprocessed recording. The upward compressor boosts quiet signals without touching loud ones, evening out this discrepancy.

### Notes

- The profile is safe to leave enabled at all times. When no audio is playing there is nothing for the AEC to subtract, so the mic passes through with only minimal processing.
- Discord, Teams, and similar apps apply their own noise canceling and gain control on top of the OS. This can interact unpredictably with the profile. If results are poor, try disabling the app's built-in voice processing and relying solely on EasyEffects.
- If performance is not acceptable for your use case, a USB audio adapter completely bypasses the problematic jack circuitry and eliminates the echo without any software processing.

### Tweaking the profile

- **Still hearing echo residue:** increase the RNNoise threshold by 5-10%.
- **Words sound clipped or jumpy:** decrease the RNNoise threshold and/or increase the release by a few tens of milliseconds.
- **Voice sounds muffled/underwater:** reduce the noise suppression level in the WebRTC module (some echo may return).
- **Voice too quiet compared to recordings without music:** raise the boost amount or threshold in the compressor.
- **Voice sounds too harsh or sibilant:** reduce the dB gains on the 2.5 and 5 kHz equalizer bands.

For a full technical writeup of the investigation behind this profile, including the reverse engineering of the Windows driver and more details about the profile, see [here](https://github.com/nadimkobeissi/16iax10h-linux-sound-saga/issues/34#issuecomment-4176480130) and [here](https://github.com/nadimkobeissi/16iax10h-linux-sound-saga/issues/34#issuecomment-4230334311).

## The loudness profile

The Legion Pro 7's woofers are driven by the AW88399 smart amp, and audio quality itself is basically identical between Linux and Windows. 
Windows does apply some software equalization, but its impact is pretty minor, and the main driving factor is the smart amp itself, unlike [other legions](/docs/other_legions_guide.md) that lack a smart amp and rely more heavily on software magic.

The most noticeable difference between Linux and Windows is in volume scaling, as the latter applies a software boost of a few dB. The practical effect is that a given volume percentage sounds noticeably louder on Windows; for example, 20% on Windows corresponds roughly to 40% on Linux. Volume values around 30% or below are particularly affected, quickly becoming nearly inaudible on Linux while still comfortable on Windows. The usable portion of the volume range is effectively compressed into the upper end of the slider.

This profile applies a +5 dB boost whose purpose it to redistribute the dynamic range, i.e. to make lower volume values usable again and bring the overall scaling closer to Windows. It is entirely optional; you can simply use higher volume values on Linux than you would on Windows to achieve the same perceived loudness. However, if you find yourself always pushing the slider past 50% just to hear anything, this profile may be more convenient.

*Note:* while the profile is active, it will also affect headphone volume unless configured differently in easyeffects, meaning you may need to use lower volume values than stock to compensate the boost when using headphones.
