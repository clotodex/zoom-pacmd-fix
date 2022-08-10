# Fix for hard zoom dependency on pacmd

Zoom requires `pacmd` to be installed on linux systems.  
This is a quick-fix to make zoom work under pipewire until zoom supports the correct implementation.

## Installation

Either execute `./install.sh` as *root*, or manually copy the `pacmd` file into `/usr/bin/` and allow it to be executable.

## Why is this necessary

Pipewire is becoming more mature and being a great choice for wayland users.  
Pipewire can serve the pulseaudio API as soundserver and also implements the common tool `pactl`.
Pacmd however is seen as obsolete and will (most likely) [never be implemented for pipewire](https://gitlab.freedesktop.org/pipewire/pipewire/-/issues/357), since it is highly specific to pulseaudio and most of its functionality can be accessed through pactl.
Some distributions (e.g. `archlinux` or `gentoo`) are thus not installing pacmd anymore when pipewire is installed with pulseaudio support.

And zoom for some reason uses pacmd internally to manage sound - thus [segfaulting](https://community.zoom.com/t5/Meetings/Unable-to-start-zoom-on-linux-with-quot-pacmd-command-not-found/m-p/61892) if `pacmd` is not installed.

## Explanation of the fix

Tl;dr: translate `pacmd` calls to `pactl` calls.

Since zoom expects there to be an executable `pacmd` in the `PATH`, there is a very easy way to track the calls to it.
Thus step 1 was to just log those calls to /tmp (feel free to uncomment that line and check for more calls should zoom be still crashing).

The only call that zoom made in my case was `pacmd list-sinks` which can we checked for and easily translated to `pactl list sinks short`, which produces the same output.
