# U-Boot: Pinephone Pro based on PKGBUILD for RK3399
# Contributor: Furkan Kardame <furkan@fkardame.com>
# Contributor: Dan Johansen <strit@manjaro.org>
# Contributor: Dragan Simic <dsimic@buserror.io>

pkgname=uboot-pinephonepro
pkgver=2022.01
pkgrel=0
epoch=1
_srcname=u-boot-pine64-pinephonepro
_commit=d637294e264adfeb29f390dfc393106fd4d41b17
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
        "0001-rockchip-Add-initial-support-for-the-PinePhone-Pro.patch"
        "0002-Correct-boot-order-to-be-USB-SD-eMMC.patch"
        "0003-Configure-USB-power-settings-for-PinePhone-Pro.patch")
sha256sums=('d69b85ee6a153cd79e17085421c5f20c8d3037c57abe84f87d8402ce9d9b3131'
            '4e59f02ccb042d5d18c89c849701b96e6cf4b788709564405354b5d313d173f7'
            '4e356b3868c0c1ac061c2c15c7ba80c627e1743214680409f418f9b4c00eb3f7'
            'de7e36cdc7ed2fb5abb9155c97f87926361aa5be87d794c9016776160f3430ec'
            'e55fb02dfb6213eabbb899b468dc5f68d36a11c05feda4c14e80282415222fea'
            '6265fb9d3bc84bf1217383b52587b1d5a36372d88a824932586a802a502f62ba'
            '05eaccb2e8ea1eba3e86a4e7fcf12fd232195b5018c049ddf36e5a82a968cc24'
            '7c3d76f4bee0e54900142043241248072e334387065212141e1f600afe0aafba'
            '017d33aac55f8a5ed22170c97b4792ba755a4dad04f6c0cdd85119bbc81e87b3'
            'b750ba47843defafd8be1cc2615983c93e9cde5a4f5a7b55308a6f00f3fe6611')

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
