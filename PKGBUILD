pkgname=mullvad-browser
pkgver=15.0.24
pkgrel=3
pkgdesc="Privacy-focused browser developed by Mullvad VPN and the Tor Project"
arch=('x86_64')
url="https://mullvad.net/browser"
license=('MPL-2.0')

depends=(
  'alsa-lib'
  'at-spi2-core'
  'bash'
  'cairo'
  'dbus'
  'fontconfig'
  'freetype2'
  'gcc-libs'
  'gdk-pixbuf2'
  'glib2'
  'glibc'
  'gtk3'
  'hicolor-icon-theme'
  'libpulse'
  'libx11'
  'libxcb'
  'libxcomposite'
  'libxcursor'
  'libxdamage'
  'libxext'
  'libxfixes'
  'libxi'
  'libxrandr'
  'libxrender'
  'libxss'
  'libxt'
  'mime-types'
  'nspr'
  'nss'
  'pango'
)

optdepends=(
  'xdg-desktop-portal: screen sharing support on Wayland'
)

conflicts=('mullvad-browser-bin')
options=('!strip')

source=(
  "https://github.com/mullvad/mullvad-browser/releases/download/${pkgver}/mullvad-browser-linux-x86_64-${pkgver}.tar.xz"
  "https://github.com/mullvad/mullvad-browser/releases/download/${pkgver}/mullvad-browser-linux-x86_64-${pkgver}.tar.xz.asc"
)

sha256sums=(
  'bca565676b7d4bae80650f367d4c3e72c450b4cbce24bac96db7ad4b7dc56fea'
  'b1b98dfed868a4ce152bcfc16d40e904ba600159c15b379cf53efcdad87cd2c6'
)

validpgpkeys=(
  'EF6E286DDA85EA2A4BA7DE684E2C6E8793298290'
)

package() {
  local appsrc="${srcdir}/mullvad-browser/Browser"
  local appdir="${pkgdir}/opt/mullvad-browser"

  if [[ ! -f "${appsrc}/mullvadbrowser" ]]; then
    error "Mullvad Browser executable not found in ${appsrc}"
    return 1
  fi

  # Make directories traversable and files readable by normal users.
  find "${appsrc}" -type d -exec chmod 755 {} +
  find "${appsrc}" -type f -perm /111 -exec chmod 755 {} +
  find "${appsrc}" -type f ! -perm /111 -exec chmod 644 {} +

  # Install the contents of Browser directly into /opt/mullvad-browser.
  install -d "${appdir}"
  cp -a "${appsrc}/." "${appdir}/"

  # Use a per-user browser profile and disable built-in self-updates.
  install -Dm644 /dev/null "${appdir}/system-install"
  install -Dm644 /dev/null "${appdir}/is-packaged-app"

  # Configure bundled fonts independently of the working directory.
  if [[ -f "${appdir}/fonts/fonts.conf" ]]; then
    sed -i \
      's|<dir prefix="cwd">fonts</dir>|<dir>/opt/mullvad-browser/fonts</dir>|' \
      "${appdir}/fonts/fonts.conf"
  fi

  # Install the command-line launcher.
  install -d "${pkgdir}/usr/bin"

  cat > "${pkgdir}/usr/bin/mullvad-browser" <<'LAUNCHER'
#!/bin/sh
unset SESSION_MANAGER
export FONTCONFIG_PATH="/opt/mullvad-browser/fonts"
export FONTCONFIG_FILE="fonts.conf"
export __GL_SHADER_DISK_CACHE=0
exec /opt/mullvad-browser/mullvadbrowser "$@"
LAUNCHER

  chmod 755 "${pkgdir}/usr/bin/mullvad-browser"

  # Install the desktop application entry.
  install -Dm644 /dev/stdin \
    "${pkgdir}/usr/share/applications/mullvad-browser.desktop" <<'DESKTOP'
[Desktop Entry]
Version=1.0
Type=Application
Name=Mullvad Browser
Comment=Privacy-focused web browser
Exec=mullvad-browser %u
Icon=mullvad-browser
Terminal=false
Categories=Network;WebBrowser;Security;
StartupNotify=true
DESKTOP

  # Install available application icons.
  local size
  for size in 16 32 48 64 128; do
    local icon="${appdir}/browser/chrome/icons/default/default${size}.png"
    if [[ -f "${icon}" ]]; then
      install -Dm644 "${icon}" \
        "${pkgdir}/usr/share/icons/hicolor/${size}x${size}/apps/mullvad-browser.png"
    fi
  done

  # Install the scalable icon if available.
  local svgicon="${appdir}/browser/chrome/icons/default/about-logo.svg"
  if [[ -f "${svgicon}" ]]; then
    install -Dm644 "${svgicon}" \
      "${pkgdir}/usr/share/icons/hicolor/scalable/apps/mullvad-browser.svg"
  fi

  # Provide the upstream license files.
  local licenses="${appdir}/MullvadBrowser/Docs/Licenses"
  if [[ -d "${licenses}" ]]; then
    install -d "${pkgdir}/usr/share/licenses"
    ln -s "/opt/mullvad-browser/MullvadBrowser/Docs/Licenses" \
      "${pkgdir}/usr/share/licenses/mullvad-browser"
  fi
}
