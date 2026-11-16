# Expo & React Native Android Troubleshooting Guide

A complete reference guide for diagnosing and fixing common connection, slow bundling, Fast Refresh, and USB debugging issues in Expo Go on Android.

---

## Table of Contents
1. ["Packager is not running" Error](#1-packager-is-not-running-error)
2. [Slow Bundle Downloading on Wi-Fi](#2-slow-bundle-downloading-on-wi-fi)
3. [App Not Auto-Rendering (Fast Refresh Issues)](#3-app-not-auto-rendering-fast-refresh-issues)
4. [Fast & Reliable Setup via USB (Recommended)](#4-fast--reliable-setup-via-usb-recommended)
5. [Quick Command Cheat Sheet](#5-quick-command-cheat-sheet)

---

## 1. "Packager is not running" Error

**Error Message:**  
`Something went wrong. Packager is not running at http://<IP_ADDRESS>:8081/status`  
*(Terminal shows: `No apps connected. Sending "reload" to all React Native apps failed.`)*

### Root Cause
Metro bundler is running on your computer, but incoming traffic from your Android phone to port `8081` is blocked by the firewall, network profile, or router settings.

### Step-by-Step Diagnosis & Fix
1. **Verify Metro Status Locally:**  
   Open `http://localhost:8081/status` in your computer's browser. It should display `packager-status:running`.
2. **Verify Local IPv4 Address:**  
   - **Windows:** Run `ipconfig` in Command Prompt.  
   - **macOS/Linux:** Run `ifconfig` or `ipconfig getifaddr en0`.  
   Ensure the IP matches the one displayed in Expo Go. If it changed, restart with `npx expo start -c`.
3. **Test Reachability from Phone:**  
   Open `http://<YOUR_COMPUTER_IP>:8081/status` in Chrome on your phone. If it times out, the local network or firewall is blocking the port.
4. **Fix Firewall & Network Profile (Windows):**  
   - Go to **Settings > Network & Internet > Wi-Fi** -> Select your network -> Change **Network profile type** from **Public** to **Private**.  
   - In **Windows Defender Firewall**, ensure **Node.js JavaScript Runtime** is allowed on Private networks.
5. **Check VPNs & Router Isolation:**  
   - Disable any active VPNs or proxies on both your phone and computer.  
   - Ensure both devices are on the exact same Wi-Fi network (not a guest network) and that **AP/Client Isolation** is disabled on your router.
6. **Fallback Tunnel Mode:**  
   If local Wi-Fi restrictions cannot be changed, run:
   ```bash
   npx expo start --tunnel
   ```

---

## 2. Slow Bundle Downloading on Wi-Fi

### Root Cause
Even a blank Expo template bundles the core React Native and Expo JavaScript runtime (~15–25 MB uncompressed). On 2.4GHz Wi-Fi or congested networks, transferring this bundle takes noticeably longer.

### Fixes
- **Switch to 5GHz Wi-Fi:** Ensure both your computer and phone are connected to the `5GHz` band rather than `2.4GHz`.
- **Avoid `--tunnel` for Local Dev:** Tunnel mode routes traffic through external internet servers, making downloads much slower than local LAN or USB.
- **Clear Expo Go Cache:** On Android, go to **Settings > Apps > Expo Go > Storage > Clear Cache** if stale cache slows down builds.
- **Disable Virtual Network Adapters:** Temporarily turn off Docker, WSL, or VPN interfaces if Expo binds to the wrong network adapter.
- **Switch to USB:** Use a USB cable for near-instant bundle transfers.

---

## 3. App Not Auto-Rendering (Fast Refresh Issues)

### Root Cause
Unstable Wi-Fi drops the WebSocket connection between your code editor/Metro server and Expo Go, or Fast Refresh is disabled.

### Fixes
1. **Enable Fast Refresh in Developer Menu:**  
   Shake your phone (or press `m` in the terminal) to open the Expo Developer Menu and verify **Enable Fast Refresh** is turned ON.
2. **Ensure File Saving:**  
   Make sure changes are saved (`Ctrl + S` / `Cmd + S`). In VS Code, enable **File > Auto Save**.
3. **Follow Component Naming Rules:**  
   React component names must start with a **Capital Letter** (e.g., `App`, `HomeScreen`) and be properly exported (`export default`).
4. **Test Manual Reload:**  
   Press `r` in the terminal. If it prints `No apps connected`, the WebSocket connection dropped—switch to USB.
5. **Clear Bundler Cache:**  
   Restart Metro with:
   ```bash
   npx expo start -c
   ```

---

## 4. Fast & Reliable Setup via USB (Recommended)

Connecting via USB eliminates Wi-Fi latency, firewall blocks, and dropped WebSocket connections.

### Step 1: Enable USB Debugging on Android
1. Go to **Settings > About Phone** and tap **Build Number** 7 times to unlock **Developer Options**.
2. Open **Settings > System / Additional Settings > Developer Options**.
3. Turn ON **USB Debugging**.
4. Connect your phone to your computer using a data-capable USB cable.
5. On the phone prompt *"Allow USB debugging?"*, check **Always allow from this computer** and tap **Allow**.

### Step 2: Verify ADB Connection
Run the following command in your project terminal:
```bash
adb devices
```
- **Expected Output:** `<device_id>    device`
- If it shows `unauthorized`, unlock your phone and accept the debugging prompt.

### Step 3: Forward Port & Launch Expo
1. Map port `8081` over USB:
   ```bash
   adb reverse tcp:8081 tcp:8081
   ```
2. Start Expo in localhost mode:
   ```bash
   npx expo start --localhost
   ```
3. Press **`a`** in the terminal to automatically launch the project in Expo Go on your Android device.

---

## 5. Quick Command Cheat Sheet

| Command | Purpose |
| :--- | :--- |
| `adb devices` | Check if Android device is connected via USB |
| `adb reverse tcp:8081 tcp:8081` | Forward Metro port `8081` directly over USB |
| `npx expo start --localhost` | Start Expo server for USB / local emulator connection |
| `npx expo start -c` | Start Expo and clear Metro bundler cache |
| `npx expo start --tunnel` | Start Expo using external tunnel (bypasses firewall) |
| `r` *(inside Metro terminal)* | Manually reload connected app |
| `m` *(inside Metro terminal)* | Open Expo Developer Menu on connected device |
