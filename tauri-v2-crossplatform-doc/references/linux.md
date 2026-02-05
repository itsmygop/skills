# Tauri V2 Linux Platform Reference

## Table of Contents
- [System Dependencies](#system-dependencies)
- [Debian Package (.deb)](#debian-package-deb)
- [RPM Package](#rpm-package)
- [AppImage](#appimage)
- [AUR (Arch User Repository)](#aur-arch-user-repository)
- [Linux-Specific APIs](#linux-specific-apis)
- [Flatpak Support](#flatpak-support)

---

## System Dependencies

### Debian/Ubuntu

```bash
sudo apt update
sudo apt install libwebkit2gtk-4.1-dev \
  build-essential \
  curl \
  wget \
  file \
  libxdo-dev \
  libssl-dev \
  libayatana-appindicator3-dev \
  librsvg2-dev
```

### Arch Linux

```bash
sudo pacman -Syu
sudo pacman -S --needed \
  webkit2gtk-4.1 \
  base-devel \
  curl \
  wget \
  file \
  openssl \
  appmenu-gtk-module \
  libappindicator-gtk3 \
  librsvg \
  xdotool
```

### Fedora

```bash
sudo dnf check-update
sudo dnf install webkit2gtk4.1-devel \
  openssl-devel \
  curl \
  wget \
  file \
  libappindicator-gtk3-devel \
  librsvg2-devel \
  libxdo-devel
sudo dnf group install "c-development"
```

### openSUSE

```bash
sudo zypper up
sudo zypper in webkit2gtk3-devel \
  libopenssl-devel \
  curl \
  wget \
  file \
  libappindicator3-1 \
  librsvg-devel
sudo zypper in -t pattern devel_basis
```

### Gentoo

```bash
sudo emerge --ask \
  net-libs/webkit-gtk:4.1 \
  dev-libs/libappindicator \
  net-misc/curl \
  net-misc/wget \
  sys-apps/file
```

### Alpine Linux

```bash
sudo apk add \
  build-base \
  webkit2gtk-4.1-dev \
  curl \
  wget \
  file \
  openssl \
  libayatana-appindicator-dev \
  librsvg
```

### Fedora Silverblue (Immutable)

```bash
sudo rpm-ostree install webkit2gtk4.1-devel \
  openssl-devel \
  curl \
  wget \
  file \
  libappindicator-gtk3-devel \
  librsvg2-devel \
  libxdo-devel \
  gcc \
  gcc-c++ \
  make
sudo systemctl reboot
```

---

## Debian Package (.deb)

```json
{
  "bundle": {
    "linux": {
      "deb": {
        "depends": ["libwebkit2gtk-4.1-0", "libayatana-appindicator3-1"],
        "recommends": [],
        "provides": [],
        "conflicts": [],
        "replaces": [],
        "section": "utils",
        "priority": "optional",
        "files": {
          "/usr/share/doc/my-app/copyright": "./LICENSE"
        },
        "desktopTemplate": "./desktop-template.desktop"
      }
    }
  }
}
```

### Desktop Entry Template

```desktop
[Desktop Entry]
Name=My App
Comment=My Tauri Application
Exec=my-app %u
Icon=my-app
Terminal=false
Type=Application
Categories=Utility;
MimeType=x-scheme-handler/my-app;
StartupWMClass=my-app
```

---

## RPM Package

```json
{
  "bundle": {
    "linux": {
      "rpm": {
        "depends": ["webkit2gtk4.1", "libappindicator-gtk3"],
        "recommends": [],
        "provides": ["coolLib.rpm"],
        "conflicts": ["oldLib.rpm"],
        "obsoletes": ["veryoldLib.rpm"],
        "epoch": 0,
        "release": "1",
        "files": {
          "/usr/share/doc/my-app/copyright": "./LICENSE"
        },
        "desktopTemplate": "./desktop-template.desktop",
        "preInstallScript": "./scripts/pre-install.sh",
        "postInstallScript": "./scripts/post-install.sh",
        "preRemoveScript": "./scripts/pre-remove.sh",
        "postRemoveScript": "./scripts/post-remove.sh"
      }
    }
  }
}
```

### ARM Cross-Compilation

```bash
# ARMv7
sudo apt install libwebkit2gtk-4.1-dev:armhf

# ARM64
sudo apt install libwebkit2gtk-4.1-dev:arm64
```

---

## AppImage

```json
{
  "bundle": {
    "linux": {
      "appimage": {
        "bundleMediaFramework": false,
        "files": {
          "/usr/lib/libcustom.so": "./libs/libcustom.so"
        }
      }
    }
  }
}
```

| Option | Description |
|--------|-------------|
| `bundleMediaFramework` | Bundle GStreamer for media playback (increases size) |
| `files` | Additional files to include |

---

## AUR (Arch User Repository)

### PKGBUILD Example

```PKGBUILD
# Maintainer: Your Name <email@example.com>
pkgname=my-app
pkgver=1.0.0
pkgrel=1
pkgdesc="My Tauri Application"
arch=('x86_64' 'aarch64')
url="https://github.com/user/my-app"
license=('MIT')
depends=(
  'cairo'
  'desktop-file-utils'
  'gdk-pixbuf2'
  'glib2'
  'gtk3'
  'hicolor-icon-theme'
  'libsoup'
  'pango'
  'webkit2gtk-4.1'
)
options=('!strip' '!debug')
source_x86_64=("${url}/releases/download/v${pkgver}/my-app_${pkgver}_amd64.deb")
source_aarch64=("${url}/releases/download/v${pkgver}/my-app_${pkgver}_arm64.deb")
sha256sums_x86_64=('SKIP')
sha256sums_aarch64=('SKIP')

package() {
  tar -xvf data.tar.gz -C "${pkgdir}"
}
```

---

## Linux-Specific APIs

### Conditional Compilation

```rust
#[cfg(target_os = "linux")]
fn linux_only() {
    // Linux-specific code
}

// Also covers BSD variants
#[cfg(any(
    target_os = "linux",
    target_os = "dragonfly",
    target_os = "freebsd",
    target_os = "netbsd",
    target_os = "openbsd"
))]
fn unix_like() {
    // Linux and BSD code
}
```

### Parent Window (GTK)

```rust
#[cfg(any(target_os = "linux", target_os = "freebsd", ...))]
impl WindowBuilder {
    fn transient_for(self, parent: &impl gtk::glib::IsA<gtk::Window>) -> Self;
}
```

### WebView Related View

```rust
#[cfg(all(feature = "wry", target_os = "linux"))]
builder = builder.with_related_view(features.opener().webview.clone());
```

### libxdo for Clipboard

Enable in `Cargo.toml`:

```toml
[dependencies]
tauri = { version = "2.9.5", features = ["linux-libxdo"] }
```

This enables Cut, Copy, Paste, and Select All menu items on Linux.

---

## Flatpak Support

Tauri V2 uses `webkit2gtk-4.1` which enables Flatpak support (GNOME runtime uses webkit2gtk-4.1).

### WebKitGTK Migration

Tauri V2 migrated from `webkit2gtk-4.0` (soup2) to `webkit2gtk-4.1` (soup3):
- Fixes subtle bugs in soup2
- Enables Flatpak support
- No API changes required for most apps

---

## Complete Linux Bundle Config

```json
{
  "bundle": {
    "linux": {
      "appimage": {
        "bundleMediaFramework": false,
        "files": {}
      },
      "deb": {
        "depends": [],
        "recommends": [],
        "provides": [],
        "conflicts": [],
        "replaces": [],
        "files": {},
        "section": "utils",
        "priority": "optional"
      },
      "rpm": {
        "depends": [],
        "recommends": [],
        "provides": [],
        "conflicts": [],
        "obsoletes": [],
        "epoch": 0,
        "release": "1",
        "files": {}
      }
    }
  }
}
```

---

## GitHub Actions for Linux

```yaml
- name: Install dependencies (Ubuntu)
  if: matrix.platform == 'ubuntu-22.04'
  run: |
    sudo apt-get update
    sudo apt-get install -y libwebkit2gtk-4.1-dev \
      libappindicator3-dev \
      librsvg2-dev \
      patchelf
```

### ARM64 Linux Build

```yaml
- platform: 'ubuntu-22.04-arm'  # Only in public repos
  args: ''
```
