# FalconLibrary's Motor Wrappers

## What do they do?
FalconLibrary offers the `FalconMotor` interface as a common interface for motors from both REV Robotics and CTRE. The interface contains methods for simply setting motor output percent to setting closed loop position setpoints and motion profile constraints, and contains an abstracted `FalconEncoder` used to track motor position. Because closed loop control is so closely integrated with the `FalconMotor`, all motors have the ability to get the current position through an integrated encoder and perform conversions between native sensor units and real-world units automatically. For more on encoders, see [this article on encoders on frc-docs](https://frc-docs.readthedocs.io/en/develop/docs/hardware/sensors/encoders-hardware.html). The "raw" native unit position conversion to real-world units (and vice versa) are done by a subclass of `NativeUnitModel` such as `NativeUnitLengthModel`, `NativeUnitRotationModel`, or the `DefaultNativeUnitModel` for motors not attached to any linkage or encoder. The `SlopeNativeUnitModel` can be used for drivetrains to not only track distance but also calculate wheel radius. Note that the implied units for any motor are in Meters/Radians/Seconds.

```kotlin
val nativeUnitModel = NativeUnitLengthModel(
    4096.nativeUnits, // per rotation
    2.inch) // wheel radius

val falconSRX = FalconSRX(5, nativeUnitModel) // to create a FalconSRX on CAN port 5 with the provided nativeUnitModel

falconSRX.setPosition(/* position */ 0.5, /* arbitrary feed forward */ 0.0) // set the wanted position setpoint 0.5 meters

// You can still use the wrapped talonSRX to call things specific to Talons
val pulseWidthPosition = falconSRX.talonSRX.sensorCollection.pulseWidthPosition
```