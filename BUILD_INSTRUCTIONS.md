# Building ICS-OpenVPN APK - Complete Guide

This guide provides detailed instructions for compiling the ICS-OpenVPN Android application into an APK file using Visual Studio Code.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Development Environment Setup](#development-environment-setup)
3. [Getting the Source Code](#getting-the-source-code)
4. [VSCode Configuration](#vscode-configuration)
5. [Building the APK](#building-the-apk)
6. [Build Variants Explained](#build-variants-explained)
7. [Output Locations](#output-locations)
8. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Required Software

1. **Java Development Kit (JDK)**
   - JDK 17 or later
   - Download from: https://adoptium.net/ or https://www.oracle.com/java/technologies/downloads/
   - Verify installation: `java -version` and `javac -version`

2. **Android SDK**
   - Minimum SDK 21 (Android 5.0)
   - Target SDK 36 (Android 14)
   - Recommended: Install via Android Studio or command-line tools
   - Download: https://developer.android.com/studio

3. **Android NDK**
   - Required version: 29.0.14206865
   - Install via Android Studio SDK Manager or standalone
   - Download: https://developer.android.com/ndk/downloads

4. **CMake**
   - Version 3.10 or later
   - Install via Android Studio SDK Manager or standalone
   - Download: https://cmake.org/download/

5. **SWIG (Simplified Wrapper and Interface Generator)**
   - Version 3.0 or later
   - Required for generating Java bindings for OpenVPN3

   **Installation by Platform:**

   - **Linux (Ubuntu/Debian):**
     ```bash
     sudo apt-get update
     sudo apt-get install swig
     ```

   - **Linux (Fedora/RHEL):**
     ```bash
     sudo dnf install swig
     ```

   - **macOS (using Homebrew):**
     ```bash
     brew install swig
     ```

   - **Windows:**
     - Download from: http://www.swig.org/download.html
     - Add SWIG to your PATH environment variable
     - On Windows, you may also need Perl for mbedtls compilation

6. **Visual Studio Code**
   - Download from: https://code.visualstudio.com/

7. **Git**
   - Required for cloning the repository and managing submodules
   - Download from: https://git-scm.com/

### Recommended VSCode Extensions

Install these extensions in VSCode for better development experience:

- **Extension Pack for Java** (Microsoft)
- **Gradle for Java** (Microsoft)
- **Android iOS Emulator** (DiemasMichiels)
- **Kotlin Language** (fwcd)
- **CMake** (twxs)

---

## Development Environment Setup

### 1. Configure Environment Variables

Set up the following environment variables:

**Linux/macOS:**

Add to your `~/.bashrc`, `~/.zshrc`, or `~/.profile`:

```bash
# Android SDK
export ANDROID_HOME=$HOME/Android/Sdk
export ANDROID_SDK_ROOT=$ANDROID_HOME

# Android NDK (adjust version if needed)
export ANDROID_NDK_HOME=$ANDROID_HOME/ndk/29.0.14206865

# Add to PATH
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_NDK_HOME
```

**Windows (PowerShell):**

```powershell
# Set as System Environment Variables or User Environment Variables
[System.Environment]::SetEnvironmentVariable("ANDROID_HOME", "C:\Users\YourUsername\AppData\Local\Android\Sdk", "User")
[System.Environment]::SetEnvironmentVariable("ANDROID_NDK_HOME", "C:\Users\YourUsername\AppData\Local\Android\Sdk\ndk\29.0.14206865", "User")

# Add to PATH
$path = [System.Environment]::GetEnvironmentVariable("PATH", "User")
[System.Environment]::SetEnvironmentVariable("PATH", "$path;$env:ANDROID_HOME\platform-tools;$env:ANDROID_HOME\cmdline-tools\latest\bin", "User")
```

### 2. Verify Installation

After setting up environment variables, verify all tools are accessible:

```bash
# Java
java -version
javac -version

# Android SDK
adb version

# SWIG
swig -version

# CMake
cmake --version

# NDK (check if directory exists)
ls $ANDROID_NDK_HOME  # Linux/macOS
dir %ANDROID_NDK_HOME%  # Windows
```

---

## Getting the Source Code

### 1. Clone the Repository

```bash
# Clone the main repository
git clone https://github.com/schwabe/ics-openvpn.git
cd ics-openvpn

# If using a fork (e.g., your modified version)
git clone https://github.com/YourUsername/ics-openvpn.git
cd ics-openvpn
```

### 2. Initialize Git Submodules

ICS-OpenVPN uses git submodules for OpenVPN, OpenSSL, and other dependencies:

```bash
# Initialize and fetch all submodules
git submodule init
git submodule update

# Alternative: Clone with submodules in one command
# git clone --recurse-submodules https://github.com/schwabe/ics-openvpn.git
```

**Important:** The submodules contain the actual OpenVPN source code and dependencies. Without them, the build will fail.

### 3. Verify Submodules

Check that submodules are properly initialized:

```bash
git submodule status
```

You should see output listing the submodules without a `-` prefix (which would indicate uninitialized).

---

## VSCode Configuration

### 1. Open Project in VSCode

```bash
code .
```

Or open VSCode and use `File > Open Folder` and select the `ics-openvpn` directory.

### 2. Configure Java Settings

Create or update `.vscode/settings.json`:

```json
{
  "java.configuration.updateBuildConfiguration": "automatic",
  "java.home": "/path/to/jdk-17",
  "java.jdt.ls.java.home": "/path/to/jdk-17",
  "gradle.nestedProjects": true,
  "files.exclude": {
    "**/.gradle": true,
    "**/build": true
  }
}
```

Replace `/path/to/jdk-17` with your actual JDK installation path.

### 3. Import Gradle Project

1. VSCode should automatically detect the Gradle project
2. If prompted, allow the Gradle extension to import the project
3. Wait for initial Gradle sync (this may take 10-15 minutes on first run as it downloads dependencies and builds native binaries)

### 4. Trust the Workspace

When prompted, click "Trust" to allow Gradle and other extensions to run.

---

## Building the APK

### Method 1: Using Gradle Command Line (Recommended)

#### Build Debug APK

Open the integrated terminal in VSCode (`Terminal > New Terminal` or `` Ctrl+` ``):

```bash
# Build all variants (debug)
./gradlew assembleDebug

# Build specific variant (UI flavor with OpenVPN 2.3)
./gradlew assembleUiOvpn23Debug

# Build specific variant (Skeleton flavor with OpenVPN 2)
./gradlew assembleSkeletonOvpn2Debug
```

**Windows:**
```cmd
gradlew.bat assembleDebug
```

#### Build Release APK (Unsigned)

```bash
# Build release APK (will need signing configuration)
./gradlew assembleRelease
```

**Note:** Release builds require signing configuration. See [Signing Configuration](#signing-configuration) below.

#### Build All Variants

```bash
# Build all debug and release variants
./gradlew build
```

### Method 2: Using VSCode Tasks

Create `.vscode/tasks.json` for quick build commands:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Build Debug APK",
      "type": "shell",
      "command": "./gradlew",
      "args": ["assembleDebug"],
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "problemMatcher": []
    },
    {
      "label": "Build UI Debug APK",
      "type": "shell",
      "command": "./gradlew",
      "args": ["assembleUiOvpn23Debug"],
      "group": "build",
      "problemMatcher": []
    },
    {
      "label": "Clean Build",
      "type": "shell",
      "command": "./gradlew",
      "args": ["clean", "assembleDebug"],
      "group": "build",
      "problemMatcher": []
    },
    {
      "label": "Build Release APK",
      "type": "shell",
      "command": "./gradlew",
      "args": ["assembleRelease"],
      "group": "build",
      "problemMatcher": []
    }
  ]
}
```

Then use `Terminal > Run Task` or `Ctrl+Shift+P` → `Tasks: Run Task` to execute builds.

### Method 3: Using Gradle Extension in VSCode

1. Open the Gradle sidebar (click Gradle icon in Activity Bar)
2. Expand the project tree
3. Navigate to `main > Tasks > build`
4. Click on `assembleDebug` or desired task

---

## Build Variants Explained

ICS-OpenVPN has multiple build variants based on two flavor dimensions:

### Implementation Dimension

1. **UI** - Full user interface variant
   - Includes complete UI with settings, profile management, graphs, etc.
   - Suitable for standalone VPN client app
   - Larger APK size (~15-20 MB)

2. **Skeleton** - Minimal variant
   - No UI components
   - Suitable for integration into other apps via AIDL API
   - Smaller APK size (~5-8 MB)

### OpenVPN Implementation Dimension

1. **ovpn23** - OpenVPN 2.3+ with OpenVPN 3 core
   - Uses newer OpenVPN 3 library
   - Better performance and features
   - Default and recommended variant

2. **ovpn2** - OpenVPN 2.x only
   - Traditional OpenVPN 2 implementation
   - Legacy compatibility
   - Separate version suffix (-o2)

### Common Build Variant Combinations

| Variant | Description | Use Case |
|---------|-------------|----------|
| `uiOvpn23Debug` | Full UI with OpenVPN 3 (Debug) | Development/testing |
| `uiOvpn23Release` | Full UI with OpenVPN 3 (Release) | Production app |
| `uiOvpn2Debug` | Full UI with OpenVPN 2 (Debug) | Legacy compatibility testing |
| `skeletonOvpn23Debug` | No UI with OpenVPN 3 (Debug) | API integration development |

**Recommendation:** Use `uiOvpn23` for most purposes.

---

## Output Locations

After successful build, APK files are located in:

```
main/build/outputs/apk/
├── ui/
│   ├── ovpn23/
│   │   ├── debug/
│   │   │   └── main-ui-ovpn23-debug.apk
│   │   └── release/
│   │       └── main-ui-ovpn23-release-unsigned.apk
│   └── ovpn2/
│       ├── debug/
│       │   └── main-ui-ovpn2-debug.apk
│       └── release/
│           └── main-ui-ovpn2-release-unsigned.apk
└── skeleton/
    ├── ovpn23/
    │   ├── debug/
    │   │   └── main-skeleton-ovpn23-debug.apk
    │   └── release/
    │       └── main-skeleton-ovpn23-release-unsigned.apk
    └── ovpn2/
        └── ...
```

### Split APKs

The build also generates split APKs for different architectures in:

```
main/build/outputs/apk/ui/ovpn23/debug/
├── main-ui-ovpn23-debug.apk (Universal APK)
├── main-ui-ovpn23-arm64-v8a-debug.apk (ARM 64-bit)
├── main-ui-ovpn23-armeabi-v7a-debug.apk (ARM 32-bit)
├── main-ui-ovpn23-x86-debug.apk (x86 32-bit)
└── main-ui-ovpn23-x86_64-debug.apk (x86 64-bit)
```

**Recommendation:** Use the universal APK for testing on all devices, or use architecture-specific APKs for smaller file sizes.

---

## Signing Configuration

### For Debug Builds

Debug builds are automatically signed with a debug keystore. No configuration needed.

### For Release Builds

Release builds require signing configuration. Create or edit `~/.gradle/gradle.properties`:

```properties
# OpenVPN 2.3 Release Signing
keystoreFile=/path/to/your/release.keystore
keystorePassword=yourKeystorePassword
keystoreAlias=yourKeyAlias
keystoreAliasPassword=yourAliasPassword

# OpenVPN 2 Release Signing (if needed)
keystoreO2File=/path/to/your/release-o2.keystore
keystoreO2Password=yourKeystorePassword
keystoreO2Alias=yourKeyAlias
keystoreO2AliasPassword=yourAliasPassword
```

**For Development/Testing Only:**

To sign release builds with debug key (not for production):

```bash
./gradlew assembleRelease -PicsopenvpnDebugSign
```

---

## Troubleshooting

### Common Issues and Solutions

#### 1. SWIG Not Found

**Error:**
```
FAILURE: Build failed with an exception.
* What went wrong:
Execution failed for task ':main:generateOpenVPN3SwiguiOvpn23Debug'.
> A problem occurred starting process 'command 'swig''
```

**Solution:**
- Verify SWIG is installed: `swig -version`
- Ensure SWIG is in your PATH
- On macOS, if using Homebrew: Check `/opt/homebrew/bin/swig` or `/usr/local/bin/swig`
- On Windows, add SWIG directory to PATH environment variable

#### 2. NDK Not Found

**Error:**
```
* What went wrong:
Execution failed for task ':main:configureCMakeDebug[arm64-v8a]'.
> No version of NDK matched the requested version 29.0.14206865
```

**Solution:**
- Install NDK version 29.0.14206865 via Android Studio SDK Manager
- Verify `ANDROID_NDK_HOME` environment variable points to correct NDK version
- Alternative: Edit `main/build.gradle.kts` and change `ndkVersion` to your installed version

#### 3. Submodules Not Initialized

**Error:**
```
FAILURE: Build failed with an exception.
* What went wrong:
Execution failed for task ':main:externalNativeBuildDebug'.
> Build command failed.
```

**Solution:**
```bash
git submodule init
git submodule update --recursive
```

#### 4. Out of Memory During Build

**Error:**
```
> Expiring Daemon because JVM heap space is exhausted
```

**Solution:**

Create or edit `gradle.properties` in project root:

```properties
org.gradle.jvmargs=-Xmx4096m -XX:MaxMetaspaceSize=512m
org.gradle.daemon=true
org.gradle.parallel=true
org.gradle.caching=true
```

#### 5. CMake Version Mismatch

**Error:**
```
CMake Error at CMakeLists.txt:X (cmake_minimum_required):
  CMake X.Y or higher is required.  You are running version X.Y.Z
```

**Solution:**
- Install CMake 3.10+ via Android Studio SDK Manager
- Or update standalone CMake installation
- Verify with: `cmake --version`

#### 6. Java Version Issues

**Error:**
```
Unsupported class file major version 61
```

**Solution:**
- Ensure Java 17 or later is installed
- Set `JAVA_HOME` environment variable to JDK 17+ installation
- In VSCode, update `.vscode/settings.json` with correct Java path

#### 7. Gradle Sync Takes Too Long

**Normal Behavior:**
- Initial Gradle sync can take 10-15 minutes
- Native binaries (OpenVPN, OpenSSL) are compiled for multiple architectures
- Be patient on first build

**If Stuck:**
```bash
# Clean and retry
./gradlew clean
# Kill Gradle daemons
./gradlew --stop
# Try build again
./gradlew assembleDebug
```

#### 8. Build Fails on Windows with Path Issues

**Solution:**
- Ensure paths don't contain spaces
- Clone repository to short path (e.g., `C:\dev\ics-openvpn`)
- Use Git Bash instead of Command Prompt
- Install Perl if building with mbedtls

---

## Installing the APK

### On Physical Device

1. Enable Developer Options on Android device
2. Enable USB Debugging
3. Connect device via USB
4. Install using ADB:
   ```bash
   adb install main/build/outputs/apk/ui/ovpn23/debug/main-ui-ovpn23-debug.apk
   ```

### On Emulator

1. Start Android Emulator
2. Install using ADB:
   ```bash
   adb install main/build/outputs/apk/ui/ovpn23/debug/main-ui-ovpn23-debug.apk
   ```

### Direct Installation

Transfer the APK to your device and open it to install (ensure "Install from Unknown Sources" is enabled).

---

## Additional Build Commands

### Clean Build

```bash
# Remove all build artifacts
./gradlew clean

# Clean and rebuild
./gradlew clean assembleDebug
```

### Check Dependencies

```bash
# View project dependencies
./gradlew main:dependencies

# Check for dependency updates
./gradlew dependencyUpdates
```

### Run Tests

```bash
# Run unit tests
./gradlew test

# Run specific test
./gradlew testUiOvpn23DebugUnitTest
```

### Lint Checks

```bash
# Run lint checks
./gradlew lint

# View lint report
open main/build/reports/lint-results.html
```

---

## Performance Tips

1. **Enable Gradle Daemon** - Already enabled by default, speeds up subsequent builds
2. **Use Parallel Builds** - Add to `gradle.properties`:
   ```properties
   org.gradle.parallel=true
   ```
3. **Enable Build Cache** - Add to `gradle.properties`:
   ```properties
   org.gradle.caching=true
   ```
4. **Increase Heap Size** - Add to `gradle.properties`:
   ```properties
   org.gradle.jvmargs=-Xmx4096m
   ```
5. **Build Specific Variant** - Don't build all variants unless needed:
   ```bash
   ./gradlew assembleUiOvpn23Debug  # Instead of assembleDebug
   ```

---

## License Notice

**Important:** ICS-OpenVPN is licensed under GPL v2. If you build a custom application based on this code:

- You **MUST** publish your source code according to GPL
- You **CANNOT** build a closed-source commercial app without acquiring a different (paid) license
- Using the AIDL API to control the app from external apps is not subject to this restriction

See `doc/LICENSE.txt` for full license terms.

---

## Getting Help

- **Official Repository:** https://github.com/schwabe/ics-openvpn
- **Documentation:** See `doc/README.txt` in the repository
- **FAQ:** https://ics-openvpn.blinkt.de/FAQ.html
- **Issues:** https://github.com/schwabe/ics-openvpn/issues

---

## Summary: Quick Build Steps

For those who already have everything set up:

```bash
# 1. Clone and prepare
git clone https://github.com/schwabe/ics-openvpn.git
cd ics-openvpn
git submodule init && git submodule update

# 2. Build (choose one)
./gradlew assembleDebug                    # All debug variants
./gradlew assembleUiOvpn23Debug           # UI with OpenVPN 3 (recommended)
./gradlew assembleSkeletonOvpn23Debug     # No UI with OpenVPN 3

# 3. Find APK
ls -la main/build/outputs/apk/ui/ovpn23/debug/

# 4. Install
adb install main/build/outputs/apk/ui/ovpn23/debug/main-ui-ovpn23-debug.apk
```

---

**Document Version:** 1.0
**Last Updated:** 2025-10-25
**Compatible with:** ICS-OpenVPN build system as of commit a114dfe
