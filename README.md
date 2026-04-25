Your **README.md** file is ready. I have structured this to be a high-quality GitHub repository, including SEO headers, clear installation steps, and technical explanations.

```markdown
# Fedora Keyboard Backlight Fix (Wayland/GNOME 49)

![Fedora](https://img.shields.io/badge/Fedora-43-blue?logo=fedora) ![GNOME](https://img.shields.io/badge/GNOME-49-green?logo=gnome) ![Wayland](https://img.shields.io/badge/Display-Wayland-orange)

A lightweight, persistent solution for enabling keyboard backlights controlled by the Scroll Lock LED on Fedora Workstation. This fix solves the common issue where the backlight turns off automatically when **Caps Lock** or other modifiers are pressed under Wayland.

---

## 🔍 The Problem
On modern Linux distributions using **Wayland** (like Fedora 43), the traditional `xset led on` command no longer works. Furthermore, many keyboards sync all LEDs at a firmware level. When you toggle Caps Lock, the system sends a refresh signal that resets the Scroll Lock LED (and your backlight) to the "off" state.

## 🚀 SEO Keywords
`Fedora 43 keyboard backlight fix`, `GNOME 49 scroll lock LED`, `Wayland keyboard light won't stay on`, `Linux keyboard backlight caps lock fix`, `enable keyboard light terminal command`.

---

## 🛠 Installation Roadmap

This process creates a high-priority background service that monitors your keyboard and forces the backlight to stay on with near-zero latency.

### 1. Create the Monitoring Script
This script checks the LED status 10 times per second and forces it back to "1" (on) if it ever drops to "0".

```bash
sudo nano /usr/local/bin/kb-light-lock
```

**Paste the following code:**
```bash
#!/bin/bash
# Wait until the keyboard hardware is initialized
until ls /sys/class/leds/*::scrolllock/brightness >/dev/null 2>&1; do 
    sleep 1
done

# Continuous monitoring loop
while true; do
    if [ "$(cat /sys/class/leds/*::scrolllock/brightness)" -eq 0 ]; then
        echo 1 > /sys/class/leds/*::scrolllock/brightness
    fi
    sleep 0.1
done
```

### 2. Set Executive Permissions
```bash
sudo chmod +x /usr/local/bin/kb-light-lock
```

### 3. Create the Systemd Service
This ensures the script starts automatically when you reach the login screen and survives every reboot.

```bash
sudo nano /etc/systemd/system/kb-light.service
```

**Paste the following configuration:**
```ini
[Unit]
Description=Force Keyboard Backlight Always On
After=display-manager.service
StartLimitIntervalSec=0

[Service]
Type=simple
ExecStart=/usr/local/bin/kb-light-lock
# High priority ensures the light turns back on instantly
Nice=-20
Restart=always
RestartSec=1

[Install]
WantedBy=graphical.target
```

### 4. Enable and Activate
Run these commands to tell Fedora to load the new service immediately and on every boot.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now kb-light.service
```

---

## 📋 Features
- **Persistence:** Works automatically after reboot and logout.
- **Fast Recovery:** Uses `Nice=-20` priority to minimize the "flicker" when pressing Caps Lock.
- **Efficient:** Uses minimal CPU resources by only writing to the hardware when a state change is detected.
- **Wayland Ready:** Bypasses X11 limitations by talking directly to the Linux Kernel LED class.

## ⚠️ Troubleshooting
If the light does not come on after a reboot:
1. **Check Service Status:** `systemctl status kb-light.service`
2. **SELinux Permissions:** If Fedora blocks the script, run:
   ```bash
   sudo restorecon -v /etc/systemd/system/kb-light.service
   ```
3. **Manual Override:** You can manually check if your hardware supports this path with:
   ```bash
   ls /sys/class/leds/*::scrolllock/
   ```

---

## 🤝 Contributing
If you have a more efficient way to lock LED registers on Fedora 43, feel free to open a Pull Request!

## 📜 License
MIT License - Free to use and modify.
```
