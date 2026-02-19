# QuickMovement

The [Harp StepperDriver](https://github.com/harp-tech/device.stepperdriver) features a special `QuickMovement` mode that can be used for extremely fast movements with low trigger latency.

> [!TIP]
> [`StepRelative`](./stepperdriver-controlmotor.md#exercise-7-move-continuously) with position limits may be sufficient for some applications requiring fast movements.

## Prerequisites

- Install the `Bonsai.Windows.Input` package from the Bonsai [package manager](https://bonsai-rx.org/docs/articles/packages.html).
- `QuickMovement` requires firmware `fw0.7-harp1.14` and later. Download the firmware from the [release](https://github.com/harp-tech/device.stepperdriver/releases) page and update it with the [Harp Toolkit](https://github.com/harp-tech/toolkit).
- Connect a stepper motor (`Motor1`) to `Stepper 1` output for these exercises. `QuickMovement` is only supported on `Stepper 1` and `Stepper 2` drivers.
- Set up the [device pattern](./stepperdriver-configuration.md#device-pattern), [configure](./stepperdriver-configuration.md) the device, and [enable](./stepperdriver-configuration.md#enable-and-disable-motor-drivers) the `Motor1` stepper driver.
- Use the [position visualizer](./stepperdriver-controlmotor.md#visualize-position) to visualize the movement.

## Driver Configuration

`QuickMovement` requires the motor driver to be configured for `DynamicMovements`. Set a higher sampling rate for the [`AccumulatedSteps`] event for visualization.

:::workflow
![StepperDriver QuickMovement Driver Configuration](../workflows/stepperdriver-quickmovement-driverconfiguration.bonsai)
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

`QuickMovement` is not yet supported by the `Harp.StepperDriver` Bonsai interface package. Instead, it can be accessed through the generic [`Bonsai.Harp`](../articles/operators.md) interface. With this interface, values for configuration are converted into [`HarpMessage`] commands by using the [`Format (Harp)`] operator. Values have been precalibrated and are defined in millimeters rather than steps. The `QuickMovement` configuration is preloaded into the `StepperDriver` and armed for quick execution.

:::workflow
![StepperDriver QuickMovement Arming](../workflows/stepperdriver-quickmovement-arming.bonsai)
:::

- Insert a [`KeyDown`] source and set the `Filter` property to `7`.

For each of the following registers:
- Insert a [`Float`] expression on a new branch and set the `Value` property to the specific default value.
- Insert a [`Format (Harp)`] operator and configure the following properties:
   - `Format` - Select `Write`.
   - `Register` - Select [`FormatMessagePayload`].
   - `Address` - Set the register address.
   - `PayloadType` - Set to `Float`.

| Register Name                      | Address | Default Values | Description   |    
| ---------------------------------  | ------- | -------------- | ------------- | 
| `Motor1QuickMovementPulseDistance` | 131     | 1.25 / 2.5     | Sets the single pulse distance for a quick movement, in millimeters, for the Motor 1. |
| `Motor1QuickMovementNominalSpeed`  | 133     | 60             | Sets the target speed for a quick movement, in millimeters per second, for the Motor 1. |
| `Motor1QuickMovementInitialSpeed`  | 135     | 2              | Sets the initial speed for a quick movement, in millimeters per second, for the Motor 1. |
| `Motor1QuickMovementAcceleration`  | 137     | 2.5            | Sets the acceleration for a quick movement, in millimeters per second^2, for the Motor 1. | 
| `Motor1QuickMovementDistance`      | 139     | 5 / -5         | Sets the travel distance of a quick movement, in millimeters, for the Motor 1. The sign of the value will determine the direction of movement. |

> [!TIP]
> The equivalent register addresses for `Motor2` are each offset by +1 (e.g. `Motor2QuickMovementPulseDistance` uses register 132).

- Insert a [`Merge`] operator to combine all the commands.
- Insert a [`MulticastSubject`] operator named `StepperDriver Commands`.

Run the workflow and press <kbd>7</kbd> to arm the `QuickMovement`.

You can also adjust the properties, and press <kbd>7</kbd> to change the `QuickMovement` configuration while the workflow is running, which will allow you to test different `QuickMovement` settings.

## Exercise 2: Trigger QuickMovement

To trigger the `QuickMovement`, use a [`CreateMessage (Harp)`] to access the `TriggerQuickMovement` register.

:::workflow
![StepperDriver QuickMovement Trigger](../workflows/stepperdriver-quickmovement-trigger.bonsai)
:::

- Insert a [`KeyDown`] source and set the `Filter` property to `H`.
- Insert a [`CreateMessage (Harp)`] operator and configure the following properties:
   - `Format` - Select `Write`.
   - `Register` - Select [`CreateMessagePayload`].
   - `Address` - Set the register address to 130.
   - `PayloadType` - Set to `U8`.
   - `Value` - Set to 2 for `Motor1`, or 4 for `Motor2`.
- Insert a [`MulticastSubject`] operator named `StepperDriver Commands`.

Run the workflow and press <kbd>H</kbd> to trigger the `QuickMovement`.

<!--Reference Style Links -->
[`AccumulatedSteps`]: xref:Harp.StepperDriver.AccumulatedSteps
[`AccumulatedStepsSamplingRatePayload`]: xref:Harp.StepperDriver.CreateAccumulatedStepsSamplingRatePayload
[`CreateMessage`]: xref:Harp.StepperDriver.CreateMessage
[`CreateMessage (Harp)`]: xref:Bonsai.Harp.CreateMessage
[`CreateMessagePayload`]: xref:Bonsai.Harp.CreateMessagePayload
[`Float`]: xref:Bonsai.Expressions.FloatProperty
[`Format (Harp)`]: xref:Bonsai.Harp.Format
[`FormatMessagePayload`]: xref:Bonsai.Harp.FormatMessagePayload
[`HarpMessage`]: xref:Bonsai.Harp.HarpMessage
[`KeyDown`]: xref:Bonsai.Windows.Input.KeyDown
[`Merge`]: xref:Bonsai.Reactive.Merge
[`Motor1OperationModePayload`]: xref:Harp.StepperDriver.CreateMotor1OperationModePayload
[`MulticastSubject`]: xref:Bonsai.Expressions.MulticastSubject
[`StepRelative`]: xref:Harp.StepperDriver.StepRelative
[`SubscribeSubject`]: xref:Bonsai.Expressions.SubscribeSubject
[`Take`]: xref:Bonsai.Reactive.Take