# jvdavim's ErgoDox EZ keymap

Personal keymap for the ZSA ErgoDox EZ (`ergodox_ez/base` variant). Everything below is CLI-only — no QMK Toolbox / GUI required.

- Keyboard: `ergodox_ez/base`
- Keymap: `jvdavim`
- MCU: `atmega32u4`, bootloader: `halfkay` (Teensy)

## 1. One-time environment setup

These steps only need to be done once per machine.

### 1.1 Install the QMK CLI and build toolchain

```sh
curl -fsSL https://install.qmk.fm | sh
```

Then, from inside this repo (`/home/jvdavim/qmk_firmware`), run setup so QMK installs the AVR toolchain (`avr-gcc`, `avrdude`, `dfu-programmer`, etc.) via `pacman` and points the CLI at this checkout:

```sh
qmk setup -y
```

Verify it worked:

```sh
qmk doctor
```

`qmk doctor` should report your build environment as ready. If `avr-gcc`/`avrdude` are still missing afterwards, install them manually:

```sh
sudo pacman -S --needed base-devel avr-binutils avrdude dfu-programmer dfu-util avr-libc
```

(`avr-gcc` on Arch usually needs to come from the AUR or an older pinned package — see `util/install/arch.sh` in this repo for the exact version QMK expects if `pacman -S avr-gcc` doesn't work.)

### 1.2 Set sane CLI defaults

So you don't have to pass `-kb`/`-km` on every command:

```sh
qmk config user.keyboard=ergodox_ez/base
qmk config user.keymap=jvdavim
```

### 1.3 Linux udev rules (flash without sudo)

```sh
util/install_udev.sh
```

This installs the udev rules from the [qmk_udev](https://github.com/qmk/qmk_udev) project so your user can access the bootloader device without `sudo`. Unplug/replug the keyboard (or reboot) after running it.

### 1.4 Install a HalfKay-compatible flasher

The ErgoDox EZ's `halfkay` bootloader needs `teensy_loader_cli` on your `$PATH`. You never run this yourself — `qmk flash` (step 3 below) calls it internally once it detects the keyboard in bootloader mode. It just needs to be installed and discoverable on `$PATH` beforehand.

**Option A — AUR (fastest, if you have an AUR helper):**

```sh
yay -S teensy-loader-cli
```

**Option B — build from source:**

```sh
sudo pacman -S --needed libusb-compat
git clone https://github.com/PaulStoffregen/teensy_loader_cli.git /tmp/teensy_loader_cli
make -C /tmp/teensy_loader_cli
sudo install -Dm755 /tmp/teensy_loader_cli/teensy_loader_cli /usr/local/bin/teensy_loader_cli
```

Confirm it's found:

```sh
which teensy_loader_cli
```

## 2. Building the firmware

From the repo root:

```sh
qmk compile -kb ergodox_ez/base -km jvdavim
```

(or, if you set the defaults in 1.2: `qmk compile`)

This produces `.build/ergodox_ez_base_jvdavim.hex`.

## 3. Flashing

The important thing to understand: **run the flash command first, then trigger the reset.** `qmk flash` sits and watches for a bootloader device to appear (it polls for up to 10 minutes) before it flashes anything. So you type the command in your terminal *before* touching the keyboard's reset button — not after. Once the keyboard drops into bootloader mode it stops working as a keyboard (it enumerates as a completely different USB device), so there's nothing to type into it at that point anyway; `qmk flash`, already running and watching, notices it and flashes automatically.

Step by step:

1. In your terminal, start the flash command. It will print something like "waiting for device..." and just sit there — this is expected:

   ```sh
   qmk flash -kb ergodox_ez/base -km jvdavim
   ```

   (or just `qmk flash` with the defaults from 1.2, or `make ergodox_ez/base:jvdavim:flash`)

2. *Now*, with the command still running, put the keyboard into bootloader mode using any one of:
   - Press the physical reset button — a small recessed button accessible through a pinhole near the top of the right-hand half's case; press it with a paperclip or SIM-eject tool. This is the reliable option and doesn't depend on the current keymap.
   - Hold both `Shift` keys and press `Pause`, or hold both `Shift` and press `B` — QMK's generic default bootloader-entry combos. These only work if the currently-*flashed* firmware on the board still has default/Fn-layer bindings for them; since this is your own customized keymap, don't rely on these unless you've verified them.
   - Press a `QK_BOOT` key if you've bound one in the layout (this keymap doesn't currently have one bound — let me know if you'd like one added, it's the most convenient long-term option).

3. Back in the terminal, `qmk flash` detects the bootloader device, flashes automatically, and exits. The keyboard resets itself back into normal typing mode when done — no further action needed.

## Troubleshooting

- `qmk doctor` — diagnoses most environment/toolchain problems.
- If `qmk flash` can't find the bootloader device, double check `util/install_udev.sh` was run and the device was replugged, and that `teensy_loader_cli` is installed and on `$PATH`.
- Full reference docs: [Getting Started](https://docs.qmk.fm/#/newbs_getting_started), [Flashing](https://docs.qmk.fm/#/newbs_flashing), [Flashing bootloaders in depth](https://docs.qmk.fm/#/flashing).
