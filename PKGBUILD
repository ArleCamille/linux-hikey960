# AArch64 HiKey960
# Maintainer: Hyeseon Oh <okw1003@gmail.com>

buildarch=8

pkgbase=linux-hikey960
_srcname=linux-6.2
_kernelname=${pkgbase#linux}
_desc="AArch64 HiKey960"
pkgver=6.2.10
pkgrel=3
arch=('aarch64')
url="http://www.kernel.org/"
license=('GPL2')
makedepends=('xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'bc' 'git' 'uboot-tools' 'dtc')
options=('!strip')
source=("http://www.kernel.org/pub/linux/kernel/v6.x/${_srcname}.tar.xz"
	"http://www.kernel.org/pub/linux/kernel/v6.x/patch-${pkgver}.xz"
	'0001-net-smsc95xx-Allow-mac-address-to-be-set-as-a-parame.patch'
	'0002-arm64-dts-rockchip-disable-pwm0-on-rk3399-firefly.patch'
	'0003-include-Mali-GPU.patch'
	'0004-HiKey960-DTS-modification.patch'
	'config'
	'linux.preset'
	'60-linux.hook'
	'90-linux.hook')
md5sums=('787862593d7bf354cf1a5c37e21fc147'
	 'e0221ea0e6eeb147c29d2fd72e987ed5'
	 '7b08a199a97e3e2288e5c03d8e8ded2d'
         'c9d4e392555b77034e24e9f87c5ff0b3'
	 '61ddd8d1586f6223acdb21311b292a45'
	 'f6719b9811185fb233612b451a142de6'
	 'SKIP' # TODO: modify
	 '41cb5fef62715ead2dd109dbea8413d6'
         '0a5f16bfec6ad982a2f6782724cca8ba'
         '3dc88030a8f2f5a5f97266d99b149f77')

prepare() {
	cd $_srcname

	echo "Setting version..."
	scripts/setlocalversion --save-scmversion
	echo "-$pkgrel" > localversion.10-pkgrel
	echo "${pkgbase#linux}" > localversion.20-pkgname

	# add upstream patch
	git apply --whitespace=nowarn ../patch-${pkgver} 

	# ALARM patches
	git apply ../0001-net-smsc95xx-Allow-mac-address-to-be-set-as-a-parame.patch
	git apply ../0002-arm64-dts-rockchip-disable-pwm0-on-rk3399-firefly.patch

	# apply custom patch
	git apply ../0003-include-Mali-GPU.patch
	git apply ../0004-HiKey960-DTS-modification.patch

	cat "${srcdir}/config" > ./.config
}

build() {
	cd ${_srcname}

	# get kernel version
	make prepare
	make -s kernelrelease > version

	# build!
	unset LDFLAGS
	make ${MAKEFLAGS} Image Image.gz modules
	# Generate device tree blobs with symbols to support applying device tree overlays in U-Boot
	make ${MAKEFLAGS} DTC_FLAGS="-@" dtbs
}

_package() {
	pkgver="${pkgver}"
	pkgdesc="The Linux kernel and modules - ${_desc}"
	depends=('coreutils' 'linux-firmware' 'kmod' 'mkinitcpio>=0.7')
	optdepends=('wireless-regdb: to set the correct wireless channels of your country')
	provides=("linux=${pkgver}" "WIREGUARD-MODULE")
	conflicts=('linux')
	backup=("etc/mkinitcpio.d/${pkgbase}.preset")
	install=${pkgname}.install

	cd $_srcname
	local kernver="$(<version)"
	local modulesdir="$pkgdir/usr/lib/modules/$kernver"

	echo "Installing boot image and dtbs..."
	install -Dm644 arch/arm64/boot/Image{,.gz} -t "${pkgdir}/boot/"
	install -Dm644 arch/arm64/boot/dts/hisilicon/hi3660-hikey960.dtb -t "${pkgdir}/boot/dtbs/"

	echo "Installing modules..."
	make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 modules_install

	# remove build and source links
	rm "$modulesdir"/{source,build}

	# sed expression for following substitutions
	local _subst="
	  s|%PKGBASE%|${pkgbase}|g
	  s|%KERNVER%|${kernver}|g
	"

	# install mkinitcpio preset file
	sed "${_subst}" ../linux.preset |
		install -Dm644 /dev/stdin "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

	# instal pacman hooks
	sed "${_subst}" ../60-linux.hook |
		install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/60-${pkgbase}.hook"
	sed "${_subst}" ../90-linux.hook |
		install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/90-${pkgbase}.hook"
}

_package-headers() {
	pkgdesc="Header files and scripts for building modules for Linux kernsl - ${_desc}"
	provides=("linux-headers=${pkgver}")
	conflicts=('linux-headers')

	cd $_srcname
	local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

	echo "Installing build files..."
	install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
		localversion.* version vmlinux
	install -Dt "$builddir/kernel" -m644 kernel/Makefile
	install -Dt "$builddir/arch/arm64" -m644 arch/arm64/Makefile
	cp -t "$builddir" -a scripts

	# add xfs and shmem for aufs building
	mkdir -p "$builddir"/{fs/xfs,mm}

	echo "Installing headers..."
	cp -t "$builddir" -a include
	cp -t "$builddir/arch/arm64" -a arch/arm64/include
	install -Dt "$builddir/arch/arm64/kernel" -m644 arch/arm64/kernel/asm-offsets.s
	mkdir -p "$builddir/arch/arm"
	cp -t "$builddir/arch/arm" -a arch/arm/include

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
    [[ $arch = */arm64/ || $arch == */arm/ ]] && continue
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
    case "$(file -bi "$file")" in
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

pkgname=("${pkgbase}" "${pkgbase}-headers")
for _p in ${pkgname[@]}; do
	eval "package_${_p}() {
		_package${_p#${pkgbase}}
	}"
done
