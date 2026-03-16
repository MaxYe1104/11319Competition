# Agent Guidelines for FRC Robot Code Repository

## IMPORTANT: FRC-SPECIFIC KNOWLEDGE REQUIREMENT

**BEFORE CODING ANY ROBOT FEATURE, YOU MUST READ THE RELEVANT DOCUMENTATION.** This is an FRC (FIRST Robotics Competition) robot code repository using specialized robotics frameworks. Agents typically lack FRC knowledge, so you must study these frameworks first:

### **Core FRC Frameworks (MUST READ BEFORE CODING):**

1. **WPILib** (`docs/agent/WPILib/`) - Foundation of everything
   - **RST files** covering robotics utilities, hardware APIs, control systems
   - **Purpose**: Core robotics library providing motor controllers, sensors, command framework, NetworkTables, kinematics, and advanced control systems

2. **PathPlanner** (`docs/agent/PathPlanner/`) - Autonomous path planning
   - **Markdown files** covering GUI and library usage
   - **Purpose**: Motion planning and autonomous path generation with pathfinding algorithms and trajectory following

3. **Phoenix6** (`docs/agent/Phoenix6/`) - CTRE hardware control library
   - **RST files** covering motor controllers, sensors, and CAN bus communication
   - **Purpose**: Control CTRE hardware including TalonFX motors, Pigeon2 IMU, CANCoder encoders, and CTRE Swerve via CAN bus

### **Command-based robot framework**

The robot uses WPILib's **command-based programming paradigm**, which structures robot code around reusable commands and subsystems. Understanding this framework is essential for modifying robot behavior:

**Boot Sequence:**
1. **Robot Initialization**: `Robot.java` extends `TimedRobot` (or `Robot` base class)
2. **RobotContainer Creation**: `RobotContainer.java` constructor runs, creating all subsystems and configuring commands
3. **Subsystem Registration**: All subsystems are registered with the `CommandScheduler`
4. **Command Binding**: Commands are bound to joystick buttons, triggers, and autonomous events
5. **Main Loop Entry**: Robot enters the main periodic loop

**Runtime Execution Flow:**
```
Robot Boots → RobotContainer Init → Subsystems Registered → Commands Bound
    ↓
Main Loop (50Hz/20ms period)
    ↓
Command Scheduler Runs (every cycle)
    ├── Executes scheduled commands
    ├── Updates subsystem periodic methods
    ├── Checks trigger conditions
    └── Schedules new commands when triggers activate
```

**Key Components:**
- **Subsystems**: Represent physical components (drive, intake, arm) with hardware control methods
- **Commands**: Represent actions (drive forward, shoot, climb) that use subsystems
- **Triggers**: Events (button presses, sensor readings, timers) that schedule commands
- **CommandScheduler**: Central coordinator that runs commands and updates subsystems

**Command Lifecycle:**
1. **Scheduled**: When a trigger activates or command is manually scheduled
2. **Initialized**: `initialize()` method runs once
3. **Executing**: `execute()` method runs every cycle (20ms)
4. **Ending**: `end()` method runs when command finishes (interrupted or completed)
5. **Finished**: Command removed from scheduler

**Common Patterns:**
- **Default Commands**: Run continuously when no other command requires the subsystem
- **Parallel Commands**: Multiple commands running simultaneously
- **Sequential Commands**: Commands running one after another
- **Conditional Commands**: Commands that run based on runtime conditions

### **CTRE Phoenix6 Framework Concepts**

The Phoenix6 library controls CTRE hardware via **CAN bus** (Controller Area Network). Key concepts:

**CAN Bus Communication:**
- Devices communicate over a shared CAN bus at 1Mbps
- Each device has a unique CAN ID (0-62 for standard, 0-63 for CANivore)
- Messages are prioritized by CAN ID (lower ID = higher priority)

**Status Signals:**
- Asynchronous data streams from hardware (position, velocity, faults)
- Use `getStatusSignal()` to subscribe to signals
- Signals update at hardware-defined rates (1ms-100ms)
- Must call `refresh()` or use `getValue()` with timeout

**Control Output:**
- Send control requests to hardware (position, velocity, duty cycle)
- Use `setControl()` with request objects
- Supports open-loop, closed-loop, motion profiling
- Requests are queued and executed by hardware

**Key CTRE Hardware Devices:**

1. **TalonFX Motor Controller**:
   - Brushless Falcon 500/ Kraken motor control
   - Integrated encoder (2048 CPR via Hall sensors)
   - Supports FOC (Field Oriented Control)
   - Built-in PID control with feedforward
   - Motion Magic for trapezoidal motion profiles
   - Current limiting and voltage compensation

2. **Pigeon2 IMU Module**:
   - 9-axis inertial measurement unit
   - Gyro, accelerometer, magnetometer
   - Yaw, pitch, roll measurements
   - Built-in sensor fusion
   - Mounting calibration
   - CAN bus or ribbon cable interface

3. **CANCoder Magnetic Encoder**:
   - Absolute magnetic position sensor
   - 4096 CPR resolution
   - Continuous rotation (no stops)
   - Magnet mounting calibration
   - Duty cycle or PWM output options
   - CAN bus communication

**CTRE Swerve Integration:**
- Use `TunerConstants.java` (auto-generated from Phoenix Tuner) for swerve configuration
- Use `CommandSwerveDrivetrain` class for swerve drive control
- Use `SwerveRequest` objects for controlling the drivetrain
- Configure swerve via `SwerveDrivetrainConstants` and `ModuleConstants`

**Common Patterns:**
- Use `TalonFXConfiguration` for motor setup
- Configure `Slot0` for PID gains
- Set `NeutralMode` to `Brake` or `Coast`
- Enable `StatorCurrentLimit` for protection
- Use `StatusSignal` for asynchronous updates

## Build/Lint/Test Commands

### **Build System**
- **Gradle-based** with WPILib GradleRIO plugin (version 2026.2.1)
- **Java 17** compatibility (sourceCompatibility = targetCompatibility = JavaVersion.VERSION_17)

### **Essential Commands**

**Linux/macOS:**
```bash
# Build the project
./gradlew build -Dorg.gradle.java.home="$HOME/wpilib/2026/jdk"

# Deploy to RoboRIO (real robot)
./gradlew deploy -Dorg.gradle.java.home="$HOME/wpilib/2026/jdk"

# Run tests (JUnit 5)
./gradlew test -Dorg.gradle.java.home="$HOME/wpilib/2026/jdk"

# Format code (runs automatically before compile)
./gradlew spotlessApply -Dorg.gradle.java.home="$HOME/wpilib/2026/jdk"

# Clean build
./gradlew clean -Dorg.gradle.java.home="$HOME/wpilib/2026/jdk"

# Build without tests
./gradlew assemble -Dorg.gradle.java.home="$HOME/wpilib/2026/jdk"
```

**Windows:**
```cmd
REM Build the project
gradlew build

REM Deploy to RoboRIO (real robot)
gradlew deploy

REM Run tests (JUnit 5)
gradlew test

REM Format code (runs automatically before compile)
gradlew spotlessApply

REM Clean build
gradlew clean

REM Build without tests
gradlew assemble
```

**Note:** If using a custom JDK installation, set `org.gradle.java.home` in `gradle.properties` or use the `-D` flag:
```bash
./gradlew build -Dorg.gradle.java.home="C:\wpilib\2026\jdk"

## Code Style Guidelines

### **Formatting (Spotless)**
- **Java**: Palantir Java Format 2.39.0 (formatJavadoc=true)
- **Gradle**: Greclipse with 4-space indentation
- **JSON**: Gson with 2-space indentation
- **Markdown**: 2-space indentation, trim trailing whitespace
- **Automatic formatting** runs before compilation (`project.compileJava.dependsOn(spotlessApply)`)

### **Function Signatures (Javadoc)**
- **All public methods must have Javadoc function signatures** that describe what the method does
- **Format**: Use standard Javadoc format with `@return` and `@param` as needed
- **Example**:
```java
/**
 * Checks if all shooter and feeder motors are connected and functioning.
 *
 * @return true if all motors are connected, false otherwise
 */
public boolean hardwareOK() { ... }
```

### **Comments**
- **Remove all unnecessary comments** - code should be self-explanatory
- **Only keep comments when something is actually tricky** or needs special explanation
- **Avoid redundant comments** like "// Update inputs" when the method is clearly named `updateInputs()`
- **Do NOT comment variable** - let the naming speak for itself

### **Imports**
```java
// Group imports in this order:
1. edu.wpi.first.* (WPILib)
2. com.ctre.* (CTRE/Phoenix)
3. com.pathplanner.* (PathPlanner)
4. Other vendor libraries
5. Java standard library
6. Project packages (frc.robot.*)
```

### **Naming Conventions**
- **Classes**: PascalCase (e.g., `DriveCommands`, `TunerConstants`)
- **Methods**: camelCase (e.g., `getPosition`, `setReference`)
- **Variables**: camelCase (e.g., `driveMotor`, `steerEncoder`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `MAX_VELOCITY`, `MODULE_OFFSETS`)
- **Packages**: lowercase (e.g., `frc.robot.subsystems.drive`)

### **Types and Error Handling**
- Use `@NonNull` and `@Nullable` annotations where appropriate
- Prefer `Optional` over null returns
- Use WPILib's `Supplier` and `Consumer` interfaces

## Project Structure

### **Key Directories**
```
src/main/java/frc/robot/
├── generated/
│   └── TunerConstants.java      # Auto-generated from Phoenix Tuner
├── subsystems/
│   └── CommandSwerveDrivetrain.java  # CTRE swerve drivetrain (generated base)
├── Robot.java                   # Main robot class
├── RobotContainer.java          # Robot container (subsystems, commands)
├── Telemetry.java               # NetworkTables telemetry
├── LimelightHelpers.java       # Limelight vision helpers
└── Main.java                   # Entry point
```

### **Vendor Dependencies** (`vendordeps/`)
1. `PathplannerLib.json` - Path planning library
2. `Phoenix6.json` - CTRE motor controllers (Talon FX)
3. `photonlib.json` - Vision processing (PhotonVision)
4. `Studica.json` - Additional vendor library
5. `WPILibNewCommands.json` - WPILib command framework

## FRC-Specific Concepts

### **Robot Modes**
```java
public static enum Mode {
    REAL,    // Running on real robot
    SIM      // Running simulation (WPILib default)
}
```

### **Command-Based Programming**
- Uses WPILib's command framework
- Commands are composed in `RobotContainer`
- Use `CommandScheduler` for command execution

### **NetworkTables**
- WPILib's communication system for robot-to-driver station
- Use `NetworkTableInstance` for publishing/subscribing data

### **Swerve Drive Mathematics**
- Uses CTRE's built-in swerve kinematics and odometry
- Uses `SwerveDriveKinematics` and `SwerveDriveOdometry` from WPILib
- Module states include `angle` and `speed`
- Field-relative vs robot-relative driving

## Documentation Index (Must Read Before Coding)

### **WPILib Documentation** (`docs/agent/WPILib/`)
- `what-is-wpilib.rst` - Introduction to WPILib, supported languages, and source code
- `frc-glossary.rst` - FRC terminology glossary
- **Advanced Controls**: Control theory, PID, feedforward, state-space control, system identification, trajectories
- **Command-Based Programming**: Framework for structuring robot code
- **Hardware APIs**: Motor controllers, sensors, pneumatics, LEDs
- **Kinematics and Odometry**: Robot movement mathematics
- **NetworkTables**: Communication system for robot-to-driver station
- **Vision Processing**: Camera integration, AprilTags, vision processing pipelines

### **PathPlanner Documentation** (`docs/agent/PathPlanner/`)
- `Home.md` - Overview and main features of PathPlanner
- `pplib-Getting-Started.md` - Library introduction for integrating PathPlanner into robot code
- `pplib-Build-an-Auto.md` - Building autonomous routines with path following and event markers
- `pplib-Pathfinding.md` - Pathfinding algorithms for navigating around obstacles

### **Phoenix6 Documentation** (`docs/agent/Phoenix6/`)
- **API Reference**: Core library usage and device-specific APIs
- **Hardware Reference**: Device specifications (TalonFX, Pigeon2, CANCoder)
- **Swerve API**: CTRE Swerve drive APIs and configuration
  - `api-reference/mechanisms/swerve/swerve-overview.rst` - Swerve drive system overview
  - `api-reference/mechanisms/swerve/swerve-builder-api.rst` - Builder pattern for swerve configuration
  - `api-reference/mechanisms/swerve/using-swerve-api.rst` - Practical swerve API usage
- **Migration**: Upgrading from Phoenix 5

## Development Workflow

### **1. Before Making Changes**
- Read relevant framework documentation
- Check existing implementations for patterns
- Understand CTRE swerve configuration in TunerConstants

### **2. When Adding New Subsystem**
- Follow existing subsystem patterns
- Add to `RobotContainer`
- Use proper WPILib command framework

### **3. Testing**
- Run `./gradlew test` after changes
- Deploy to robot for hardware testing

### **4. Code Review Checklist**
- [ ] Follows naming conventions
- [ ] Properly formatted (Spotless)
- [ ] Uses CTRE swerve properly
- [ ] Documentation updated if needed

## Common Pitfalls

1. **Hardcoding hardware values** - Use constants or configuration
2. **Ignoring thread safety** - WPILib has specific threading requirements
3. **Not testing on real robot** - Simulation may not catch all issues
4. **Incorrect swerve configuration** - Verify in TunerConstants

## Team Information
- **Team Number**: 5516 (from `.wpilib/wpilib_preferences.json`)
- **Project Year**: 2026
- **Template**: CTRE Swerve with PathPlanner

## Additional Resources
- [WPILib Documentation](https://docs.wpilib.org)
- [CTRE Phoenix Documentation](https://docs.ctr-electronics.com)
- [PathPlanner Documentation](https://pathplanner.dev)

**REMEMBER**: This is REAL ROBOT CODE that will control physical hardware. Safety and reliability are paramount. Understand the physics implications and follow FRC best practices.