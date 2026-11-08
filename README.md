<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/3/3f/Fedora_logo.svg" alt="Fedora Logo" height="80" />
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  <img src="https://upload.wikimedia.org/wikipedia/commons/c/cb/GnomeLogoHorizontal.svg" alt="GNOME Logo" height="80" />
</p>

# Setup Guide: Fedora Linux 45 (Fullstack & Mobile)

**OS:** Fedora Linux 45 (GNOME 51.1 - Wayland) | **Secure Boot:** Disabled  
**Stack:** Java, Node.js, Python, Go, Rust, Flutter, Native Android

---

## Phase 1: DNF & Base Update (Crucial)

Configure DNF for maximum speed and ensure you are running the latest kernel before installing hardware drivers.

```bash
# 1. Optimize DNF Configuration
sudo bash -c 'cat <<EOF >> /etc/dnf/dnf.conf
max_parallel_downloads=10
fastestmirror=True
defaultyes=True
EOF'

# 2. Perform full system upgrade
sudo dnf update -y
```
> **⚠️ IMPORTANT:** Reboot your system now if the `kernel` package was updated before proceeding to Phase 2.

---

## Phase 2: Firmware & Repositories

### Firmware Updates (LVFS)
```bash
sudo fwupdmgr get-devices
sudo fwupdmgr refresh --force
sudo fwupdmgr get-updates
sudo fwupdmgr update
```
> **⚠️ Note:** Firmware updates require a reboot to take effect.

### RPM Fusion & Base Packages
RPM Fusion provides essential non-free drivers and multimedia codecs.
```bash
# Free & Non-Free Repositories
sudo dnf install -y \
  https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

# AppStream Metadata
sudo dnf install -y rpmfusion-free-appstream-data rpmfusion-nonfree-appstream-data
sudo dnf check-update

# Core Development Tools & Utilities
sudo dnf groupinstall "Development Tools" -y
sudo dnf install -y git curl wget gnupg2 flatpak fontconfig fzf gnome-keyring openssl p7zip p7zip-plugins unrar unzip
```

### Flathub Setup
```bash
# Remove limited Fedora Flatpak repo to prevent duplicate listings
flatpak remote-delete fedora 2>/dev/null || true

# Add complete Flathub repository
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

---

## Phase 3: NVIDIA & Graphics Drivers

### Prerequisites & Driver Installation
```bash
# Install kernel headers and dev packages
sudo dnf install -y kernel-devel kernel-headers gcc make dkms acpid libglvnd-glx libglvnd-opengl libglvnd-devel pkgconfig

# Install NVIDIA Drivers & CUDA (Secure Boot Disabled)
sudo dnf install -y akmod-nvidia xorg-x11-drv-nvidia-cuda
```
> **⏳ WAIT:** Mappings take 5-10 minutes to compile. Verify completion by executing:  
> `modinfo -F version nvidia`  
> *If a version number (e.g., 550.xx) returns, reboot your system.*

> **💡 Note on Hybrid Graphics:** On Wayland, the **Intel iGPU** renders the GNOME UI by default for battery savings. The RTX 3050 operates on-demand via PRIME Offload.

### Intel (Video Acceleration)
```bash
sudo dnf install -y mesa-vulkan-drivers vulkan-loader mesa-libGLU libva-utils intel-media-driver
```

---

## Phase 4: Full Multimedia & Codecs Stack

```bash
# 1. Swap to full FFmpeg from RPM Fusion
sudo dnf swap ffmpeg-free ffmpeg --allowerasing -y

# 2. Update Multimedia Group (GStreamer Plugins + Audio/Video Codecs)
# H.264, H.265, FLAC, FAAC, AAC, MP3 (LAME)
sudo dnf update @multimedia --setopt="install_weak_deps=False" --exclude=PackageKit-gstreamer-plugin -y
sudo dnf groupupdate sound-and-video -y
sudo dnf install -y gstreamer1-plugins-bad-free gstreamer1-plugins-ugly vlc

# 3. Hardware Acceleration (VA-API)
sudo dnf install -y ffmpeg-libs libva libva-utils libva-intel-driver libva-nvidia-driver
sudo dnf swap libva-intel-media-driver intel-media-driver --allowerasing -y

# 4. Firefox OpenH264 Support
sudo dnf install -y openh264 gstreamer1-plugin-openh264 mozilla-openh264
sudo dnf config-manager --set-enabled fedora-cisco-openh264
sudo dnf update -y
```

---

## Phase 5: Developer Fonts, Terminal & Shell Setup

### Developer & Microsoft Fonts
```bash
# System Repositories
sudo dnf install -y fira-code-fonts cascadia-code-fonts jetbrains-mono-fonts msttcore-fonts-installer

# Google Sans Suite (Local User Fonts)
mkdir -p ~/.local/share/fonts/google-fonts
cd ~/.local/share/fonts/google-fonts

wget -qO GoogleSans.zip "https://fonts.google.com/download?family=Google%20Sans"
wget -qO GoogleSansCode.zip "https://fonts.google.com/download?family=Google%20Sans%20Code"
wget -qO GoogleSansFlex.zip "https://fonts.google.com/download?family=Google%20Sans%20Flex"

unzip -qo GoogleSans.zip
unzip -qo GoogleSansCode.zip
unzip -qo GoogleSansFlex.zip
rm -f *.zip

fc-cache -fv
cd -
```

### Oh My Posh & Mise Setup
```bash
# Terminal Prompt (Oh My Posh)
curl -s https://ohmyposh.dev/install.sh | bash -s
echo 'eval "$(oh-my-posh init bash)"' >> ~/.bashrc

# Version Manager (Mise)
curl https://mise.run | sh
echo 'eval "$(~/.local/bin/mise activate bash)"' >> ~/.bashrc

# Starship Prompt (Alternative)
sudo dnf install -y starship
```

---

## Phase 6: Containers & Cloud Infrastructure

```bash
# Podman Container Suite
sudo dnf install -y podman podman-compose podman-docker
systemctl --user enable --now podman.socket

# Cloud & Kubernetes CLI Tools
sudo dnf install -y kubectl
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip -q awscliv2.zip
sudo ./aws/install && rm -rf aws awscliv2.zip

# Python & Native Build Dependencies
sudo dnf install -y gcc make zlib-devel bzip2-devel readline-devel sqlite-devel openssl-devel tk-devel libffi-devel xz-devel
```

---

## Phase 7: Programming Languages (Mise)

```bash
mise registry

# Global Runtime Versions (LTS prioritized)
mise use --global java@21      # LTS
mise use --global node@22      # LTS
mise use --global python@latest
mise use --global go@latest
mise use --global rust@latest

# Node Package Managers
corepack enable
```

---

## Phase 8: Mobile Ecosystem (Android & Flutter)

### KVM Virtualization & Flutter SDK
```bash
# KVM Hardware Acceleration
sudo dnf install -y qemu-kvm libvirt virt-manager
sudo systemctl enable --now libvirtd
sudo usermod -aG libvirt,kvm $USER

# Flutter SDK (Stable)
sudo dnf install -y clang cmake ninja-build pkgconf-pkg-config gtk3-devel
git clone https://github.com/flutter/flutter.git -b stable ~/.local/flutter
```

Add the following to your `~/.bashrc`:
```bash
export PATH="$PATH:$HOME/.local/flutter/bin"
export ANDROID_HOME="$HOME/Android/Sdk"
export PATH="$PATH:$ANDROID_HOME/emulator"
export PATH="$PATH:$ANDROID_HOME/platform-tools"
```
*Run `source ~/.bashrc` and execute `flutter doctor --android-licenses`.*

---

## Phase 9: Applications, IDEs & AppImages

### Primary Editors & Browsers
```bash
# Official Google Chrome & VSCode Repositories
sudo dnf install -y fedora-workstation-repositories
sudo dnf config-manager --set-enabled google-chrome
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'

sudo dnf install -y google-chrome-stable firefox code DBeaver
```

### AI Toolchain Setup
```bash
# OpenCode CLI & Desktop
curl -fsSL https://opencode.ai/v2/install | bash
sudo dnf install -y https://opencode.ai/update/api/latest/desktop/opencode/opencode-desktop-linux-x86_64.rpm

# Antigravity CLI
curl -fsSL https://antigravity.google/cli/install.sh | bash

# Antigravity IDE & Desktop Integration
mkdir -p ~/.local/opt/antigravity-ide ~/.local/opt/antigravity-app
wget -qO- "https://edgedl.me.gvt1.com/edgedl/release2/j0qc3/antigravity/stable/2.6.0-5526304964083712/linux-x64/Antigravity%20IDE.tar.gz" | tar -xz -C ~/.local/opt/antigravity-ide --strip-components=1
wget -qO- "https://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/linux-x64/Antigravity.tar.gz" | tar -xz -C ~/.local/opt/antigravity-app --strip-components=1

cat <<EOF > ~/.local/share/applications/antigravity-ide.desktop
[Desktop Entry]
Name=Antigravity IDE
Exec=$HOME/.local/opt/antigravity-ide/antigravity-ide %U
Terminal=false
Type=Application
Icon=$HOME/.local/opt/antigravity-ide/icon.png
Categories=Development;IDE;
EOF

cat <<EOF > ~/.local/share/applications/antigravity-app.desktop
[Desktop Entry]
Name=Antigravity Desktop
Exec=$HOME/.local/opt/antigravity-app/antigravity-app %U
Terminal=false
Type=Application
Icon=$HOME/.local/opt/antigravity-app/icon.png
Categories=Utility;Development;
EOF
```

### Zed Editor & AppImages (Obsidian, Bitwarden, Postman, Insomnia, Draw.io, Typora)
```bash
# Zed Editor
curl -f https://zed.dev/install.sh | sh

# AppImage Infrastructure (FUSE & Gearlever)
sudo dnf install -y fuse fuse-libs
flatpak install -y flathub it.mijorus.gearlever

# Bitwarden CLI
sudo dnf install -y bitwarden-cli 2>/dev/null || npm install -g @bitwarden/cli

# Directory for AppImages
mkdir -p ~/.local/bin ~/.local/opt
```

*  **JetBrains Toolbox (Android Studio & IntelliJ):** [Download Toolbox](https://www.jetbrains.com/toolbox-app/)
*  **Cursor IDE (.AppImage):** [Download Cursor](https://cursor.com/download)

---

## Phase 10: GNOME Desktop & Ptyxis Customizations

### GNOME Behavior & Interface Settings
```bash
# Enable Dark Mode & Night Light (8 PM - 8 AM)
gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'
gsettings set org.gnome.settings-daemon.plugins.color night-light-enabled true

# Window Buttons & Fixed Workspaces (3 Workspaces)
gsettings set org.gnome.desktop.wm.preferences button-layout 'appmenu:minimize,maximize,close'
gsettings set org.gnome.mutter dynamic-workspaces false
gsettings set org.gnome.desktop.wm.preferences num-workspaces 3

# UI Animation Removal for Maximum Speed
gsettings set org.gnome.desktop.interface enable-animations false

# Visual Feedback on Sudo Password Entry (Asterisks)
echo 'Defaults pwfeedback' | sudo tee /etc/sudoers.d/pwfeedback

# GNOME Tweaks & Extension Manager
sudo dnf install -y gnome-tweaks
flatpak install -y flathub com.mattjakeman.ExtensionManager
```

### Ptyxis Terminal Tuning
```bash
# Custom Keybinding (Super+T for Ptyxis)
gsettings set org.gnome.settings-daemon.plugins.media-keys custom-keybindings "['/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/']"
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ name 'Terminal'
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ command 'ptyxis'
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ binding '<Super>t'
```

---

## Phase 11: Advanced Kernel & I/O Optimizations

### 1. Kernel Tuning (`sysctl`)
```bash
sudo bash -c 'cat <<EOF > /etc/sysctl.d/99-developer-performance.conf
# Favor RAM over ZRAM swap
vm.swappiness=10

# Increase VFS cache pressure for SSDs
vm.vfs_cache_pressure=50

# Lower dirty page writeback latency
vm.dirty_ratio=10
vm.dirty_background_ratio=3

# Increase inotify file watch limit for IDEs (VSCode/IntelliJ)
fs.inotify.max_user_watches=524288

# TCP Backlog Tuning for Containers
net.core.somaxconn=1024
EOF'

sudo sysctl --system
```

### 2. I/O Schedulers (Udev Rules)
```bash
sudo bash -c 'cat <<EOF > /etc/udev/rules.d/60-ioschedulers.rules
# NVMe Drives (Hardware queuing)
ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/scheduler}="none"

# SATA SSDs (Low latency queuing)
ACTION=="add|change", KERNEL=="sd[a-z]|mmcblk[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="mq-deadline"

# Mechanical HDDs (Budget Fair Queueing)
ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/rotational}=="1", ATTR{queue/scheduler}="bfq"
EOF'

sudo udevadm control --reload-rules && sudo udevadm trigger
```

### 3. Journal Management & SSD TRIM
```bash
# Restrict Systemd Journal Logs to 100MB
sudo bash -c 'cat <<EOF > /etc/systemd/journald.conf.d/size-limit.conf
[Journal]
SystemMaxUse=100M
SystemMaxFiles=5
EOF'

# Enable Weekly & Instant SSD TRIM
sudo systemctl enable --now fstrim.timer
sudo fstrim -av
```

### 4. Disable Unnecessary Startup Services (Saves 900MB RAM)
```bash
# Disable NetworkManager Wait Online (Faster boot)
sudo systemctl disable NetworkManager-wait-online.service

# Prevent GNOME Software Background Autostart
mkdir -p ~/.config/autostart
cp /usr/share/applications/org.gnome.Software.desktop ~/.config/autostart/
echo "X-GNOME-Autostart-enabled=false" >> ~/.config/autostart/org.gnome.Software.desktop
dconf write /org/gnome/desktop/search-providers/disabled "['org.gnome.Software.desktop']"
```

---

## Phase 12: System Cleanup

Clean DNF package cache and remove orphaned dependencies:
```bash
sudo dnf clean all
sudo dnf autoremove -y
```

---

## 🔥 Bonus: Force NVIDIA GPU on Android Emulator

1. In Android Studio, set your Emulator AVD **Graphics** to `Hardware - GLES 2.0`.
2. Add this alias to your `~/.bashrc`:
   ```bash
   alias emulador-nvidia='__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia emulator'
   ```
   *Usage:* `emulador-nvidia -avd YOUR_AVD_NAME`
