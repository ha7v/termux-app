# Termux Toolbar Implementation Analysis

This document serves as context for AI assistants modifying the Termux application, specifically focusing on the "Terminal Toolbar" (Extra Keys & Text Input) implementation.

## 1. Architectural Overview

The Termux Toolbar acts as a bridge between touch interactions/native Android input and the terminal emulator. It is implemented not as a static view, but as a `ViewPager` (`TerminalToolbarViewPager`) located at the bottom of `TermuxActivity`.

It has two distinct modes (Pages):
1.  **Page 0 (Default): Extra Keys View**
    *   Displays a grid of virtual buttons (ESC, TAB, CTRL, ALT, Arrows, etc.).
    *   Implemented via `ExtraKeysView` (from `termux-shared`).
2.  **Page 1: Text Input View**
    *   Displays a native Android `EditText`.
    *   Allows usage of soft keyboard features (autocorrect, swipe typing, etc.) that are normally unavailable in a raw terminal view.
    *   Text committed here is sent to the terminal session.

## 2. Key Classes & Responsibilities

### A. `TermuxActivity.java` (Controller)
*   **Initialization**: Calls `setTerminalToolbarView()` during `onCreate`.
*   **Visibility**: `toggleTerminalToolbar()` shows/hides the entire ViewPager based on user preferences (`mPreferences.shouldShowTerminalToolbar()`).
*   **Height Calculation**: `setTerminalToolbarHeight()` dynamically calculates the ViewPager height based on the number of rows in `ExtraKeysView` and a user-defined scaling factor.
*   **State Management**: Preserves the text in the input view across configuration changes via `ARG_TERMINAL_TOOLBAR_TEXT_INPUT`.
*   **Switching Logic**: (New) `toggleTerminalToolbarViewPager()` allows programmatic switching between Page 0 and Page 1.

### B. `TerminalToolbarViewPager.java` (View & Adapter)
Located in `app/src/main/java/com/termux/app/terminal/io/TerminalToolbarViewPager.java`.

*   **`PageAdapter`**:
    *   **`instantiateItem(position)`**:
        *   `position == 0`: Inflates `R.layout.view_terminal_toolbar_extra_keys`. Configures `ExtraKeysView` with `mActivity.getTermuxTerminalExtraKeys()`.
        *   `position == 1`: Inflates `R.layout.view_terminal_toolbar_text_input`. Finds `R.id.terminal_toolbar_text_input` (EditText).
    *   **Input Handling (Page 1)**: Sets an `OnEditorActionListener` on the EditText. When the "Enter/Done" action is triggered:
        1.  Gets text from EditText.
        2.  Sends text to current session: `session.write(textToSend)`.
        3.  Clears EditText.
*   **`OnPageChangeListener`**:
    *   Handles focus management when swiping between pages.
    *   **Page 0 Selected**: Requests focus for `mTerminalView`.
    *   **Page 1 Selected**: Requests focus for the `EditText` input view.

### C. `TermuxTerminalViewClient.java` (Input Interceptor)
*   **Key Handling**: `onKeyDown` intercepts hardware keyboard events.
*   **Shortcut Registration**: Maps `Ctrl` + `Alt` + `i` to `mActivity.toggleTerminalToolbarViewPager()`.

## 3. Detailed Logic Flows

### Initialization
1.  `TermuxActivity.onCreate()` calls `setTerminalToolbarView(savedInstanceState)`.
2.  `mTermuxTerminalExtraKeys` is instantiated (loads button configuration from properties).
3.  `TerminalToolbarViewPager` adapter is set.
4.  Visibility is determined by `TermuxAppSharedPreferences`.
5.  Height is calculated via `setTerminalToolbarHeight()`, which depends on the `ExtraKeysInfo` matrix (number of rows).

### Text Input to Terminal
The path for text entering the terminal via Page 1:
1.  User types in `EditText` (Page 1).
2.  User presses Enter on soft keyboard.
3.  `PageAdapter.instantiateItem` -> `OnEditorActionListener`:
    ```java
    String textToSend = editText.getText().toString();
    if (textToSend.length() == 0) textToSend = "\r"; // Handle empty enter
    session.write(textToSend); // Send raw bytes to PTY
    editText.setText(""); // Clear input
    ```

### Mode Switching (Swipe vs Shortcut)
*   **Swipe**: Native `ViewPager` behavior. `OnPageChangeListener` updates focus automatically.
*   **Shortcut (Ctrl+Alt+i)**:
    1.  `TermuxTerminalViewClient.onKeyDown` detects keys.
    2.  Calls `TermuxActivity.toggleTerminalToolbarViewPager()`.
    3.  Activity checks `pager.getCurrentItem()`:
        *   If 0 -> Set to 1.
        *   If 1 -> Set to 0.
    4.  `OnPageChangeListener` triggers, handling focus transfer.

## 4. Configuration
*   **Show/Hide**: Controlled by `termux.properties` or GUI preferences.
*   **Extra Keys Layout**: JSON configuration in `~/.termux/termux.properties` (handled by `TermuxTerminalExtraKeys`).

## 5. Recent Modifications
*   Added `toggleTerminalToolbarViewPager()` in `TermuxActivity`.
*   Added `Ctrl+Alt+i` shortcut handler in `TermuxTerminalViewClient`.
