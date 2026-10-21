# Maintainer: Nokzes
pkgname=sirius-schedule
pkgver=1.1.0
pkgrel=1
pkgdesc="Сегодняшнее расписание группы Сириус колледжа: окно GTK4 и модуль waybar"
arch=('any')
url="https://github.com/Nokzes/sirius-schedule-arch-hyprland-waybar"
license=('MIT')
depends=('python' 'python-gobject' 'gtk4')
source=('sirius-schedule' 'sirius-waybar' 'sirius-schedule.desktop')
sha256sums=('SKIP' 'SKIP' 'SKIP')

package() {
  install -Dm755 sirius-schedule "$pkgdir/usr/bin/sirius-schedule"
  install -Dm755 sirius-waybar "$pkgdir/usr/bin/sirius-waybar"
  install -Dm644 sirius-schedule.desktop "$pkgdir/usr/share/applications/sirius-schedule.desktop"
}
