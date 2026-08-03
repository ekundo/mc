# Vector-06C disk images in Midnight Commander

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

Two extfs plugins that turn disk images of the Soviet-era
[Vector-06C](https://en.wikipedia.org/wiki/Vector-06C) home computer into ordinary
directories: press Enter on an image in mc and walk into it, copy files in and out,
delete them — no mounting, no emulator, no separate tool.

[![asciicast](https://asciinema.org/a/OGqfSFJljih71xtAEb10getNv.svg)](https://asciinema.org/a/OGqfSFJljih71xtAEb10getNv)

| plugin | handles | what you get |
| --- | --- | --- |
| `ufdd` | floppy images, `*.fdd` | the files of the CP/M filesystem inside; listing, copying both ways, deleting |
| `uhdd` | hard disk images, `*.hdd` | the floppies the image is made of, as `disk_0001.fdd` … ; copying out and replacing |

CP/M user numbers show up as directories `user_1` … `user_15`, user 0 being the root of
the image. A slash inside a CP/M name is shown as `∕` (U+2215), since a real slash would
be a directory separator to mc.

## Install on macOS

```bash
brew tap ekundo/mc-fdd
brew install ekundo/mc-fdd/mc-fdd
mc-fdd install
```

Homebrew is only allowed to write inside its own prefix, so the last step is a separate
command: it links the plugins into the mc user data directory and registers `*.fdd` and
`*.hdd` in the mc user extension file. `mc-fdd uninstall` undoes it, `mc-fdd status`
shows what is installed where. The tap lives in
[ekundo/homebrew-mc-fdd](https://github.com/ekundo/homebrew-mc-fdd).

## Install on Debian, Ubuntu, Raspberry Pi OS

```bash
type -p curl >/dev/null || sudo apt install curl -y
curl -s --compressed "https://ekundo.github.io/mc/ppa/KEY.gpg" | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/ekundo_ppa.gpg >/dev/null \
&& sudo curl -s --compressed -o /etc/apt/sources.list.d/ekundo_ppa_list_file.list "https://ekundo.github.io/mc/ppa/list_file.list" \
&& sudo apt update \
&& sudo apt install mc-fdd mc-hdd -y
```

The packages put the plugins into `/usr/lib/mc/extfs.d` and add the entries to the mc
extension file of the system, keeping the original around through `dpkg-divert`, so
nothing of mc is overwritten. `sudo apt remove mc-fdd mc-hdd` puts everything back.

## Install by hand

```bash
mkdir -p ~/.local/share/mc/extfs.d
cp packages/mc-fdd/ufdd packages/mc-hdd/uhdd ~/.local/share/mc/extfs.d
```

Then add the entries below to `~/.config/mc/mc.ext.ini`, right after `Version=`. mc reads
that file *instead of* the system one, not in addition to it, so start from a copy of the
system `mc.ext.ini` — `mc -F` prints where both live. Note the doubled backslash: mc since
4.8.30 reads this file as an ini and a single one is silently ignored.

```ini
[fdd]
Regex=\\.fdd$
RegexIgnoreCase=true
Open=%cd %p/ufdd://

[hdd]
Regex=\\.hdd$
RegexIgnoreCase=true
Open=%cd %p/uhdd://
```

On mc older than 4.8.30 the same goes into `~/.config/mc/mc.ext` in the old syntax:

```
regex/\.([fF][dD][dD])$
  Open=%cd %p/ufdd://
```

## Requirements

`ufdd` needs [cpmtools](https://www.moria.de/~michael/cpmtools/) and gawk. The v06 disk
format is not one cpmtools knows, so the plugin carries the definition itself and does
not care what is in the system `diskdefs`. `uhdd` needs nothing beyond bash and
coreutils.

## How the images are laid out

A floppy image is 839680 bytes: 164 tracks of 5 sectors of 1024 bytes, the first 8 tracks
being the system area, the rest a CP/M filesystem with 2048 byte blocks and 128 directory
entries.

A hard disk image is a stack of floppies sharing one boot area: a 16 bit little endian
count of them sits at offset 132, the boot area holds the first seven tracks of every
floppy, and the data of the floppies follows one after another. Reading a disk out of a
container therefore means gluing the shared boot area and the data of that disk back into
a plain `.fdd` image, which is what `uhdd` hands over to mc.

## Repository layout

```
packages/mc-fdd/    ufdd and its Debian packaging
packages/mc-hdd/    uhdd and its Debian packaging
brew/mc-fdd         the command that hooks the plugins into mc for one user
ppa/                the apt repository served from GitHub Pages
build               builds both packages and refreshes ppa/
```

`./build` needs a Debian machine with `dpkg-buildpackage` and `config-package-dev`, plus
the gpg key the repository is signed with.
