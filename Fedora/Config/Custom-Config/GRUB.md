# Custome configuration for GRUB

Open the file

bash: sudo nano /etc/default/grub

## Change the Timeout

Look for the line that say: GRUB_TIMEOUT=5 (the last number does not matter) 
The Timeout measure in Second, change it to the preferred value, Then save and exit

### Updating GRUB

Run: 

bash: sudo grub2-mkconfig -o /boot/grub2/grub.cfg

After you run that command, reboot your computer!