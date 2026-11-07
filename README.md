<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/3/3f/Fedora_logo.svg" alt="Fedora Logo" height="80" />
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  <img src="https://upload.wikimedia.org/wikipedia/commons/c/cb/GnomeLogoHorizontal.svg" alt="GNOME Logo" height="80" />
</p>

# Setup Guide: Fedora Linux 45 (Fullstack & Mobile)

**OS:** Fedora Linux 45 (GNOME 51.1 - Wayland) | **Secure Boot:** Disabled

**Stack:** Java, Node.js, Python, Go, Rust, Flutter, Native Android

---

## Phase 1: Base Update (Crucial)

We need to update because fresh installs always have outdated packages. Ensure you are running the latest kernel before installing any drivers.

```bash
# Update everything
sudo dnf update -y
```
> **⚠️ IMPORTANT:** It is recommended to reboot after updating, especially if the kernel was updated.

---

## Phase 2: Firmware & Repositories

### Firmware Updates
Your hardware probably has newer firmware available. This actually matters for things like WiFi and battery life.
```bash
# See what can be updated
sudo fwupdmgr get-devices
# Refresh the firmware database
sudo fwupdmgr refresh --force
# Check for updates
sudo fwupdmgr get-updates
# Apply them
sudo fwupdmgr update
```
> **⚠️ Note:** After firmware updates, you need a reboot.

### RPM Fusion
Fedora ships pretty bare-bones because of legal reasons. RPM Fusion is where all the actually useful stuff lives.
```bash
# Get the free repository
sudo dnf install -y https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm

# Get the nonfree repository (NVIDIA drivers, some codecs)
sudo dnf install -y https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

# Install AppStream metadata
sudo dnf install -y rpmfusion-free-appstream-data rpmfusion-nonfree-appstream-data
sudo dnf check-update

# Install basic compile tools
sudo dnf groupinstall "Development Tools" -y
```

### Flathub Setup
Fedora comes with a neutered version of Flatpak. Flathub is where the actual apps are.
```bash
# Remove the limited Fedora repo to prevent conflicts
flatpak remote-delete fedora

# Get access to the full Flathub repository
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

---

## Phase 3: NVIDIA & Graphics Drivers

### Prerequisites
Install kernel headers and dev tools:
```bash
sudo dnf install -y kernel-devel kernel-headers gcc make dkms acpid libglvnd-glx libglvnd-opengl libglvnd-devel pkgconfig
```

### NVIDIA Drivers (Secure Boot Disabled)
NVIDIA drivers on Fedora are best installed via RPM Fusion's `akmod-nvidia`.
```bash
# Install drivers and CUDA support (Crucial for AI and Emulators)
sudo dnf install -y akmod-nvidia xorg-x11-drv-nvidia-cuda
```
> **⏳ WAIT:** The module compilation happens in the background. Wait 5-10 minutes and run:
> `modinfo -F version nvidia`
> *If it returns the driver version (e.g., 550.xx), it was successful. **Reboot your system.** *

> **💡 Note on Hybrid Graphics:** On Wayland, the **Intel integrated GPU** takes care of rendering the GNOME UI by default, maximizing battery life. The NVIDIA GPU stays suspended and only kicks in on-demand for heavy tasks.

### Intel (Video Acceleration)
Make sure media works optimally on the integrated Intel i5 processor.
```bash
sudo dnf install -y mesa-vulkan-drivers vulkan-loader mesa-libGLU libva-utils
# Intel video acceleration (for newer Intel GPUs)
sudo dnf install -y intel-media-driver
```

---

## Phase 4: Media, Archives & Optimizations

### Video Codecs
Fedora ships with basically no codecs because of patent issues.
```bash
# Swap to full FFmpeg
sudo dnf swap ffmpeg-free ffmpeg --allowerasing -y

# Update multimedia group (GStreamer full + codecs)
sudo dnf update @multimedia --setopt="install_weak_deps=False" --exclude=PackageKit-gstreamer-plugin -y

# Optional sound/video group
sudo dnf groupupdate sound-and-video -y
```

### Hardware Acceleration (VA-API)
```bash
sudo dnf install -y ffmpeg-libs libva libva-utils
sudo dnf swap libva-intel-media-driver intel-media-driver --allowerasing -y
sudo dnf install -y libva-intel-driver
# NVIDIA specific VA-API
sudo dnf install -y libva-nvidia-driver
```

### Firefox Video Fix
```bash
sudo dnf install -y openh264 gstreamer1-plugin-openh264 mozilla-openh264
sudo dnf config-manager --set-enabled fedora-cisco-openh264
sudo dnf update -y
```
> **⚠️ Important:** Restart Firefox and check that the OpenH264 plugin is enabled in `about:addons`.

### Archives & AppImages
```bash
# Archive utilities
sudo dnf install -y unzip p7zip p7zip-plugins unrar

# Install FUSE (Required for AppImages like Cursor)
sudo dnf install -y fuse fuse-libs

# Optional: AppImage manager (Gearlever)
flatpak install -y flathub it.mijorus.gearlever
```

### System Optimizations
```bash
# Disable UI Animations for maximum speed
gsettings set org.gnome.desktop.interface enable-animations false

# Disable NetworkManager-wait-online.service (Faster boot)
sudo systemctl disable NetworkManager-wait-online.service
```
**Disable GNOME Software Autostart (Saves RAM):**
```bash
mkdir -p ~/.config/autostart
cp /usr/share/applications/org.gnome.Software.desktop ~/.config/autostart/
echo "X-GNOME-Autostart-enabled=false" >> ~/.config/autostart/org.gnome.Software.desktop
dconf write /org/gnome/desktop/search-providers/disabled "['org.gnome.Software.desktop']"
```

---

## Phase 5: Utilities & GNOME Tweaks

```bash
# Vital terminal utilities
sudo dnf install -y curl wget htop btop git jq

# GNOME Tweaks & Extension Manager
sudo dnf install -y gnome-tweaks
flatpak install -y flathub com.mattjakeman.ExtensionManager
```

### Developer Fonts
```bash
# 1. Install Fedora Repository Fonts
sudo dnf install -y fira-code-fonts cascadia-code-fonts jetbrains-mono-fonts

# 2. Download and Install Google Fonts directly to local user
mkdir -p ~/.local/share/fonts/google-fonts
cd ~/.local/share/fonts/google-fonts

wget -qO GoogleSans.zip "https://fonts.google.com/download?family=Google%20Sans"
wget -qO GoogleSansCode.zip "https://fonts.google.com/download?family=Google%20Sans%20Code"
wget -qO GoogleSansFlex.zip "https://fonts.google.com/download?family=Google%20Sans%20Flex"

unzip -qo GoogleSans.zip
unzip -qo GoogleSansCode.zip
unzip -qo GoogleSansFlex.zip
rm -f *.zip

# Refresh Font Cache
fc-cache -fv
cd -
```

### Oh My Posh & Mise
```bash
# Oh My Posh (Ultra-fast prompt)
curl -s https://ohmyposh.dev/install.sh | bash -s
echo 'eval "$(oh-my-posh init bash)"' >> ~/.bashrc

# Mise (Environment & Tool Manager)
curl https://mise.run | sh
echo 'eval "$(~/.local/bin/mise activate bash)"' >> ~/.bashrc
```

---

## Phase 6: Containers & Python Build Dependencies

```bash
# Native Podman ecosystem
sudo dnf install -y podman podman-compose podman-docker
systemctl --user enable --now podman.socket

# Heavy dependencies for local compilation (Crucial for Python via mise)
sudo dnf install -y gcc make zlib-devel bzip2-devel readline-devel sqlite-devel openssl-devel tk-devel libffi-devel xz-devel
```

---

## Phase 7: Programming Languages (Mise)

Install the LTS versions (or latest stable if LTS doesn't apply) for your stack.
```bash
mise registry

# Install global versions
mise use --global java@21      # LTS
mise use --global node@22      # LTS
mise use --global python@latest
mise use --global go@latest
mise use --global rust@latest

# Enable yarn and pnpm natively
corepack enable
```

---

## Phase 8: Mobile Ecosystem (Android & Flutter)

### 1. KVM (Android Emulator Acceleration)
```bash
sudo dnf install -y qemu-kvm libvirt virt-manager
sudo systemctl enable --now libvirtd
sudo usermod -aG libvirt,kvm $USER
```
*(You must log out and log back in for group changes to take effect)*

### 2. Flutter SDK
```bash
sudo dnf install -y clang cmake ninja-build pkgconf-pkg-config gtk3-devel
git clone https://github.com/flutter/flutter.git -b stable ~/.local/flutter
```

### 3. Environment Variables
Add these to your `~/.bashrc`:
```bash
# Mobile PATHs
export PATH="$PATH:$HOME/.local/flutter/bin"
export ANDROID_HOME="$HOME/Android/Sdk"
export PATH="$PATH:$ANDROID_HOME/emulator"
export PATH="$PATH:$ANDROID_HOME/platform-tools"
```
Run `source ~/.bashrc`, then accept licenses using `flutter doctor --android-licenses`.

---

## Phase 9: IDEs, Browsers & AI Tools

### Browsers and VS Code
```bash
sudo dnf install -y fedora-workstation-repositories
sudo dnf config-manager --set-enabled google-chrome
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'

sudo dnf install -y google-chrome-stable firefox code
```

### OpenCode
**CLI:**
```bash
curl -fsSL https://opencode.ai/v2/install | bash
```
**Desktop App:**
```bash
sudo dnf install -y https://opencode.ai/update/api/latest/desktop/opencode/opencode-desktop-linux-x86_64.rpm
```

### Antigravity
**CLI:**
```bash
curl -fsSL https://antigravity.google/cli/install.sh | bash
```
**IDE & App (Installed natively via .desktop entries):**
```bash
mkdir -p ~/.local/opt/antigravity-ide ~/.local/opt/antigravity-app

# Extract IDE
wget -qO- "https://edgedl.me.gvt1.com/edgedl/release2/j0qc3/antigravity/stable/2.6.0-5526304964083712/linux-x64/Antigravity%20IDE.tar.gz" | tar -xz -C ~/.local/opt/antigravity-ide --strip-components=1

# Extract Desktop App
wget -qO- "https://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/linux-x64/Antigravity.tar.gz" | tar -xz -C ~/.local/opt/antigravity-app --strip-components=1

# Create GNOME App Launcher entries
cat <<EOF> ~/.local/share/applications/antigravity-ide.desktop
[Desktop Entry]
Name=Antigravity IDE
Exec=$HOME/.local/opt/antigravity-ide/antigravity-ide %U
Terminal=false
Type=Application
Icon=$HOME/.local/opt/antigravity-ide/icon.png
Categories=Development;IDE;
EOF

cat <<EOF> ~/.local/share/applications/antigravity-app.desktop
[Desktop Entry]
Name=Antigravity Desktop
Exec=$HOME/.local/opt/antigravity-app/antigravity-app %U
Terminal=false
Type=Application
Icon=$HOME/.local/opt/antigravity-app/icon.png
Categories=Utility;Development;
EOF

update-desktop-database ~/.local/share/applications/
```

### JetBrains & Cursor
*   **JetBrains Toolbox (Android Studio & IntelliJ):** [Download](https://www.jetbrains.com/toolbox-app/)
*   **Cursor IDE (.AppImage):** [Download](https://cursor.com/download) *(Use Gearlever to manage AppImages)*

---

## Phase 10: System Cleanup

Clean package cache and remove orphaned packages:
```bash
sudo dnf clean all
sudo dnf autoremove -y
```

---

## 🔥 Bonus: Force NVIDIA on Android Emulator (Prime Offload)

To ensure your Android emulator runs smoothly using the RTX 3050 instead of Intel Graphics:
1. In Android Studio, edit the Emulator AVD and set **Graphics** to `Hardware - GLES 2.0`.
2. Add this alias to your `~/.bashrc` to launch the emulator directly from the terminal using the dedicated GPU:
   ```bash
   alias emulador-nvidia='__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia emulator'
   ```
   *Usage:* `emulador-nvidia -avd YOUR_AVD_NAME`
