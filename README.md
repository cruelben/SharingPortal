# SharingPortal 🌐

**SharingPortal** is a lightweight, serverless, single-file web application designed for fast, seamless, and private file sharing. It supports direct **P2P (Peer-to-Peer)** browser-to-browser transfers as well as an anonymous **Temporary Cloud Storage** fallback. 

Built entirely with HTML, Tailwind CSS, and vanilla JavaScript, **SharingPortal** requires **no account, no registration, and zero setup**. You can host it for free on **GitHub Pages** or run it locally by simply opening the `index.html` file in any modern web browser.

---

## ✨ Features

- 🚀 **Dual Transfer Modes:**
  - **P2P Direct Transfer:** Browser-to-browser transfer without uploading files to any permanent server. Unlimited file sizes.
  - **Temporary Storage Deposit:** Upload files anonymously to a temporary cloud deposit so receivers can download them even if you go offline.
- 📁 **Multi-File Support:** Select or drag and drop multiple files at once.
- 🔑 **Custom & Random Redeem Codes:** Use custom redemption codes (e.g., `pippo`, `PROJECT2026`) or generate short 5-character randomized keys.
- 🔐 **Privacy First & Local Queue Management:** Track and manage your uploaded files in the **Queue** tab. Storage privacy is kept using browser `localStorage` tokens—you only see and manage the files you uploaded.
- 🎨 **Relaxing Dark & Light Themes:** Elegant Slate and Sky-Blue visual palette designed for low eye strain and high contrast readability.
- 🌐 **Auto-Language Detection:** Automatically detects browser language settings (supports Italian & English).
- ⚙️ **Customizable Options:** Set temporary storage limits (default 500 MB / 1 GB), toggle sound notifications, and customize your experience.
- 📦 **100% Client-Side & GitHub Pages Ready:** Runs directly in the user's browser with zero backend build steps or server maintenance.

---

## 🛠️ Architecture & How It Works

```
                     +---------------------------------------+
                     |            SharingPortal              |
                     |             (index.html)              |
                     +-------------------+-------------------+
                                         |
               +-------------------------+-------------------------+
               |                                                   |
       [ P2P Direct Mode ]                                 [ Deposit Mode ]
               |                                                   |
   WebRTC Peer-to-Peer Transfer                     Anonymous Temporary Cloud Upload
(Sender & Receiver online simultaneously)         (Receiver downloads anytime via Code)
```

1. **P2P Direct Transfer:** Establishes a direct browser connection between sender and receiver. Data flows directly without middle storage.
2. **Deposit Mode (Anonymous API):** Uploads files to anonymous storage providers (e.g., Pixeldrain API) and maps them to your redemption code.
3. **Queue Privacy:** A unique anonymous user token generated in `localStorage` ensures that your queue displays only files associated with your session.

---

## 🚀 Quick Start & Installation

### Option 1: Run Locally
1. Clone or download this repository:
   ```bash
   git clone https://github.com/your-username/SharingPortal.git
   ```
2. Navigate to the project directory and open `index.html` using a local web server (e.g., VS Code *Live Server* or Python's `http.server`):
   ```bash
   python -m http.server 8000
   ```
3. Open `http://localhost:8000` in your web browser.

> **Note:** Testing external network uploads/downloads locally via raw `file://` paths may be blocked by browser CORS security policies. Always use a local HTTP server or deploy to GitHub Pages.

---

### Option 2: Deploy to GitHub Pages (Free Hosting)

1. Create a public repository named `SharingPortal` on GitHub.
2. Upload the `index.html` file to the `main` branch.
3. Go to **Repository Settings** ➔ **Pages**.
4. Set **Source** to `Deploy from a branch` and select `main` / `/ (root)`.
5. Click **Save**. Your portal will be live at `https://your-username.github.io/SharingPortal/` within minutes!

---

## ⚙️ Configuration & Default Settings

| Option | Default Setting | Description |
| :--- | :--- | :--- |
| **Theme** | Dark (Slate & Sky Blue) | Customizable to Light Mode in Options |
| **Language** | Auto-Detect | Defaults to Italian / English based on browser settings |
| **Deposit Limit** | **1 GB** | Adjustable to 500 MB or 2 GB |
| **P2P Limit** | Unlimited | No maximum size for direct P2P transfers |
| **Sound Alerts** | Enabled 🔔 | Audio feedback on transfer completion |
| **Queue Cleanup** | Manual 🗑️ | Control your queue directly from the Queue tab |

---

## 📄 License

This project is open-source and available under the [MIT License](LICENSE). Feel free to fork, modify, and customize it for your needs!