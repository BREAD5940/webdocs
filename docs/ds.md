# DriverStation
This document outlines various features of DriverStation and its networking protocol

## Using DriverStation
 - [Viewing Laptop-side Logs](https://wpilib.screenstepslive.com/s/currentCS/m/troubleshooting/l/599678-driver-station-log-file-viewer)
 - [Using the logs to prove that a brownout happened](https://wpilib.screenstepslive.com/s/currentCS/m/troubleshooting/l/599749-roborio-brownout-and-understanding-current-draw)

## Comms Protocol
 - [DS -> RIO](https://frcture.readthedocs.io/en/latest/driverstation/ds_to_rio.html)
 - [RIO -> DS](https://frcture.readthedocs.io/en/latest/driverstation/rio_to_ds.html)
 - [DS -> FMS](https://frcture.readthedocs.io/en/latest/driverstation/ds_to_fms.html)
 - [FMS -> DS](https://frcture.readthedocs.io/en/latest/driverstation/fms_to_ds.html)

## The "Secret DS protocol"
There is a protocol built into the driverstation that is used by gradleRIO and NetowrkTables to find the robot's ip address. To read this data, open a TCP connection to `localhost` on port `1742` on a computer running driverstation. A JSON string will be returned containing the magical data!

This was found in the [NetworkTables Native library](https://github.com/wpilibsuite/allwpilib/blob/221011494d202770ad275c88cd7380119505e65d/ntcore/src/main/native/cpp/DsClient.cpp#L73-L144)

<!-- Global site tag (gtag.js) - Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=UA-139497732-2"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'UA-139497732-2');
</script>
