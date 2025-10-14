# Maintainer: Darkfish Tech <arch at darkfish dot com dot au>
# Modified by: Miguel Cárcamo Vásquez <Universidad de Santiago de Chile>

pkgname=globalprotect-bin
pkgver=6.2.1.1
pkgrel=7
pkgdesc="GlobalProtect VPN Client Agent (UoM) with Ubuntu-compatible HIP support for Arch/Manjaro"
arch=('x86_64')
url="https://www.itservices.manchester.ac.uk/ourservices/popular/vpn/"
license=('custom')
depends=('qt5-webkit' 'wmctrl' 'openssl' 'glibc')
install=globalprotect.install
source=(
    "GlobalProtect_UI_focal_deb-$pkgver-$pkgrel.deb"
    "uom-pangps.xml"
)
noextract=("GlobalProtect_UI_focal_deb-$pkgver-$pkgrel.deb")

prepare() {
    mkdir -p "$srcdir/compat-libs"
    cd "$srcdir/compat-libs" || exit 1

    echo "Downloading compatibility libraries (Ubuntu 22.04)..."

    urls=(
      "https://security.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2.24_amd64.deb"
      "https://launchpad.net/ubuntu/+archive/primary/+files/libcurl4_7.81.0-1ubuntu1.21_amd64.deb"
      "https://security.ubuntu.com/ubuntu/pool/main/n/nss/libnss3_3.98-0ubuntu0.22.04.2_amd64.deb"
    )

    for url in "${urls[@]}"; do
        file=$(basename "$url")
        echo "Downloading $file"
        wget -q --show-progress "$url" -O "$file" || { echo "ERROR: Failed to download $url"; exit 1; }
    done

    echo "Extracting libraries..."
    for f in *.deb; do
        echo "  - Extracting $f"
        tmpdir=$(mktemp -d)
        bsdtar -xf "$f" -C "$tmpdir" || { echo "WARNING: Could not open $f"; exit 1; }

        # Detect if there is data.tar.xz or data.tar.zst
        if [[ -f "$tmpdir/data.tar.xz" ]]; then
            bsdtar -xf "$tmpdir/data.tar.xz" --strip-components=3 -C . ./usr/lib/x86_64-linux-gnu || true
        elif [[ -f "$tmpdir/data.tar.zst" ]]; then
            bsdtar -xf "$tmpdir/data.tar.zst" --strip-components=3 -C . ./usr/lib/x86_64-linux-gnu || true
        else
            echo "WARNING: No data.tar.* found in $f"
        fi

        rm -rf "$tmpdir"
    done

    echo "SUCCESS: Libraries extracted successfully to $srcdir/compat-libs"
}

package() {
    tar -xf data.tar.xz -C "$pkgdir/"
    GPDIR="$pkgdir/opt/paloaltonetworks/globalprotect"

    # Install services
    install -Dm644 "$GPDIR/gpd.service" "$pkgdir/usr/lib/systemd/system/gpd.service"
    install -Dm644 "$GPDIR/gpa.service" "$pkgdir/usr/lib/systemd/user/gpa.service"

    # Desktop files
    install -Dm644 "$GPDIR/gp.desktop" "$pkgdir/usr/share/applications/gp.desktop"
    install -Dm644 "$GPDIR/PanGPUI.desktop" "$pkgdir/etc/xdg/autostart/PanGPUI.desktop"

    # Main executables
    install -Dm755 "$GPDIR/globalprotect" "$pkgdir/usr/bin/globalprotect"
    install -Dm755 "$GPDIR/PanGPUI" "$pkgdir/usr/bin/PanGPUI"

    # University of Manchester configuration
    install -Dm644 "$srcdir/uom-pangps.xml" "$GPDIR/pangps.xml"

    # Ubuntu compatible libraries
    mkdir -p "$GPDIR/lib-compat"

    echo "Searching for extracted libraries in compat-libs..."
    found_libs=$(find "$srcdir/compat-libs" -type f -name "*.so*" 2>/dev/null)

    if [[ -n "$found_libs" ]]; then
        echo "SUCCESS: Copying found libraries:"
        echo "$found_libs" | sed 's/^/   - /'
        while IFS= read -r f; do
            cp -a "$f" "$GPDIR/lib-compat/" || echo "WARNING: Could not copy $f"
        done <<< "$found_libs"
    else
        echo "WARNING: No extracted libraries found — verify compat-libs structure"
    fi

    # HIP shim launcher
    cat > "$GPDIR/launch-hip.sh" <<'EOF'
#!/bin/bash
LD_LIBRARY_PATH="/opt/paloaltonetworks/globalprotect/lib-compat:$LD_LIBRARY_PATH"
export LD_LIBRARY_PATH

# Execute primary HIP first, then multiprocess if it exists
if [ -x /opt/paloaltonetworks/globalprotect/PanGpHip ]; then
  /opt/paloaltonetworks/globalprotect/PanGpHip "$@" >> /opt/paloaltonetworks/globalprotect/PanGpHip.log 2>&1 &
fi
if [ -x /opt/paloaltonetworks/globalprotect/PanGpHipMp ]; then
  /opt/paloaltonetworks/globalprotect/PanGpHipMp "$@" >> /opt/paloaltonetworks/globalprotect/PanGpHipMp.log 2>&1 &
fi
EOF
    chmod +x "$GPDIR/launch-hip.sh"
    ln -sf /opt/paloaltonetworks/globalprotect/launch-hip.sh "$pkgdir/usr/bin/launch-hip"

    # LSB release fake (only if it doesn't exist in the system)
    if [ ! -f /etc/lsb-release ]; then
        mkdir -p "$pkgdir/etc"
        cat > "$pkgdir/etc/lsb-release" <<EOF
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=22.04
DISTRIB_CODENAME=jammy
DISTRIB_DESCRIPTION="Ubuntu 22.04 LTS (emulated for HIP)"
EOF
    else
        echo "WARNING: Skipping creation of /etc/lsb-release (already exists in Manjaro)"
    fi

}


md5sums=('a285cb0cdb1bd14ddc999c7278cb9a6b'
         'ad5a1512c694d09f76913c7c7e5f9c67')
