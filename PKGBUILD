# Maintainer: Jakub Schmidtke <sjakub-at-gmail.com>

pkgname=kfs
pkgver=0.5
pkgrel=4
pkgdesc="KosmosFS - distributed Filesystem"
arch=('i686' 'x86_64')
url="http://code.google.com/p/kosmosfs"
license=('APACHE')
depends=('log4cpp' 'boost' 'python2>=2.7')
makedepends=('cmake' 'java-environment')
conflicts=('kfs-svn')
install=kfs.install
source=("http://kosmosfs.googlecode.com/files/kfs-${pkgver}.tar.gz" "kfs-chunkserv.rc" "kfs-metaserv.rc" "kfs.confd" "kfs.conf" "kfswebui.conf")
# backup=('etc/conf.d/kfs' 'etc/kfs.conf' 'etc/kfswebui.conf')

md5sums=('2178209298db5b8bf3b07dcbfef46acb'
         '5beed96cdc4ce543eab49c110ee6e383'
         'cb7d402e865bc0f9498a7b837daca7d9'
         '644ffa98afcb016cc2ec73ef5dc12cd0'
         'aaacdbd30410f8ab3b264519b944e263'
         'd99232fa0c729896f9ae9c213b4631ea')

build() {
  rm -rf "${srcdir}/build"
  cd "${srcdir}/kfs-${pkgver}"

  echo "target_link_libraries(qcdio rt)" >> src/cc/qcdio/CMakeLists.txt
  echo "target_link_libraries(qcdio-shared rt)" >> src/cc/qcdio/CMakeLists.txt

  sed -i "s|'/home/.*/src/cc/'|'${srcdir}/kfs-${pkgver}/src/cc/'|" src/cc/access/kfs_setup.py
  sed -i "s|'/usr/.*boost.*/'|'/usr/include/boost/'|" src/cc/access/kfs_setup.py

  find -name \*.py -type f -exec sed -i 's|#!/usr/bin/env python$|#!/usr/bin/python2|' {} \;
  find -name \*.py -type f -exec sed -i 's|#!/usr/bin/python$|#!/usr/bin/python2|' {} \;
  sed -i 's|COMMAND python |COMMAND /usr/bin/python2 |' src/cc/common/CMakeLists.txt

  mkdir -p "${srcdir}/build"
  cd "${srcdir}/build"

  cmake -Wno-dev -D CMAKE_BUILD_TYPE=Release ../kfs-${pkgver}
  make
}

package() {
  cd "${srcdir}/build"

  make install

  mkdir -p "${pkgdir}/usr/bin"
  mkdir -p "${pkgdir}/usr/lib"
  mkdir -p "${pkgdir}/usr/include/kfs/common"
  mkdir -p "${pkgdir}/usr/share/kfs/"{bin/{tests,emulator},scripts,webui}

  install -m0755 bin/tests/* "${pkgdir}/usr/share/kfs/bin/tests/"
  rm -rf bin/tests

  install -m0755 bin/emulator/* "${pkgdir}/usr/share/kfs/bin/emulator/"
  rm -rf bin/emulator

  install -m0755 bin/tools/* "${pkgdir}/usr/bin/"
  rm -rf bin/tools

  install -m0755 bin/* "${pkgdir}/usr/share/kfs/bin/"

  install -m0644 lib/static/* "${pkgdir}/usr/lib/"
  rm -rf lib/static

  install -m0755 lib/* "${pkgdir}/usr/lib/"

  install -m0644 include/kfs/common/* "${pkgdir}/usr/include/kfs/common/"
  rm -rf include/kfs/common

  install -m0644 include/kfs/* "${pkgdir}/usr/include/kfs/"

  install -m0755 "${srcdir}/kfs-${pkgver}/scripts"/*.sh "${pkgdir}/usr/share/kfs/scripts/"

  install -D -m0755 "${srcdir}/kfs-chunkserv.rc" "${pkgdir}/etc/rc.d/kfs-chunkserv"
  install -D -m0755 "${srcdir}/kfs-metaserv.rc" "${pkgdir}/etc/rc.d/kfs-metaserv"
  install -D -m0644 "${srcdir}/kfs.confd" "${pkgdir}/etc/conf.d/kfs.default"
  install -D -m0644 "${srcdir}/kfs.conf" "${pkgdir}/etc/kfs.conf.default"
  install -D -m0644 "${srcdir}/kfswebui.conf" "${pkgdir}/etc/kfswebui.conf.default"

  cp -a "${srcdir}/kfs-${pkgver}/webui/files" "${pkgdir}/usr/share/kfs/webui/"

  install -D -m0755 "${srcdir}/kfs-${pkgver}/webui/kfsstatus.py" "${pkgdir}/usr/share/kfs/webui/"

  ( cd "${srcdir}/kfs-${pkgver}/src/cc/access" &&
    python2 kfs_setup.py "${srcdir}/build/lib/" build &&
    python2 kfs_setup.py "${srcdir}/build/lib/" install --prefix=/usr --root="${pkgdir}" ) || return 1
}
