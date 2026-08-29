# Post-install profiles

The common image does not guess which machine it is running on. Optional or
machine-specific behavior is exposed through `ujust` and remains off until the
user explicitly enables it.

Run `ujust --choose` to browse recipes, or use the commands below directly.

## Common user setup

```bash
ujust setup-zsh
ujust setup-virtualization
```

New accounts receive Zsh and the image-baked Zim module tree automatically.
`setup-zsh` is for upgraded accounts and preserves existing `.zshrc` and
`.zimrc` files. To also select Zsh as the login shell, run `ujust setup-zsh
true`; every retained deployment must continue to include `/usr/bin/zsh` for
rollback-safe login.

`setup-virtualization` enables libvirt's modular sockets and default NAT
network. Administrators in `wheel` are authorized by the image's
polkit rule, so a separate group change and logout are not required. The command
also grants the `qemu` service account traversal-only access to your private
home directory so ISO images selected from `~/Downloads` can be opened.

## RamaLama

```bash
ujust ramalama-setup
# or, to install the CLI and download every selected model:
ujust ramalama-setup-all
```

Both options first check rootless Podman, Python, `jq`, and the AMD Vulkan
environment. `ramalama-setup` creates a user-local virtual environment in
`~/.local/share/ramalama-cli`, exposes `ramalama` through `~/.local/bin`, and
does not download weights. `ramalama-setup-all` also pulls the selected GGUF
models, which requires at least 145 GiB free in the model store (by default
`~/ai/models`). Set `RAMALAMA_STORE=/path/on/a/large/disk` before
running either command to relocate that store.

Use an individual `ramalama-pull-*` recipe to download a model later. Run
`ujust ramalama-list` to inspect local models and `ujust ramalama-smoke` after
the Liquid model is present. `ujust --choose` groups the install, pull, and run
helpers under RamaLama.

## Lossless Scaling

The `lsfg-vk` layer is installed, but frame generation still requires the
purchased Windows Lossless Scaling application in the native Steam library.
Use `lsfg-vk-ui` to select its `Lossless.dll` and configure game profiles.

The upstream default configuration includes a `vkcube` test profile. Until the
DLL is installed, bypass that profile when checking the baseline Vulkan stack:

```bash
DISABLE_LSFG=1 vkcube
```

## Labeled internal drive automount

```bash
ujust automount status
ujust automount enable
```

This opts the machine into Universal Blue's service for labeled, non-removable
Btrfs/ext4 partitions. It mounts eligible drives under `/run/media/system` and
does not alter `/etc/fstab`. It is intended for the Ryzen AI MAX host's
`gamedrive` disk and is disabled by default elsewhere.

## Streaming

```bash
ujust install-streaming moonlight
ujust install-streaming sunshine
# or
ujust install-streaming both
```

These are per-user Flatpaks. Sunshine also runs the upstream-required
`additional-install.sh` host integration; reboot before testing Wayland capture,
audio, mouse, and controller input.

## SSH and Wake-on-LAN

```bash
ujust ssh-server enable
ujust wake-on-lan
ujust wake-on-lan eth0 enable
```

The SSH command enables `sshd` and the firewalld SSH service. Confirm key or
password authentication locally before depending on remote access.

The Wake-on-LAN command lists candidate interfaces when no interface is given.
It verifies magic-packet support and updates the active NetworkManager wired
connection. Firmware settings and power-state support must still be checked on
each machine.

## Ryzen AI MAX+ 395 legacy memory profile

```bash
ujust setup-ai-max status
ujust setup-ai-max apply
```

The apply path refuses to run unless both the Ryzen AI MAX+ 395 CPU and the
GMKtec NucBox EVO-X2 DMI identity match. It removes `iommu=off`, explicitly
enables AMD IOMMU, and restores the former large TTM/GTT values in a new atomic
deployment. The GTT argument is deprecated and is retained only as a temporary
compatibility setting pending real AI workload tests. Use
`ujust setup-ai-max undo` to remove the profile.

Do not use this profile on the Ryzen 6550U or Intel/Arc machines.
