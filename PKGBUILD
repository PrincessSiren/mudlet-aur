# SPDX-License-Identifier: GPL-3.0-or-later
# Contributor: PrincessSiren <48842744+PrincessSiren@users.noreply.github.com>
# Contributor: Jaroslav Lichtblau <dragonlord@aur.archlinux.org>
# Contributor: Christoph Zeiler <archNOSPAM_at_moonblade.dot.org>
# Contributor: Ryon Sherman <ryon.sherman@gmail.com>
# Contributor: Dardo Marasca <gefarion@gmail.com>
# Contributor: Kevin Kyzer <kev@k3v.in>
# Contributor: Xabre <xabre @archlinux.info>

pkgname=mudlet
pkgver=5.1.1
pkgrel=1
_lrexlib_ver=2.9.4
_luasql_ver=2.6.0     # Mudlet's DB.lua breaks on LuaSQL >= 2.8 (system lua51-sql-sqlite)
_luazip_ver=0.2.0
_luayajl_ver=2.0
pkgdesc="A modern MUD client with a graphical user interface and built-in Lua scripting"
arch=('x86_64')
url="https://www.mudlet.org"
license=('GPL-2.0-or-later' 'GPL-3.0-or-later' 'MIT' 'BSD-3-Clause')
depends=('qt6-base' 'qt6-5compat' 'qt6-multimedia' 'qt6-speech' 'qt6-svg' 'qt6-tools'
         'qtkeychain-qt6' 'lua51' 'lua51-filesystem' 'lua51-lpeg' 'lua51-luautf8'
         'assimp' 'hunspell' 'libzip' 'oniguruma' 'pcre2' 'pugixml' 'sqlite' 'yajl'
         'glu' 'ttf-font')
makedepends=('cmake' 'ninja' 'boost' 'luarocks' 'git')
optdepends=('ttf-bitstream-vera: default font'
            'ttf-ubuntu-font-family: default font'
            'noto-fonts-emoji: emoji font support'
            'hunspell-en_us: spell checking')
conflicts=('mudlet-git')
source=("https://www.mudlet.org/wp-content/files/Mudlet-${pkgver}.tar.xz"
        "lrexlib::git+https://github.com/rrthomas/lrexlib.git#tag=rel-${_lrexlib_ver//./-}"
        "luasql::git+https://github.com/lunarmodules/luasql.git#tag=${_luasql_ver}"
        "lua-zip::git+https://github.com/brimworks/lua-zip.git#tag=v${_luazip_ver}"
        "lua-yajl::git+https://github.com/brimworks/lua-yajl.git#tag=v${_luayajl_ver}"
        "https://luarocks.org/lrexlib-pcre2-${_lrexlib_ver}-1.rockspec"
        "https://luarocks.org/luasql-sqlite3-2.6.1-3.rockspec"
        "https://luarocks.org/lua-zip-0.2-0.rockspec"
        "https://luarocks.org/lua-yajl-${_luayajl_ver}-1.rockspec")
noextract=("Mudlet-${pkgver}.tar.xz")
sha256sums=('5efab5442de50f835c9af0da8714b48627879a3e421f932817ad5eb95eb073b9'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            '85156588dd70480dc02b99d6fdc1a4cbf8b8b0a9f9a3febf524e3b193094756b'
            'cb4f274bb9062becefa4a2e5c63159d09bad5b71bae959cce5cfeaca4867c8e1'
            '5528e825812c783297358a564608e55d010a638c6cdd67ca287bae935b342100'
            'b7b36b948763485787020e8f0a1e55282d3c7019798616d9f889e182df07c040')

prepare() {
    # The upstream tarball has no top-level directory
    mkdir -p mudlet
    bsdtar -xf "Mudlet-${pkgver}.tar.xz" -C mudlet
}

build() {
    # Lua 5.1 modules Arch doesn't package (or packages at an incompatible
    # version), built offline from the pinned sources above. luarocks would
    # otherwise bake an RPATH to the library search dir into each module.
    printf 'gcc_rpath = false\n' > "$srcdir/luarocks-config.lua"
    export LUAROCKS_CONFIG="$srcdir/luarocks-config.lua"
    local rock
    for rock in lrexlib:lrexlib-pcre2-${_lrexlib_ver}-1 luasql:luasql-sqlite3-2.6.1-3 lua-zip:lua-zip-0.2-0 \
                lua-yajl:lua-yajl-${_luayajl_ver}-1; do
        # lrexlib and libssh both export buffer_free, and libssh gets loaded
        # into Mudlet through Qt Multimedia's ffmpeg backend. Bind lrexlib's
        # own calls locally so they don't end up in libssh's copy.
        local libflag="-shared"
        [[ ${rock%%:*} == lrexlib ]] && libflag+=" -Wl,-Bsymbolic-functions"
        (cd "${rock%%:*}" && luarocks --lua-version 5.1 --tree "$srcdir/rocks" \
            make --deps-mode=none "$srcdir/${rock#*:}.rockspec" LIBFLAG="$libflag")
    done

    # CMake runs translations/translated/generate-translation-stats.lua, which
    # needs yajl; without it the language statistics in Settings are empty
    export LUA_CPATH="$srcdir/rocks/lib/lua/5.1/?.so;;"

    cd mudlet
    # Unset, CMake tags the build "-dev-<sha>"; upstream's release builds set
    # it to an empty string to get a plain release version
    export MUDLET_VERSION_BUILD=
    export WITH_OWN_QTKEYCHAIN=NO
    cmake -S . -B build -G Ninja \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DCMAKE_INSTALL_LIBDIR=lib \
        -DLUA_INCLUDE_DIR=/usr/include/lua5.1 \
        -DUSE_SANITIZER="" \
        -DBUILD_TESTING=OFF \
        -DUSE_UPDATER=OFF \
        -DUSE_FONTS=OFF
    cmake --build build
}

package() {
    cd mudlet
    DESTDIR="$pkgdir" cmake --install build

    # On Linux Mudlet prepends <binary dir>/lib/?.so to package.cpath, so
    # the rocks stay private and can't clash with system Lua 5.1 modules.
    # Qt takes <binary dir> from the path the program was started as, so a
    # symlink would make it /usr/bin; exec the real binary by its full path.
    install -d "$pkgdir/usr/lib/mudlet"
    mv "$pkgdir/usr/bin/mudlet" "$pkgdir/usr/lib/mudlet/mudlet"
    printf '#!/bin/sh\nexec /usr/lib/mudlet/mudlet "$@"\n' > "$pkgdir/usr/bin/mudlet"
    chmod 755 "$pkgdir/usr/bin/mudlet"
    cp -a "$srcdir/rocks/lib/lua/5.1" "$pkgdir/usr/lib/mudlet/lib"

    local licdir="$pkgdir/usr/share/licenses/$pkgname"
    install -Dm644 3rdparty/edbee-lib/LICENSE "$licdir/LICENSE.edbee-lib"
    install -Dm644 3rdparty/communi/LICENSE "$licdir/LICENSE.communi"
    install -Dm644 3rdparty/qt-tags-widget/LICENSE "$licdir/LICENSE.qt-tags-widget"
    install -Dm644 "$srcdir/lrexlib/LICENSE" "$licdir/LICENSE.lrexlib"
    install -Dm644 "$srcdir/luasql/doc/us/license.html" "$licdir/LICENSE.luasql.html"
    head -n 26 "$srcdir/lua-zip/README.txt" > "$licdir/LICENSE.lua-zip"
    head -n 26 "$srcdir/lua-yajl/README" > "$licdir/LICENSE.lua-yajl"
}
