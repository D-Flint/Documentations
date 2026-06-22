# This is the documentation on brightness control for Fedora

## Installed brrightnessctl

Make sure that you have brightnessctl installed.
You can verify the installation by running

---
bash: brightnessctl --version
---

If you do not have it installed, you can installed it by running

--- 
bash: sudo dnf installed bightnessctl
---

To change the brightness of a monitor via a command line or bind it to a key, you must first know the name of the display
Run this command to check for the name, it should be the first line after the word "Monitor"

---
bash: hyprctl monitor
---

It should loop something like:
"Monitor eDP-1 (ID 0):"

After that run this command to check for what drivers are controlling your backlight

---
ls /sys/class/backlight/
---

It should return something like "amdgpu_bl1 nvidia_0"

After you have the driver list, try it out! Whichever work, that is the driver that control your backlight.
run the command:

---
bash: brightnessctl -d amdgpu_bl1 set +100%
---

run brightnessctl --help for more information