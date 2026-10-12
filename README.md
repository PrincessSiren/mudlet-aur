# mudlet-aur

An Arch Linux PKGBUILD for [Mudlet](https://www.mudlet.org) 5.0.1, the MUD client.

The [`mudlet` AUR package](https://aur.archlinux.org/packages/mudlet) is stuck at 4.19.1 and no longer builds, because upstream removed that release's tarball. This is a rewrite for Mudlet 5, which moved to Qt 6 and CMake. I plan to take over the AUR package once AUR registration reopens; until then, you can build it from here.

## Install

```sh
git clone https://github.com/PrincessSiren/mudlet-aur.git
cd mudlet-aur
makepkg -si
```

This replaces an installed 4.19.1 `mudlet` package. The compile takes a while.

## What's tested

- Builds in a clean chroot (`extra-x86_64-build`) with only official-repo dependencies, so no AUR helpers are needed.
- namcap reports no errors.
- At runtime, the database API works (`db:create`, `db:add`, `db:fetch`), and the bundled Lua modules load (regex, JSON, zip, LuaSQL).

## How it differs from the 4.19.1 package

- **Qt 6 and CMake:** replace Qt 5 and qmake; upstream needs Qt 6.8.2 or newer.
- **Four Lua 5.1 modules bundled:** lrexlib-pcre2, LuaSQL-SQLite3 2.6.0, lua-zip and lua-yajl. They're built offline from pinned git tags and installed privately in `/usr/lib/mudlet/lib/`, so they can't clash with repo packages.
  - Mudlet's `DB.lua` breaks with LuaSQL 2.8, the version in the repo's `lua51-sql-sqlite`. That package can stay installed; Mudlet uses its own copy.
  - lua-yajl is only in the AUR, so bundling it keeps the build working in a clean chroot.
- **Launcher:** `/usr/bin/mudlet` is a small script that runs `/usr/lib/mudlet/mudlet`. Mudlet looks for its private modules next to the path it was started from, and a symlink would point that at `/usr/bin`.

## Licenses

The packaging files in this repo are licensed under [GPL-3.0-or-later](LICENSE).

The package itself is `GPL-2.0-or-later AND GPL-3.0-or-later AND MIT AND BSD-3-Clause`:

- **GPL-2.0-or-later:** Mudlet
- **GPL-3.0-or-later:** the bundled lcf code formatter
- **MIT:** the bundled Lua modules, edbee-lib and qt-tags-widget
- **BSD-3-Clause:** communi

The MIT and BSD license texts are installed to `/usr/share/licenses/mudlet/`.

## Credits

The earlier AUR package was maintained by rantnar. Its contributors are credited in the PKGBUILD header: Jaroslav Lichtblau, Christoph Zeiler, Ryon Sherman, Dardo Marasca, Kevin Kyzer and Xabre.
