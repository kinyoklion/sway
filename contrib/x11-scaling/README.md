# X11 self-scaling for sway (`xwayland_scaling`)

This branch adds KDE-style "X11 applications apply scaling themselves" to
sway. On hidpi outputs, stock sway/wlroots upscale Xwayland windows and they
come out blurry. With `xwayland_scaling`, Xwayland is handed a server-side
scale override `S` (the maximum output scale, or a fixed factor) and X11
clients see a screen of `logical * S` pixels, so toolkits that honor `Xft.dpi`
render crisp at native density. The option defaults to `off`, where behavior
is identical to upstream sway.

Research and design notes live outside this repo; the short version of the
mechanism is:

- wlroots (`wlroots/*.patch`): `wlr_server_scale_set(client, S)` marks one
  `wl_client` (the Xwayland connection). Its buffers are displayed at `1/S`
  (scene dest size + opaque region), surface-local input coordinates are
  multiplied by `S`, xdg-output logical geometry is sent multiplied by `S`,
  and fractional-scale / preferred-buffer-scale hints are suppressed for it.
- sway: `sway/desktop/xwayland_scaling.c` computes `S` from the config and
  the outputs, applies it (on Xwayland ready and on every output change),
  re-sends xdg-output, re-configures mapped X11 views and runs
  `xwayland_scale_command`. `sway/desktop/xwayland.c` converts between
  logical and X11-native units at every boundary (configure, commit
  geometry, natural size, size hints, unmanaged window positions).

## Files

| Path | Purpose |
|---|---|
| `wlroots/0001-*.patch` | Required companion patch for wlroots. Applies on top of the commit in `wlroots/UPSTREAM_REV` (wlroots 0.21.0-dev). |
| `sway-xwayland-setup` | Helper for `xwayland_scale_command`: merges `Xft.dpi = 96*S` and `Xcursor.size = base*S` into the X resource database with `xrdb`. |
| `nested-test.conf` | Config for trying the feature nested inside an existing session. |

## Configuration

```
xwayland_scaling auto                      # off | auto | <factor>; default off
xwayland_scale_command sway-xwayland-setup # run with SWAY_XWAYLAND_SCALE set
```

See `sway(5)` for the full description of both commands.

## Building

sway on this branch requires wlroots `>=0.21.0` (git master) **with the
patch applied**:

```
git clone https://gitlab.freedesktop.org/wlroots/wlroots.git
cd wlroots
git checkout "$(cat ../sway/contrib/x11-scaling/wlroots/UPSTREAM_REV)"
git am ../sway/contrib/x11-scaling/wlroots/*.patch
```

Then build sway as usual (`meson setup build && ninja -C build`), either
against an installed patched wlroots or with `subprojects/wlroots` pointing
at the patched checkout.

The `master` branch of <https://github.com/kinyoklion/wlroots> already has this
patch. As an alternative, clone that branch and do not apply the patch.

### NixOS sketch

```nix
let
  swaySrc = pkgs.fetchFromGitHub { owner = "kinyoklion"; repo = "sway"; rev = "<commit>"; hash = "<sha256>"; };
  wlrootsSrc = pkgs.fetchFromGitLab {
    domain = "gitlab.freedesktop.org"; owner = "wlroots"; repo = "wlroots";
    rev = "<contents of contrib/x11-scaling/wlroots/UPSTREAM_REV>"; hash = "<sha256>";
  };
  wlroots-x11scale = pkgs.wlroots_0_20.overrideAttrs (old: {
    version = "0.21.0-dev"; src = wlrootsSrc;
    patches = (old.patches or [ ]) ++ [ "${swaySrc}/contrib/x11-scaling/wlroots/0001-Add-server-side-per-client-scale-override-for-Xwayla.patch" ];
  });
  sway-x11scale-unwrapped = (pkgs.sway-unwrapped.override { wlroots_0_20 = wlroots-x11scale; })
    .overrideAttrs (old: { version = "git-x11scale"; src = swaySrc; });
in
pkgs.sway.override { sway-unwrapped = sway-x11scale-unwrapped; }
```

(wlroots master currently needs `libdrm >= 2.4.134`; pick a nixpkgs that has it.)

## Trying it nested

```
sway -c contrib/x11-scaling/nested-test.conf
```

This opens a 1600x1200 window that is an 800x600 logical output at scale 2
with `xwayland_scaling auto`. `Mod4+Return` starts `xterm`, `Mod4+d` prints
`xdpyinfo` dimensions (expect 1600x1200 and 192 dpi), `Mod4+Shift+e` exits.

## Status / known gaps

- X11 client cursors render `S` times too large (wlroots cursor scaling for
  the overridden client is not done yet).
- Only integer scale factors have been tested; fractional `S` is untested.
- Drag-and-drop and mixed-scale multi-output setups are not verified yet.
- The wlroots hook is a single-client global, not an upstream-shaped API.

## Updating the wlroots patch

From a checkout of <https://github.com/kinyoklion/wlroots>, branch `master`,
with upstream wlroots as the `upstream` remote:

```
git format-patch -o ../sway/contrib/x11-scaling/wlroots upstream/master..master
git rev-parse upstream/master > ../sway/contrib/x11-scaling/wlroots/UPSTREAM_REV
```
