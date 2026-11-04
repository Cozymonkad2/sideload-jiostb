# Jio STB ADB Setup Guide (Windows 11 & 10)

A beginner-friendly, clutter-free walkthrough to enable **network ADB debugging** and sideload custom launchers/apps on supported Jio Set-Top Boxes using a Windows PC and your home Wi-Fi router.

> [!TIP]
> **Need Hands-On Help or Want Me to Do It For You?**
> * **100% Free Self-Help:** This guide and any remote assistance or troubleshooting you need are completely **free of cost**.
> * **Mail-in Service:** If you find this guide too technical or complicated, you can ship your box to me and I will set it up for you! You can pay me whatever your heart desires + return shipping.
> * **Reddit:** Reach out directly at [**u/ilikechocopies**](https://www.reddit.com/user/ilikechocopies/).
> * **Instagram:** [**@soloxsurfer**](https://instagram.com/soloxsurfer).

> [!NOTE]
> **Supported Hardware & Firmware:**
> * **JMSC200Av2** and **JMSB200A** across firmware builds `22.1.09`, `24.1.39`, `25.1.9`, and older.
> * **JHS series (JHSC200 v1 / JioOS 2.0)** on firmware build `stb-jhcs200-11.3.46`.
>
> *Both box families use the exact same `./stage1.sh` workflow. The only minor difference is in **Step 8** (launching the proxy listener).*

> [!TIP]
> **Don't Panic If Something Goes Wrong:**
> * **Box acting up?** Simply factory reset your Jio STB to restore it back to default settings, then start over.
> * **Wi-Fi not connecting properly?** Go into network settings, choose **Forget Network**, re-enter your password, and reconnect. Everything will work fine.

> [!IMPORTANT]
> **Package Installer Limitation (Read This!):**
> Jio disabled the stock Android package installer UI on the box. Because of this, you cannot simply click an APK file directly on the TV to install it. **Every time you want to install apps, you must install them via ADB** from your PC using:
>
> ```bash
> adb install "filename.apk"
> ```

> ❤️ **A quick note from Vishal Beniwal / Instagram ([@soloxsurfer](https://instagram.com/soloxsurfer)) / Reddit ([u/ilikechocopies](https://www.reddit.com/user/ilikechocopies/)):**
> The guide and all troubleshooting assistance are completely free! If you find the process too complicated and prefer me to do it for you, you can ship the box over—pay whatever your heart desires + shipping. Otherwise, just keep me in your blessings for future success!

---

## ✨ What Happens After Running the Script?

Once you execute Stage 1, the script performs several handy quality-of-life modifications on your Jio STB:

* 🔓 **ADB Enabled:** Network ADB is activated (expires in 2037) so you can manage and sideload apps.
* 🚀 **Monet Launcher Activated:** Replaces the cluttered stock interface with a clean, customizable TV launcher. The stock launcher is suspended to prevent boot ad reversion.
* ⚡ **Shevery (Shizuku-Compatible API) Installed:** Standalone manager allowing supported apps to access privileged system APIs over ADB.
* 🛑 **Automatic Firmware Updates Blocked:** OTA auto-updates are disabled (`com.jio.*` on JMSC/JMSB and `com.iwedia.*` on JHS) so your setup doesn't get overwritten.
* 🚫 **Startup Boot Ads Disabled:** Annoying commercial promo and intro ads that display when powering on the STB are suppressed.

---

## ⚡ Quick Navigation

* [Prerequisites](#-prerequisites)
* [Step-by-Step Guide](#-step-by-step-guide)
  * [1. Network Pairing](#step-1-network-pairing)
  * [2. Get Host IP](#step-2-get-host-ip)
  * [3. Configure STB DNS via USB Bug](#step-3-configure-stb-dns-via-usb-bug)
  * [4. Isolate Network & Temporarily Disable Firewall](#step-4-isolate-network--temporarily-disable-windows-firewall)
  * [5. Run Docker](#step-5-run-docker)
  * [6. Setup Technitium DNS](#step-6-setup-technitium-dns)
  * [7. Deploy TLS Containers](#step-7-deploy-tls-containers)
  * [8. Launch Proxy Listener (Model Specific)](#step-8-launch-proxy-listener)
  * [9. Initialize Git Bash & Health Check](#step-9-initialize-git-bash--health-check)
  * [10. Execute Stage 1](#step-10-execute-stage-1)
  * [11. Verify ADB Connection](#step-11-verify-adb-connection)
  * [12. Teardown & Reset](#step-12-teardown--reset)
* [How to Install Apps via ADB](#-how-to-install-apps-via-adb)
* [Advanced CLI & JHS Options](#-advanced-cli-options)
* [Get Hands-On Help / Mail-In Setup](#-get-hands-on-help--mail-in-setup)
* [Credits & Acknowledgments](#-credits--acknowledgments)

---

## 🛠 Prerequisites

Install these utilities on your Windows system before starting:

| Tool | Purpose | Download | 
| ----- | ----- | ----- | 
| **Docker Desktop** | Hosts local GenieACS backend services | [Official Download](https://docs.docker.com/desktop/setup/install/windows-install/) | 
| **Python 3** | Runs proxy and diagnostic scripts (`Add to PATH` checked) | [python.org](https://www.python.org/downloads/) | 
| **Technitium DNS** | Overrides local domain resolution | [technitium.com](https://technitium.com/dns/) | 
| **Git for Windows** | Bash terminal environment | [gitforwindows.org](https://gitforwindows.org/) | 

*(Note: Android platform-tools ADB binaries are already bundled inside this folder).*

---

## 🚀 Step-by-Step Guide

### Step 1: Network Pairing

* Connect your Windows PC to your primary home Wi-Fi.
* Connect your Jio STB to the same router (via standard Wi-Fi or Ethernet).
* **Do not use Guest Wi-Fi** (guest networks isolate connected devices from talking to each other).

### Step 2: Get Host IP

Open Command Prompt (`Win + R` $\rightarrow$ `cmd`) and type:

```cmd
ipconfig
