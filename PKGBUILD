# U-Boot: Pinephone Pro based on PKGBUILD for RK3399
# Contributor: Furkan Kardame <furkan@fkardame.com>

pkgname=uboot-pinephonepro
pkgver=2021.01rc3
pkgrel=4
epoch=1
_srcname=u-boot-pine64-pinephonepro
_commit=0719bf42931033c3109ecc6357e8adb567cb637b
_tfaver=2.6
pkgdesc="U-Boot for Pinephone Pro"
arch=('aarch64')
url='https://git.sr.ht/~martijnbraam/u-boot'
license=('GPL')
makedepends=('git' 'arm-none-eabi-gcc' 'dtc' 'bc')
provides=('uboot')
conflicts=('uboot')
install=${pkgname}.install
source=("u-boot-$_commit.tar.gz::https://source.denx.de/u-boot/u-boot/-/archive/${_commit}/u-boot-${_commit}.tar.gz"
        "https://git.trustedfirmware.org/TF-A/trusted-firmware-a.git/snapshot/trusted-firmware-a-$_tfaver.tar.gz"
        0001-PPP.patch
        0002-Add-ppp-dt.patch
        0003-Config-changes.patch
        0004-Add-kconfig-include.patch
        0005-Add-pinephone-pro-rk3399.h.patch
        0006-Added-dts-to-makefile.patch
        0007-u-boot.dtsi-fixes.patch
        0008-fix-boot-order.patch
        0009-Correct-boot-order-to-be-USB-SD-eMMC.patch)
sha256sums=('6b196b6592fabed060b7c5b1fa05a743f9be131d11389b762b7d0e2beebbd381'
            '4e59f02ccb042d5d18c89c849701b96e6cf4b788709564405354b5d313d173f7'
            'f2e9d4efd24b7a6d94ccfe8c1a6fd0fac04776483be7a2d343f5b7a6b50a8ff2'
            'c889eb1b55868a3d007be6f5c618823faca70905ce4a4047cd98bdcbfb48d6ab'
            '355444b20346bb5adbc531b1a8813483bb8e8e6a0b884139479dbf2ad342ef79'
            'ded1b2e6effbea181ed5c875bd63905db07daba268db48f0a75e9643c830c949'
            'ba42e43fa471154f6ea4fb5f731557a5f2494668afe05797450a43b82b82ab2f'
            '0e96af517f2f7a085412c659ab672e61912ff59d92f009ed119832c7c790d6d8'
            '3aa7c3b4aa1233d604cb9177fc9bc56f85714c5d69f9432690dc7c50e06c105b'
            '4aadc4f07f4ae62d5fe11cfabe1c5f917f77ce8014800ae3a107f9bcc551bc5b'
            '017d33aac55f8a5ed22170c97b4792ba755a4dad04f6c0cdd85119bbc81e87b3')

prepare() {
  cd u-boot-${_commit}
  local src
  for src in "${source[@]}"; do
      src="${src%%::*}"
      src="${src##*/}"
      [[ $src = *.patch ]] || continue
      msg2 "Applying patch: $src..."
      patch -Np1 < "../$src"
  done
}

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

