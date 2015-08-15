# Maintainer: ponsfoot <cabezon dot hashimoto at gmail dot com>
# Contributor: Jaroslav Lichtblau <dragonlord@aur.archlinux.org>
# Contributor: Michal Krenek <mikos@sg1.cz>

## If you want to use X less version, uncomment below.
#_X="no"

_root=trunk
_mod=boinc
## If you want to use stable release, uncomment below 2 lines.
#_root=tags
#_mod=boinc_core_release_7_0_28

_apply_patch="yes"

_svntrunk="http://boinc.berkeley.edu/svn/${_root}/${_mod}"
_svnmod="$_mod"

pkgname=boinc-svn
pkgver=26153 
pkgrel=2
arch=('i686' 'x86_64')
url="http://boinc.berkeley.edu/"
license=('LGPL')
pkgdesc="Berkeley Open Infrastructure for Network Computing"
depends=('curl' 'sqlite3')
makedepends=('pkg-config' 'libxslt' 'perl-xml-sax' 'subversion')
provides=('boinc')
conflicts=('boinc' 'boinc-nox')
options=('!libtool')
install=boinc.install
changelog=ChangeLog
source=(boinc.rc
        boinc.bash
        boinc.desktop
        boinc.service
        r24240-ignore_ret_value.patch
        r25690-boinc-client-menu.patch
        r26153-disable_xss.patch
)
md5sums=('43605168e310f50cf426d2f9a7b39847'
         '05ed267db973ef7cbaf1118bb20bf9ce'
         'db62de2f08117e6379a3c613b58fa7ff'
         '485c1abef2c9b98ea589f35da67e437e'
         '68bb3dfac41c20251b543665659f0281'
         '19dc2fb68d5013fcf95c59fb528bd86e'
         'aacca6f4fe612edf767e699544b7a0d0')

if [[ "$_X" != "no" ]]; then
  depends+=('wxgtk>=2.8.11.0' 'libnotify')
fi

build() {
  cd "$srcdir"

  msg "Connecting to boinc.berkeley.edu SVN server..."
  if [[ -d ${_svnmod}/.svn ]]; then
    (cd $_svnmod ; svn update)
  else
    svn co $_svntrunk --config-dir ./ -r $pkgver $_svnmod
  fi
  msg "SVN checkout done or server timeout."

  msg "Starting make..."
  rm -rf "${srcdir}/${_svnmod}-build"
  cp -r "${srcdir}/${_svnmod}" "${srcdir}/${_svnmod}-build"
  cd "${srcdir}/${_svnmod}-build"

  if [[ "$_apply_patch" == "yes" ]]; then
    # prevent unnecessary changes in compile flags
    sed -i -e 's:BOINC_SET_COMPILE_FLAGS::' configure.ac
    # build fails if set -D_FORTIFY_SOURCE=2
    patch -p1 -i "${srcdir}/r24240-ignore_ret_value.patch"
    # disable xscreensaver
    patch -p1 -i "${srcdir}/r26153-disable_xss.patch"
    sed -i 's/^ *CLIENTGUI_SUBDIRS += clientscr/#&/' Makefile.am

    if [[ "$_mod" == "boinc_core_release_7_0_28" ]]; then
      # build fails if automake >=1.12
      sed -i -e 's:AC_PROG_CC:&\nAC_PROG_OBJCXX:' configure.ac
      # Force a redraw of the menu on advanced frame of boincmgr
      patch -p1 -i "${srcdir}/r25690-boinc-client-menu.patch"
    fi
  fi

  if [[ "$_X" == "no" ]]; then
    _copt="--disable-manager --without-wxdir"
  else
    _copt="--with-x --with-wxdir=/usr/lib --with-wx-config=`which wx-config`"
  fi

  #configure
  LC_ALL=C ./_autosetup # Possibility to fail ver. check depending on the localization.
  ./configure --prefix=/usr \
              --sysconfdir=/etc \
              --localstatedir=/var \
              --disable-server \
              --enable-unicode \
              --with-ssl \
              --enable-dynamic-client-linkage \
              --disable-static \
              $_copt
  make
}

package() {
  cd "${srcdir}/${_svnmod}-build"

  make DESTDIR=$pkgdir install

  #install unit file for systemd
  install -D -m644 "${srcdir}/boinc.service" ${pkgdir}/usr/lib/systemd/system/boinc.service

  #install rc-script
  install -D -m755 "${srcdir}/boinc.rc" ${pkgdir}/etc/rc.d/boinc

  #install bash-completion
  install -D -m644 "${srcdir}/boinc.bash" ${pkgdir}/usr/share/bash_completion.d/boinc

  if [[ "$_X" != "no" ]]; then
    #install .desktop File
    install -D -m644 "${srcdir}/boinc.desktop" \
      ${pkgdir}/usr/share/applications/boinc.desktop

    #install icons
    install -D -m644 "${srcdir}/${_svnmod}-build/clientgui/res/boincmgr.48x48.png" \
      ${pkgdir}/usr/share/pixmaps/boinc.png
  fi

  #killing /etc/init.d directory
  rm -rf ${pkgdir}/etc/init.d
}
