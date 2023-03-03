# Installing on Linux

## Official sources

### Debian, Ubuntu Linux, Raspberry Pi OS (apt)

Install:

```bash
type -p curl >/dev/null || sudo apt install curl -y
curl -s --compressed "https://ekundo.github.io/mc/ppa/KEY.gpg" | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/ekundo_ppa.gpg >/dev/null \
&& sudo curl -s --compressed -o /etc/apt/sources.list.d/ekundo_ppa_list_file.list "https://ekundo.github.io/mc/ppa/list_file.list" \
&& sudo apt update \
&& sudo apt install mc-fdd -y
```
