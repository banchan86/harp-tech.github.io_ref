# Control Motor

The follow exercises will demonstrate how to control motor speed and rotational position with the [Harp StepperDriver](https://github.com/harp-tech/device.stepperdriver).

> [!WARNING]
> Ensure that you are familiar with the operation of the `StepperDriver` and motors before connecting external loads. Consider using end-of-travel switches in conjunction with a [digital input](xref:Harp.StepperDriver.CreateEnableDigitalInputsPayload) [configuration](xref:Harp.StepperDriver.CreateInput0OpModePayload). Improper use of the `StepperDriver` and motors may result in damage to equipment.

## Prerequisites

- Install the `Bonsai.Windows.Input` package from the Bonsai [package manager](https://bonsai-rx.org/docs/articles/packages.html).
- Connect a stepper motor to the `Stepper 1` output on the `StepperDriver`, set up the [device pattern](./stepperdriver-configuration.md#device-pattern), [configure](./stepperdriver-configuration.md) the device, and [enable](./stepperdriver-configuration.md#enable-and-disable-motors) the `Motor1` stepper driver.

## Position visualizer

For these exercises, it helps to track the rotational position of the motor. The `StepperDriver` can broadcast a stream of [`AccumulatedSteps`] events, which can be displayed in a visualizer for this purpose.

:::workflow
![StepperDriver Position Visualizer](../workflows/stepperdriver-controlmotor-visualizer.bonsai)
:::

Enable the [`AccumulatedSteps`] event register and configure the dispatch rate:

- Insert a [`SubscribeSubject`] operator named `StepperDriver Events`.
- Insert a [`Take`] combinator and set the `Count` property to 1.
- Insert a [`CreateMessage`] operator and configure these properties: 
   - `Payload` - Set it to [`AccumulatedStepsSamplingRatePayload`].
   - `AccumulatedStepsSamplingRate` - Set it to the desired sampling rate (e.g. `Rate10Hz` for coarse movements).
- Insert a [`MulticastSubject`] operator named `StepperDriver Commands`.

Read the [`AccumulatedSteps`] and display it in a visualizer:

- Insert a [`SubscribeSubject`] operator named `StepperDriver Events`.
- Insert a [`Parse`] operator and set the `Register` property to [`AccumulatedSteps`].
- Right-click on the [`Parse`] operator, select "Output (Harp.StepperDriver.AccumulatedStepsPayload)" > "Motor1" from the context menu.
- Insert a [`VisualizerWindow`] node.

## Exercise 1: Set acceleration profile

Motor motion is driven by a series of step pulses. Speed can be controlled by specifying the interval between steps (in μs). For any given movement, the acceleration profile is defined by three parameters:

- the initial/final step interval
- the target step interval at nominal speed
- the change in interval per step during the acceleration/deceleration phase

:::workflow
![StepperDriver Acceleration Profile](../workflows/stepperdriver-controlmotor-acceleration.bonsai)
:::

- Insert a [`KeyDown`] source and set the `Filter` property to `3`. 
- Insert a [`CreateMessage`] operator and configure these properties:
   - `Payload` - Set it to [`Motor1MaximumStepIntervalPayload`].
   - `Motor1MaximumStepInterval` - Set this to the initial/final step interval (e.g. 2000).
- Insert a [`CreateMessage`] operator on a new branch and configure these properties:
   - `Payload` - Set it to [`Motor1StepIntervalPayload`].
   - `Motor1StepInterval` - Set this to the target step interval (e.g. 250).
- Insert a [`CreateMessage`] operator on a new branch and configure these properties:
   - `Payload` - Set it to [`Motor1StepAccelerationIntervalPayload`].
   - `Motor1StepAcceleration` - Set this to the change in step interval (e.g. 10).
- Combine the three messages with a [`Merge`] combinator.
- Insert a [`MulticastSubject`] operator named `StepperDriver Commands`.

Run the workflow and press the <kbd>3</kbd> key to set the acceleration profile. You can also adjust the properties, and press the <kbd>3</kbd> key to update the acceleration profile while the workflow is running. This allows you to test different speeds with the movement commands.

## Exercise 2: Move relative steps

The [`MoveRelative`] register moves the motor by a specified number of steps from its current position. Positive values move in one direction, negative values in the other.

:::workflow
![StepperDriver Move Relative](../workflows/stepperdriver-controlmotor-moverelative.bonsai)
:::

To move the motor in the positive direction:

- Insert a [`KeyDown`] source and set the `Filter` property to `A`.
- Insert a [`CreateMessage`] operator and configure these properties:
   - `Payload` - Set it to [`Motor1MoveRelativePayload`].
   - `Motor1MoveRelative` - Set this to the number of steps to move (e.g. 3000).
- Insert a [`MulticastSubject`] operator named `StepperDriver Commands`.

To move the motor in the negative direction, set up a separate pipeline:

- Insert a [`KeyDown`] source and set the `Filter` property to `S`.
- Insert a [`CreateMessage`] operator and configure these properties:
   - `Payload` - Set it to [`Motor1MoveRelativePayload`].
   - `Motor1MoveRelative` - Set this to a negative value (e.g. -3000).
- Insert a [`MulticastSubject`] operator named `StepperDriver Commands`.

Run the workflow and press <kbd>A</kbd> to move the motor forward and <kbd>S</kbd> to move it back. Observe the accumulated steps in the visualizer to track the motor's position. 

**Optional**: Change the acceleration profile in the previous exercise (e.g. increase the initial and target step interval). Rerun the exercise. What do you observe?

> [!TIP]
> To move multiple motors simultaneously, use the [`MoveRelativePayload`].

<!--Reference Style Links -->
[`AccumulatedSteps`]: xref:Harp.StepperDriver.AccumulatedSteps
[`AccumulatedStepsSamplingRatePayload`]: xref:Harp.StepperDriver.CreateAccumulatedStepsSamplingRatePayload
[``BehaviourSubject`1``]: xref:Bonsai.Reactive.BehaviorSubject
[`CreateMessage`]: xref:Harp.StepperDriver.CreateMessage
[`Device`]: xref:Harp.StepperDriver.Device
[`HarpMessage`]: xref:Bonsai.Harp.HarpMessage
[`KeyDown`]: xref:Bonsai.Windows.Input.KeyDown
[`Merge`]: xref:Bonsai.Reactive.Merge
[`MoveRelative`]: xref:Harp.StepperDriver.MoveRelative
[`MoveRelativePayload`]: xref:Harp.StepperDriver.CreateMoveRelativePayload
[`Motor1MoveRelativePayload`]: xref:Harp.StepperDriver.CreateMotor1MoveRelativePayload
[`Motor1StepAccelerationIntervalPayload`]: xref:Harp.StepperDriver.CreateMotor1StepAccelerationIntervalPayload
[`Motor1MaximumStepIntervalPayload`]: xref:Harp.StepperDriver.CreateMotor1MaximumStepIntervalPayload
[`Motor1Motor1StepInterval`]: xref:Harp.StepperDriver.CreateMotor1StepIntervalPayload
[`MulticastSubject`]: xref:Bonsai.Expressions.MulticastSubject
[`Parse`]: xref:Harp.StepperDriver.Parse
[`PublishSubject`]: xref:Bonsai.Reactive.PublishSubject
[`SubscribeSubject`]: xref:Bonsai.Expressions.SubscribeSubject
[`Take`]: xref:Bonsai.Reactive.Take
[`Timer`]: xref:Bonsai.Reactive.Timer
[`VisualizerWindow`]: xref:Bonsai.Design.VisualizerWindow