# plasma-hud

Provides a way to run menubar commands through
[`rofi`](https://github.com/davatorium/rofi), much like the Unity 7
Heads-Up Display (HUD). `plasma-hud` was forked from [`mate-hud`](https://github.com/ubuntu-mate/mate-hud) which was based on
[`i3-hud-menu`](https://github.com/RafaelBocquet/i3-hud-menu). There's also a [`gnome-hud`](https://github.com/hardpixel/gnome-hud). This is a fork by uszie that works on Wayland. Update to Plasma 6 by Felagund.

If you are interested in Unity's other feature, locally integrated menus in window titlebars, then that currently does not work on Wayland, but should be implemented soon: https://invent.kde.org/plasma/breeze/-/merge_requests/529 If you are still on X, you may want to check out the [Material KWin decoration](https://github.com/Zren/material-decoration) which has that feature.

![](https://i.imgur.com/M3YUONc.png)
![](https://i.imgur.com/sE0i8IE.png)

## What is a HUD and why should I care?

A Heads-Up Display (HUD) allows you to search through an application's
appmenu. So if you're trying to find that single filter in Gimp but
can't remember which filter category it fits into or if you can't
recall if preferences sits under File, Edit or Tools on your favourite
browser, you can just search for it rather than hunting through the
menus. Note that on Wayland, this does not work with GTK applications and most browsers.

## Install via GitHub

### Dependencies

#### apt (Kubuntu / KDE Neon)

```
sudo apt install rofi python3 python3-dbus python3-setproctitle python3-xlib
sudo apt install appmenu-qt5 # Qt5
```
#### pacman (Arch)

```
pacman -S rofi python python-dbus python-setproctitle python-xlib python-gobject gobject-introspection
```

#### DNF (Fedora)
```
sudo dnf install rofi python3 python3-dbus python3-setproctitle python3-xlib appmenu-qt5 appmenu-qt6
```

### Manual Install
```
# Download
git clone https://github.com/felagund/plasma-hud
cd plasma-hud
# Instal PLasma HUD
sudo mkdir -p /usr/lib/plasma-hud
sudo cp usr/lib/plasma-hud/plasma-hud /usr/lib/plasma-hud/plasma-hud
sudo mkdir -p /usr/share/plasma-hud
sudo cp usr/share/plasma-hud/breeze.rasi /usr/share/plasma-hud/breeze.rasi
sudo cp usr/share/plasma-hud/kwin-plasma-hud-helper.js /usr/share/plasma-hud/kwin-plasma-hud-helper.js
# Install the helper widget
mkdir ~/.local/share/plasma/
cp -pr usr/share/plasma/plasmoids/com.github.zren.PlasmaHUD/ ~/.local/share/plasma/plasmoids
# Set up shortcuts
cp net.local.launch_plasma_hud.sh.desktop ~/.local/share/applications/
kwriteconfig6 --file ~/.config/kglobalshortcutsrc --group services --group net.local.launch_plasma_hud.sh.desktop --key _launch Alt
# Enable through systemd
cp plasma-hud.service ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable plasma-hud.service
systemctl --user start plasma-hud.service
```
Now add Plasma HUD widget to your panel. It will be invisible but currently is the only known way to acces manues on Wayland.

Now either log off and on or restart kwin and plasma shell (after restarting kwin, invoke krunner with ALT+F2 to run commands) or run:
```
kwin --replace
plasmashell --replace
```
Plasma HUD should not be bound to ALT and autostart via systemd (there is also /etc/zdg/autostart provided, byt systemd makes sure Plasma HUD gets restarted should it crash or something). There is also a `setup.py` but that only installs the program itself, not the helper files.

You can also for testing run Plasma HUD with 
```
/usr/lib/plasma-hud/plasma-hud
```

And invoke it with
```
qdbus com.github.zren.PlasmaHUD /PlasmaHUD com.github.zren.PlasmaHUD.toggleHUD
```

### Manual Uninstall
```
systemctl --user disable plasma-hud.service
systemctl --user stop plasma-hud.service
kwriteconfig6 --file ~/.config/kglobalshortcutsrc --group services --group net.local.launch_plasma_hud.sh.desktop --key _launch --delete
sudo rm /usr/lib/plasma-hud/plasma-hud /usr/share/plasma-hud/breeze.rasi /usr/share/plasma-hud/kwin-plasma-hud-helper.js
rm ~/.local/share/plasma/plasmoids/com.github.zren.PlasmaHUD ~/.local/share/applications/net.local.launch_plasma_hud.sh.desktop ~/.config/systemd/user/plasma-hud.service
```
## Settings

If you manally create `~/.config/plasmahudrc` you can change any of the following settings.

```
[General]
Matching=fuzzy
Sort=true
Lines=20

[Icons]
Enabled=true
Theme=breeze-dark

[Shortcuts]
Enabled=true

[Style]
Font=Sans 10
Title=::
Theme=breeze
# place any normal rofi theme in the folder /usr/share/plasma-hud
# The Theme argument must be the same as the theme filename without the .rasi extension.
# At runtime plasma-hud will inject the colors of the current desktop theme into the rasi theme.
# Possible injection values are `<BACKGROUND>`, `<FOREGROUND>`, `<BORDERS>`,
# `<VIEW_BACKGROUND>`, `<DECORATION_FOCUS>` and `<DECORATION_FOCUS_INACTIVE>`.

[Colors]
Background=#111111
Foreground=#eeeff0
SelectedBackground=#062d25
SelectedForeground=#1abc9c
Borders=#000000
ShortcutForeground=#888888
```

* `[General] Matching=fuzzy` can be either `fuzzy` matching or `normal` matching where it matches a keyword exactly. See `man rofi | grep "\-matching" -A20` for more info.

* `[Style] Title=::` will change the `HUD:` prompt text to `:::` which is roughly the width of an icon.

![](https://i.imgur.com/OrDieG2.png)
