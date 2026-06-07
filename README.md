# ChronosBackend

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![FRC](https://img.shields.io/badge/FRC-WPILib_2026-red.svg)](https://wpimath.wpi.edu/)
[![JitPack](https://jitpack.io/v/Daboss-1/ChronosBackend.svg)](https://jitpack.io/#Daboss-1/ChronosBackend)

`ChronosBackend` is a lightweight, high-performance Java utility library designed for FRC robot projects. It serves as the official telemetry and command orchestration backend for the **Chronos Dashboard**. Utilizing WPILib's native architecture, NetworkTables v4, and reflective metadata indexing, it enables bidirectional dashboard binding, pre-match diagnostic verification, smart `.wpilog` telemetry streaming, and automated structural integration with AdvantageScope.

---

## Key Features

* **Automated Data Logging (`MatchRecorder`)**: Hooks into `DriverStation` lifecycle states to automatically trigger `DataLogManager` logging, writing optimized binary `.wpilog` files to a flash drive or flash memory instantly when a match starts.
* **AdvantageScope Structural Sync**: Flattens and maps active `Pose2d` arrays, match time elapsed markers, and selected auto routines directly to `/AdvantageKit/RealOutputs` at 20Hz for immediate visualization without overhead.
* **Remote Command & Keybind Execution**: Intercepts structural action queues across NetworkTables to trigger, lifecycle-track, and cancel commands safely inside the robot's `CommandScheduler`.
* **Annotation-Driven Run-Time Tuning**: Exposes robot fields to immediate dashboard runtime tuning via simple `@DashboardTunable` reflection fields on active class systems.
* **Telemetry Routing Matrix**: Structured handlers for multi-tab streaming of numbers, booleans, live strings, and dynamic camera streams (including native Limelight mapping).
* **Pre-Match Checklists & Toasts**: Unified interfaces to verify system health prior to a match alongside structured real-time alert priority states.

---

## Installation

`ChronosBackend` is distributed as an official FRC Vendor Dependency via JitPack architecture. 

### Online Installation (VS Code)

1. Open your FRC robot project directory in **VS Code**.
2. Press `Ctrl+Shift+P` (Windows/Linux) or `Cmd+Shift+P` (Mac) to trigger the Command Palette.
3. Select **WPILib: Manage Vendor Libraries**.
4. Choose **Install new libraries (online)**.
5. Paste the raw configuration endpoint configuration below and hit **Enter**:
   ```text
   [https://raw.githubusercontent.com/Daboss-1/ChronosBackend/main/ChronosBackend.json](https://raw.githubusercontent.com/Daboss-1/ChronosBackend/main/ChronosBackend.json)
   ```

### Dependency Sync

Run a compilation build sequence to instruct GradleRIO to fetch and cache the internal `.jar` library binaries:
```bash
./gradlew build
```

---

## Initialization

To bring the automated recording matrix and tracking pipelines online, initialize the `MatchRecorder` subsystem single instance within your core robot runtime bootstrap lifecycle.

Open your `RobotContainer.java` or `Robot.java` initialization sequence and execute a registration sequence:

```java
import com.github.daboss_1.MatchRecorder;

public class RobotContainer {
    
    public RobotContainer() {
        // Automatically boots DataLogManager, begins NT structural capture, 
        // and synchronizes the 20Hz AdvantageScope stream infrastructure
        MatchRecorder.INSTANCE.register();
        
        configureButtonBindings();
    }
}
```

---

## Deep-Dive Usage & API Examples

### 1. Static and Supplier Telemetry Mapping
Publish real-time telemetry variables to custom named tabs. Values can be bound via continuous evaluation lambda suppliers.

```java
import com.github.daboss_1.Dashboard;

// Stream dynamic motor numeric telemetry to a specific dashboard tab
Dashboard.INSTANCE.putNumber("Drive", "LeftVelocity", () -> m_drive.getLeftEncoderVelocity());

// Stream boolean diagnostic metrics
Dashboard.INSTANCE.putBoolean("Intake", "HasNote", () -> m_intake.sensorTriggered());

// Stream text execution state indicators
Dashboard.INSTANCE.putString("Shooter", "State", () -> m_shooter.getCurrentStateName());

// Route an active camera stream link directly to the video canvas layout matrix
Dashboard.INSTANCE.putCameraStream("Vision", "DriverCam", "http://10.TE.AM.11:1181/?action=stream");
Dashboard.INSTANCE.putLimelightStream("Vision", "limelight-front");
```

### 2. On-The-Fly Property Modification (Tunables)
Expose variables to interactive dashboard widgets to allow dynamic runtime configuration adjustments on the field.

```java
// Native property tuning hook with variable state delta interception
Dashboard.INSTANCE.putNumberTunable("Tuning", "Shooter_kP", 0.004, (newKP) -> {
    m_shooter.updatePIDControllerKP(newKP);
});

Dashboard.INSTANCE.putBooleanTunable("Testing", "EnableTestMode", false, (testState) -> {
    m_systems.toggleInternalTestSequence(testState);
});
```

### 3. Reflective Subsystem Tuning (`@DashboardTunable`)
Rather than mapping individual fields manually, annotate your subsystem primitive variables directly. Registering the parent instance will automatically configure bidirectional NetworkTable mapping bindings.

```java
import com.github.daboss_1.Dashboard;
import com.github.daboss_1.Dashboard.DashboardTunable;
import com.github.daboss_1.Dashboard.DashboardTunableConstants;
import edu.wpi.first.wpilibj2.command.SubsystemBase;

@DashboardTunableConstants(name = "FlywheelConstants", tab = "Developer")
public class ShooterSubsystem extends SubsystemBase {

    @DashboardTunable(name = "TargetRPM")
    private double targetRPM = 3500.0;

    @DashboardTunable(name = "IsLaserEnabled")
    private boolean laserOverride = true;

    public ShooterSubsystem() {
        // Automatically discovers fields across hierarchy models and hooks NT updates
        Dashboard.INSTANCE.register(this);
    }
}
```

### 4. Remote Command Orchestration & Smart Keybinds
Map commands directly to named dashboard action nodes or trigger active keybind maps dynamically over NetworkTables.

```java
import com.github.daboss_1.Dashboard;
import edu.wpi.first.wpilibj2.command.Commands;

// Map a specific autonomous command string target to the dashboard selection hub
Dashboard.INSTANCE.putAutonomousCommand("ThreeNoteAuto", "Executes three note collection", m_drive.getAutoRoutine());
Dashboard.INSTANCE.putDefaultAutonomousCommand("DoNothing", "Standard safe baseline fallback", Commands.none());

// Map complex inline execution sequences directly to structural button arrays
Dashboard.INSTANCE.putCommand("OperatorActions", "PurgeSystem", m_intake.purgeCommand());

// Put interactive Keybind monitors equipped with safety execution lifecycles
Dashboard.INSTANCE.putKeybind("F", "Toggles structural target tracking locking", 
    m_vision.lockTargetCommand(), 
    m_vision.releaseLockCommand()
);
```

### 5. Multi-Robot Field Positional Mapping
Map robot locations or vision target detections cleanly across target mapping assets on the frontend canvas layout.

```java
import com.github.daboss_1.Dashboard;
import edu.wpi.first.math.geometry.Pose2d;

// Continuously stream the main odometry position tracking state
Dashboard.INSTANCE.putField("MatchMap", "Field")
                 .withRobot("MainBot", () -> m_drive.getPose());

// Publish distinct peripheral elements or visualization references side-by-side
Dashboard.INSTANCE.putRobot("MatchMap", "Field", "GhostTarget", () -> new Pose2d(5.0, 3.0, m_angle));
```

### 6. Diagnostics, Pre-Match Checklists, and Critical Alerts
Manage pre-match testing checklists and register active toast alert hooks for the drive team.

```java
// Register structured checklist nodes evaluating state behaviors
Dashboard.INSTANCE.putChecklistItem("Pneumatics Charge", 
    () -> m_pcm.getPressure() > 60.0, 
    () -> "Current PSI: " + m_pcm.getPressure()
);

// Register highly visible real-time error toasts 
Dashboard.INSTANCE.putAlert(
    "CAN_BUS_OVERLOAD", 
    "critical", 
    "CAN bus usage utilization threshold crossed past 85%!", 
    () -> m_pdp.getBusUtilization() > 0.85
);
```

---

## NetworkTables Structural Architecture

Data layout mapping hierarchy trees follow a rigid pathing outline across NetworkTables v4 specifications to simplify frontend parsing:

```text
/
├── ChronosDashboard/            <-- Configurable Global Dashboard Telemetry Tree
│   ├── battery/                 
│   │   └── voltage              [Double] Current operational battery state
│   ├── selectedAutonomous/      
│   │   └── Match                [String] Active selected autonomous identifier
│   ├── autonomousCommands/      [Table] Struct representations of auto paths
│   ├── commands/                [Table] Input trigger pipelines & remote hooks
│   ├── fields/                  [Table] Coordinate data points for canvas arrays
│   ├── tunableNumbers/          [Table] Modification vectors for numbers
│   ├── tunableBooleans/         [Table] Modification vectors for states
│   └── advantagescope/          
│       └── ready                [Boolean] State status verification tag
│
├── NFRDashboard/
│   └── recorder/                <-- MatchRecorder Status Nodes
│       ├── active               [Boolean] Execution indicator status
│       └── filename             [String] Output destination target file index
│
└── AdvantageKit/
    └── RealOutputs/             <-- Flattened AdvantageScope Direct Streams
        ├── Timestamp            [Double] Elapsed relative run time metrics
        ├── AutoRoutine          [String] Current cached auto routine name
        └── Drive/
            └── Pose             [DoubleArray] Flattened coordinate array [x, y, heading_rad]
```

---

## Modifying & Re-Deploying the Backend

If you fork or pull this backend project to adjust layout tables or implement native subsystems for your own custom implementation, handle releases through your repository line.

1. Ensure changes match internal packages: `package com.github.daboss_1;`
2. Validate files compile cleanly locally:
   ```bash
   ./gradlew compileJava
   ```
3. Commit adjustments and push files to your public GitHub repository:
   ```bash
   git add .
   git commit -m "Optimize dashboard action orchestration schemas"
   git push origin main
   ```
4. Create a new formal **Release Tag** (e.g., `v1.0.2`) on GitHub.
5. Re-index and pull your distribution tag down via the **JitPack UI** console to finalize cloud compilation distribution blocks.
