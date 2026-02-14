# Play Sound

The [Harp SoundCard](https://github.com/harp-tech/device.soundcard) supports playback of waveforms stored in its onboard memory. It also includes an internal sine wave generator for pure tones. The following exercises demonstrate how to play these sounds in Bonsai.

> [!WARNING]
> When adding these operators to the workflow, make sure to use the device-specific versions, e.g. `Device (Harp.SoundCard)` instead of `Device (Harp)`. If correctly selected, the names of these operators in the workflow panel will change to reflect either the name of the device or the selected register/payload.

## Prerequisites

- Install the `Bonsai.Windows.Input` package from the Bonsai [package manager](https://bonsai-rx.org/docs/articles/packages.html).

## Device pattern

Set up the standard Harp [device pattern](../articles/operators.md#device-pattern) to initialize the device, log data, broadcast events, and send commands to the `SoundCard`.

:::workflow
![SoundCard Device Pattern](../workflows/soundcard-playsound-devicepattern.bonsai)
:::

- Insert a [`Device`] operator, set the `PortName` property to the communications port for the device.
- Insert a [`DeviceDataWriter`] sink and set the `Path` property (e.g. `SoundCard.harp`). 
   - This will save the data in the standard Harp logging format, which can be loaded with [`harp-python`](../articles/python.md).
- Insert a [`PublishSubject`] operator and name it `SoundCard Events`.
- Right-click the [`Device`] operator, select "Create Source (Bonsai.Harp.HarpMessage)" > "BehaviorSubject". 
   - Name the generated [``BehaviourSubject`1``] [source subject](https://bonsai-rx.org/docs/articles/subjects.html#source-subjects) `SoundCard Commands`. 
   - Connect it as input to the [`Device`] operator.

### Exercise 1 - Play sound index

Sounds can be played from the `SoundCard` onboard memory by using the [`PlaySoundOrFrequency`] register.

:::workflow
![Play Sound Index Keydown](../workflows/soundcard-playsound-indexkeydown.bonsai)
:::

- Insert a [`KeyDown`] source and set the `Filter` property to `A`. 
- Insert a [`CreateMessage`] operator to construct a [`HarpMessage`] command to send to the device and configure these properties:
    - `Payload` - Select [`PlaySoundOrFrequencyPayload`] from the property dropdown menu.
    - `PlaySoundOrFrequency` - Set this to the index of the sound you want to play from the `SoundCard` onboard memory (2-31).
- Insert a [`MulticastSubject`] operator to send [`HarpMessage`] commands to named subjects, and configure the `Name` property to `SoundCard Commands`.

Run the workflow and press the <kbd>A</kbd> key to play the sound. Sound duration is determined by the length of the stored waveform.

You can replace [`KeyDown`] with other operators to trigger sound playback on other events in Bonsai.

:::workflow
![Play Sound Index Timer](../workflows/soundcard-playsound-indextimer.bonsai)
:::

- Replace the [`KeyDown`] source with a [`Timer`] source and set the `DueTime` property to 0.
- Insert a [`SubscribeWhen`] operator after `SoundCard Commands`.
- Insert a [`SubscribeSubject`] operator, configure the `Name` property to `SoundCard Events`, and connect it to [`SubscribeWhen`].

> [!TIP]
> The `SubscribeWhen` > `SoundCard Events` pattern is useful for ensuring that [`HarpMessage`] commands are only sent after the [`Device`] has been initialized. It relies on the `DumpRegisters` property being set to `True` in [`Device`]. Use it when needed, for instance, if sounds are being played at the start of the workflow.

### Exercise 2 - Play pure tone

The [`PlaySoundOrFrequency`] register can also be used to play pure tones using the internal sine wave generator.

:::workflow
![Play Sound Frequency](../workflows/soundcard-playsound-frequency.bonsai)
:::

- Insert a [`KeyDown`] source and set the `Filter` property to `A`. 
- Insert a [`CreateMessage`] operator, select [`PlaySoundOrFrequencyPayload`] for the `Payload` property, and set the `PlaySoundOrFrequency` property to the desired frequency in Hz (e.g. 1000).
- Insert a [`MulticastSubject`] operator named `SoundCard Commands`.

Unlike playback of sounds from the onboard memory, the pure tone will continue playing until it is terminated via the [`Stop`] register.

- Insert a [`KeyDown`] source and set the `Filter` property to `S`.
- Insert a [`CreateMessage`] operator, select [`StopPayload`] for the `Payload` property, and set the `Stop` property to 1 (or any other value than 0).
- Insert a [`MulticastSubject`] operator named `SoundCard Commands`.

Run the workflow, press the <kbd>A</kbd> key to play the sound, and press the <kbd>S</kbd> key to stop playback.

> [!WARNING]
> The [`Stop`] register can only be used to stop playback from the internal sine wave generator, not sounds from the onboard memory.

### Exercise 3 - Lower sound playback volume

The [`PlaySoundOrFrequency`] register plays the sound at the amplitude of the stored waveform or at maximum amplitude for pure tones. To lower the volume, use the [`AttenuationAndPlaySoundOrFreq`] register instead to set the attenuation in 0.1 dB steps.

:::workflow
![Play Sound Attenuation](../workflows/soundcard-playsound-attenuation.bonsai)
:::

- Insert a [`KeyDown`] source and set the `Filter` property to `A`. 
- Insert a [`CreateMessage`] operator and select [`AttenuationAndPlaySoundOrFreqPayload`] for the `Payload` property.
   - Click on the dialog button for the `AttenuationAndPlaySoundOrFreq` field in the property grid to open the member collection editor. Add three members:
      - The sound index or pure tone frequency to be played (e.g. 2).
      - The attenuation of the left channel (e.g. 200 = -20 dB).
      - The attenuation of the right channel (e.g. 200 = -20 dB).
- Insert a [`MulticastSubject`] operator named `SoundCard Commands`.

Run the workflow and press the <kbd>A</kbd> key to play the sound at reduced volume.

> [!TIP]
> Pure tone playback must be stopped explicitly via the [`Stop`] register.

### Exercise 4 - Trigger sound index playback with digital inputs

The `SoundCard` also features digital input channels that can be configured to trigger sound index playback.

:::workflow
![Play Sound Digital Input](../workflows/soundcard-playsound-configureDI.bonsai)
:::

- Connect a TTL signal from another device to the digital input channel `DI0` (5 V tolerant) and `GND` on the `SoundCard`.
- Insert a [`SubscribeSubject`] operator and configure the `Name` property to `SoundCard Events`.
- Insert a [`Take`] combinator and set the `Count` property to 1.
- Insert a [`CreateMessage`] operator, select [`ConfigureDI0Payload`] for the `Payload` property, and set the `ConfigureDI0` property to `StartSound`.
- Insert a second [`CreateMessage`] operator on a new branch, select [`SoundIndexDI0Payload`] for the `Payload` property, and set the `SoundIndexDI0` property to the sound index for playback.
- Combine both messages with a [`Merge`] combinator.
- Insert a [`MulticastSubject`] operator named `SoundCard Commands`.

Run the workflow and send the TTL signal from the other device to trigger sound playback.

> [!WARNING]
> Only sound index playback is supported currently.

> [!TIP]
> The `SoundCard Events` > `Take(1)` is another useful pattern for ensuring that configuration commands are sent as soon as the `SoundCard` has initialized. It also relies on the `DumpRegisters` property being set to `True` in the [`Device`] operator.

<!--Reference Style Links -->
[`AttenuationAndPlaySoundOrFreq`]: xref:Harp.SoundCard.AttenuationAndPlaySoundOrFreq
[`AttenuationAndPlaySoundOrFreqPayload`]: xref:Harp.SoundCard.CreateAttenuationAndPlaySoundOrFreqPayload
[``BehaviourSubject`1``]: xref:Bonsai.Reactive.BehaviorSubject
[`ConfigureDI0Payload`]: xref:Harp.SoundCard.CreateConfigureDI0Payload
[`SoundIndexDI0Payload`]: xref:Harp.SoundCard.CreateSoundIndexDI0Payload
[`CreateMessage`]: xref:Harp.SoundCard.CreateMessage
[`Device`]: xref:Harp.SoundCard.Device
[`DeviceDataWriter`]: xref:Harp.SoundCard.DeviceDataWriter
[`HarpMessage`]: xref:Bonsai.Harp.HarpMessage
[`KeyDown`]: xref:Bonsai.Windows.Input.KeyDown
[`Merge`]: xref:Bonsai.Reactive.Merge
[`MulticastSubject`]: xref:Bonsai.Expressions.MulticastSubject
[`PlaySoundOrFrequency`]: xref:Harp.SoundCard.PlaySoundOrFrequency
[`PlaySoundOrFrequencyPayload`]: xref:Harp.SoundCard.CreatePlaySoundOrFrequencyPayload
[`PublishSubject`]: xref:Bonsai.Reactive.PublishSubject
[`Stop`]: xref:Harp.SoundCard.Stop
[`StopPayload`]: xref:Harp.SoundCard.CreateStopPayload
[`SubscribeSubject`]: xref:Bonsai.Expressions.SubscribeSubject
[`SubscribeWhen`]: xref:Bonsai.Reactive.SubscribeWhen
[`Take`]: xref:Bonsai.Reactive.Take
[`Timer`]: xref:Bonsai.Reactive.Timer