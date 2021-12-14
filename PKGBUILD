# U-Boot: Pinephone Pro based on PKGBUILD for RK3399
# Contributor: Furkan Kardame <furkan@fkardame.com>
# Contributor: Dan Johansen <strit@manjaro.org>
# Contributor: Dragan Simic <dsimic@buserror.io>

pkgname=uboot-pinephonepro
pkgver=2021.01rc3
pkgrel=6
epoch=1
_srcname=u-boot-pine64-pinephonepro
_commit=0719bf42931033c3109ecc6357e8adb567cb637b
_tfaver=2.6
pkgdesc="U-Boot for Pine64 PinePhone Pro"
arch=('aarch64')
url='https://git.sr.ht/~martijnbraam/u-boot'
license=('GPL')
makedepends=('git' 'arm-none-eabi-gcc' 'dtc' 'bc')
depends=('uboot-tools')
provides=('uboot')
conflicts=('uboot')
install=${pkgname}.install
source=("u-boot-$_commit.tar.gz::https://source.denx.de/u-boot/u-boot/-/archive/${_commit}/u-boot-${_commit}.tar.gz"
        "https://git.trustedfirmware.org/TF-A/trusted-firmware-a.git/snapshot/trusted-firmware-a-$_tfaver.tar.gz"
        "boot.txt"
        "ppp-prepare-fstab"
        "ppp-prepare-fstab.service"
        "ppp-uboot-flash"
        "ppp-uboot-mkscr"
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
            '4e356b3868c0c1ac061c2c15c7ba80c627e1743214680409f418f9b4c00eb3f7'
            'de7e36cdc7ed2fb5abb9155c97f87926361aa5be87d794c9016776160f3430ec'
            'e55fb02dfb6213eabbb899b468dc5f68d36a11c05feda4c14e80282415222fea'
            '6265fb9d3bc84bf1217383b52587b1d5a36372d88a824932586a802a502f62ba'
            '05eaccb2e8ea1eba3e86a4e7fcf12fd232195b5018c049ddf36e5a82a968cc24'
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
  # Avoid build warnings by editing a .config option in place instead of
  # appending an option to .config, if an option is already present
  update_config() {
    if ! grep -q "^$1=$2$" .config; then
      if grep -q "^# $1 is not set$" .config; then
        sed -i -e "s/^# $1 is not set$/$1=$2/g" .config
      elif grep -q "^$1=" .config; then
        sed -i -e "s/^$1=.*/$1=$2/g" .config
      else
        echo "$1=$2" >> .config
      fi
    fi
  }

  unset CFLAGS CXXFLAGS CPPFLAGS LDFLAGS

  cd trusted-firmware-a-$_tfaver

  echo -e "\nBuilding TF-A for Pine64 PinePhone Pro...\n"
  make PLAT=rk3399
  cp build/rk3399/release/bl31/bl31.elf ../u-boot-${_commit}

  cd ../u-boot-${_commit}

  echo -e "\nBuilding U-Boot for Pine64 PinePhone Pro...\n"
  make pinephone-pro-rk3399_defconfig

  update_config 'CONFIG_IDENT_STRING' '" Manjaro Linux ARM"'
  update_config 'CONFIG_BOOTDELAY' '0'
  update_config 'CONFIG_USB_EHCI_HCD' 'n'
  update_config 'CONFIG_USB_EHCI_GENERIC' 'n'
  update_config 'CONFIG_USB_XHCI_HCD' 'n'
  update_config 'CONFIG_USB_XHCI_DWC3' 'n'
  update_config 'CONFIG_USB_DWC3' 'n'
  update_config 'CONFIG_USB_DWC3_GENERIC' 'n'

  make EXTRAVERSION=-${pkgrel}
}

package() {
  cd u-boot-${_commit}

  install -D -m 0644 idbloader.img u-boot.itb -t "${pkgdir}/boot"
  install -D -m 0644 "${srcdir}/boot.txt" -t "${pkgdir}/boot"
  install -D -m 0755 "${srcdir}/ppp-uboot-mkscr" -t "${pkgdir}/usr/bin"

  install -D -m 0755 "${srcdir}/ppp-prepare-fstab" -t "${pkgdir}/usr/bin"
  install -D -m 0644 "${srcdir}/ppp-prepare-fstab.service" -t "${pkgdir}/usr/lib/systemd/system"
  install -D -m 0755 "${srcdir}/ppp-uboot-flash" -t "${pkgdir}/usr/bin"
}
