<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/3/3f/Fedora_logo.svg" alt="Fedora Logo" height="80" />
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  <img src="https://upload.wikimedia.org/wikipedia/commons/c/cb/GnomeLogoHorizontal.svg" alt="GNOME Logo" height="80" />
</p>

# Setup Guide: Fedora Linux 45 (Fullstack & Mobile)

**Hardware:** Acer Nitro AN515-58 | i5-12450H | 32GB RAM | RTX 3050  
**OS:** Fedora Linux 45 (GNOME 51.1 - Wayland) | **Secure Boot:** Disabled  
**Stack:** Java, Node.js, Python, Go, Rust, Flutter, Native Android  

---

## Phase 1: DNF Optimization and Base Update (Crucial)

Before updating the system, let's optimize the package manager for faster downloads by using the fastest mirrors and allowing parallel downloads.

```bash
# Optimize DNF speed
echo -e "max_parallel_downloads=10\nfastestmirror=True" | sudo tee -a /etc/dnf/dnf.conf

# Update everything because fresh installs always have outdated packages
sudo dnf upgrade --refresh -y
```

> **⚠️ IMPORTANT:** It is highly recommended to restart after this initial update, especially if the `kernel` package was updated.
```bash
sudo reboot
```

---

## Phase 2: Firmware and Repositories

Fedora ships pretty bare-bones because of legal reasons. RPM Fusion is where all the actually useful stuff lives (codecs, drivers, etc.).

```bash
# See what hardware can be updated
sudo fwupdmgr get-devices

# Refresh the firmware database, check for updates, and apply them
sudo fwupdmgr refresh --force
sudo fwupdmgr get-updates
sudo fwupdmgr update
```
> **⚠️ Note:** After firmware updates, you need a reboot.

```bash
# Get the free and nonfree repositories (NVIDIA drivers, some codecs)
sudo dnf install -y https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

# Install AppStream metadata
sudo dnf install rpmfusion-free-appstream-data rpmfusion-nonfree-appstream-data -y
sudo dnf check-update

# Remove the limited Fedora flatpak repo to prevent conflicts
flatpak remote-delete fedora

# Get access to everything Flathub (Full Repository)
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

---

## Phase 3: NVIDIA, CUDA, and Video Acceleration

Install prerequisites and drivers for your RTX 3050 (essential for emulators and local AI), plus Vulkan and Intel acceleration.

```bash
# Install kernel headers and dev tools (Prerequisites)
sudo dnf install kernel-devel kernel-headers gcc make dkms acpid libglvnd-glx libglvnd-opengl libglvnd-devel pkgconfig -y

# Install NVIDIA drivers and CUDA support
sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda -y

# Core Packages (AMD & Intel): Vulkan and basic acceleration
sudo dnf install mesa-vulkan-drivers vulkan-loader mesa-libGLU libva-utils -y
```

> **⏳ WAIT:** The NVIDIA module compilation happens in the background. Wait about 5 to 10 minutes and run the command below:
```bash
modinfo -F version nvidia
```
*If the command returns the driver version (e.g., 550.xx), the process finished successfully. **Restart your computer again** to load the UI using the NVIDIA GPU.*

---

## Phase 4: System Optimizations, Codecs, and Useful Stuff

Tweaks to save battery life, speed up the UI, install missing codecs, and add core utilities.

```bash
# 1. Archive Support (For extracting .rar, .7z, etc.)
sudo dnf install -y p7zip p7zip-plugins unrar

# 2. AppImage Support (Mandatory to run Cursor IDE)
sudo dnf install -y fuse fuse-libs
flatpak install -y flathub it.mijorus.gearlever # Optional: AppImage manager

# 3. GNOME Utilities
sudo dnf install -y gnome-tweaks
flatpak install -y flathub com.mattjakeman.ExtensionManager

# 4. Multimedia Codecs and Video Acceleration (Intel i5 & NVIDIA VA-API)
sudo dnf swap 'ffmpeg-free' 'ffmpeg' --allowerasing -y
sudo dnf update @multimedia --setopt="install_weak_deps=False" --exclude=PackageKit-gstreamer-plugin -y
sudo dnf groupupdate sound-and-video -y
sudo dnf install -y ffmpeg-libs libva libva-utils libva-nvidia-driver
sudo dnf swap libva-intel-media-driver intel-media-driver --allowerasing -y
sudo dnf install -y libva-intel-driver

# 5. Firefox Video Fix (Cisco Codec for H.264)
sudo dnf install -y openh264 gstreamer1-plugin-openh264 mozilla-openh264
sudo dnf config-manager --set-enabled fedora-cisco-openh264
sudo dnf update -y
```
> **⚠️ Important:** Restart Firefox and check that the OpenH264 plugin is enabled in `about:addons`.

```bash
# 6. Disable GNOME animations for maximum fluidity
gsettings set org.gnome.desktop.interface enable-animations false

# 7. Enable Fractional Scaling on Wayland (Ideal for external monitors)
gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"

# 8. Speed up boot time (by ~15s-20s) by disabling network wait
sudo systemctl disable NetworkManager-wait-online.service
```

**Disable GNOME Software on Autostart (Saves up to 900MB of RAM):**
```bash
mkdir -p ~/.config/autostart
cp /usr/share/applications/org.gnome.Software.desktop ~/.config/autostart/
echo "X-GNOME-Autostart-enabled=false" >> ~/.config/autostart/org.gnome.Software.desktop
dconf write /org/gnome/desktop/search-providers/disabled "['org.gnome.Software.desktop']"
```

---

## Phase 5: Developer Fonts

Install the best fonts with ligature support for your IDEs and Terminal:

```bash
sudo dnf install fira-code-fonts cascadia-code-fonts jetbrains-mono-fonts -y
```

---

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

---

## Phase 7: Containers and Python Compilation

```bash
# Podman ecosystem (native)
sudo dnf install podman podman-compose podman-docker -y
systemctl --user enable --now podman.socket

# Heavy dependencies for local compilation (Essential for Python via mise)
sudo dnf install gcc make zlib-devel bzip2-devel readline-devel sqlite-devel openssl-devel tk-devel libffi-devel xz-devel -y
```

---

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

---

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

---

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
*   **JetBrains Toolbox (Android Studio and IntelliJ IDEA):** [Download](https://www.jetbrains.com/toolbox-app/)
*   **Cursor IDE (.AppImage):** [Download](https://cursor.com/download) - *You can use Gearlever (installed in Phase 4) to manage it easily!*
*   **OpenCode CLI:** `npm install -g @opencode/cli`
*   **Antigravity:** Download the official desktop/CLI, move the executable to `~/.local/bin/` and run `chmod +x`.

---

## Phase 11: System Cleanup

After installing everything, it's a good idea to clean up residual files and orphaned packages to keep your system lean.

```bash
# Clean package cache and remove orphaned packages
sudo dnf clean all
sudo dnf autoremove -y
```

---

## 🔥 Bonus: Force NVIDIA on the Emulator (Prime Offload)

To make the emulator run smoothly using the RTX 3050 instead of Intel Graphics:

1.  In Android Studio, edit the Emulator and change **Graphics** to `Hardware - GLES 2.0`.
2.  Add this alias to your `~/.bashrc` to launch standalone emulators from the terminal using the dedicated GPU:
    ```bash
    alias emulador-nvidia='__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia emulator'
    ```
    *Usage:* `emulador-nvidia -avd YOUR_AVD_NAME`
