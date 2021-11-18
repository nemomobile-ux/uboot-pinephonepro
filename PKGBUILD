# U-Boot: Pinephone Pro based on PKGBUILD for RK3399
# Contributor: Furkan Kardame <furkan@fkardame.com>

pkgname=uboot-pinephonepro
pkgver=2021.01rc3
pkgrel=1
epoch=1
_srcname=u-boot-pine64-pinephonepro
_commit="19150d65b1bed6831ba92a4cf3e7262518f1049f"
_tfaver=2.5
pkgdesc="U-Boot for Pinephone Pro"
arch=('aarch64')
url='https://git.sr.ht/~martijnbraam/u-boot'
license=('GPL')
makedepends=('git' 'arm-none-eabi-gcc' 'dtc' 'bc')
provides=('uboot')
conflicts=('uboot')
install=${pkgname}.install
source=("$_srcname-$_commit.tar.gz::https://git.sr.ht/~martijnbraam/u-boot/archive/$_commit.tar.gz"
        "https://git.trustedfirmware.org/TF-A/trusted-firmware-a.git/snapshot/trusted-firmware-a-$_tfaver.tar.gz")
sha256sums=('1858fd6dd4d7f44a43f3945ef7a2751b59ef1d94c8696b3523f566ec3fe037da'
            'ad8a2ffcbcd12d919723da07630fc0840c3c2fba7656d1462e45488e42995d7c')

#prepare() {
#  cd u-boot-${pkgver/rc/-rc}
# Patches based on the work of dhivael and Nadia
#  cd ../trusted-firmware-a-$_tfaver
#}

build() {
  cd trusted-firmware-a-$_tfaver
  unset CFLAGS CXXFLAGS CPPFLAGS LDFLAGS
  make PLAT=rk3399
  cp build/rk3399/release/bl31/bl31.elf ../u-boot-${_commit}
  cd ../u-boot-${_commit}
  unset CFLAGS CXXFLAGS CPPFLAGS LDFLAGS
  make pinephone-pro-rk3399_defconfig
  echo 'CONFIG_IDENT_STRING=" Manjaro ARM"' >> .config
  echo 'CONFIG_USB_EHCI_HCD=n' >> .config
  echo 'CONFIG_USB_EHCI_GENERIC=n' >> .config
  echo 'CONFIG_USB_XHCI_HCD=n' >> .config
  echo 'CONFIG_USB_XHCI_DWC3=n' >> .config
  echo 'CONFIG_USB_DWC3=n' >> .config
  echo 'CONFIG_USB_DWC3_GENERIC=n' >> .config

  make EXTRAVERSION=-${pkgrel}
}

package() {
  cd u-boot-${_commit}

  mkdir -p "${pkgdir}/boot/extlinux"
  cp idbloader.img u-boot.itb  "${pkgdir}/boot/"
}

