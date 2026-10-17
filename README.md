# Fork notice

This repository is a fork of [sway](https://github.com/swaywm/sway). The
default branch, `master`, has the upstream code and one additional feature:
scale control for X11 windows.

## Purpose of the fork

Standard sway shows X11 windows at scale 1. On a HiDPI output, sway then makes
the windows larger. The image is not clear.

This fork permits the X11 clients to draw at the scale of the output. Sway then
shows each window at the correct size, and the image is clear. KDE Plasma uses
the same method.

The feature adds two configuration commands:

- `xwayland_scaling off|auto|<factor>` sets the scale for the X11 clients. The
  default is `off`.
- `xwayland_scale_command <command>` starts a command each time the scale
  changes. The command gets the scale in the `SWAY_XWAYLAND_SCALE` environment
  variable.

A change to wlroots is also necessary for this feature. The change is in the
patch file in `contrib/x11-scaling/wlroots/`. The same change is in the
`master` branch of [kinyoklion/wlroots](https://github.com/kinyoklion/wlroots).

## Branches

- `master` is the default branch. It has upstream sway and the feature.
- `x11-scaling` has the feature on the upstream commit of the first tests. Use
  `master` for new work.

## Related repositories

- [kinyoklion/wlroots](https://github.com/kinyoklion/wlroots): wlroots with
  the necessary change.
- [kinyoklion/swayfx](https://github.com/kinyoklion/swayfx): SwayFX with the
  same feature.

## More information

Read [contrib/x11-scaling/README.md](contrib/x11-scaling/README.md). It gives
the design, the build procedure, and the test procedure.

The text that follows is the upstream README.

---

# sway

**[English][en]** - [عربي][ar] - [Azərbaycanca][az] - [Česky][cs] - [Deutsch][de] - [Dansk][dk] - [Español][es] - [Français][fr] - [ქართული][ge] - [Ελληνικά][gr] - [हिन्दी][hi] - [Magyar][hu] - [فارسی][ir] - [Italiano][it] - [日本語][ja] - [한국어][ko] - [Nederlands][nl] - [Norsk][no] - [Polski][pl] - [Português][pt] - [Română][ro] - [Русский][ru] - [Српски][sr] - [Svenska][sv] - [Türkçe][tr] - [Українська][uk] - [中文-简体][zh-CN] - [中文-繁體][zh-TW]

sway is an [i3]-compatible [Wayland] compositor. Read the [FAQ]. Join the
[IRC channel] \(#sway on irc.libera.chat).

## Release Signatures

Releases are signed with [E88F5E48] and published [on GitHub][GitHub releases].

## Installation

### From Packages

Sway is available in many distributions. Try installing the "sway" package for
yours.

### Compiling from Source

Check out [this wiki page][Development setup] if you want to build the HEAD of
sway and wlroots for testing or development.

Install dependencies:

* meson \*
* [wlroots]
* wayland
* wayland-protocols \*
* pcre2
* json-c
* pango
* cairo
* gdk-pixbuf2 (optional: additional image formats for system tray)
* [swaybg] (optional: wallpaper)
* [scdoc] (optional: man pages) \*
* git (optional: version info) \*

_\* Compile-time dep_

Run these commands:

    meson setup build/
    ninja -C build/
    sudo ninja -C build/ install

## Configuration

If you already use i3, then copy your i3 config to `~/.config/sway/config` and
it'll work out of the box. Otherwise, copy the sample configuration file to
`~/.config/sway/config`. It is usually located at `/etc/sway/config`.
Run `man 5 sway` for information on the configuration.

## Running

Run `sway` from a TTY or from a display manager.

[en]: https://github.com/swaywm/sway#readme
[ar]: README.ar.md
[az]: README.az.md
[cs]: README.cs.md
[de]: README.de.md
[dk]: README.dk.md
[es]: README.es.md
[fr]: README.fr.md
[ge]: README.ge.md
[gr]: README.gr.md
[hi]: README.hi.md
[hu]: README.hu.md
[ir]: README.ir.md
[it]: README.it.md
[ja]: README.ja.md
[ko]: README.ko.md
[nl]: README.nl.md
[no]: README.no.md
[pl]: README.pl.md
[pt]: README.pt.md
[ro]: README.ro.md
[ru]: README.ru.md
[sr]: README.sr.md
[sv]: README.sv.md
[tr]: README.tr.md
[uk]: README.uk.md
[zh-CN]: README.zh-CN.md
[zh-TW]: README.zh-TW.md
[i3]: https://i3wm.org/
[Wayland]: http://wayland.freedesktop.org/
[FAQ]: https://github.com/swaywm/sway/wiki
[IRC channel]: https://web.libera.chat/gamja/?channels=#sway
[E88F5E48]: https://keys.openpgp.org/search?q=34FF9526CFEF0E97A340E2E40FDE7BE0E88F5E48
[GitHub releases]: https://github.com/swaywm/sway/releases
[Development setup]: https://github.com/swaywm/sway/wiki/Development-Setup
[wlroots]: https://gitlab.freedesktop.org/wlroots/wlroots
[swaybg]: https://github.com/swaywm/swaybg/
[scdoc]: https://git.sr.ht/~sircmpwn/scdoc
