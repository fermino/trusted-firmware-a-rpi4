# TF-A for the Raspberry Pi 4 with PSCI core parking fixes
# Maintainer: Fermín Olaiz <fermin@olaiz.net>

pkgname=trusted-firmware-a-rpi4
_tag=v2.16-rc0
_dtc=8f48565e5cfedc74d3f7512f1e0188e9d85dc1de
pkgver=${_tag#v}
pkgver=${pkgver//-/}
pkgrel=2
pkgdesc='Trusted Firmware-A BL31 for the Raspberry Pi 4, as an armstub providing PSCI'
arch=(any)
url='https://www.trustedfirmware.org/projects/tf-a/'
license=('BSD-3-Clause' 'BSD-2-Clause OR GPL-2.0-or-later')
depends=(bash util-linux)
optdepends=('raspberrypi-bootloader-rpi4: start4.elf and fixup4.dat on the boot partition')
options=(!buildflags)
makedepends=(git)
source=(
    "trusted-firmware-a::git+https://git.trustedfirmware.org/TF-A/trusted-firmware-a.git#tag=$_tag"
    "dtc::git+https://git.kernel.org/pub/scm/utils/dtc/dtc.git#commit=$_dtc"
    trusted-firmware-a-rpi4-install
    trusted-firmware-a-rpi4-install.hook
)
sha256sums=('c65c4909379e69968594966b92ff81454ce159fa37c47aec6a4d02070fec0918'
            'eebcca9d985422a2e244650eaf766880329111a11314509a3678c08e0799636a'
            'f4c2ed541957d41c04602c6571c5e178a67d714da3b5060773df3b9f97661107'
            'bdb922719d3cc5b07803700acc8074bef3bd641370bd3781c3044a9027c6f42a')

prepare() {
    # Check that the DTC clone matches what TF-A expects
    local pinned
    pinned=$(git -C trusted-firmware-a rev-parse "HEAD:contrib/dtc")
    if [[ $pinned != "$_dtc" ]]; then
        echo "prepare: TF-A $_tag pins dtc at $pinned, but _dtc is $_dtc" >&2
        return 1
    fi

    git -C trusted-firmware-a submodule init contrib/dtc
    git -C trusted-firmware-a config submodule.dtc.url "$srcdir/dtc"
    git -C trusted-firmware-a -c protocol.file.allow=always \
        submodule update --no-recommend-shallow contrib/dtc
}

build() {
    cd trusted-firmware-a

    if [[ $CARCH != aarch64 ]]; then
        export CROSS_COMPILE=aarch64-linux-gnu-
    fi

    make \
        PLAT=rpi4 \
        DEBUG=0 \
        bl31
}

package() {
    cd trusted-firmware-a

    install -Dm644 build/rpi4/release/bl31.bin "$pkgdir/usr/lib/$pkgname/boot/tfa-bl31.bin"

    install -Dm755 -t "$pkgdir/usr/share/libalpm/scripts" "$srcdir/trusted-firmware-a-rpi4-install"
    install -Dm644 -t "$pkgdir/usr/share/libalpm/hooks" "$srcdir/trusted-firmware-a-rpi4-install.hook"

    install -Dm644 docs/license.rst "$pkgdir/usr/share/licenses/$pkgname/LICENSE.tf-a"
    install -Dm644 contrib/dtc/BSD-2-Clause "$pkgdir/usr/share/licenses/$pkgname/LICENSE.libfdt"
}
