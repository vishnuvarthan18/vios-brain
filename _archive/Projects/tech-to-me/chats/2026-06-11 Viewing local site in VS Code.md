---
tags: chat
date: 2026-06-11
source: Claude personal account
uuid: aeaff5d2-2349-4bc4-859e-97388faff907
---
# Viewing local site in VS Code

## Summary
**Conversation Overview**

The person is building a website using VS Code and needed help viewing their local development site on a mobile device. The conversation progressed from a general question about previewing sites locally, to a specific goal of accessing the local server from a phone. Through a clarifying exchange, it was established that the person is using macOS and the Live Server VS Code extension as their dev server.

Claude provided a detailed step-by-step guide tailored to that exact setup, covering how to find the Mac's local IP address using the terminal command `ipconfig getifaddr en0`, how to configure Live Server to bind to `0.0.0.0` by editing VS Code's User Settings JSON, how to restart Live Server and confirm the port number, and how to access the site from a phone browser using the local IP and port (e.g., `http://192.168.1.42:5500`). The guide also included a troubleshooting note about macOS firewall potentially blocking the connection, with instructions for temporarily disabling it to test. The person's communication style suggests they prefer clear, step-by-step instructions rather than high-level overviews, which prompted Claude to structure the final response in a numbered format with exact commands and settings.

## Chat

**Vishnu:** is ther any waay to see the site that is in local i am using vs code for building

**Claude:** Yes — VS Code has several ways to preview a site locally:

**Static HTML/CSS/JS:**
- **Live Server extension** (most common). Install from Extensions panel, then right-click your `.html` file → "Open with Live Server". Auto-reloads on save at `http://127.0.0.1:5500`.
- **Live Preview extension** (by Microsoft). Opens a preview pane inside VS Code itself. Click the preview icon top-right of an open HTML file.

**Framework/build-tool projects** (React, Vite, Next, etc.) — use the dev server in VS Code's integrated terminal:
```bash
npm run dev      # Vite / many setups
npm start        # Create React App
```
Then open the printed URL (e.g. `http://localhost:5173`) in your browser.

Which type of project is it? That determines the right answer.

**Vishnu:** i want to see in my mobile

**Claude:** To view your local site on your phone, both devices must be on the **same Wi-Fi network**, then you access your computer's local IP instead of `localhost`.

**Steps:**

1. **Find your computer's local IP:**
```bash
ipconfig getifaddr en0    # macOS Wi-Fi
hostname -I               # Linux
ipconfig                  # Windows → look for "IPv4 Address"
```
Gives something like `192.168.1.42`.

2. **Bind your dev server to your network**, not just localhost:
- **Live Server:** add to VS Code `settings.json` → `"liveServer.settings.host": "0.0.0.0"`, then restart it.
- **Vite:** run `npm run dev -- --host`
- **Create React App:** `HOST=0.0.0.0 npm start`

3. **On your phone's browser**, go to:
```
http://192.168.1.42:5500
```
(use your actual IP + the port your server printed)

**If it doesn't load:** your computer's firewall is likely blocking the port. Allow it, or temporarily disable the firewall to test.

Easier alternative if Wi-Fi/firewall fights you: **Cloudflare Tunnel** or **ngrok** gives a public URL to your local server in one command — works over any network.

What's your OS and which server are you running? I'll give you the exact command.

**Vishnu:** tell me step by step

**Claude:** Need your OS and dev server to give exact commands.

_[tool: ask_user_input_v0]_

**Vishnu:** Q: What's your computer's OS?
A: macOS

Q: What are you running to serve the site?
A: Live Server (VS Code)

**Claude:** macOS + Live Server, step by step:

1. **Find your Mac's IP.** Open VS Code's terminal (`Ctrl+~`) and run:
```bash
ipconfig getifaddr en0
```
Note the result, e.g. `192.168.1.42`. (If it returns nothing, try `en1` instead of `en0`.)

2. **Make Live Server listen on the network.** Open VS Code settings JSON: `Cmd+Shift+P` → type "Preferences: Open User Settings (JSON)" → Enter. Add this inside the `{ }`:
```json
"liveServer.settings.host": "0.0.0.0",
"liveServer.settings.useLocalIp": true
```
Save the file.

3. **Restart Live Server.** If it's running, click the port number in the bottom status bar to stop it, then right-click your `.html` file → "Open with Live Server" again.

4. **Confirm the port.** Look at the browser tab Live Server opened on your Mac — the URL ends in a port, usually `:5500`.

5. **Make sure your phone is on the same Wi-Fi** as your Mac.

6. **On your phone's browser, type:**
```
http://192.168.1.42:5500
```
Use your actual IP from step 1 + the port from step 4.

**If it won't load:** macOS firewall is blocking it. Go to System Settings → Network → Firewall → turn it off temporarily to test. If it loads with the firewall off, turn it back on and add an allow rule for VS Code.
