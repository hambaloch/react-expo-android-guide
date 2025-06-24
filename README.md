# Complete Guide: Building and Debugging React Native/Expo Android Apps

*From Development to Production: A Real-World Journey*

## Table of Contents
1. [Overview](#overview)
2. [Initial Setup](#initial-setup)
3. [Common Build Issues](#common-build-issues)
4. [Debugging APK Crashes](#debugging-apk-crashes)
5. [Environment Configuration](#environment-configuration)
6. [Build Process](#build-process)
7. [Testing on Emulator](#testing-on-emulator)
8. [Lessons Learned](#lessons-learned)
9. [Troubleshooting Checklist](#troubleshooting-checklist)

## Overview

This guide documents the complete process of building, debugging, and deploying a React Native/Expo app to Android. Based on real-world experience with a production app, we'll cover everything from initial setup to resolving complex build issues.

## Initial Setup

### Prerequisites
- Node.js (v18 or higher)
- npm or yarn
- Expo CLI (`npm install -g expo-cli`)
- Android Studio with Android SDK
- EAS CLI (`npm install -g eas-cli`)

### Project Structure
```
andalus-express/
├── app/              # Main application screens and navigation
├── assets/          # Images, fonts, and other static files
├── components/      # Reusable React components
├── constants/       # App-wide constants and configuration
├── hooks/           # Custom React hooks
├── android/         # Native Android files (generated)
├── ios/             # Native iOS files (generated)
└── scripts/         # Utility scripts
```

## Common Build Issues

### 1. Expo Doctor Warnings
**Problem:** `expo doctor` reports dependency issues
```
✖ Validate packages against React Native Directory package metadata
✖ Check that packages match versions required by installed Expo SDK
```

**Solution:**
- Run `npx expo install --check` to identify mismatched versions
- Update dependencies to match Expo SDK requirements
- Remove manually installed `expo-modules-core` (let Expo manage it)

### 2. Dependency Conflicts
**Problem:** npm install fails with ERESOLVE errors
```
npm error ERESOLVE unable to resolve dependency tree
```

**Solution:**
- Use `npm install --legacy-peer-deps` for temporary fixes
- Align all navigation packages to compatible versions
- Consider using `npx expo install` for Expo-specific packages

### 3. Android SDK Location Issues
**Problem:** Build fails with "SDK location not found"
```
com.android.builder.errors.EvalIssueException: SDK location not found
```

**Solution:**
- Set `ANDROID_HOME` environment variable
- Create `android/local.properties` file
- Ensure SDK path is correct and accessible

## Debugging APK Crashes

### Why APKs Crash When Development Works

Development builds (`expo start`) run in Expo Go with pre-built native modules, while standalone APKs use your project's native modules. Version mismatches cause runtime crashes.

### Debugging Steps

1. **Use Android Studio Logcat**
   - Open Android Studio → View → Tool Windows → Logcat
   - Filter by your app's package name
   - Look for `FATAL EXCEPTION` errors

2. **Common Crash Patterns**
   ```
   java.lang.NoSuchMethodError: No virtual method getConverters()
   at expo.modules.blur.BlurModule.definition(BlurModule.kt:45)
   ```
   This indicates version mismatches between Expo modules.

3. **Fix Module Version Conflicts**
   ```bash
   npx expo install expo-blur
   npx expo install expo-modules-core
   npx expo install expo
   ```

## Environment Configuration

### Android SDK Setup

1. **Set Environment Variables**
   Add to `~/.zshrc` (or `~/.bash_profile`):
   ```bash
   export ANDROID_HOME=$HOME/Library/Android/sdk
   export PATH=$PATH:$ANDROID_HOME/emulator
   export PATH=$PATH:$ANDROID_HOME/tools
   export PATH=$PATH:$ANDROID_HOME/tools/bin
   export PATH=$PATH:$ANDROID_HOME/platform-tools
   ```

2. **Create local.properties**
   In `android/local.properties`:
   ```
   sdk.dir=/Users/yourusername/Library/Android/sdk
   ```

3. **Verify Setup**
   ```bash
   echo $ANDROID_HOME
   emulator -list-avds
   ```

### Android Emulator Setup

1. **List Available Emulators**
   ```bash
   emulator -list-avds
   ```

2. **Start an Emulator**
   ```bash
   emulator -avd <emulator_name>
   ```

3. **Install APK**
   ```bash
   adb install path/to/your-app.apk
   ```

## Build Process

### EAS Build (Recommended)

1. **Preview Build (for testing)**
   ```bash
   eas build -p android --profile preview
   ```

2. **Production Build (for Play Store)**
   ```bash
   eas build -p android --profile production
   ```

3. **Local Build (for debugging)**
   ```bash
   eas build --platform android --local
   ```

### Build Outputs

- **AAB (.aab)**: Android App Bundle for Play Store
- **APK (.apk)**: Direct installation on devices/emulators

### Converting AAB to APK

For testing on emulators, convert AAB to APK:

1. **Download bundletool**
   ```bash
   # Download from https://github.com/google/bundletool/releases
   ```

2. **Generate APK**
   ```bash
   java -jar bundletool.jar build-apks --bundle=your-app.aab --output=output.apks --mode=universal --ks=~/.android/debug.keystore --ks-key-alias=androiddebugkey --ks-pass=pass:android
   ```

3. **Extract and Install**
   ```bash
   unzip output.apks
   adb install universal/*.apk
   ```

## Testing on Emulator

### Debugging with Android Studio

1. **Open Logcat**
   - Android Studio → View → Tool Windows → Logcat
   - Select your device/emulator
   - Filter by package name

2. **Identify Issues**
   - Look for red error messages
   - Focus on `FATAL EXCEPTION` entries
   - Note the stack trace for debugging

3. **Common Log Messages**
   ```
   [73] ItemStore: getItems RPC failed for item com.yourapp
   ```
   This is a system warning, not a crash.

### Development vs Production

| Environment | Native Modules | Debugging | Use Case |
|-------------|----------------|-----------|----------|
| Development | Expo Go's modules | Easy | Development |
| Production | Your app's modules | Requires APK | Testing/Release |

## Lessons Learned

### 1. Version Management is Critical
- Always use `npx expo install` for Expo packages
- Keep all Expo modules aligned with SDK version
- Don't manually install `expo-modules-core`

### 2. Environment Variables Matter
- `ANDROID_HOME` must be set for local builds
- EAS local builds run in isolated environments
- Always verify environment setup

### 3. Debugging Strategy
- Start with `expo doctor` to identify issues
- Use Android Studio Logcat for crash analysis
- Test with development builds before production

### 4. Build Types
- Use AAB for Play Store distribution
- Use APK for direct testing and debugging
- Local builds help isolate environment issues

## Troubleshooting Checklist

### Before Building
- [ ] Run `expo doctor` and fix all issues
- [ ] Verify `ANDROID_HOME` is set
- [ ] Check `android/local.properties` exists
- [ ] Ensure all Expo modules are compatible

### During Build
- [ ] Monitor for dependency conflicts
- [ ] Check for SDK location errors
- [ ] Verify environment variables are loaded

### After Build
- [ ] Test APK on emulator
- [ ] Check Logcat for crashes
- [ ] Verify all features work as expected

### Common Commands
```bash
# Check project health
npx expo doctor

# Update dependencies
npx expo install --check

# Build for testing
eas build -p android --profile preview

# Build locally
eas build --platform android --local

# Debug on emulator
adb logcat | grep "your.package.name"
```

## Conclusion

Building and debugging React Native/Expo Android apps requires attention to detail in dependency management, environment configuration, and systematic debugging. The key is to understand the differences between development and production environments and to use the right tools for each stage of the process.

By following this guide, you can avoid common pitfalls and create a robust build process for your team.

---

*This guide is based on real-world experience building and debugging a production React Native/Expo application. The solutions provided have been tested and proven effective in resolving complex build and runtime issues.* 
