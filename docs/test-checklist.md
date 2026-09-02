# Vimmite V5 physical test checklist

Record the image digest, machine, firmware version, Secure Boot state, and test
date for every run. Do not promote a build until the blocking checks pass.

## Build artifact gate

- [x] `bluebuild validate recipes/vimmite.yml` passes.
- [x] A clean `bluebuild build --no-sign recipes/vimmite.yml` passes.
- [x] The image uses `ghcr.io/ublue-os/kinoite-main:44` and Fedora's standard
      kernel, not an OGC/Bazzite kernel.
- [x] Firefox is present; Gamescope and NVIDIA-specific packages are absent.
- [ ] LACT, input-remapper, and their enabled services are present.
- [ ] `brew` initializes successfully through BlueBuild's native module, including `hf --help`.
- [ ] No EVDI or DisplayLink packages, modules, services, scripts, or helpers
      remain in the image.
- [x] Steam, 32/64-bit MangoHud, controller rules, and `lsfg-vk` are present.
- [x] `modprobe -c` reports `options hid_apple fnmode=2`, and the image
      initramfs contains the setting.
Verified locally on 2026-08-25 against image
`sha256:71fedc25c0c0fa5058770aef2575fbca2535688200562bffed17d556b0e24362`.
The previous artifact used kernel `7.1.10-200.fc44.x86_64`; `dnf check` also
completed successfully. Re-run this gate for the current change set before
installation. Static artifact verification is not approval to install or
rebase.

## Installation and rollback

- [ ] Install on the explicitly designated test machine without changing the
      current primary host.
- [ ] Installer completes without the previously observed grey screen.
- [ ] First reboot reaches SDDM and Plasma on every connected native display.
- [ ] After the first image update, `rpm-ostree status` retains the prior
      deployment and it can be selected from the boot menu.
- [ ] `rpm-ostree status` reports the expected signed image origin/digest.
- [ ] Btrfs root/home layout and encryption match the chosen installer plan.

## Portable baseline on every machine

- [ ] Wi-Fi, Ethernet, Bluetooth, audio, camera, keyboard, and touchpad work.
- [ ] Suspend/resume succeeds ten consecutive times, including an overnight
      suspend where practical.
- [ ] Boot offline, observe a failed Flatpak reconciliation, connect Wi-Fi, and
      verify all required Flatpaks install without rebooting.
- [ ] Firefox, Brave, Bitwarden, Warehouse, and LocalSend launch.
- [ ] A new user starts in Zsh with working Zim and `~/dev`, `~/sync`, and `~/ai`.
- [ ] Zsh/Zim setup preserves pre-existing dotfiles on a rerun.
- [ ] Distrobox can create, enter, update, and remove a disposable test box.
- [ ] No AI MAX TTM/GTT arguments appear on Ryzen 6550U or Intel systems.

## Strix Halo local AI

- [ ] `ujust strix-halo-ai` opens the 15-choice `ugum` menu.
- [ ] Unsupported AMD hardware is refused unless the documented diagnostic override is set.
- [ ] Vulkan uses `vulkan-radv`, `/dev/dri`, `keep-groups`, and unconfined seccomp.
- [ ] ROCm uses `rocm-10.0`, `/dev/dri`, `/dev/kfd`, `keep-groups`, and unconfined seccomp.
- [ ] Both `llama-cli --list-devices` calls report gfx1151/Radeon 8050S or 8060S.
- [ ] The version menu option reports the toolbox image and `llama-server` build for each installed backend.
- [ ] Host and both containers see the same sentinel below `~/ai/models`.
- [ ] Every host wrapper selects its intended backend and supplies overridable defaults.
- [ ] A localhost llama-server responds from the host on a non-conflicting test port.
- [ ] Bare Vulkan and ROCm server wrappers enter router mode rooted at `~/ai/models` and list the three managed IDs.
- [ ] Qwen 3.6 and 3.8 use `draft-mtp` with `spec-draft-n-max = 3`; Muse has no MTP preset.
- [ ] Update/recreate and Remove leave the model sentinel and downloaded weights intact.
- [ ] Reinstall after removal restores the backend and host wrappers cleanly.
- [ ] No host ROCm/llama.cpp package, udev mode rule, kernel argument, or BIOS change is introduced.

## Graphics and gaming

- [ ] `vulkaninfo` identifies the intended AMD or Intel GPU without software
      rendering.
- [ ] Native Steam launches and sees internal/external game libraries.
- [ ] A native Linux game and a Proton game launch.
- [ ] MangoHud works for both a 64-bit game and a 32-bit/Proton title.
- [ ] The EPOMAKER EA75 function row behaves normally with `fnmode=2`.
- [ ] ProtonPlus can install a compatibility tool visible to Steam.
- [ ] Bottles creates and launches a disposable test bottle.
- [ ] Lossless Scaling's Vulkan layer is discoverable and passes one real game
      test. Gamescope is not required for acceptance.

## Controllers

- [ ] The Chicken Run receiver enumerates in its `054c:09cc` PlayStation mode.
- [ ] Steam Input sees buttons, sticks, triggers, touchpad, motion, and rumble.
- [ ] Reconnect the receiver and repeat after resume.
- [ ] If the receiver exposes `3537:0575`, capture `udevadm`, `libinput`, and
      Steam Input results before adding a workaround.
- [ ] Pair and test the primary controller over Bluetooth.
- [ ] Pair and test an Xbox controller over Bluetooth.
- [ ] Test a Steam Controller when hardware becomes available.

## Audio and microphone

- [ ] `ujust hyperx-mic status` reports disabled before explicit opt-in.
- [ ] `ujust hyperx-mic enable` enables the per-user path unit and is safe to rerun.
- [ ] HyperX playback and capture appear after receiver reconnect and resume.
- [ ] The microphone starts at 90%.
- [ ] Vesktop cannot lower the source volume during calls or device changes.
- [ ] OBS, browser capture, and normal volume controls still work.

## Virtualization and optional profiles

- [ ] virt-manager connects to `qemu:///system` as the non-root user.
- [ ] The Arch ISO in `~/Downloads` reaches its boot menu with KVM, UEFI, and
      the default NAT network.
- [ ] Create a UEFI VM, a TPM-backed VM, and a NAT-connected VM.
- [ ] VM shutdown and host suspend/resume do not leave stale libvirt state.
- [ ] Labeled-drive automount mounts `gamedrive` under `/run/media/system` only
      on the opted-in host.
- [ ] Moonlight client streaming works if installed.
- [ ] Sunshine Wayland capture, audio, and remote controller input work if
      installed.
- [ ] SSH and Wake-on-LAN remain disabled until explicitly enabled, then pass a
      same-LAN test and can be disabled again.
- [ ] LACT connects to `lactd`; input-remapper lists devices and answers its
      control handshake; both services survive reboot.

## Evidence to collect on failure

Capture `rpm-ostree status`, `journalctl -b`, the previous boot journal when a
resume fails, `inxi -Fz`, `lsusb`, `lspci -nnk`, and the exact command/output for
the failing component. Avoid copying credentials or unrelated user data.
