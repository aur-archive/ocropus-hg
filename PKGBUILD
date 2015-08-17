# Contributor: Ilya Mezhirov <mezhirov@iupr.com>

pkgname=ocropus-hg
pkgver=50
pkgrel=1
pkgdesc="An OCR system for documents and books"
arch=('i686' 'x86_64')
url="http://code.google.com/p/ocropus/"
license=('APACHE')
makedepends=('mercurial' 'scons' 'swig')
depends=('giflib' 'sqlite3' 'python2' 'python2-numpy' 'sdl_gfx'
         'sdl_image' 'libjpeg-turbo' 'flann' 'pyopenfst-hg' 'ocropus-data-hg'
         'python2-scipy' 'python2-matplotlib' 'python-pytables' 
         'gcc') # yes, gcc - it calls gcc dynamically from 'pynative'
conflicts=('ocropus' 'iulib' 'iulib-hg' 'ocrofst' 'ocropy' 'ocropy-hg'
           'ocroswig' 'ocroswig-hg')
source=(ocrolseg.patch ocrorast.patch ocrofst.patch ocrolib-common.patch)
options=()
md5sums=('dae6a79a8832f9d28414ba45febb404a'
         '691c9ce8d9b048d8cc5f08a6fb60de38'
         'a6d347a6be6e0f16b7752e7c3ae720d8'
         'f788b8112d7213c1a994ade2bef2dcc6')



_hgroot="https://ocropus.googlecode.com/hg"
_hgrepo="ocropus"

_incdir="$pkgdir/usr/include"
_libdir="$pkgdir/usr/lib"
_swigopts=-I"$pkgdir/$(swig -swiglib)"

run_make() {
    make CXX="g++ -I$_incdir -L$_libdir" $@
}

run_scons() {
    scons iulib="$pkgdir/usr" prefix="$pkgdir/usr" sdl=1 || exit
    scons iulib="$pkgdir/usr" prefix="$pkgdir/usr" sdl=1 install
}

py_ext() {
    SWIG_OPTS=$_swigopts python2 setup.py build_ext \
        --include-dirs "$_incdir" \
        --library-dirs "$_libdir" || exit
    python2 setup.py install --root "$pkgdir"
}


build() {
    cd "$srcdir"
    msg "Connecting to Mercurial server...."

    if [ -d $_hgrepo ] ; then
        cd $_hgrepo
        hg pull -u || return 1
        msg "The local files are updated."
    else
        hg clone $_hgroot $_hgrepo || return 1
    fi

    msg "Mercurial checkout done or server timeout"

    rm -rf "$srcdir/$_hgrepo-build"
    cp -r "$srcdir/$_hgrepo" "$srcdir/$_hgrepo-build"
    cd "$srcdir/$_hgrepo-build"

    ##############################################
    patch -N -p1 -i "$srcdir/ocrolseg.patch"
    patch -N -p1 -i "$srcdir/ocrorast.patch"
    patch -N -p1 -i "$srcdir/ocrofst.patch"
    patch -N -p1 -i "$srcdir/ocrolib-common.patch"
    ##############################################

    cd iulib
    run_scons || exit
    cd pyswig
    py_ext || exit
    cd ../..

    cd ocrolseg
    run_make libocrolseg.a || exit
    py_ext || exit
    cd ..

    cd ocrorast
    run_make || exit
    py_ext || exit
    cd ..

    cd ocropy
    py_ext || exit
    cd ..

    cd llpy
    py_ext || exit
    cd ..

    cd ocrofst
        cd ocrofstll
        run_scons || exit
        cd ..
    py_ext || exit
    mv "$pkgdir/usr/bin/test-main" "$pkgdir/usr/bin/ocropus-test-main"
    cd ..

    # pyopenfst is provided by pyopenfst-hg package
    # model files are provided by ocropus-data-hg package
}
