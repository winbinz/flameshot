# Sway and wlroots support
Flameshot currently supports Sway and other wlroots based Wayland compositors through [xdg-desktop-portal-wlr](https://github.com/emersion/xdg-desktop-portal-wlr). However, due to the way dbus works, there may be some extra steps required for the integration to work properly.

## Basic steps
The following packages need to be installed: `xdg-desktop-portal xdg-desktop-portal-wlr grim`. Please ensure your distro packages these, or install them manually.

Ensure that environment variables are set properly. If your distro does not set them automatically, use a launch script to export `XDG_CURRENT_DESKTOP=sway` or `XDG_CURRENT_DESKTOP=river` **before** Sway or River is launched.
```sh
#!/bin/bash
export SDL_VIDEODRIVER=wayland
export _JAVA_AWT_WM_NONREPARENTING=1
export QT_QPA_PLATFORM=wayland
export XDG_CURRENT_DESKTOP=sway
export XDG_SESSION_DESKTOP=sway
exec sway
```

or


```sh
#!/bin/bash
export SDL_VIDEODRIVER=wayland
export _JAVA_AWT_WM_NONREPARENTING=1
export QT_QPA_PLATFORM=wayland
export XDG_CURRENT_DESKTOP=river
export XDG_SESSION_DESKTOP=river
exec river
```

You will also need to ensure that systemd/dbus is aware of these environment variables; this should be done **in your sway or river config** so that the DISPLAY and WAYLAND_DISPLAY variables are defined.

(taken from [Sway wiki](https://github.com/swaywm/sway/wiki#gtk-applications-take-20-seconds-to-start)):
```sh
exec systemctl --user import-environment DISPLAY WAYLAND_DISPLAY SWAYSOCK
exec hash dbus-update-activation-environment 2>/dev/null && \
     dbus-update-activation-environment --systemd DISPLAY WAYLAND_DISPLAY SWAYSOCK
```

To ensure that Flameshot is correctly positioned on multiple outputs (monitors) add this rule to your Sway config:
```
for_window [app_id="flameshot"] border pixel 0, floating enable, fullscreen disable, move absolute position 0 0
```

and add the following on your River config:

```
riverctl rule-add -app-id "flameshot" float
```

Otherwise, flameshot will not take all of the screen and tiles its window instead like a normal application. Note however, that some clipboard stuff is broken so it might be good to save your screenshot as a file while having it copied to a clipboard in case if clipboard does some weird stuff like not pasting the overall screenshot.


Starting from 0.17.0 xdg-desktop-portal requires a configuration file (e.g. in ~/.config/xdg-desktop-portal/sway-portals.conf):

(take from [issues#3363](https://github.com/flameshot-org/flameshot/issues/3363))
```sh
[preferred]
# use xdg-desktop-portal-gtk for every portal interface
default=gtk
# except for the xdg-desktop-portal-wlr supplied interfaces
org.freedesktop.impl.portal.Screencast=wlr
org.freedesktop.impl.portal.Screenshot=wlr
```

## Troubleshooting

Q) Flameshot doesn't take a screenshot, it just hangs!

A) Please ensure that the packages are installed, and that the variables are exported.
This is usually caused by Flameshot receiving no response from the desktop portal. This can be verified by running `dbus-monitor --session sender=org.freedesktop.portal.Desktop destination=org.freedesktop.portal.Desktop`.

Q) Flameshot takes one screenshot, then won't take anymore!

A) There is a bug in xdg-desktop-portal-wlr and Flameshot causing calls with the same token to fail. If you see a sdbus vtable error in the xdpw logs, either used the [patched version](https://github.com/nullobsi/xdg-desktop-portal-wlr/tree/improve-screenshot) or update Flameshot to the latest master.

# River wlroots support

Like mentioned above, Flameshot now works on wlroots based Wayland compositors, however, there is a weird problem with river and that is when setting `XDG_CURRENT_DESKTOP=river`, Flameshot won't work. The fix is you need to trick Flameshot that you are on `sway`. Hence, you need to run river like so:

```sh
XDG_CURRENT_DESKTOP=sway dbus-run-session river
```

and add the following on your config such as in `$HOME/.config/river/init`

```
riverctl float-filter-add "flameshot"
```

Otherwise, Flameshot will not take all of the screen and tiles its window instead like a normal application.

#### For more information, please refer to https://github.com/emersion/xdg-desktop-portal-wlr/wiki/%22It-doesn't-work%22-Troubleshooting-Checklist

