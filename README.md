# Vector-06C disk images in Midnight Commander

Two extfs plugins that let mc walk into disk images of the Soviet-era
[Vector-06C](https://en.wikipedia.org/wiki/Vector-06C) home computer as if they were
directories:

* `ufdd` — floppy images (`*.fdd`), a CP/M filesystem in the "v06" format. Files can
  be listed, copied in and out and deleted. CP/M user numbers show up as `user_1` …
  `user_15` directories, user 0 being the root of the image.
* `uhdd` — hard disk images (`*.hdd`), a stack of floppy images sharing one boot area.
  Each disk shows up as `disk_NNNN.fdd` and can be copied out or replaced.

Press Enter on an image and mc opens it.

## Installing on macOS (Homebrew)

```bash
brew tap ekundo/mc-fdd
brew install ekundo/mc-fdd/mc-fdd
mc-fdd install
```

Homebrew may only write inside its own prefix, so the last step is a separate command:
it links the plugins into the mc user data directory and teaches the mc user extension
file about `*.fdd` and `*.hdd`. `mc-fdd uninstall` undoes it, `mc-fdd status` shows what
is where.

## Installing on Linux (Debian, Ubuntu, Raspberry Pi OS)

```bash
type -p curl >/dev/null || sudo apt install curl -y
curl -s --compressed "https://ekundo.github.io/mc/ppa/KEY.gpg" | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/ekundo_ppa.gpg >/dev/null \
&& sudo curl -s --compressed -o /etc/apt/sources.list.d/ekundo_ppa_list_file.list "https://ekundo.github.io/mc/ppa/list_file.list" \
&& sudo apt update \
&& sudo apt install mc-fdd mc-hdd -y
```

## Installing by hand

```bash
mkdir -p ~/.local/share/mc/extfs.d
cp packages/mc-fdd/ufdd packages/mc-hdd/uhdd ~/.local/share/mc/extfs.d
```

and add this to `~/.config/mc/mc.ext.ini`, right below `Version=`. Note that mc reads
the user file instead of the system one, so start from a copy of the system
`mc.ext.ini` (`mc -F` prints where both live):

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

`ufdd` needs [cpmtools](https://www.moria.de/~michael/cpmtools/) and gawk; it carries
the v06 disk format itself and does not need the format to be in the diskdefs of
cpmtools. `uhdd` needs nothing beyond bash and coreutils.

## Building the Debian packages

`./build` builds both packages and refreshes the apt repository under `ppa/`. It needs
`dpkg-buildpackage`, `config-package-dev` and a gpg key for signing the release.

### Demo

[![asciicast](https://asciinema.org/a/OGqfSFJljih71xtAEb10getNv.svg)](https://asciinema.org/a/OGqfSFJljih71xtAEb10getNv)
