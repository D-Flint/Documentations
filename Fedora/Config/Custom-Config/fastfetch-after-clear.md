# Automatically execute the fastfetch command after the clear command

The file path is ~/.zshrc

## Adding the fastfetch autoexecution

### 1st  step:

Add this line to the file under the "# Display Pokemon-colorscript"

---
pokemon-colorscripts --no-title -s -r | fastfetch -c $HOME/.config/fastfetch/config-pokemon.jsonc ...
---

### 2nd step:

After that find the line "# Override the type 'clear' command", and add

---
alias clear="clear && fastfetch"
---

### 3rd step:

Add this Custom Function

---

### Custom function to clear and run fastfetch
clear-and-fetch() {
    clear
    fastfetch
    zle rgb-refresh 2>/dev/null || zle redisplay
}

## Removing it

To remove it, just comment out the line of text in the 1st step,
and remove " && fastfetch" from the the text in the 2nd step, and finally in the custom function in the 3rd step, just remove fastfect.

### Save and Apply

Save the file and exit, then apply the change with the commanf:

---
bash: source ~/.zshrc
---


