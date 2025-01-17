# Sway builds for Ubuntu 21.04

Ubuntu 21.04 build system for sway and related tools.

Even though most of these tools (including sway and wlroots) are now available in Ubuntu, they move and evolve pretty quickly and I personally prefer to keep up to date with those.

This repository contains a Makefile based build system for all of these. We are NOT building deb packages (see my [old repository which did](https://github.com/luispabon/sway-ubuntu-deb-build) if you want to do so), but we're directly building from source and installing as root.

This means you should make sure you do not install any of the ubuntu provided packages, and indeed dependents (for instance other tools that depend on wlroots) should also be compiled here.

Apps provided (make sure you do not install these via Ubuntu's package repos):

  * Sway
  * wlroots
  * clipman
  * kanshi
  * mako
  * nwg-panel
  * swaylock-effects
  * waybar
  * wf-recorder
  * wofi
  * xdg-desktop-portal-wlr (for screen sharing)
  * wayfire / wf-config / wcm
  * wf-shell

Debs:

  * network-manager-gnome: supersedes Ubuntu hirsute's version, hides unmanaged interfaces (eg virtualbox, docker, etc)
  * wayland 1.19 (required for sway 1.6+)

# Prepare your system's environment

You must make sure that

```
LD_LIBRARY_PATH=/usr/local/lib/x86_64-linux-gnu/
```

is set on your environment prior to starting Sway. This is required so that any apps you compile here can find each other's library, as they're placed somewhere else than Ubuntu's default library path.

# Note: `sudo`

Some operations require root to complete. While building `sudo` will be run at some point to do so and your password will be asked.

# Note: `meson` and `ninja`

Make sure you uninstall `meson` and `ninja` if you've already installed them via Ubuntu's package manager. We need newer versions than what's available in 20.10 and we'll be installing the latest versions using `pip` instead.

# Dependencies

You need `make`. That's it really, everything else can be installed via the provided targets in the [Makefile](Makefile).

# Building stuff

First time, you should probably run

```
make yolo
```

This will clone all the app's git repos, install dev dependencies and tools required to build everything, then it proceeds to build each project in sequence.

Have a look at the [Makefile](Makefile) for all the different build targets, in case you want to build this or the other app. I for instance build `wlroots` and `sway` once a week, for which I have

```
make core
```

If you just want to update the apps (not wlroots and sway):

```
make apps
```

## Updating repositories before building

Simply pass `-e UPDATE=true` to `make`:

```
make mako -e UPDATE=true
```

## App versions

At the top of the [Makefile](Makefile) you'll see one variable per app that defines which version of that app to build that you can override via environment. By version, I mean either a git hash, or a branch, or a tag - we will simply be running `git checkout $APP_VERSION` before building that app.

For instance, if I wanted to build wlroots `0.11.0`, sway `1.5` and swaylock-effects `master`, while making sure we're on the absolute latest commits for each:

```
make core swaylock-build -e SWAY_VERSION=1.5 -e WLROOTS_VERSION=0.11.0 -e UPDATE=true
```

Note the lack of `SWAYLOCK_VERSION` up there - `master` is already the default.

# Uninstalling stuff

When installing the stuff we're compiling, `ninja` will be copying the relevant files wherever they need to be in the system, without creating a `deb` package. Therefore, `apt autoremove app` won't work.

So far all the apps in the repo except for clipman use `meson` and `ninja` for building. As long as you don't delete the `APP/build` repository you can uninstall from the system anything ninja installs:

```
cd APP
sudo ninja -C build uninstall
```

If you deleted the `build` folder on the app, simply build the app again before running the command above.

# wlroots dependencies

This goes without saying, but if you're updating `wlroots` make sure it's built first so that any of the other apps that link against it (like `sway`) have the right version to link against instead of linking against the version you're replacing.

# Screen sharing

Ubuntu 21.04 finally comes with all the plumbing to make it all work:
  * pipewire 0.3
  * xdg-desktop-portal-gtk with the correct build flags


## Limitations

xdg-desktop-portal-wlr does not support window sharing, [only entire outputs](https://github.com/emersion/xdg-desktop-portal-wlr/wiki/FAQ). No way around this. Apps won't show anything on the window list.

## How to install

```
make xdg-desktop-portal-wlr -e UPDATE=true
```

This will compile & install & make available the wlr portal to xdg-desktop-portal.

After that, make sure systemd has the following env var `XDG_CURRENT_DESKTOP=sway`. This won't work by merely setting that env var before you start sway. The best way is to create a file containing that at `~/.config/environment.d/xdg.conf`, [like so](https://github.com/luispabon/sway-dotfiles/blob/master/configs/environment.d/xdg.conf). Then reboot.

## Choosing an output to share

When choosing to share a screen from an app, xdpw won't give it a list of available windows or screens to the app to display and for you to choose from. Instead, you'll need to tell your app to share everything and after that the xdpw's output chooser will kick in.

By default it'll be `slurp` - your cursor will change to a crosshairs and you'll be able to click on a screen to share only that one.

The chooser is configurable, see docs here:
https://github.com/emersion/xdg-desktop-portal-wlr/blob/master/xdg-desktop-portal-wlr.5.scd#output-chooser

For instance, if you'd like to use wofi/dmenu, place the following on `~/config/xdg-desktop-portal-wlr/config`

```
[screencast]
chooser_type=dmenu
chooser_cmd=wofi --show=dmenu
```

The actual defaults (if you had no config file) are:

```
[screencast]
chooser_type=simple
chooser_cmd="slurp -f %o -o"
```

## Firefox

Should work out of the box on Firefox 84+ using the wayland backend.

When you start screensharing, on the dialog asking you what to share tell it to "Use operating system settings" when prompted. After that, the output chooser for xdpw will kick in, as explained on the previous section.

## Chromium

Ubuntu's Chromium snap currently does not seem to have webrtc pipewire support.

## Chrome

Open `chrome://flags` and flip `WebRTC PipeWire support` to `enabled`. Should work after that.

### Note
It looks like this option has disappeared and is not available anymore.

# Known issues
Nothing at the moment.
