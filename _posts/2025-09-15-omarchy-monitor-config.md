---
title: Updating Omarchy monitor configuration on connect/disconnect
date: 2025-09-15 17:29:00
categories: [Linux, Omarchy]
tags: [linux, arch-linux, omarchy, monitor, hyprland]
---

After installing [Omarchy](https://omarchy.org/) as my OS which comes with the tiling window manager [Hyprland](https://hypr.land/), I realized that having a multi-monitor setup compounds the learning curve when working with a completely new UI with new keybindings. To solve this, I wanted to configure my setup to automatically disable my internal [Framework 13](https://frame.work/) laptop display whenever an external display was connected (and re-enable it when it was disconnected).

## Configuring monitors in Omarchy

Hyprland offers a CLI-only approach to configuring monitors. It does make an attempt to automatically configure new monitors when connected with somewhat sensible defaults, but any further changes need to be made by editing the configuration located at `~/.config/hypr/monitors.conf`. This is what `monitors.conf` looks like after you install Omarchy:

```conf
# See https://wiki.hyprland.org/Configuring/Monitors/
# List current monitors and resolutions possible: hyprctl monitors
# Format: monitor = [port], resolution, position, scale
# You must relaunch Hyprland after changing any envs (use Super+Esc, then Relaunch)

# Optimized for retina-class 2x displays, like 13" 2.8K, 27" 5K, 32" 6K.
env = GDK_SCALE,2
monitor=,preferred,auto,auto

# Good compromise for 27" or 32" 4K monitors (but fractional!)
# env = GDK_SCALE,1.75
# monitor=,preferred,auto,1.666667

# Straight 1x setup for low-resolution displays like 1080p or 1440p
# env = GDK_SCALE,1
# monitor=,preferred,auto,1

# Example for Framework 13 w/ 6K XDR Apple display
# monitor = DP-4, 3840x3384@60, preferred, auto, 2
# monitor = eDP-1, 2880x1920@120, auto, 2
```

For a static workstation that always has the same setup, setting your configuration here may work fine. But I wanted to use different configurations depending on whether or not an external screen was connected.

To solve this, I used the wonderful [hyprland-monitor-attached](https://github.com/coffebar/hyprland-monitor-attached) Hyprland plugin that listens for `monitoradded` and `monitorremoved` events to execute bash script.

## Step-by-step solution

### 1. Install hyprland-monitor-attached

First, install the `hyprland-monitor-attached` package from the AUR repository:

```terminal
yay -S hyprland-monitor-attached
```

### 2. Create an executable bash script

Next, create the bash script that will execute when a monitor is added or removed. I chose to create mine at `~/.local/bin/monitor-config.sh`:

```bash
#!/bin/bash
LAPTOP_DISPLAY="eDP-1" # most likely eDP-1, but you can confirm with `hyprctl monitors`
USER="YOUR_USERNAME" # get username with `echo $USER`
UUID="YOUR_UID" # get UID with `echo $UID`

# Get the active Hyprland instance/signature. This is needed in case the script is executed as root
active_instance=""
for instance in /run/user/$UUID/hypr/*; do
  if [[ -d "$instance" ]]; then
    instance_name=$(basename "$instance")
    if sudo -u "$USER" HYPRLAND_INSTANCE_SIGNATURE="$instance_name" hyprctl activewindow &>/dev/null; then
      active_instance="$instance_name"
      break
    fi
  fi
done

# Exit if no active Hyprland instance is found
if [[ -z "$active_instance" ]]; then
  echo "No active Hyprland instance found"
  exit 1
fi

export HYPRLAND_INSTANCE_SIGNATURE="$active_instance"

# Get the external monitor count
monitor_count=$(sudo -u "$USER" HYPRLAND_INSTANCE_SIGNATURE="$active_instance" hyprctl monitors -j | jq -r ".[].name" | grep -v "$LAPTOP_DISPLAY" | wc -l)

# If any number of external monitors are connected, disable the internal laptop display; otherwise re-enable it
if [[ "$monitor_count" -gt 0 ]]; then
  sudo -u "$USER" HYPRLAND_INSTANCE_SIGNATURE="$active_instance" hyprctl keyword monitor "$LAPTOP_DISPLAY,disable"
else
  sudo -u "$USER" HYPRLAND_INSTANCE_SIGNATURE="$active_instance" hyprctl keyword monitor "$LAPTOP_DISPLAY,preferred,auto,2" # adjust scale accordingly
fi
```

Make sure you adjust this script for your particular use case:

* Replace `YOUR_USERNAME` and `YOUR_UID` with your actual user/UID (see inline comments explaining how to get these values for your system)
* Adjust line 33 to set the scale factor accordingly for your internal laptop's display. I set mine to `2` since I have a high resolution display for my internal laptop screen, but if you have a 1080p or similar lower resolution internal display, you may be better off setting this to `1`. In that case, line 33 would look like this:

```bash
sudo -u "$USER" HYPRLAND_INSTANCE_SIGNATURE="$active_instance" hyprctl keyword monitor "$LAPTOP_DISPLAY,preferred,auto,1" 
```
{: .nolineno }

Don't forget to make your bash script executable with `sudo chmod +x ~/.local/bin/monitor-config.sh`.

### 3. Update Hyprland configuration

Add the following lines to your Hyprland configuration at `~/.config/hypr/autostart.conf`:

```conf
exec-once = bash /home/YOUR_USERNAME/.local/bin/monitor-config.sh # configure monitors on launch
exec-once = hyprland-monitor-attached /home/YOUR_USERNAME/.local/bin/monitor-config.sh # configure monitors when monitors change
```

> Replace `YOUR_USERNAME` with your actual username.
{: .prompt-info }

### 4. Restart Hyprland

Hyprland should automatically restart when it detects a change to the configuration; otherwise, you can restart your session by selecting "Relaunch" from the Omarchy "System" menu.

After relaunching, your system should now be configured to change your monitor configuration automatically when you connect/disconnect your external monitor.

## Troubleshooting additional issues

* Verify you replaced all of the variables in your bash script with the correct files
* Make sure you made your bash script executable
* Confirm that the path to your bash script as set in your Hyprland `autostart.conf` is correct

## Conclusion

This should properly configure monitors on boot as well as when monitors are attached/removed. You can tweak this bash script for many other use cases as well, but hopefully this helps for anyone looking to have a similar setup.

**Questions or comments?** Sign in with GitHub to comment below!
