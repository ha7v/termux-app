# Termux App

## Project Overview

Termux is an Android terminal application and Linux environment. This repository (`termux-app`) contains the source code for the Android application itself, which handles the user interface, terminal emulation, and integration with the Android OS. The actual Linux packages installable inside the app are managed in a separate repository (`termux-packages`).

### Architecture

The project is a multi-module Android application using Gradle:

*   **`app`**: The main application module containing the UI, `TermuxActivity`, `TermuxService`, and bootstrap installation logic.
*   **`termux-shared`**: A library module containing shared constants, utilities, and helpers used by the main app and Termux plugins (e.g., Termux:API, Termux:Widget).
*   **`terminal-emulator`**: A module handling the backend terminal emulation logic. It likely contains the JNI bindings to interact with the pseudo-terminal (pty).
*   **`terminal-view`**: A module responsible for rendering the terminal screen and handling user input (keyboard/touch).

## Building and Running

The project uses the Gradle build system. A wrapper is provided (`gradlew`).

### Common Commands

*   **Build Debug APK:**
    ```bash
    ./gradlew assembleDebug
    ```
    Output APKs will be in `app/build/outputs/apk/debug/`.

*   **Build Release APK:**
    ```bash
    ./gradlew assembleRelease
    ```

*   **Run Unit Tests:**
    ```bash
    ./gradlew test
    ```

*   **Run Instrumented Tests (requires connected device/emulator):**
    ```bash
    ./gradlew connectedAndroidTest
    ```

*   **Clean Build Directory:**
    ```bash
    ./gradlew clean
    ```

### Dependency Management

*   **JitPack**: The project uses JitPack for some dependencies.
*   **Bootstrap**: The app downloads a "bootstrap" zip file (minimal Linux environment) during the build or runtime to populate the initial file system. This is configured in `app/build.gradle`.

## Development Conventions

*   **Language**:
    *   **Java**: The primary language for the Android application logic.
    *   **C/C++**: Used via JNI (Java Native Interface) for low-level terminal operations and bootstrap handling (see `src/main/cpp` in modules).
*   **Versioning**: The project follows [Semantic Versioning](https://semver.org/). The `versionName` is defined in `app/build.gradle`.
*   **Commit Messages**: The project strictly follows the [Conventional Commits](https://www.conventionalcommits.org) specification (e.g., `Fix: ...`, `Feat: ...`). This is crucial for automated changelog generation.
*   **Code Style**: Standard Android and Java coding conventions apply.
*   **Shared Logic**: Reusable code should be placed in the `termux-shared` module to facilitate code sharing with Termux plugins.

## Key Files and Directories

*   `app/src/main/AndroidManifest.xml`: Defines the app's components (Activities, Services, Providers) and permissions.
*   `app/src/main/java/com/termux/app/TermuxActivity.java`: The main activity hosting the terminal view.
*   `app/src/main/java/com/termux/app/TermuxService.java`: The foreground service ensuring the terminal session persists in the background.
*   `app/src/main/cpp/`: Contains native C/C++ code.
*   `settings.gradle`: Defines the included Gradle modules.
*   `build.gradle` (root): Global build configuration and repositories.
