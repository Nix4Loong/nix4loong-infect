# nix4loong-infect

Turn a running loongarch64 Linux host (AOSC OS, Arch Linux for Loong64, etc.) into NixOS in place, using the [nix4loong](https://nix4loong.cn) ecosystem:

- Installer: [Nix4Loong/experimental-nix-installer](https://github.com/Nix4Loong/experimental-nix-installer)
- Channel: `https://download.nix4loong.cn/nix-channels/loong-master`
- Binary cache: `https://cache.nix4loong.cn`

This is a loongarch64-focused port of [`elitak/nixos-infect`](https://github.com/elitak/nixos-infect) — see it for background, caveats, and the security disclaimer (this script wipes the host!).

## Usage

```bash
curl -fsSL https://github.com/Nix4Loong/nix4loong-infect/raw/refs/heads/main/nix4loong-infect | sudo bash -x
```

### Environment variables

- `doNetConf` — when non-empty, capture the host's current interface addresses, gateways, nameservers, and MAC-to-ifname rules into `/etc/nixos/networking.nix` and import it. ⚠️ **Required** if the host uses a **static IP** (otherwise NixOS's default DHCP will pick up nothing and you lose the box). Safe to leave unset on DHCP-configured hosts.
- `NIX_INSTALL_URL` — override installer URL (default `https://install.nix4loong.cn`).
- `NIX_CHANNEL_URL` — override channel URL (default `https://download.nix4loong.cn/nix-channels/loong-master`). Accepts a full URL, unlike upstream's `NIX_CHANNEL` which takes a name.
- `NIXOS_CONFIG` — path or HTTP URL of a pre-baked `configuration.nix`. When set, `makeConf` still writes the default config first, then this replaces it inside `infect`.
- `NIXOS_IMPORT` — extra Nix import added to the generated `configuration.nix`'s `imports` list.
- `NO_SWAP` — don't create a temporary 1 GiB swap file during the install.
- `NO_INFECT` — run config generation only, skip the Nix install / lustrate.
- `NO_REBOOT` — don't reboot at the end.

Combine `NO_INFECT=1 NO_REBOOT=1 NO_SWAP=1` for a dry run that only writes `/etc/nixos/*.nix` without touching anything else.

## Prerequisites

- Root access.
- Running on loongarch64 (`uname -m == loongarch64`); the script aborts otherwise.
- `/root/.ssh/authorized_keys` (or `$SUDO_USER`'s) populated — that's the only way to log back in after reboot.
- **New-world** loongarch with UEFI boot recommended. Non-EFI MBR/GRUB fallback is inherited from upstream but untested on loongarch.

## Differences from upstream nixos-infect

- Targets the nix4loong installer, channel, and binary cache.
- Declares `nix.settings.extra-substituters` + `extra-trusted-public-keys` in the generated `configuration.nix` so post-reboot rebuilds can pull from `cache.nix4loong.cn`.
- Provider-specific logic (DigitalOcean, Hetzner, Lightsail, Hostinger, …) stripped out — no loongarch VPS providers.
- Architecture check up front; no `isX86_64` branching.
- Package-manager bootstrap block extended with AOSC (`oma`) and Arch (`pacman`) fallbacks.

## License

GPLv3, inherited from upstream nixos-infect. See [LICENSE](./LICENSE).
