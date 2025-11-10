# Silent Coder - Complete Setup & Run Guide

## 📋 Project Overview

**Silent Coder** is an Electron desktop application that helps you solve coding problems using AI. It:
- Takes screenshots of coding problems
- Uses AI (OpenAI or Google Gemini) to extract problem details
- Generates solutions with explanations, time/space complexity analysis
- Provides debugging help for your code

## 🛠️ Prerequisites

Before you begin, make sure you have the following installed:

### 1. **Node.js** (v18 or higher)
   - Download from: https://nodejs.org/
   - Verify installation:
     ```bash
     node --version
     npm --version
     ```

### 2. **pnpm** (Package Manager)
   - The project uses `pnpm` as specified in `package.json`
   - Install pnpm globally:
     ```bash
     npm install -g pnpm
     ```
   - Or using standalone installer:
     - Windows: `iwr https://get.pnpm.io/install.ps1 -useb | iex`
     - Or visit: https://pnpm.io/installation
   - Verify installation:
     ```bash
     pnpm --version
     ```

### 3. **API Key** (Required for functionality)
   You need **ONE** of the following:
   
   **Option A: OpenAI API Key**
   - Sign up at: https://platform.openai.com/signup
   - Get your API key from: https://platform.openai.com/api-keys
   - Format: Starts with `sk-` (e.g., `sk-...`)
   
   **Option B: Google Gemini API Key**
   - Get your API key from: https://aistudio.google.com/app/apikey
   - Format: Starts with `AIzaSyB...`

## 📦 Installation Steps

### Step 1: Clone/Navigate to Project
```bash
cd D:\course\Learning_Coding_Projects\silent-coder
```

### Step 2: Install Dependencies
```bash
pnpm install
```

This will:
- Install all Node.js dependencies
- Install Electron and its dependencies
- Set up the build environment

**Note:** The first installation may take a few minutes as it downloads Electron binaries.

### Step 3: Verify Installation
Check that all dependencies are installed correctly:
```bash
pnpm list --depth=0
```

## 🚀 Running the Application

### Development Mode
To run the app in development mode with hot-reload:
```bash
pnpm dev
```

This will:
- Start the Electron app
- Enable hot-reload for React components
- Open the application window

### Production Build
To build the application for your platform:

**For Windows:**
```bash
pnpm build:win
```

**For macOS:**
```bash
pnpm build:mac
```

**For Linux:**
```bash
pnpm build:linux
```

The built application will be in the `dist` folder.

## ⚙️ Initial Setup (First Run)

### 1. Configure API Key
When you first run the app:
1. The welcome screen will appear
2. Click **"Open Settings"** button
3. Select your API provider (OpenAI or Gemini)
4. Enter your API key
5. Click **"Test & Save"** to validate the key
6. The app will automatically detect the provider based on your API key format

### 2. Configure Models (Optional)
In Settings, you can choose different AI models:
- **OpenAI Models:** `gpt-4o-mini`, `gpt-4o`
- **Gemini Models:** `gemini-2.0-flash`, `gemini-1.5-pro`

### 3. Select Programming Language
Choose your preferred programming language (default: Python)

## ⌨️ Keyboard Shortcuts

The app uses global keyboard shortcuts (work system-wide):

| Shortcut | Action |
|----------|--------|
| `Ctrl+B` / `Cmd+B` | Toggle window visibility |
| `Ctrl+H` / `Cmd+H` | Take screenshot |
| `Ctrl+L` / `Cmd+L` | Delete last screenshot |
| `Ctrl+Enter` / `Cmd+Enter` | Process screenshots |
| `Ctrl+R` / `Cmd+R` | Reset view / Cancel request |
| `Ctrl+Q` / `Cmd+Q` | Quit application |
| `Ctrl+Left/Right/Up/Down` | Move window position |
| `Ctrl+[` / `Ctrl+]` | Decrease/Increase opacity |
| `Ctrl+-` / `Ctrl+=` | Zoom out/in |
| `Ctrl+0` | Reset zoom |

## 📖 How to Use

### Basic Workflow:

1. **Take Screenshots:**
   - Press `Ctrl+H` (or `Cmd+H` on Mac) to capture a screenshot
   - The app will hide itself while taking the screenshot
   - Take multiple screenshots if the problem spans multiple screens

2. **Process Screenshots:**
   - Press `Ctrl+Enter` (or `Cmd+Enter` on Mac) to process
   - The AI will:
     - Extract problem statement, constraints, examples
     - Generate a solution with code
     - Provide time/space complexity analysis
     - Explain the approach

3. **Debug Mode:**
   - If your solution has errors, take screenshots of the error messages
   - Add them to the queue and process again
   - The app will provide debugging help

4. **View Results:**
   - Solutions are displayed with:
     - Complete code
     - Key insights and thoughts
     - Time complexity explanation
     - Space complexity explanation

## 🗂️ Project Structure

```
silent-coder/
├── src/
│   ├── main/              # Electron main process
│   │   ├── index.ts      # Main entry point
│   │   └── lib/          # Core functionality
│   │       ├── config-manager.ts      # API key & settings
│   │       ├── processing-manager.ts  # AI processing
│   │       ├── screenshot-manager.ts # Screenshot handling
│   │       └── keyboard-shortcut.ts   # Global shortcuts
│   ├── preload/          # Preload scripts
│   └── renderer/         # React UI
│       └── src/
│           ├── components/  # React components
│           └── App.tsx     # Main React app
├── build/                # Build assets (icons, etc.)
├── package.json          # Dependencies & scripts
├── electron.vite.config.ts  # Build configuration
└── README.md            # Basic project info
```

## 🔧 Configuration Files

The app stores configuration in:
- **Windows:** `%APPDATA%\silent-coder\config.json`
- **macOS:** `~/Library/Application Support/silent-coder/config.json`
- **Linux:** `~/.config/silent-coder/config.json`

This file contains:
- API key (encrypted/stored securely)
- API provider (openai/gemini)
- Model selections
- Language preference
- Window opacity settings

## 🐛 Troubleshooting

### Issue: "pnpm: command not found"
**Solution:** Install pnpm globally:
```bash
npm install -g pnpm
```

### Issue: "API key invalid" error
**Solutions:**
- Verify your API key is correct
- Check if you have sufficient API credits/quota
- Ensure the API key format matches the provider:
  - OpenAI: Must start with `sk-`
  - Gemini: Must start with `AIzaSyB`

### Issue: Screenshots not working
**Solutions:**
- Check if you have screen capture permissions (macOS/Linux)
- On Windows, ensure the app has proper permissions
- Try running as administrator if needed

### Issue: App window not visible
**Solution:**
- Press `Ctrl+B` (or `Cmd+B`) to toggle visibility
- Check opacity settings in the app

### Issue: Build fails
**Solutions:**
- Ensure you're using the correct Node.js version (v18+)
- Clear cache and reinstall:
  ```bash
  pnpm store prune
  rm -rf node_modules
  pnpm install
  ```

### Issue: Dependencies installation fails
**Solutions:**
- Check your internet connection
- Try clearing npm/pnpm cache:
  ```bash
  pnpm store prune
  ```
- On Windows, you might need to run PowerShell as Administrator

## 📝 Available Scripts

| Command | Description |
|---------|-------------|
| `pnpm dev` | Start development server |
| `pnpm build` | Build for production (all platforms) |
| `pnpm build:win` | Build Windows executable |
| `pnpm build:mac` | Build macOS app |
| `pnpm build:linux` | Build Linux app |
| `pnpm lint` | Run ESLint |
| `pnpm format` | Format code with Prettier |
| `pnpm typecheck` | Run TypeScript type checking |

## 🔒 Security Notes

- **API Keys:** Your API keys are stored locally in the config file
- **Never commit:** Don't commit your API keys to version control
- **Permissions:** The app needs screen capture permissions to take screenshots

## 📚 Additional Resources

- **Electron Documentation:** https://www.electronjs.org/
- **React Documentation:** https://react.dev/
- **OpenAI API Docs:** https://platform.openai.com/docs
- **Google Gemini API Docs:** https://ai.google.dev/docs

## 🎯 Quick Start Checklist

- [ ] Install Node.js (v18+)
- [ ] Install pnpm globally
- [ ] Run `pnpm install` in project directory
- [ ] Get an API key (OpenAI or Gemini)
- [ ] Run `pnpm dev` to start the app
- [ ] Configure API key in Settings
- [ ] Test by taking a screenshot (`Ctrl+H`) and processing it (`Ctrl+Enter`)

---

**Need Help?** Check the troubleshooting section above or review the code comments in the source files.

