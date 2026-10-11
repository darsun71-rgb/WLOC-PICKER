# WLOC Location Spoofer for Loon

Intercepts Apple's network-location responses (`gs-loc.apple.com`) and replaces the
coordinates handed to iOS, so map/social apps on the phone read a virtual location.
The response-rewriting script is the community-maintained
[zxishere/wloc](https://github.com/zxishere/wloc) project; this `.lpx` file is an
English-language Loon module that wires it up.

## What it affects

- **Only** Wi-Fi / cell-tower network positioning — the coordinates Apple computes
  from nearby Wi-Fi and base stations.
- It does **not** spoof GPS hardware. Outdoors with a strong GPS signal, the real
  position may win. Works best indoors / on Wi-Fi.

## Coordinate picker

`wloc-picker.html` in this folder is a self-contained picker page: tap the map (or
drag the pin, or search a place) to get validated **WGS84** GPS coordinates, with
one-tap copy buttons for the Loon arguments and the Shadowrocket argument string.
It also converts Amap (GCJ-02) and Baidu (BD-09) coordinates — including extracting
them from an Amap share link — so the numbers you feed the module are correct.
Open it in any browser, including on the phone itself.

## Shadowrocket

Use `wloc-shadowrocket.module` in this folder instead of the `.lpx`:

1. In Shadowrocket: **Config → Modules → +**, paste the module URL
   (`https://raw.githubusercontent.com/BH0sec/wloc/refs/heads/main/modules/wloc.module`
   works out of the box) or import the local `wloc-shadowrocket.module` file.
2. Turn on **HTTPS Decryption**, install the generated CA, and enable full trust
   under iOS **Settings → General → About → Certificate Trust Settings**.
3. The module appends (`%APPEND%`) the Apple location hostnames to your existing
   MITM list, so it won't clobber other modules.
4. Set coordinates by editing the `argument=` values in the module
   (`longitude=…&latitude=…`), or use the same picker page / iOS Shortcuts as the
   Loon version — the settings-save interception works identically.

## Install in Loon

1. In Loon: **Config → Plugins → + → Add URL**, and paste:
   `https://raw.githubusercontent.com/zxishere/wloc/main/wloc.lpx`
   (or import the `wloc.lpx` file in this folder the same way if you host it
   somewhere reachable by the phone).
2. Enable the plugin.

## Enable HTTPS decryption (required)

The module must read Apple's encrypted location traffic on your own device:

1. In Loon, turn on **MITM / HTTPS Decryption** and generate the CA certificate.
2. Install the certificate profile, then open iOS **Settings → General → About →
   Certificate Trust Settings** and enable full trust for the Loon root CA.
   (Installing the profile alone is not enough — the trust toggle is the step
   people miss.)
3. Make sure the MITM hostnames include: `gs-loc.apple.com`,
   `gs-loc-cn.apple.com`, `gsp-ssl.ls.apple.com` (the module declares these).

## Set your virtual location

Two options; the picker/shortcut overrides the module arguments:

- **Module arguments (simplest):** in Loon, open the plugin's settings and fill in
  `longitude` / `latitude` (WGS84) plus accuracy.
- **Point-and-pick page:** open `https://zxishere.github.io/wloc/` in Safari, tap
  the map, and save — the module intercepts the save request locally on the
  device, so the coordinates never leave your phone. There are also iOS Shortcuts
  ("set location" / "clear location") in the repo README for one-tap switching.

## Verify and troubleshoot

- Open Apple Maps and check the blue dot.
- If the map still shows the old spot, restart the phone — iOS `locationd` caches
  aggressively; toggling airplane mode often isn't enough.
- Check the Loon plugin log (`logLevel` argument) for "modified" entries to
  confirm interception is working.
- Note: newer iOS versions are tightening network-location paths; if it stops
  working after an iOS update, check the upstream repo for fixes.

## Notes

- Defaults ship with a Shenzhen example coordinate and pass through unmodified
  until you set your own — set the arguments or use the picker before expecting
  any change.
- For your own device only. Spoofing location can violate the terms of some apps
  and services; use responsibly.
- Want zero third-party hosting? The upstream repo documents how to self-host the
  scripts and picker page (Cloudflare Worker/Pages) — the `.lpx` here can be
  repointed at your own URLs.
