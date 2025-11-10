# Silent Coder - Complete Codebase Explanation

## Overview

**Silent Coder** is an Electron-based desktop application designed to help solve coding problems during online interviews/exams (like Google Meet) without being detected. It captures screenshots of coding problems, uses AI (OpenAI or Google Gemini) to extract problem details, and generates solutions.

---

## 🏗️ Architecture Overview

### Technology Stack

- **Framework**: Electron (v35.0.0) - Desktop app framework
- **Frontend**: React 19 + TypeScript
- **UI**: TailwindCSS
- **AI Integration**: OpenAI API & Google Gemini API
- **Screenshot Library**: `screenshot-desktop` + native PowerShell (Windows)

### Project Structure

```
silent-coder/
├── src/
│   ├── main/              # Electron main process (Node.js)
│   │   ├── index.ts       # Main entry point, window creation
│   │   └── lib/
│   │       ├── screenshot-manager.ts    # Handles screenshot capture
│   │       ├── keyboard-shortcut.ts     # Global keyboard shortcuts
│   │       ├── processing-manager.ts    # AI processing logic
│   │       ├── config-manager.ts        # Configuration management
│   │       └── ipc-handler.ts          # IPC communication
│   ├── preload/           # Bridge between main and renderer
│   │   └── index.ts       # Exposes safe APIs to renderer
│   └── renderer/          # React frontend (browser process)
│       └── src/
│           ├── App.tsx
│           ├── components/
│           └── providers/
└── package.json
```

---

## 🔄 How It Works - Step by Step

### 1. **Application Initialization** (`src/main/index.ts`)

When the app starts:

1. **Window Creation** (lines 44-169):
   - Creates a transparent, frameless Electron window
   - Window is configured with special properties for stealth:
     ```typescript
     {
       alwaysOnTop: true,        // Stays above other windows
       frame: false,              // No window frame/chrome
       transparent: true,        // Transparent background
       skipTaskbar: true,         // Hidden from taskbar
       type: 'panel',             // Special window type
       show: false,               // Initially hidden
     }
     ```

2. **Stealth Features**:
   - `setContentProtection(true)` - Prevents screen recording/sharing from capturing content
   - `setHiddenInMissionControl(true)` - Hidden on macOS Mission Control
   - `setVisibleOnAllWorkspaces(true)` - Works across virtual desktops
   - `setAlwaysOnTop(true, 'screen-saver', 1)` - Highest priority

3. **Manager Initialization**:
   - `ScreenshotManager` - Handles screenshot capture
   - `ProcessingManager` - Handles AI processing
   - `KeyboardShortcutHelper` - Registers global shortcuts

### 2. **Screenshot Capture** (`src/main/lib/screenshot-manager.ts`)

**Key Method: `takeScreenshot()`** (lines 225-285)

**Process:**

1. **Hide Window** (line 230):

   ```typescript
   hideMainWindow() // Sets opacity to 0, ignores mouse events
   ```

2. **Wait for Window to Disappear** (line 232-233):

   ```typescript
   const hideDelay = process.platform === 'win32' ? 500 : 300
   await new Promise((resolve) => setTimeout(resolve, hideDelay))
   ```

   - Ensures window is completely hidden before screenshot

3. **Capture Screenshot** (line 237):
   - **Windows**: Uses PowerShell script (lines 179-201) or `screenshot-desktop` library
   - **Other Platforms**: Uses `screenshot-desktop` library
   - The screenshot captures the **entire screen**, not just the app window

4. **Save Screenshot** (lines 243-276):
   - Saves to `screenshots/` directory
   - Maintains a queue (max 5 screenshots)
   - Returns file path

5. **Show Window Again** (line 282):
   ```typescript
   showMainWindow() // Restores opacity and position
   ```

**Why This Works:**

- The window is **completely hidden** (opacity 0) when screenshot is taken
- Screenshot captures the **entire desktop**, including the problem statement
- Google Meet screen sharing only sees what's visible - since the window is hidden, it's not detected

### 3. **AI Processing** (`src/main/lib/processing-manager.ts`)

**Two-Phase Processing:**

#### Phase 1: Problem Extraction (lines 306-486)

1. **Read Screenshots** (lines 175-188):
   - Converts screenshots to base64
   - Prepares for AI API

2. **Send to AI** (lines 323-441):
   - **OpenAI**: Uses GPT-4o with vision capabilities
   - **Gemini**: Uses Gemini 2.0 Flash with vision
   - Extracts: problem statement, constraints, examples

3. **Parse Response** (lines 364-378):
   - Extracts JSON from AI response
   - Stores in `problemInfo`

#### Phase 2: Solution Generation (lines 488-677)

1. **Generate Solution** (lines 507-592):
   - Sends problem info to AI
   - Requests: code, thoughts, time/space complexity

2. **Parse Solution** (lines 594-654):
   - Extracts code from markdown
   - Parses complexity explanations
   - Formats response

3. **Send to Frontend** (line 212):
   - Emits `SOLUTION_SUCCESS` event
   - Frontend displays solution

### 4. **Global Keyboard Shortcuts** (`src/main/lib/keyboard-shortcut.ts`)

**Registered Shortcuts** (lines 44-139):

- `Cmd/Ctrl + H`: Take screenshot
- `Cmd/Ctrl + Enter`: Process screenshots (generate solution)
- `Cmd/Ctrl + B`: Toggle window visibility
- `Cmd/Ctrl + L`: Delete last screenshot
- `Cmd/Ctrl + Left/Right/Up/Down`: Move window
- `Cmd/Ctrl + [/]`: Adjust opacity
- `Cmd/Ctrl + R`: Reset (cancel processing, clear queues)

**Why This Matters:**

- Shortcuts work **globally** - even when window is hidden
- User can trigger actions without revealing the app
- Works during screen sharing because shortcuts are system-level

### 5. **Frontend (React)** (`src/renderer/src/`)

**Components:**

- `App.tsx`: Main app wrapper, handles initialization
- `main-app.tsx`: Routes between queue and solutions views
- `screenshots-view.tsx`: Displays screenshot queue
- `solutions/`: Displays AI-generated solutions
- `queue-commands.tsx`: UI for taking screenshots and processing

**State Management:**

- React Query for data fetching
- IPC events for real-time updates
- Local state for UI

---

## 🕵️ How Google Meet Can't Detect It

### 1. **Window Stealth Configuration**

**Content Protection** (`src/main/index.ts:120`):

```typescript
state.mainWindow.setContentProtection(true)
```

- **What it does**: Prevents screen recording/sharing APIs from capturing the window content
- **Why it works**: Google Meet uses screen sharing APIs that respect this flag
- **Result**: The window content is **not visible** in screen shares

**Skip Taskbar** (`src/main/index.ts:75`):

```typescript
skipTaskbar: true
```

- Window doesn't appear in taskbar/dock
- Harder to detect in process lists

**Hidden in Mission Control** (`src/main/index.ts:128`):

```typescript
state.mainWindow.setHiddenInMissionControl(true)
```

- On macOS, window is hidden from window switcher
- Can't be seen when switching windows

### 2. **Opacity Control**

**Dynamic Opacity** (`src/main/index.ts:274-305`):

- Window can be set to opacity 0 (completely invisible)
- User can adjust opacity with `Cmd/Ctrl + [/]`
- When opacity is 0, window is effectively invisible

**Initial Opacity** (lines 155-168):

- Can start with opacity 0
- Window appears "closed" but is actually running

### 3. **Screenshot Timing**

**Hide Before Screenshot** (`src/main/lib/screenshot-manager.ts:230`):

```typescript
hideMainWindow() // Opacity = 0
await new Promise((resolve) => setTimeout(resolve, 500)) // Wait
const screenshotBuffer = await this.captureScreenshot() // Capture
showMainWindow() // Restore
```

**Why This Works:**

- Window is **completely hidden** (opacity 0) when screenshot is taken
- Google Meet screen sharing only captures **visible windows**
- Since the window is invisible, it's not in the screen share
- Screenshot captures the **entire desktop** (including the problem), but the app window itself is hidden

### 4. **System-Level Screenshot APIs**

**Windows Implementation** (`src/main/lib/screenshot-manager.ts:148-223`):

- Uses PowerShell with `System.Drawing.Graphics.CopyFromScreen()`
- This is a **system-level API** that captures the screen buffer directly
- **Not** a window capture API - it captures the raw screen
- Google Meet screen sharing doesn't intercept these APIs

**Why Screen Sharing Doesn't See It:**

- Screen sharing typically captures **window content** or **display output**
- The app window is hidden (opacity 0) during screenshot
- The screenshot API captures the **raw screen buffer**, not window content
- Google Meet's screen sharing doesn't monitor these low-level APIs

### 5. **Global Shortcuts Work When Hidden**

**Keyboard Shortcut Registration** (`src/main/lib/keyboard-shortcut.ts:44`):

```typescript
globalShortcut.register('CommandOrControl+H', async () => {
  // Take screenshot even when window is hidden
})
```

**Why This Works:**

- `globalShortcut` is an Electron API that registers **system-level** shortcuts
- These shortcuts work **even when the window is hidden**
- User can press `Cmd+H` during a Google Meet call
- Window briefly hides, takes screenshot, shows again
- Google Meet doesn't detect keyboard shortcuts (they're system-level)

### 6. **Window Positioning**

**Off-Screen Positioning** (`src/main/index.ts:245-272`):

- Window can be moved partially or completely off-screen
- Can be positioned at negative coordinates
- Even if visible, can be outside the visible area

**Why This Helps:**

- If opacity isn't 0, window can be positioned off-screen
- Not visible to user or screen sharing
- Still functional (can receive shortcuts, process data)

### 7. **No Network Activity to Google Meet**

**Local Processing:**

- Screenshots are saved **locally** (in `appData/silent-coder/screenshots/`)
- AI API calls go directly to OpenAI/Gemini (not through Google Meet)
- No communication with Google Meet servers
- Google Meet has **no way to detect** the app's network activity

### 8. **Process Name Stealth**

**Electron App:**

- Runs as a standard Electron process
- Process name is typically "Electron" or the app name
- Not obviously named "cheat-app" or similar
- Blends in with other Electron apps

---

## 🔍 Technical Deep Dive

### IPC Communication Flow

**Main Process → Renderer:**

1. Main process captures screenshot
2. Sends `screenshot-taken` event via IPC
3. Renderer receives event and updates UI

**Renderer → Main Process:**

1. User clicks button in React UI
2. Calls `window.electronAPI.triggerScreenshot()`
3. Preload script forwards to main process via IPC
4. Main process handles request

**Why Context Isolation Matters:**

- `contextIsolation: true` (line 88 in `index.ts`)
- Prevents renderer from accessing Node.js APIs directly
- Only exposes safe APIs via `contextBridge`
- Security best practice

### Screenshot Capture Methods

**Windows (Primary Method):**

```typescript
// Uses screenshot-desktop library
await screenshot({ path: tempFilePath })
```

**Windows (Fallback):**

```typescript
// PowerShell script using System.Drawing
$graphics.CopyFromScreen($bounds.Left, $bounds.Top, 0, 0, $bounds.Size)
```

**Why Two Methods:**

- Primary method uses library (cross-platform)
- Fallback uses native Windows API (more reliable)
- Ensures screenshot works even if library fails

### AI Processing Pipeline

**Step 1: Extract Problem** (lines 306-486)

- Input: Screenshot images (base64)
- AI Model: GPT-4o or Gemini 2.0 Flash
- Output: JSON with problem details

**Step 2: Generate Solution** (lines 488-677)

- Input: Problem details + language preference
- AI Model: GPT-4o or Gemini 1.5 Flash
- Output: Code + explanations

**Step 3: Debug Mode** (lines 703-902)

- User takes additional screenshots (errors, test cases)
- AI analyzes and provides debugging help
- Combines original screenshots with debug screenshots

### Configuration Management

**Config File Location:**

- `appData/silent-coder/config.json`
- Stores: API keys, model preferences, language, opacity

**Security:**

- API keys stored in plain text (local file)
- Not encrypted (could be improved)
- Only accessible to the app

---

## 🎯 Key Anti-Detection Features Summary

1. ✅ **Content Protection**: `setContentProtection(true)` - Hides from screen sharing
2. ✅ **Transparent Window**: Can be made completely invisible (opacity 0)
3. ✅ **Skip Taskbar**: Hidden from taskbar/dock
4. ✅ **Hidden in Mission Control**: Not visible in window switcher
5. ✅ **Screenshot While Hidden**: Window hidden when screenshot is taken
6. ✅ **System-Level Screenshot API**: Uses low-level screen capture
7. ✅ **Global Shortcuts**: Work even when window is hidden
8. ✅ **Off-Screen Positioning**: Can be positioned outside visible area
9. ✅ **No Network to Google Meet**: All communication is local or to AI APIs
10. ✅ **Stealth Process Name**: Blends in with other apps

---

## 🚨 Limitations & Detection Risks

### What Could Potentially Detect It:

1. **Screen Recording Software**:
   - Some screen recorders might capture the window even with content protection
   - Solution: Keep opacity at 0 during use

2. **Process Monitoring**:
   - If someone checks running processes, they might see "Electron" or app name
   - Solution: Rename the process or use a generic name

3. **Network Monitoring**:
   - If network traffic is monitored, AI API calls might be visible
   - Solution: Use VPN or ensure network monitoring isn't active

4. **Screen Sharing with Multiple Monitors**:
   - If sharing a specific monitor, app on another monitor won't be visible
   - But if sharing entire desktop, might see window if opacity > 0

5. **Keyboard Shortcut Detection**:
   - Some proctoring software monitors keyboard shortcuts
   - Solution: Use mouse clicks instead of shortcuts (if implemented)

### Best Practices for Stealth:

1. **Set Opacity to 0** before joining Google Meet
2. **Position Window Off-Screen** as backup
3. **Use Keyboard Shortcuts** instead of visible UI
4. **Close Window** (hide it) when not actively using
5. **Monitor Network** if in strict environment

---

## 📝 Code Flow Example

### User Takes Screenshot During Google Meet:

1. **User presses `Cmd+H`** (or clicks button)
2. **Keyboard shortcut handler** (`keyboard-shortcut.ts:65`) fires
3. **Main process** calls `takeScreenshot()`
4. **Window hides** (`hideMainWindow()` - sets opacity to 0)
5. **Wait 500ms** for window to disappear
6. **Screenshot captured** (entire desktop, including problem statement)
7. **Window shows again** (`showMainWindow()` - restores opacity)
8. **Screenshot saved** to local file
9. **UI updates** via IPC event
10. **Google Meet screen sharing** never saw the window (it was hidden)

### User Processes Screenshots:

1. **User presses `Cmd+Enter`** (or clicks "Solve")
2. **ProcessingManager.processScreenshots()** called
3. **Screenshots read** from disk, converted to base64
4. **Sent to AI** (OpenAI or Gemini) with vision API
5. **Problem extracted** from images
6. **Solution generated** by AI
7. **Displayed in UI** (window can be hidden while processing)
8. **User copies solution** to coding platform

---

## 🔧 Configuration

### Settings Available:

- **API Provider**: OpenAI or Gemini
- **API Key**: Your API key for the chosen provider
- **Models**:
  - Extraction model (for problem extraction)
  - Solution model (for code generation)
  - Debugging model (for error analysis)
- **Language**: Preferred programming language (Python, Java, etc.)
- **Opacity**: Window opacity (0.1 to 1.0)

### Keyboard Shortcuts:

- `Cmd/Ctrl + H`: Take screenshot
- `Cmd/Ctrl + Enter`: Process screenshots (solve)
- `Cmd/Ctrl + B`: Toggle window visibility
- `Cmd/Ctrl + L`: Delete last screenshot
- `Cmd/Ctrl + R`: Reset (clear all)
- `Cmd/Ctrl + [/]`: Decrease/increase opacity
- `Cmd/Ctrl + Left/Right/Up/Down`: Move window

---

## 🎓 Conclusion

**Silent Coder** is a sophisticated Electron application that uses multiple stealth techniques to avoid detection during online interviews:

1. **Window-level stealth**: Content protection, transparency, hidden from taskbar
2. **Timing-based stealth**: Hides window during screenshot capture
3. **System-level APIs**: Uses low-level screenshot APIs that screen sharing doesn't intercept
4. **Global shortcuts**: Works even when window is hidden
5. **Local processing**: No communication with proctoring software

The combination of these techniques makes it very difficult for Google Meet or similar screen sharing software to detect the application, especially when used with opacity set to 0 and proper window positioning.

**Important Note**: This tool is designed for educational purposes. Using it to cheat on exams or interviews may violate academic or professional integrity policies.
