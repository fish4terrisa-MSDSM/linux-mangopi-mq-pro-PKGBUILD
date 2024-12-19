# Maintainer: Jan Alexander Steffens (heftig) <heftig@archlinux.org>

# For MangoPi MQ-Pro
pkgbase=linux-mangopi
pkgver=6.8.mangopi1
pkgrel=1
pkgdesc='Linux for MangoPi MQ-Pro'
url='https://github.com/archlinux/linux'
arch=(riscv64)
license=(GPL-2.0-only)
makedepends=(
  bc
  cpio
  gettext
  libelf
  pahole
  perl
  python
  tar
  xz

  # htmldocs
  graphviz
  imagemagick
  python-sphinx
  python-yaml
  texlive-latexextra
)
options=(
  !debug
  !strip
)
_srcname=linux-${pkgver%.*}
_srctag=v${pkgver%.*}-${pkgver##*.}
source=(
  https://cdn.kernel.org/pub/linux/kernel/v${pkgver%%.*}.x/${_srcname}.tar.xz
  "git+https://github.com/lwfinger/rtl8723ds.git"
)
sha256sums=('SKIP'
            'SKIP')

export KBUILD_BUILD_HOST=archlinux
export KBUILD_BUILD_USER=$pkgbase
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

patch_config() {
    # must be called when inside the `linux` dir
    key="$1"
    val="$2"

    if [ -z "$key" ] || [ -z "$val" ]; then
        exit 1
    fi

    case "$val" in
    'y')
        _OP='--enable'
        ;;
    'n')
        _OP='--disable'
        ;;
    'm')
        _OP='--module'
        ;;
    *)
        echo "Unknown kernel option value '$KERNEL'"
        exit 1
        ;;
    esac

    if [ -z "$_OP" ]; then
        exit 1
    fi

    ./scripts/config --file ".config" "$_OP" "$key"
}

prepare() {
  cd $_srcname
  echo "Setting version..."
  touch .scmversion
  sed -i 's/EXTRAVERSION =/EXTRAVERSION = -mangopi1/' Makefile
  echo "-$pkgrel" > localversion.10-pkgrel
  echo "${pkgbase#linux}" > localversion.20-pkgname
  local src

  echo "Setting config..."
  make defconfig
  # patch necessary options
        # patch_config LOCALVERSION_AUTO n #### not necessary with a release kernel

        # enable WiFi
        patch_config CFG80211 m

        # There is no LAN, so let there be USB-LAN
        patch_config USB_NET_DRIVERS m
        patch_config USB_CATC m
        patch_config USB_KAWETH m
        patch_config USB_PEGASUS m
        patch_config USB_RTL8150 m
        patch_config USB_RTL8152 m
        patch_config USB_LAN78XX m
        patch_config USB_USBNET m
        patch_config USB_NET_AX8817X m
        patch_config USB_NET_AX88179_178A m
        patch_config USB_NET_CDCETHER m
        patch_config BT y
        patch_config BT_HCIUART m
        patch_config USB_NET_CDC_EEM m
        patch_config USB_NET_CDC_NCM m
        patch_config USB_NET_HUAWEI_CDC_NCM m
        patch_config USB_NET_CDC_MBIM m
        patch_config USB_NET_DM9601 m
        patch_config USB_NET_SR9700 m
        patch_config USB_NET_SR9800 m
        patch_config USB_NET_SMSC75XX m
        patch_config USB_NET_SMSC95XX m
        patch_config USB_NET_GL620A m
        patch_config USB_NET_NET1080 m
        patch_config USB_NET_PLUSB m
        patch_config USB_NET_MCS7830 m
        patch_config USB_NET_RNDIS_HOST m
        patch_config USB_NET_CDC_SUBSET_ENABLE m
        patch_config USB_NET_CDC_SUBSET m
        patch_config USB_ALI_M5632 y
        patch_config USB_AN2720 y
        patch_config USB_BELKIN y
        patch_config USB_ARMLINUX y
        patch_config USB_EPSON2888 y
        patch_config USB_KC2190 y
        patch_config USB_NET_ZAURUS m
        patch_config USB_NET_CX82310_ETH m
        patch_config USB_NET_KALMIA m
        patch_config USB_NET_QMI_WWAN m
        patch_config USB_NET_INT51X1 m
        patch_config USB_IPHETH m
        patch_config USB_SIERRA_NET m
        patch_config USB_VL600 m
        patch_config USB_NET_CH9200 m
        patch_config USB_NET_AQC111 m
        patch_config USB_RTL8153_ECM m

        # enable systemV IPC (needed by fakeroot during makepkg)
        patch_config SYSVIPC y
        patch_config SYSVIPC_SYSCTL y

        # enable swap
        patch_config SWAP y
        patch_config ZSWAP y

        # enable Cedrus VPU Drivers
        patch_config MEDIA_SUPPORT y
        patch_config MEDIA_CONTROLLER y
        patch_config MEDIA_CONTROLLER_REQUEST_API y
        patch_config V4L_MEM2MEM_DRIVERS y
        patch_config VIDEO_SUNXI_CEDRUS y

        # enable binfmt_misc
        patch_config BINFMT_MISC y
  make olddefconfig

  #diff -u ../config .config || :

  make -s kernelrelease > version
  echo "Prepared $pkgbase version $(<version)"
}

build() {
  cd $_srcname
  make DTC_FLAGS="-@" all
  # make -C tools/bpf/bpftool vmlinux.h feature-clang-bpf-co-re=1
  # make htmldocs
  # Build rtl8723ds driver
  cd ../rtl8723ds
  make KSRC="../$_srcname" modules || true

}

_package() {
  pkgdesc="The $pkgdesc kernel and modules"
  depends=(
    coreutils
    initramfs
    kmod
  )
  optdepends=(
    'linux-firmware: firmware images needed for some devices'
    'scx-scheds: to use sched-ext schedulers'
    'wireless-regdb: to set the correct wireless channels of your country'
  )
  provides=(
    KSMBD-MODULE
    VIRTUALBOX-GUEST-MODULES
    WIREGUARD-MODULE
  )

  cd $_srcname
  local modulesdir="$pkgdir/usr/lib/modules/$(<version)"

  echo "Installing boot image..."
  # MangoPi MQ-Pro currently cannot boot from a compressed kernel.
  # Doesn't really know why(probably something related with the
  # devicetree)
  # But we'll just use the uncompressed one instead :-)
  install -Dm644 arch/riscv/boot/Image.gz "$modulesdir/Image.gz"
  install -Dm644 arch/riscv/boot/Image "$modulesdir/Image"
  install -Dm644 arch/riscv/boot/Image "$modulesdir/vmlinuz"

  # Used by mkinitcpio to name the kernel
  echo "$pkgbase" | install -Dm644 /dev/stdin "$modulesdir/pkgbase"

  echo "Installing modules..."
  make INSTALL_MOD_PATH="$pkgdir/usr" KERNELRELEASE="$(<version)" \
    DEPMOD=/doesnt/exist modules_install  # Suppress depmod
  echo "Installing rtl8723ds module..."
  install -p -Dm644 ../rtl8723ds/8723ds.ko "$pkgdir/usr/lib/modules/$(<version)/kernel/drivers/net/wireless/8723ds.ko"
  echo "Installing dtbs..."
  make INSTALL_DTBS_PATH="$pkgdir/usr/lib/modules/$(<version)/dtb" dtbs_install

  # remove build link
  rm "$modulesdir"/build
}

_package-headers() {
  pkgdesc="Headers and scripts for building modules for the $pkgdesc kernel"
  depends=(pahole)

  cd $_srcname
  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  echo "Installing build files..."
  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
    localversion.* version vmlinux
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/riscv" -m644 arch/riscv/Makefile
  cp -t "$builddir" -a scripts
  ln -srt "$builddir" "$builddir/scripts/gdb/vmlinux-gdb.py"


  # required when DEBUG_INFO_BTF_MODULES is enabled
  # install -Dt "$builddir/tools/bpf/resolve_btfids" tools/bpf/resolve_btfids/resolve_btfids

  echo "Installing headers..."
  cp -t "$builddir" -a include
  cp -t "$builddir/arch/riscv" -a arch/riscv/include
  install -Dt "$builddir/arch/riscv/kernel" -m644 arch/riscv/kernel/asm-offsets.s
  install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
  install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h

  # https://bugs.archlinux.org/task/13146
  install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

  # https://bugs.archlinux.org/task/20402
  install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h

  # https://bugs.archlinux.org/task/71392
  install -Dt "$builddir/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

  echo "Installing KConfig files..."
  find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

  echo "Removing unneeded architectures..."
  local arch
  for arch in "$builddir"/arch/*/; do
    [[ $arch = */riscv/ ]] && continue
    echo "Removing $(basename "$arch")"
    rm -r "$arch"
  done

  echo "Removing documentation..."
  rm -r "$builddir/Documentation"

  echo "Removing broken symlinks..."
  find -L "$builddir" -type l -printf 'Removing %P\n' -delete

  echo "Removing loose objects..."
  find "$builddir" -type f -name '*.o' -printf 'Removing %P\n' -delete

  echo "Stripping build tools..."
  local file
  while read -rd '' file; do
    case "$(file -Sib "$file")" in
      application/x-sharedlib\;*)      # Libraries (.so)
        strip -v $STRIP_SHARED "$file" ;;
      application/x-archive\;*)        # Libraries (.a)
        strip -v $STRIP_STATIC "$file" ;;
      application/x-executable\;*)     # Binaries
        strip -v $STRIP_BINARIES "$file" ;;
      application/x-pie-executable\;*) # Relocatable binaries
        strip -v $STRIP_SHARED "$file" ;;
    esac
  done < <(find "$builddir" -type f -perm -u+x ! -name vmlinux -print0)

  echo "Adding symlink..."
  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}

_package-docs() {
  pkgdesc="Documentation for the $pkgdesc kernel"

  cd $_srcname
  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  echo "Installing documentation..."
 # local src dst
 # while read -rd '' src; do
 #   dst="${src#Documentation/}"
 #   dst="$builddir/Documentation/${dst#output/}"
 #   install -Dm644 "$src" "$dst"
 # done < <(find Documentation -name '.*' -prune -o ! -type d -print0)

 # echo "Adding symlink..."
 # mkdir -p "$pkgdir/usr/share/doc"
 # ln -sr "$builddir/Documentation" "$pkgdir/usr/share/doc/$pkgbase"
}

pkgname=(
  "$pkgbase"
  "$pkgbase-headers"
  "$pkgbase-docs"
)
for _p in "${pkgname[@]}"; do
  eval "package_$_p() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#$pkgbase}
  }"
done

# vim:set ts=8 sts=2 sw=2 et:
