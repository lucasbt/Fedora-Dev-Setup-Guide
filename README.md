# Setup Guide: Fedora Linux 45 (Fullstack & Mobile)

**OS:** Fedora Linux 45 (GNOME 51.1 - Wayland) | **Secure Boot:** Disabled

**Stack:** Java, Node.js, Python, Go, Rust, Flutter, Native Android

## Phase 1: DNF Optimization and Base Update (Crucial)

Before updating the system, let's optimize the package manager for faster downloads by using the fastest mirrors and allowing parallel downloads.

```bash
# Optimize DNF speed
echo -e "max_parallel_downloads=10\nfastestmirror=True" | sudo tee -a /etc/dnf/dnf.conf

# Update the entire system
sudo dnf upgrade --refresh -y
```

> **⚠️ IMPORTANT:** If the command above updated the `kernel` package, **RESTART YOUR COMPUTER NOW** before proceeding to Phase 2.

## Phase 2: Firmware and Repositories

```bash
# Update BIOS and hardware firmware (LVFS)
fwupdmgr refresh --force
fwupdmgr get-updates
fwupdmgr update

# Install RPM Fusion (Free and Non-Free)
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm -y

# Install basic development tools
sudo dnf groupinstall "Development Tools" -y

# Enable Flathub (Global and updated Flatpak repository)
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

## Phase 3: NVIDIA and CUDA

Proprietary driver installation for the RTX 3050 (essential for emulators and local AI).

```bash
# Install drivers and CUDA support
sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda -y
```

> **⏳ WAIT:** The module compilation happens in the background. Wait about 5 to 10 minutes and run the command below:

```bash
modinfo -F version nvidia
```

*If the command returns the driver version (e.g., 550.xx), the process finished successfully. **Restart your computer again** to load the UI using the NVIDIA GPU.*

## Phase 4: System Optimizations, Codecs, and AppImages

Tweaks to save battery life, speed up the UI, improve display compatibility, and clean up the system.

```bash
# 1. AppImage Support (Mandatory to run Cursor IDE)
sudo dnf install fuse-libs -y

# 2. Multimedia Codecs and Video Acceleration (Intel i5)
sudo dnf swap 'ffmpeg-free' 'ffmpeg' --allowerasing -y
sudo dnf update @multimedia --setopt="install_weak_deps=False" --exclude=PackageKit-gstreamer-plugin -y
sudo dnf group install -y sound-and-video
sudo dnf install ffmpeg-libs libva libva-utils openh264 gstreamer1-plugin-openh264 mozilla-openh264 -y
sudo dnf swap libva-intel-media-driver intel-media-driver --allowerasing -y
sudo dnf install libva-intel-driver -y
sudo dnf config-manager setopt fedora-cisco-openh264.enabled=1

# 3. Disable GNOME animations for maximum fluidity
gsettings set org.gnome.desktop.interface enable-animations false

# 4. Enable Fractional Scaling on Wayland (Ideal for external monitors)
gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"

# 5. Speed up boot time by disabling network wait
sudo systemctl disable NetworkManager-wait-online.service
```

**Disable GNOME Software on Autostart (Saves up to 900MB of RAM):**

```bash
mkdir -p ~/.config/autostart
cp /usr/share/applications/org.gnome.Software.desktop ~/.config/autostart/
echo "X-GNOME-Autostart-enabled=false" >> ~/.config/autostart/org.gnome.Software.desktop
dconf write /org/gnome/desktop/search-providers/disabled "['org.gnome.Software.desktop']"
```

## Phase 5: Developer Fonts

Install the best fonts with ligature support for your IDEs and Terminal:

```bash
sudo dnf install fira-code-fonts cascadia-code-fonts jetbrains-mono-fonts -y
```

## Phase 6: Utilities and Terminal Core

```bash
# Essential utilities
sudo dnf install curl wget htop btop git jq -y

# Install Oh My Posh (Ultra-fast prompt)
curl -s https://ohmyposh.dev/install.sh | bash -s
echo 'eval "$(oh-my-posh init bash)"' >> ~/.bashrc

# Install Mise (Unified version/language manager)
curl https://mise.run | sh
echo 'eval "$(~/.local/bin/mise activate bash)"' >> ~/.bashrc
```

*(Apply the changes by running `source ~/.bashrc`)*

## Phase 7: Containers and Python Compilation

```bash
# Podman ecosystem (native)
sudo dnf install podman podman-compose podman-docker -y
systemctl --user enable --now podman.socket

# Heavy dependencies for local compilation (Essential for Python via mise)
sudo dnf install gcc make zlib-devel bzip2-devel readline-devel sqlite-devel openssl-devel tk-devel libffi-devel xz-devel -y
```

## Phase 8: Programming Languages (Mise)

```bash
mise registry

# Install language versions
mise use --global java@21
mise use --global node@22
mise use --global python@latest
mise use --global go@latest
mise use --global rust@latest

# Enable yarn and pnpm natively via Node
corepack enable
```

## Phase 9: Mobile Ecosystem (Android and Flutter)

**1. KVM (Acceleration for Android Emulator)**

```bash
sudo dnf install qemu-kvm libvirt virt-manager -y
sudo systemctl enable --now libvirtd
sudo usermod -aG libvirt,kvm $USER
```

*(Log out and log back in to apply the group changes)*

**2. Flutter SDK**

```bash
sudo dnf install clang cmake ninja-build pkgconf-pkg-config gtk3-devel -y
git clone https://github.com/flutter/flutter.git -b stable ~/.local/flutter
```

**3. Environment Variables**
Add this to your `~/.bashrc`:

```bash
# Mobile PATHs
export PATH="$PATH:$HOME/.local/flutter/bin"
export ANDROID_HOME="$HOME/Android/Sdk"
export PATH="$PATH:$ANDROID_HOME/emulator"
export PATH="$PATH:$ANDROID_HOME/platform-tools"
```

Run `source ~/.bashrc` and then accept the licenses with `flutter doctor --android-licenses`.

## Phase 10: IDEs, Browsers, and AI

**Installation via DNF (Official Repositories)**

```bash
sudo dnf install fedora-workstation-repositories -y
sudo dnf config-manager --set-enabled google-chrome
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'

sudo dnf install google-chrome-stable firefox code -y
```

**Manual Tools / AI:**

* **JetBrains Toolbox (Android Studio and IntelliJ IDEA):** [Download](https://www.jetbrains.com/toolbox-app/)
* **Cursor IDE (.AppImage):** [Download](https://cursor.com/download)
* **OpenCode CLI:** `npm install -g @opencode/cli`
* **Antigravity:** Download the official desktop/CLI, move the executable to `~/.local/bin/` and run `chmod +x`.

## 🔥 Bonus: Force NVIDIA on the Emulator (Prime Offload)

To make the emulator run smoothly using the RTX 3050 instead of Intel Graphics:

1. In Android Studio, edit the Emulator and change **Graphics** to `Hardware - GLES 2.0`.
2. Add this alias to your `~/.bashrc` to launch standalone emulators from the terminal using the dedicated GPU:

   ```bash
   alias emulador-nvidia='__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia emulator'
   ```

   *Usage:* `emulador-nvidia -avd YOUR_AVD_NAME`
