# Maintainer: Darkfish Tech <arch at darkfish dot com dot au>

pkgname=globalprotect-bin
pkgver=6.1.4.0
pkgrel=711
pkgdesc="GlobalProtect VPN Client Agent for The University of Manchester"
arch=('x86_64')
url="https://www.itservices.manchester.ac.uk/ourservices/popular/vpn/"
license=('custom')
groups=()
depends=('qt5-webkit' 'wmctrl')
options=()
install=globalprotect.install
source=(
    "GlobalProtect_UI_focal_deb-$pkgver-$pkgrel.deb"
    "uom-pangps.xml"
    )
package() {
    tar -xf data.tar.xz -C "$pkgdir/"

    # Adapted for Arch Linux from debian package postinst
    GPDIR="$pkgdir/opt/paloaltonetworks/globalprotect"
    
    # Install system and user services
    install -Dm644 $GPDIR/gpd.service "$pkgdir/usr/lib/systemd/system/gpd.service"
    install -Dm644 $GPDIR/gpa.service "$pkgdir/usr/lib/systemd/user/gpa.service"

    # Install desktop and autostart files
    install -Dm644 $GPDIR/gp.desktop "$pkgdir/usr/share/applications/gp.desktop"
    install -Dm644 $GPDIR/PanGPUI.desktop "$pkgdir/etc/xdg/autostart/PanGPUI.desktop"

    # Install executables
    install -Dm755 $GPDIR/globalprotect "$pkgdir/usr/bin/globalprotect"
    install -Dm755 $GPDIR/PanGPUI "$pkgdir/usr/bin/PanGPUI"

    # Install custom configuration for UoM
    install -Dm644 "$srcdir"/uom-pangps.xml $GPDIR/pangps.xml

}
md5sums=('d5265bd523b137334baa1511dc499698'
         'ad5a1512c694d09f76913c7c7e5f9c67')
