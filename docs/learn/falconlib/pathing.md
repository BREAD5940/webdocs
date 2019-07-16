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

## Following Paths

Path following is governed by a `TrajectoryTracker`, which uses a pre-generated Trajectory and attempts to follow it using a selection of different path following algorithms. The trackers offered by FalconLibrary are:
 - FeedForwardTracker, which uses no feedback about robot pose
 - PurePursuitTracker, which uses the Pure Pursuit algorithm and robot pose
 - RamseteTracker, which uses the RAMSETE time-varying non linear reference controller to correct for disturbances in real time
   - This sentence basically won us Innovation in Control in 2019

To use a trajectory tracker, call the `reset` method on the tracker with the new trajectory to follow, and call the `nextState` method repeatedly to calculate Chassis speeds (TODO link Kinematics article). These should next be fed into your robot's `DifferentialDrive` model to calculate velocities for the left and right wheels, and the velocities and feed forwards fed to the appropriate motor controllers. Note that the `TrajectoryTrackerDriveBase` and `TankDriveSubsystem` contain methods to abstract this calculation away and will take the raw `TrajectoryTrackerOutput` and perform feedforward calculations for you. This example command will follow a path using a drive base which inherits `TrajectoryTrackerDriveBase`.

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