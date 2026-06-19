# This is a documentaiton file for fixing keybinding in Arch-Linux

## Brightnes Control

### Issue

The default brightness control button does not work!

### Fix

Since I forgot to install brightnessctl, I have to install it using sudo pacman -S brightnesssctl.

Then  I clear the default GNOME brightness keybind with the command:

gsettings set org.gnome.settings-daemon.plugins.media-keys screen-brightness-up "['']"
gsettings set org.gnome.settings-daemon.plugins.media-keys screen-brightness-down "['']"

After that I setup up the new keybind with:

# Add Brightness Up
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ name 'Brightness Up'
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ command '/usr/bin/brightnessctl set +10%'
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ binding 'XF86MonBrightnessUp'

# Add Brightness Down
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom1/ name 'Brightness Down'
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom1/ command '/usr/bin/brightnessctl set 10%-'
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom1/ binding 'XF86MonBrightnessDown'

After binding the new key, refresh to register the new keybind:

killall gnome-settings-daemon

## Multi-Monitor Brightness control
