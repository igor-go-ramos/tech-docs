# :material-arch: Arch Linux
## :fontawesome-solid-cubes: Pacotes Úteis:

```bash
nano tmux pacman-contrib go git zsh base-devel man-pages tldr htop zip gzip atool ttf-jetbrains-mono kdeconnect xdg-desktop-portal
```

## :material-cog: Configurações
`/etc/systemd/logind.conf` - `handlelid` pode ser configurado para `ignore` assim quando a tampa do laptop for fechada nada é feito

```bash title="/etc/vconsole.conf"
FONT=ter-v24n
KEYMAP=us-acentos
XKBLAYOUT=us
XKBMODEL=us-acentos
```

## :material-expansion-card: NVIDIA
Instalar driver:

```bash
pacman -S nvidia
```
Verificar a saída do comando (sugere habilitar o nvidia-resume)

```bash
systemctl enable --now nvidia-resume.service
```

Em alguns casos é possível encontrar drivers mais atualizados no AUR:

```bash
yay -S nvidia-driver-xxx
```

Remover `kms` do `mkinitcpio.conf` (`/etc/mkinitcpio.conf`) e executar:

```bash
mkinitcpio -P
```

## :material-vlc: VLC
Para instalar o programa e todos os plugins:

```bash
pacman -S vlc vlc-plugins-all
```