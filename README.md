# Jio STB ADB Setup Guide (Windows 11)

A beginner-friendly, clutter-free walkthrough to enable **ADB debugging** on supported Jio Set-Top Boxes using a Windows 11 PC and a local Wi-Fi router.

> [!NOTE]
> **Tested Hardware & Firmware:**
> Verified working on **JMSC200Av2** and **JMSB200A** across firmware builds `22.1.09`, `24.1.39`, and `25.1.9`. It should work on older firmware versions as well.

> [!CAUTION]
> **Incompatible Hardware:**
> This exploit **will not work on JHS series** STB models.

> [!TIP]
> **Don't Panic If Something Goes Wrong:**
>
> * **Box acting up?** Simply factory reset your Jio STB to restore it back to default settings, then start over.
>
> * **Wi-Fi not connecting properly?** Go into network settings, choose **Forget Network**, re-enter your password, and reconnect. Everything will work fine.

> [!IMPORTANT]
> **Package Installer Limitation (Read This!):**
> I tried everything possible, but could **not** enable the default Android package installer. Because of this, you cannot simply click an APK on the box to install it. **Every time you want to install apps, you must install them via ADB** from your PC using:
>
> ```bash
> adb install "filename.apk"
> ```

> ❤️ **A quick note from Vishal Beniwal:**
>
> If this guide helped you and you'd like to support me, I don't want any money—just keep me in your blessings for future success!

## ✨ What Happens After Running the Script?

Once you execute Stage 1, the script performs several handy quality-of-life modifications on your Jio STB:

* 🔓 **ADB Enabled:** Network ADB is activated so you can manage and sideload apps.
* 🚀 **Monet Launcher Activated:** Replaces the cluttered default view with a clean, customizable launcher. Stock launcher is suspended to prevent boot ad reversion.
* ⚡ **Shevery (Shizuku-Compatible API) Installed:** Standalone Shizuku-compatible manager allowing supported apps to access privileged APIs over ADB.
* 🛑 **Automatic Firmware Updates Blocked:** OTA auto-updates are disabled so your setup doesn't get overwritten.
* 🚫 **Startup Boot Ads Disabled:** Annoying commercial promo and intro ads that display when powering on the STB are suppressed.

## ⚡ Quick Navigation

* [Prerequisites](#-prerequisites)
* [Repository Setup](#-repository-setup)
* [Step-by-Step Guide](#-step-by-step-guide)
  * [1. Network Pairing](#step-1-network-pairing)
  * [2. Get Host IP](#step-2-get-host-ip)
  * [3. Configure STB DNS via USB Bug](#step-3-configure-stb-dns-via-usb-bug)
  * [4. Isolate Network & Disable Firewall](#step-4-isolate-network--temporarily-disable-windows-firewall)
  * [5. Run Docker](#step-5-run-docker)
  * [6. Setup Technitium DNS](#step-6-setup-technitium-dns)
  * [7. Deploy TLS Containers](#step-7-deploy-tls-containers)
  * [8. Launch Proxy Listener](#step-8-launch-proxy-listener)
  * [9. Initialize Git Bash](#step-9-initialize-git-bash)
  * [10. Execute Stage 1](#step-10-execute-stage-1)
  * [11. Verify Connection](#step-11-verify-connection)
  * [12. Teardown & Reset](#step-12-teardown--reset)
* [Installing Apps via ADB](#-how-to-install-apps-via-adb)
* [Important Disclaimers](#-important-disclaimers)
* [Credits & Acknowledgments](#-credits--acknowledgments)

## 🛠 Prerequisites

Install these utilities on your Windows 11 system before starting:

| Tool | Purpose | Download | 
| ----- | ----- | ----- | 
| **Docker Desktop** | Hosts ACS backend services | [Official Download](https://docs.docker.com/desktop/setup/install/windows-install/) | 
| **Python 3** | Runs payload handlers (`Add to PATH` checked) | [python.org](https://www.python.org/downloads/) | 
| **Technitium DNS** | Resolves local spoofed domains | [technitium.com](https://technitium.com/dns/) | 
| **Git for Windows** | Bash emulation environment | [gitforwindows.org](https://gitforwindows.org/) | 
| **Platform-Tools** | Official ADB binaries | [Android Developer Portal](https://developer.android.com/tools/releases/platform-tools) | 

## 📦 Repository Setup

1. Grab the repository ZIP from Here
2. Extract the archive into your `Downloads` directory.

## 🚀 Step-by-Step Guide

### Step 1: Network Pairing

* Connect your Windows 11 PC to your primary home Wi-Fi.
* Connect your Jio STB to the same router (Ethernet or standard Wi-Fi).
* **Avoid Guest Wi-Fi** to prevent device isolation issues.

### Step 2: Get Host IP

Open Command Prompt (`Win + R` $\rightarrow$ `cmd`) and execute:

```cmd
ipconfig
```

Locate **Wireless LAN adapter Wi-Fi** and copy your **IPv4 Address** (e.g., `192.168.1.20`).

### Step 3: Configure STB DNS via USB Bug

> A blank USB pendrive is required strictly to access the hidden stock Android settings menu.

1. Plug any USB thumb drive into the Jio STB.
2. Boot the STB $\rightarrow$ Launch **My Music** $\rightarrow$ **Settings** $\rightarrow$ Select **Eject USB**.
3. Tap **Internal Storage** $\rightarrow$ Open **Apps** $\rightarrow$ Select the stock **Settings** app (gear icon).
4. Navigate to **Network** $\rightarrow$ Select your active Wi-Fi connection $\rightarrow$ Open **Advanced IP Settings**.
5. Change IP Settings from **DHCP** to **Static**.
6. Keep IP Address, Gateway, Subnet, and DNS 2 intact.
7. Set **DNS 1** to your Windows 11 IPv4 Address from Step 2.
8. Save and ensure the STB reconnects.

> [!WARNING]
> If the Gateway or IP fields auto-clear upon selecting Static, switch back to DHCP, note the default values, switch to Static, and re-enter them manually before saving.

### Step 4: Isolate Network & Temporarily Disable Windows Firewall

1. **Unplug Internet Cable:**
   Unplug the **WAN / incoming fiber cable** from your router.
   * Keep the router powered on.
   * Both devices must communicate over the local subnet without active internet access.

2. **Temporarily Disable Windows Defender Firewall:**
   > [!IMPORTANT]
   > Windows Firewall frequently blocks incoming traffic from the STB on port **53** (DNS) and port **8443/9443** (TLS proxy), causing the exploit to silently fail. You must temporarily disable it:
   >
   > 1. Press `Win + R`, type `firewall.cpl`, and press **Enter**.
   > 2. Click **"Turn Windows Defender Firewall on or off"** on the left menu.
   > 3. Select **"Turn off Windows Defender Firewall (not recommended)"** for both *Private* and *Public* network settings.
   > 4. Click **OK**. *(You will turn it back on in Step 12 once finished.)*

### Step 5: Run Docker

Launch **Docker Desktop** from the Start Menu and wait until the engine status switches to running.

### Step 6: Setup Technitium DNS

1. Open the Technitium console at [`http://localhost:5380`](http://localhost:5380).
2. Go to **Zones** $\rightarrow$ Click **Add Zone**:
   * **Zone Name:** `jio.com`
   * **Zone Type:** `Primary Zone`
3. Click into `jio.com` $\rightarrow$ Click **Add Record**:
   * **Name:** `acs.oss`
   * **Type:** `A`
   * **IP Address:** *Your Windows 11 IPv4 Address*
4. Save the record and ensure the zone is **Enabled**.

### Step 7: Deploy TLS Containers

Open File Explorer, go to `sideload-jiostb/tls`, type `cmd` in the address bar, and press Enter:

```cmd
docker-compose up -d
```

### Step 8: Launch Proxy Listener

In that same terminal window:

```cmd
python -m pip install pycryptodome
python pass_front.py
```

*(Leave this window open. Note: `pass_front.py` includes an automatic precomputed fallback so it continues to work reliably even if `pycryptodome` is absent.)*

### Step 9: Initialize Git Bash & Health Check

1. Open the root folder `sideload-jiostb`.
2. Right-click blank space $\rightarrow$ **Show more options** $\rightarrow$ **Open Git Bash here**.
3. *(Optional but highly recommended)* Verify the PC-side chain:
   ```bash
   python doctor.py --quick
   python doctor.py --simulate
   ```
   `--simulate` sends a synthetic Inform through the TLS front and GenieACS, proving your PC setup is 100% operational before you touch the TV!

### Step 10: Execute Stage 1

Ensure Docker, Technitium, and `pass_front.py` are active, then run:

```bash
./stage1.sh --list-devices
```

Select your STB when prompted. To run only the required payload without stripping extras, you can alternatively use:

```bash
./stage1.sh --tasks-only
```

> [!IMPORTANT]
> **Cold Power-Cycle Is Mandatory (Unplug Wall Power):**
> The Jio STB only contacts the ACS server during **boot**. Its periodic background check-in takes hours, and remote wake is impossible because `NATDetected=True` closes incoming port 7547 on the STB.
>
> 1. Unplug the STB power adapter from the wall (do not just put it in standby with the remote).
> 2. Wait 5 seconds $\rightarrow$ Plug it back in.
> 3. Allow it to boot up completely.

### Step 11: Verify ADB Connection

> [!NOTE]
> **"Allow USB Debugging?" Dialog on TV:**
> No prompt will appear on your TV until the STB boots and applies the CWMP task. Once connected, your TV screen will display an RSA key prompt.
> Check **"Always allow from this computer"** and select **OK**.
>
> If running `adb connect` initially shows `failed to authenticate` or `unauthorized`, that is completely normal before you click OK on the TV. Simply run `adb connect <STB_IP>:5555` again after approving.

In Git Bash, confirm your connection:

```bash
adb devices
```

Your device should now be listed with the status **`device`**.

### Step 12: Teardown & Reset

Once ADB is confirmed and your setup is complete:

1. Stop `pass_front.py` by pressing `Ctrl + C` in its window (or run `./stage1.sh --stop`).
2. In Git Bash, terminate Docker containers:
   ```bash
   docker compose down
   ```
3. Reconnect your router's incoming WAN/Internet cable.
4. **Turn Windows Defender Firewall back on:** Press `Win + R` $\rightarrow$ `firewall.cpl` $\rightarrow$ Turn Windows Defender Firewall back on.

### 🧰 Advanced CLI Options for `stage1.sh`

| Option | Purpose |
|---|---|
| `./stage1.sh --doctor` | Run full diagnostic checklist of ports, NBI, and DNS |
| `./stage1.sh --simulate` | Test synthetic CWMP session through TLS front & GenieACS |
| `./stage1.sh --watch` | Real-time watcher polling for STB Inform and port 5555 |
| `./stage1.sh --tasks` | Inspect all pending TR-069 tasks queued for your STB |
| `./stage1.sh --clear-tasks` | Clear duplicate or stale tasks from GenieACS queue |
| `./stage1.sh --stop` | Stop background `pass_front.py` process |
| `./stage1.sh --yes-disable-tr069` | Explicitly permit disabling `com.jio.tr069client` in Stage 6 |

## 📲 How to Install Apps via ADB

Since the built-in package installer cannot be re-enabled, install any Android TV APK using your terminal:

1. Connect your PC to the STB over ADB (if not already connected):
   ```bash
   adb connect <STB_IP_ADDRESS>:5555
   ```
2. Sideload your preferred application:
   ```bash
   adb install "C:\path\to\your_app.apk"
   ```
3. The app will immediately show up inside **Monet Launcher**.

## ⚠️ Important Disclaimers

* **Scope:** Stage 1 performs modifications such as enabling ADB, queueing downgrade flags, installing Monet Launcher and Shizuku, disabling TR-069 settings, and disabling startup ads.
* **Root/Bootloader:** This guide deliberately excludes `unlock.sh` and `pull_boot.sh`. Do not run Stage 2 utilities unless you know what you are doing.
* **Liability:** Use only on hardware you own or have explicit authorization to test.

## 🌟 Credits & Acknowledgments

* **Original Vulnerability Research & Tooling:**
  Massive respect to **Bihaan Sen** for the reverse-engineering and original [Housekeeping-PoC](https://github.com/JFC-Group/Housekeeping-PoC/) repository. Please star and support his project!

* **Documentation & Windows 11 Streamlining:**
  Maintained by **Vishal Beniwal**.  
  *No donations needed—just wish me luck and keep me in your blessings!*
