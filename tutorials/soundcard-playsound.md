# Play Sound

> [!WARNING]
> When adding these operators to the workflow, make sure to use the device-specific versions, e.g. `Device (Harp.SoundCard)` instead of `Device (Harp)`. If correctly selected, the names of these operators in the  workflow panel will change to reflect either the name of the device or the selected register.

## Prerequisites

- Install the `Bonsai.Windows.Input` package from the Bonsai [package manager](https://bonsai-rx.org/docs/articles/packages.html).

## Device pattern

Set up the standard Harp [device pattern](../articles/operators.md#device-pattern) to initialize the device, log data, receive events, and send commands to the `SoundCard`.

- Insert a [`Device`] operator, set the `PortName` property to the communications port for the device.
- Insert a [`DeviceDataWriter`] and set the `Path` property (e.g. `SoundCard.harp`). This will log the data in the standard `.harp` format that can be analyzed with [`harp-python`](../python.md).
- Insert a [`PublishSubject`] operator and name it `SoundCard Events`.
- Right-click the [`Device`] operator, select `Create Source (Bonsai.Harp.HarpMessage)` > [`BehaviorSubject`]. Name the generated ``BehaviourSubject`1`` operator `SoundCard Commands`. Connect it as input to the [`Device`] operator.

## Play sound index

<!--Reference Style Links -->
[`Device`]: xref:Harp.SoundCard.Device
[`DeviceDataWriter`]: xref:Harp.SoundCard.DeviceDataWriter
[`PublishSubject`]: xref:Bonsai.Reactive.PublishSubject
[`BehaviorSubject`]: xref:Bonsai.Reactive.BehaviorSubject