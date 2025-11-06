# Quick Start Guide

This guide will help you set up your development environment and start working with CaveDroid.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Getting the Code](#getting-the-code)
3. [Building the Project](#building-the-project)
4. [Running the Game](#running-the-game)
5. [Development Workflow](#development-workflow)
6. [Common Issues](#common-issues)

---

## Prerequisites

### Required

- **JDK 17** or higher
  - Download from [Adoptium](https://adoptium.net/) or use your system's package manager
  - Verify: `java -version` should show version 17 or higher

### Optional (Platform-Specific)

- **Android Development**
  - Android SDK (API level 34)
  - Android Studio (recommended for Android builds)

- **iOS Development** (macOS only)
  - Xcode
  - RoboVM (handled by Gradle)

### Recommended Tools

- **IDE:** IntelliJ IDEA (Community or Ultimate)
- **Git:** For version control
- **Gradle:** Bundled with project (use `./gradlew`)

---

## Getting the Code

### Clone the Repository

```bash
git clone https://github.com/your-username/cavedroid.git
cd cavedroid
```

### Verify Project Structure

You should see:
```
cavedroid/
├── android/
├── assets/
├── buildSrc/
├── core/
├── desktop/
├── ios/
├── gradle/
├── build.gradle.kts
└── settings.gradle.kts
```

---

## Building the Project

### Desktop Build

The desktop version is the easiest to build and test during development.

```bash
# Build the desktop JAR
./gradlew desktop:dist

# The JAR will be in: desktop/build/libs/desktop-1.0.jar
```

### Android Build

```bash
# Build debug APK
./gradlew android:assembleDebug

# The APK will be in: android/build/outputs/apk/debug/android-debug.apk
```

### Clean Build

If you encounter issues:
```bash
# Clean all build artifacts
./gradlew clean

# Rebuild everything
./gradlew desktop:dist
```

---

## Running the Game

### From IDE (IntelliJ IDEA)

#### Desktop

1. Open the project in IntelliJ IDEA
2. Wait for Gradle sync to complete
3. Navigate to: `desktop/src/main/kotlin/ru/fredboy/cavedroid/desktop/DesktopLauncher.kt`
4. Right-click and select "Run 'DesktopLauncherKt'"
5. The game should launch in a window

**Tip:** Set working directory to the project root in Run Configuration:
- Run → Edit Configurations → Working directory: `$PROJECT_DIR$/assets`

#### Android

1. Connect an Android device or start an emulator
2. Select "android" run configuration
3. Click Run button
4. The APK will be installed and launched

### From Command Line

#### Desktop

```bash
# Run directly (no JAR created)
./gradlew desktop:run

# Or run the built JAR
java -jar desktop/build/libs/desktop-1.0.jar
```

**Command-Line Options:**
```bash
# Enable touch controls (for testing mobile on desktop)
java -jar desktop-1.0.jar --touch

# Enable debug mode
java -jar desktop-1.0.jar --debug

# Enable verbose logging
java -jar desktop-1.0.jar --verbose

# Combine options
java -jar desktop-1.0.jar --touch --debug
```

#### Android

```bash
# Install and run
./gradlew android:installDebug
adb shell am start -n ru.fredboy.cavedroid/.AndroidLauncher
```

### Game Data Location

**Desktop:**
- Linux/Mac: `~/.cavedroid/`
- Windows: `%USERPROFILE%\.cavedroid\`

**Android:**
- Internal: `/data/data/ru.fredboy.cavedroid/files/`

**Files:**
- `preferences.properties` - Settings
- `saves/` - World save files

---

## Development Workflow

### 1. Make Changes

Edit code in the `/core` directory for game logic, or platform-specific directories for platform code.

### 2. Format Code

CaveDroid uses ktlint for code formatting:

```bash
# Auto-format all code
./gradlew ktlintFormat

# Check formatting without changes
./gradlew ktlintCheck
```

**Note:** ktlint runs automatically before compilation.

### 3. Build and Test

```bash
# Quick build and run
./gradlew desktop:run

# Full build with checks
./gradlew build
```

### 4. Debugging

#### Enable Debug Mode

```kotlin
// In DesktopLauncher or set via command line
args.debug = true  // Shows FPS, coordinates, hitboxes
```

#### In-Game Debug Keys

When debug mode is enabled:

- **F3** - Toggle debug overlay
- **F1** - Toggle hitbox rendering
- **G** - Toggle creative mode (during development)

#### Logging

```kotlin
import ru.fredboy.cavedroid.common.CommonLogger

CommonLogger.info("MyTag", "Something happened")
CommonLogger.debug("MyTag", "Debug info")
CommonLogger.error("MyTag", "Error message")
```

Set log level:
```bash
./gradlew desktop:run --verbose  # Shows debug logs
```

---

## Common Issues

### Issue: "Cannot find Java 17"

**Solution:**
```bash
# Check Java version
java -version

# Set JAVA_HOME (Linux/Mac)
export JAVA_HOME=/path/to/jdk-17

# Set JAVA_HOME (Windows)
set JAVA_HOME=C:\path\to\jdk-17
```

### Issue: "Assets not found"

**Cause:** Working directory is incorrect.

**Solution:**
- For IDE: Set working directory to project root
- For JAR: Run from project root directory
- For Gradle: Use `./gradlew desktop:run` (handles automatically)

### Issue: Build fails with "OutOfMemoryError"

**Solution:** Increase Gradle memory in `gradle.properties`:
```properties
org.gradle.jvmargs=-Xmx2048m
```

### Issue: Android build fails with SDK errors

**Solution:**
```bash
# Set Android SDK path
export ANDROID_HOME=/path/to/android-sdk

# Or create local.properties in project root
echo "sdk.dir=/path/to/android-sdk" > local.properties
```

### Issue: Black screen on launch

**Possible causes:**
1. Assets not loading - Check working directory
2. OpenGL not supported - Update graphics drivers
3. Crash during initialization - Check logs

**Check logs:**
```bash
# Desktop
./gradlew desktop:run --verbose

# Android
adb logcat | grep CaveDroid
```

### Issue: "Could not resolve dependencies"

**Solution:**
```bash
# Refresh dependencies
./gradlew --refresh-dependencies

# Use Gradle wrapper (ensures correct Gradle version)
./gradlew clean build
```

---

## Next Steps

Now that you have the game running:

1. **Explore the Codebase**
   - Read [ARCHITECTURE.md](ARCHITECTURE.md) to understand the code structure
   - Browse `/core` modules to see how systems are organized

2. **Make Your First Change**
   - Try the guides in `/docs/guides/`:
     - [Adding New Blocks](guides/ADDING_BLOCKS.md)
     - [Adding New Items](guides/ADDING_ITEMS.md)
     - [Modifying Game Mechanics](guides/MODIFYING_MECHANICS.md)

3. **Join Development**
   - Read existing code before making changes
   - Follow the coding style (ktlint enforced)
   - Test changes on desktop before Android/iOS
   - Write clear commit messages

---

## Development Tips

### Fast Iteration

```bash
# Terminal 1: Keep this running
./gradlew desktop:run --continuous

# Terminal 2: Make changes, then
./gradlew desktop:classes
# Game will auto-reload
```

### Asset Development

```
assets/
├── textures/        # Edit PNG files here
│   ├── blocks/      # Block textures (16x16)
│   ├── mobs/        # Mob sprites
│   └── items/       # Item icons
├── json/            # Edit JSON configs here
│   ├── blocks.json  # Block definitions
│   └── items.json   # Item definitions
└── sfx/             # Add sound files here
```

**After adding assets:**
- No rebuild needed for JSON changes (read at runtime)
- PNG changes require restart

### Testing Different Platforms

**Desktop is fastest for development:**
- Instant builds and launches
- Easy debugging
- Full keyboard/mouse control

**Test on Android occasionally:**
- Touch controls
- Performance on mobile hardware
- Screen size variations

### Useful Gradle Tasks

```bash
# List all tasks
./gradlew tasks

# Show dependencies
./gradlew dependencies

# Build all platforms
./gradlew build

# Clean and rebuild
./gradlew clean build
```

---

## Getting Help

- **Documentation:** Check `/docs` directory
- **Code Examples:** Look at existing implementations
- **Logs:** Enable verbose mode for detailed output
- **Issues:** Search GitHub issues for similar problems

Happy coding!
