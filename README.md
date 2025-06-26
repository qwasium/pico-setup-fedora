# Raspberry Pi Pico dev environment on Fedora

- C/C++ development for Raspberry Pi Pico
- Fedora Linux (`dnf` package manager)
- `direnv` to manage environment variables

If you don't use `direnv`, see `.envrc` to know what's going on.

## pico 1 vs pico 2

|              | pico 1        | pico 2 |
| ------------ | ------------- | ------ |
| processor    | RP2040        | RP2350 |
| ARM core     | Cortex-M0+    | Cortex-M33 |
| RISC-V core  | n/a           | Hazard3 |
| SRAM         | 264KB 6 banks | 520KB 10 banks |
| flash memory | 2MB           | 4MB |
| official variants | H, W, WH | W |

Check which one you're using; compilation flags are different!

## Documentation and References

- [Getting started with Raspberry Pi Pico-series](https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf)
- [Official setup script](https://github.com/raspberrypi/pico-setup/blob/master/pico_setup.sh)
- [Raspberry Pi Pico-series C/C++ SDK](https://rptl.io/pico-c-sdk)
- [Blog post explaining GNU ARM toolchain](https://embeddedinventor.com/a-complete-beginners-guide-to-the-gnu-arm-toolchain-part-1/)

## About git submodules

Directory name for `picotool` repo is `pico-tool` to avoid name collision with zsh auto_cd.

`picotool` and `pico-sdk` is on `develop` branch because the fix for picotool discovery is still not in the `master` branch.

See [GitHub issue](https://github.com/raspberrypi/pico-sdk/issues/1818).

WIP: Edited `pico-sdk/tools/Findpicotool.cmake`, also see `.envrc`.

## Setup

Install dependencies.

```bash
sudo dnf update
sudo dnf install development-tools c-development
sudo dnf install cmake direnv libusb1
```

### GNU ARM Embedded Toolchain

#### Package Manager

```
# dnf search arm
sudo dnf install arm-none-eabi-gcc-cs arm-none-eabi-gcc-cs-c++
sudo dnf install gcc-arm-linux-gnu gcc-c++-arm-linux-gnu
```

### Direct download

Downloading the GNU ARM toolchain directly from the official arm webpage and adding it to `$PATH` will also work.

Which one?

- AArch32 bare-metal target/x86_64 Linux host
  - `arm-none-eabi`: for ARM bare-metal systems
  - `x86_64-linux`: host system; choose whatever you're running
  - RP2040 doesn't have hardware float; don't choose gnueabihf
  - `AArch32`: 32-bit

```bash
# First download
tar -xvJf path/to/toolchain.tar.xz
```

### direnv

Setup direnv if newly installed: [docs](https://direnv.net/).

```bash
git clone https://github.com/qwasium/pico-setup-fedora.git
git submodule update --init --recursive
```

### pico-tool

CLI tool for working with RP2040/RP2350 binaries.

`pico-sdk>2.0.0` uses `picotool` to do the ELF to UF2 conversion.

```bash
cd pico-tool # ->pico-setup-fedora/pico-tool/
mkdir build
cd build  # ->pico-setup-fedora/pico-tool/build/
cmake ..
make
cd ..  # ->pico-setup-fedora/pico-tool/

sudo cp udev/99-picotool.rules /etc/udev/rules.d
cd .. # ->pico-setup-fedora/
```

## pico examples

```bash
cd pico-examples
mkdir build
cd build
cmake ..  # FIXME: cmake can't find picotool error
```

---

WIP from here

```bash
cd blink
make -j4
```

## pico extras/playground

```bash
git clone -b master git@github.com:raspberrypi/pico-extras.git
git clone -b master git@github.com:raspberrypi/pico-playground.git
```

## debuggerprobe

```bash
git clone git@github.com:raspberrypi/debugprobe.git
cd debugprobe
git submodule update --init --recursive
mkdir build
cd build
# for debug on pico2
cmake -DDEBUG_ON_PICO=ON -DPICO_BOARD=pico2 ..
make
```

