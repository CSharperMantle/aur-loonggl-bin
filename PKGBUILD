# Maintainer: csmantle <aur at csmantle dot top>
# shellcheck shell=bash
# shellcheck disable=SC2034,SC2154,SC2164

_pkgname=loonggl
pkgname="${_pkgname}-bin"
_origver='1.0.2-lnd25.1~rc1.7'
_pkgver="$(printf '%s' "$_origver" | tr -- '-~' '_.')"
pkgver="$_pkgver"
pkgrel=2
pkgdesc='OpenGL runtime components for LoongGPU'
arch=('loong64')
url='https://pkg.loongnix.cn/'
license=('custom')
depends=(
  'elfutils'
  'expat'
  'gcc-libs'
  'glibc'
  'libdrm'
  'libldrm'
  'libloong-gpucomp'
  'lm_sensors'
  'libx11'
  'zlib'
  'libglvnd'
)
options=('!strip')
install="${_pkgname}.install"

_debname="${_pkgname}_${_origver}_loong64.deb"
source=(
  "${_debname}::https://pkg.loongnix.cn/loongnix/25/pool/non-free/l/loonggpu-graphics-drivers/${_debname}"
  'loonggl-hack'
  'loonggl-hack.service'
  '40-loonggl-hack.preset'
)
sha256sums=('7ad0eba229921643d11dcf58f43ce3ff164f032aa2ba7d61dcf642c9cdc1d221'
            'd4196284eddcceec132c9aca763785fe054252785012a575590ee8c5996ee9c3'
            'df07e32e38a7b82140dae103b33fd1c90a04bd1ceee758240ebbf58460d84726'
            'bfe173cb7718584f0aa9262d90441ec69c2302ae8f78263e6f77b4b6fa3217f4')

package() {
  echo 'Extracting .deb archive...'
  bsdtar -xvf "$_debname"
  tar -xvf data.tar.* -C "$pkgdir"

  echo 'Moving libraries to the correct location ...'
  mv -v "$pkgdir"/usr/lib/loongarch64-linux-gnu/* "$pkgdir"/usr/lib/
  rmdir "$pkgdir"/usr/lib/loongarch64-linux-gnu

  echo 'Setting executable bits for shared objects ...'
  chmod -v +x "$pkgdir"/usr/lib/{,dri/,loonggpu/,gsgpu/}*.so*

  # FIXME: X.Org Server does not earch for DRI in /usr/lib/dri.
  # Whereas LoongGPU expects DRI modules to be stored in this path.
  echo 'Moving X11 DRI module to the correct location ...'
  install -d "$pkgdir"/usr/lib/xorg/modules/dri
  ln -svf ../../../gsgpu/dri/gsgpu_dri.so "$pkgdir"/usr/lib/xorg/modules/dri/gsgpu_dri.so
  ln -svf ../../../loonggpu/dri/loonggpu_dri.so "$pkgdir"/usr/lib/xorg/modules/dri/loonggpu_dri.so

  echo 'Creating a symlink to libglapidispatch ...'
  ln -svf loonggpu/libglapidispatch.so.0 "$pkgdir"/usr/lib/libglapidispatch.so.0
  
  echo 'Creating a symlink to libgsgpu_glapi.so.0 ...'
  ln -svf gsgpu/libgsgpu_glapi.so.0 "$pkgdir"/usr/lib/libgsgpu_glapi.so.0
  ln -svf loonggpu/libloonggpu_glapi.so.0 "$pkgdir"/usr/lib/libloonggpu_glapi.so.0

  # FIXME: See the loonggl-hack script for this stupidity.
  echo 'Renaming JSON files for loonggl-hack ...'
  for json in "$pkgdir"/usr/share/glvnd/egl_vendor.d/40*.json; do
    [ -e "$json" ] || continue
    base="$(basename -- "$json")"
    mv -v "$json" "$pkgdir"/usr/share/glvnd/egl_vendor.d/"${base/40/60}"
  done

  install -Dvm644 -t "$pkgdir"/usr/lib/systemd/system/ "$srcdir"/loonggl-hack.service
  install -Dvm644 -t "$pkgdir"/usr/lib/systemd/system-preset/ "$srcdir"/40-loonggl-hack.preset
  install -Dvm755 -t "$pkgdir"/usr/libexec/ "$srcdir"/loonggl-hack
}
