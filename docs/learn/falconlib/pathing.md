# Auto Pathing with FalconLibrary

## Generating paths

Following paths can be Fun and Easy with FalconLibrary. Before starting, the drivetrain must both be charicterized with the [Robot Charicterization Toolsuite](docs/learn/characterization), and Velocity closed loop must already be tuned to get reliable results. 

Paths are generated using the `TrajectoryGenerator` class. To generate a trajectory, use the `DefaultTrajectoryGenerator` unless you have a good reason not to. On a technical note, the splies are evaluated using recursive arc subdivision (from Team 254). These splines are constrained using waypoints represented as `Pose2d`s, maximum velocities/accelerations, and other user specified constraints. Paths can be previewed visually [using FalconDashboard](docs/learn/faclinlib/falcondash). An example trajectory which might be used in robot code follows:

```Kotlin
val baseline = DefaultTrajectoryGenerator.generateTrajectory(
    wayPoints = listOf(
        Pose2d(1.5.feet, 23.feet, 0.degree),
        Pose2d(11.5.feet, 23.feet, 0.degree)
    ),
    constraints = listOf(CentripetalAccelerationConstraint(4.0.feet.acceleration),
    startVelocity = 0.0.feet.velocity,
    endVelocity = 0.0.feet.velocity,
    maxVelocity = 10.0.feet.velocity,
    maxAcceleration = 4.0.feet.acceleration,
    reversed = false
)
```

## Constraining paths

The path generator takes a `List<TimingConstraint<Pose2dWithCurvature>>`, which are used to constrain the trajectory to ensure the robot can safely follow it. The constraints offered by FalconLibrary are:
 - AngularAccelerationConstraint, which bounds angular acceleration below a specified limit
 - CentripetalAccelerationConstraint, which bounds centripetal acceleration below a specified limit
 - DifferentialDriveDynamicsConstraint, which constrains both velocity and accelerations to fit a `DifferentialDrive` at a specified max voltage
 - VelocityLimitRadiusConstraint, which limits velocity to a maximum in a radius around a specified `Translation2d`
 - VelocityLimitRegionConstraint, which limits velocity within a specified `Rectangle2d` (for example, the Hab from 2019)

We used the `DifferentialDriveDynamicsConstraint` and `CentripetalAccelerationConstraint` to initially constrain the trajectory, and used the `VelocityLimitRegionConstraint` to limit Hab velocity and `VelocityLimitRadiusConstraint` to slow the trajectory for vision placement.

## Tracking a trajectory

Path following is governed by a `TrajectoryTracker`, which uses a pre-generated Trajectory and attempts to follow it using a selection of different path following algorithms. The trackers offered by FalconLibrary are:
 - FeedForwardTracker, which uses no feedback about robot pose
 - PurePursuitTracker, which uses the Pure Pursuit algorithm and robot pose
 - RamseteTracker, which uses the RAMSETE time-varying non linear reference controller to correct for disturbances in real time
   - This sentence basically won us Innovation in Control in 2019

To use a trajectory tracker, call the `reset` method on the tracker with the new trajectory to follow, and call the `nextState` method repeatedly to calculate a `TrajectoryTrackerOutput`. The output contains linear and angular velocities and accelerations These outptus should next be fed into your robot's [DifferentialDrive](docs/learn/falconlib/kinematics#DifferentialDrive) model to calculate velocities for the left and right wheels, and the velocities and feed forwards fed to the appropriate motor controllers. This process is further detailed below. Note that the `DifferentialTrackerDriveBase` and `TankDriveSubsystem` contain methods to abstract this calculation away and will take the raw `TrajectoryTrackerOutput` and perform feedforward calculations for you. This example command will follow a path using a drive base which inherits `DifferentialTrackerDriveBase`:

```Kotlin
class StandardTrajectoryTracker(
    val trajectorySource: Source<Trajectory<Time, TimedEntry<Pose2dWithCurvature>>>
) : FalconCommand(DriveSubsystem) {

    private var trajectoryFinished = false
    
    @Suppress("LateinitUsage")
    private lateinit var trajectory: Trajectory<Time, TimedEntry<Pose2dWithCurvature>>

    override fun isFinished() = trajectoryFinished

    override fun initialize() {
        trajectory = trajectorySource()
        DriveSubsystem.trajectoryTracker.reset(trajectory)
        trajectoryFinished = false
        LiveDashboard.isFollowingPath = true
    }
    
    override fun execute() {
        val nextState = DriveSubsystem.trajectoryTracker.nextState(DriveSubsystem.robotPosition)

        DriveSubsystem.setOutput(nextState)

        // update LiveDashboard and FalconDashboard
        val referencePoint = DriveSubsystem.trajectoryTracker.referencePoint
        if (referencePoint != null) {
            val referencePose = referencePoint.state.state.pose

            // Update Current Path Location on Live Dashboard
            LiveDashboard.pathX = referencePose.translation.x / SILengthConstants.kFeetToMeter
            LiveDashboard.pathY = referencePose.translation.y / SILengthConstants.kFeetToMeter
            LiveDashboard.pathHeading = referencePose.rotation.radian
        }

        trajectoryFinished = DriveSubsystem.trajectoryTracker.isFinished
    }
    
    override fun end(interrupted: Boolean) {
        DriveSubsystem.zeroOutputs()
        LiveDashboard.isFollowingPath = false
    }
}
```

## Converting Trajectory Tracker Outputs to robot motion

### Converting outputs to desired velocities and feedforwards with a DifferentialDrive

The process of converting linear and angler velocities and accelerations into individual wheel velocity setpoints and voltages is done using the robot's `DifferentialDrive`. The following example code is from the `DifferentialTrackerDriveBase` interface, which will accept a raw `TrajectoryTrackerOutput` from the `setOutput` method, which solves inverse dynamics to calculate wheel velocities and voltages:

```Kotlin
val dynamics = differentialDrive.solveInverseDynamics(output.differentialDriveVelocity, output.differentialDriveAcceleration)

leftMotor.setVelocity(
        dynamics.wheelVelocity.left * differentialDrive.wheelRadius,
        dynamics.voltage.left
)
rightMotor.setVelocity(
        dynamics.wheelVelocity.right * differentialDrive.wheelRadius,
        dynamics.voltage.right
)
```

However, `DifferentialTrackerDriveBase` contains all of this code in the `setOutput(output: TrajectoryTrackerOutput)` method. You shouldn't ever need to use the above code, but should simply use the `setOutput` method. Furthermore, this relies on an accurate velocity closed loop in addition to calculated feed forwards to accurately track trajectories.

### Tuning a velocity PID loop to track trajectories

The process is similar to any PID loop. However, any feed forward term should be set to zero, as it is replaced by the calculations done by the `DifferentialDrive`. Begin with a small P value, no I or D, and try following a short baseline trajectory. Observe the motor setpoint and actual velocities using [Phoenix Tuner](https://github.com/CrossTheRoadElec/Phoenix-Releases) for Talons and SmartDashboard prints for Spark MAXes. A detailed guide for how to tune a velocity closed loop gain is avalible on the [Phoenix documentation](https://phoenix-documentation.readthedocs.io/en/latest/ch16_ClosedLoop.html?highlight=motion%20magic#dialing-kp).

For a quick and dirty tune for people already familiar with PID tuning: Begin with a P value of 0.05 for talons, and begin by doubling P until high frequency oscillations occur. Next, set D to 10x P, and increase to up to 25x P, while making sure that the motor will still track the setpoint even if it changes rapidly (ex. during deceleration). In 2019, or low gear kp was 0.45 with a kd of 9 (Talon SRXes geared to 8ft/sec), and a high gear kp of 1.2 with a kd of 24. 