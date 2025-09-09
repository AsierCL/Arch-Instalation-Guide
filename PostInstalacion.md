# Configuracion de AsierCL de Arch Linux post instalación

## Configuracion de pacman
En Misc options, descomento:
- Color
- VerbosePkgList
- Cambio el ParallelDownloads a 10
- Agrego ILoveCandy

En los repositorios, descomento el multilib


## Creamos directorios de usuarios

```
pacman -S xdg-user-dirs
xdg-user-dirs-update
su miusuario -c "xdg-user-dirs-update"
```

## Instalamos YAY
```
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

## Instalamos todos los programas necesarios para los dotfiles
```
yay -S 7zip adobe-source-code-pro-fonts bat bc blueman bluez brightnessctl btop cava cliphist corectrl fastfetch gamemode gnome-system-monitor grim gtk-engine-murrine hypridle hyprland hyprlock hyprpolkitagent imv jq keyd kitty kvantum less libnotify lxappearance loupe lsd ly nano neovim noto-fonts noto-fonts-emoji nvtop nwg-displays nwg-look otf-font-awesome openssh pamixer pavucontrol pipewire-alsa pipewire-audio pipewire-pulse qalculate-gtk qt5ct qt6ct rofi-wayland slurp spice spicetify-cli spotify-launcher swappy swaync swww thunar tree ttf-droid ttf-fantasque-nerd ttf-fira-code ttf-jetbrains-mono ttf-jetbrains-mono-nerd ttf-victor-mono visual-studio-code-bin wakeonlan wallust waybar wireplumber wlogout  xdg-desktop-portal-gtk xdg-desktop-portal-hyprland yadm zen-browser-bin zsh zsh-autosuggestions zsh-sudo zsh-syntax-highlighting 

systemctl enable --now ly
systemctl enable --now keyd
```

Metemos lo siguiente en el archivo _/etc/keyd.default.conf_
```
[ids]
*=*

[main]
capslock = overload(control, esc)
```

## Configuramos cred de github
```
ssh-keygen -t ed25519 -C "tuemail@ejemplo.com"
```
Clonamos el repo
```
yadm clone https://github.com/AsierCL/Dotfiles.git
yadm reset --hard 
```

## Asignamos ZSH como shell por defecto
```
chsh -s $(which zsh) $USER
```
Si no funciona, ejecuta 
```
sudo sh -c "echo $(which zsh) >> /etc/shells"
```
y repite.