# QuickMovement

The [Harp StepperDriver](https://github.com/harp-tech/device.stepperdriver) features a `QuickMovement` mode that can be used for extremely fast movements with low trigger latency.

> [!TIP]
> [`StepRelative`](./stepperdriver-controlmotor.md#exercise-7-move-continuously) with position limits may be sufficient for some applications requiring fast movements.

## Prerequisites

- Install the `Bonsai.Windows.Input` package from the Bonsai [package manager](https://bonsai-rx.org/docs/articles/packages.html).
- `QuickMovement` requires firmware `fw0.7-harp1.14` and later. Download the firmware from the [release](https://github.com/harp-tech/device.stepperdriver/releases) page and update it with the [Harp Toolkit](https://github.com/harp-tech/toolkit).
- Connect a stepper motor (`Motor1`) to `Stepper 1` output for these exercises. `QuickMovement` is only supported on `Stepper 1` and `Stepper 2` drivers.
- Set up the [device pattern](./stepperdriver-configuration.md#device-pattern), [configure](./stepperdriver-configuration.md) the device, and [enable](./stepperdriver-configuration.md#enable-and-disable-motor-drivers) the `Motor1` stepper driver.
- Use the [position visualizer](./stepperdriver-controlmotor.md#visualize-position) to visualize the movement.

## Driver Configuration for QuickMovement

`QuickMovement` requires the motor driver to be configured for `DynamicMovements`. It also helps to set a higher sampling rate for the [`AccumulatedSteps`] event for visualization.

:::workflow
![StepperDriver QuickMovement Driver Configuration](../workflows/stepperdriver-quickmovements-driverconfiguration.bonsai)
:::

- Insert a [`SubscribeSubject`] operator named `StepperDriver Events`.
- Insert a [`Take`] combinator and set the `Count` property to 1.
- Insert a [`CreateMessage`] operator and configure these properties:
   - `Payload` - Select [`Motor1OperationModePayload`].
   - `Motor1OperationMode` - Set to `DynamicMovements`.
- Insert a [`CreateMessage`] operator on another branch and configure these properties:
   - `Payload` - Select [`AccumulatedStepsSamplingRatePayload`].
   - `AccumulatedStepsSamplingRate` - Set to `Rate100Hz`.
- Combine the two messages with a [`Merge`] combinator.
- Insert a [`MulticastSubject`] operator named `StepperDriver Commands`.

## Exercise 1: Arm QuickMovement 

## Exercise 2: Trigger QuickMovement


<!--Reference Style Links -->
[`AccumulatedSteps`]: xref:Harp.StepperDriver.AccumulatedSteps
[`AccumulatedStepsSamplingRatePayload`]: xref:Harp.StepperDriver.CreateAccumulatedStepsSamplingRatePayload
[`CreateMessage`]: xref:Harp.StepperDriver.CreateMessage
[`Merge`]: xref:Bonsai.Reactive.Merge
[`Motor1OperationModePayload`]: xref:Harp.StepperDriver.CreateMotor1OperationModePayload
[`MulticastSubject`]: xref:Bonsai.Expressions.MulticastSubject
[`StepRelative`]: xref:Harp.StepperDriver.StepRelative
[`SubscribeSubject`]: xref:Bonsai.Expressions.SubscribeSubject
[`Take`]: xref:Bonsai.Reactive.Take