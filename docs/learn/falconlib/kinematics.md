# 254 Kinematic classes

FalconLibrary offers the `DCMotorTransmission` model from Team 254, as well as `DifferentialDrive`, which are used extensively for feedforward calculations in [path following](docs/learn/falconlib/pathing). A `DCMotorTransission` is used to model a motor connected to something which rotates such as a drivetrain wheel connected to a motor (or motors), a moving arm or elevator. The `DifferentialDrive` models a drivetrain and contains methods for converting between a `ChassisState` of linear and angular velocities and wheel speeds for each motor. For calculating the constants used in the `DCMotorTransmission`, see [the characterization article](docs/learn/characterization).

## DCMotorTransmission

The `DCMotorTransmission` class is constructed with the `speedPerVolt`, the `torquePerVolt` and the `frictionVoltage` of the linkage, and contains methods for calculating speeds from voltages, torques from voltages and voltages from torques. For example, the `DCMotorTransmission` of one side of a drivetrain would be constructed as follows:
```Kotlin
val kRightTransmissionModelLowGear = DCMotorTransmission(
        1 / kVDrive,
        kWheelRadius.meter * kWheelRadius.meter * kRobotMass /* kg */ / (2.0 * kADrive),
        kVIntercept)
```

## DifferentialDrive

The `DifferentialDrive` class is a model of a robot's drivetrain, composed of two DCMotorTransmissions, one for each side, as well as the robot's mass, moment of inertia, angular drag, wheel radius, and track width. The class can make calculations decomposing a desired `ChassisState` containing linear (forward/backward) and angular (turn left/right) velocities into a `WheelState` containing speeds for the left and right motors, calcuating the robot's `ChassisState` from the current wheel speeds, and more torque calculations. These are used in the `DifferentialTrackerDriveBase` to calculate wheel speeds and feedforwards from commanded ChassisStates output by the trajectory tracker. The methods contained include:
- solveForwardKinematics, which calculates chassis motion from wheel states (either velocity or acceleration)
- solveInverseKinematics, which gets wheel motion from chassis motion (either velocity or acceleration)
- solveForwardDynamics, which solves forward dynamics for torques and accelerations given chassis motion and voltages
- getVoltagesFromkV, which get the voltage simply from the Kv and the friction voltage of the transmissions
- solveForwardDynamics
  - Solve forward dynamics for torques and accelerations given wheel velocities and voltages
  - Solve forward dynamics for torques and accelerations given wheel velocities, chassis velocities, curvature and wheel voltages 
- solveInverseDynamics
  - Solves inverse dynamics for torques and voltages given chassis velocity and acceleration
  - Solves inverse dynamics for torques and voltages given wheel velocities and accelerations
- solveInverseDynamics for torques and voltages given wheel and chassis velocities and acceleration, curvature, and the derivative of curvature
- getMaxAbsVelocity, which solves for the max absolute velocity that the drivetrain is capable of given a max voltage and curvature
- getMinMaxAcceleration, which gets the min/max accelerations given a velocity, curvature and maximum voltage

whew