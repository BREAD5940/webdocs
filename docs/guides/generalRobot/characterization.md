# Characterizing a feedforward model of a joint using Robotpy's characterization toolsuite

The [RobotPy Characterization toolsuite](https://github.com/robotpy/robot-characterization/) is a python based tool used to characterize various linkages on FRC robots. The chariceterization creates a model which can be used to accurately predict the voltage that should be applied to the linkage or drivetrain to achieve a given velocity/acceleration state. This "feedforward" calculation is vital to path following, as it takes the "load" of compensating for physical aspects of robot performance such as static friction and inertial off of the P term of the velocity or position loop and increase performance.

The python tool characterizes drivetrains, arms or (soon) elevators. The characterization tools consist of a python application that runs on the user's PC, and matching robot code that runs on the user's robot. The PC application will send control signals to the robot over network tables, while the robot sends data back to the application. The application then processes the data and determines characterization parameters for the user's robot mechanism, as well as producing diagnostic plots. The tool runs the given linkage forward and backward multiple times to determine it's physical characteristics. Once this is complete, the script will produce three numbers: kV, kA and vStatic. vStatic represents the voltage required to overcome static friction, kV the speed per volt, and Ka the acceleration per volt. The units of velocity and acceleration are the same units that the position and velocity are reported in on the robot side code - for example, if the drivetrain reported distance in meters and velocity in meters per second, kV and kA would be in meters/sec/volt and meters/sec^2/volt.

## Prerequisites (PC side)

The following is required to use the data_logger.py/data_analyzer.py scripts for all of the characterization tools included in this repository:
- Install Python 3.6 on your data gathering computer that will be connected to the robot's network
- Once finished, install pynetworktables, matplotlib, scipy, and statsmodels

On Windows the command to install pynetworktables, matplotlib, scipy, and statsmodels is as follows:

```py -3 -m pip install pynetworktables matplotlib scipy statsmodels```

On Mac with Python 3.7 installed, run 
```python3.7 -m pip install pynetworktables matplotlib scipy statsmodels```

## Running the characterization

To log data, call the corresponding data logger. For example, to characterize a drivetrain, use the command `python3.7 drive-characterization/data_logger.py`. Next, enter the team number (5940) to connect to the robot. Next, enable the robot in autonomous, being careful of obstacles or humans in the way. The robot will drive forward and backward twice, collecting data. The data analyzer will automatically be run and constants calculated. If you wish to analyze a previous run, call `python3.7 data_analyzer.py PATH_TO_LOG.json`. 

## Using with 254 drive models

Once the left and right kV, kA and vStatic have been calculated, they can be used to create a model of the drivetrain using the `DifferentialDrive` class. The following example is from our 2019 code for Croissant:

```Kotlin
private const val kRobotMass = (50.0 /* Robot, kg */ + 5.0 /* Battery, kg */ + 2.0 /* Bumpers, kg */).toDouble()
private const val kRobotMomentOfInertia = 10.0 // kg m^2 // TODO Tune
private const val kRobotAngularDrag = 12.0 // N*m / (rad/sec)

private val kWheelRadius = (2.0).inch // falconlib units
private val kTrackWidth = (26.0).inch // falconlib units

private const val kVDriveLeftLow = 0.274 // Volts per radians per second
private const val kADriveLeftLow = 0.032 // Volts per radians per second per second
private const val kVInterceptLeftLow = 1.05 // Volts

private const val kVDriveRightLow = 0.265 // Volts per radians per second
private const val kADriveRightLow = 0.031 // Volts per radians per second per second
private const val kVInterceptRightLow = 1.02 // Volts

private val kLeftTransmissionModelLowGear = DCMotorTransmission(1 / kVDriveLeftLow,
        kWheelRadius.meter * kWheelRadius.meter * kRobotMass / (2.0 * kADriveLeftLow),
        kVInterceptLeftLow)

private val kRightTransmissionModelLowGear = DCMotorTransmission(1 / kVDriveRightLow,
        kWheelRadius.meter * kWheelRadius.meter * kRobotMass / (2.0 * kADriveRightLow),
        kVInterceptRightLow)

val kLowGearDifferentialDrive = DifferentialDrive(kRobotMass, kRobotMomentOfInertia,
        kRobotAngularDrag, kWheelRadius.meter, kTrackWidth.meter / 2.0, kLeftTransmissionModelLowGear, kRightTransmissionModelLowGear)
```
