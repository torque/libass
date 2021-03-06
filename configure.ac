AC_INIT(libass, 0.12.3)
AM_INIT_AUTOMAKE
AC_CONFIG_MACRO_DIR([m4])
# Disable Fortran checks
define([AC_LIBTOOL_LANG_F77_CONFIG], [:])
LT_INIT
AC_CONFIG_SRCDIR([libass/ass.c])
AC_CONFIG_HEADER([config.h])

# Checks for programs.
AC_PROG_CC
AC_PROG_CPP
AM_PROG_CC_C_O
AM_PROG_AS

# Checks for header files.
AC_HEADER_STDC
AC_HEADER_STDBOOL
AC_CHECK_HEADERS([stdint.h unistd.h iconv.h])

# Checks for library functions.
AC_CHECK_FUNCS([strdup strndup])

# Checks for libraries.
AC_SEARCH_LIBS([libiconv_open], [iconv], AC_DEFINE(CONFIG_ICONV, 1, [use iconv]))
AC_SEARCH_LIBS([iconv_open], [iconv], AC_DEFINE(CONFIG_ICONV, 1, [use iconv]))
AC_CHECK_LIB([m], [fabs])

# Check for libraries via pkg-config
AC_ARG_ENABLE([test], AS_HELP_STRING([--enable-test],
    [enable test program (requires libpng) @<:@default=no@:>@]))
AC_ARG_ENABLE([profile], AS_HELP_STRING([--enable-profile],
    [enable profiling program @<:@default=no@:>@]))
AC_ARG_ENABLE([enca], AS_HELP_STRING([--disable-enca],
    [disable enca (charset autodetect) support @<:@default=check@:>@]))
AC_ARG_ENABLE([fontconfig], AS_HELP_STRING([--disable-fontconfig],
    [disable fontconfig support @<:@default=enabled@:>@]))
AC_ARG_ENABLE([directwrite], AS_HELP_STRING([--disable-directwrite],
    [disable DirectWrite support (win32 only) @<:@default=check@:>@]))
AC_ARG_ENABLE([coretext], AS_HELP_STRING([--disable-coretext],
    [disable CoreText support (OSX only) @<:@default=check@:>@]))
AC_ARG_ENABLE([require-system-font-provider], AS_HELP_STRING([--disable-require-system-font-provider],
    [allow compilation even if no system font provider was found @<:@default=enabled:>@]))
AC_ARG_ENABLE([harfbuzz], AS_HELP_STRING([--disable-harfbuzz],
    [disable HarfBuzz support @<:@default=check@:>@]))
AC_ARG_ENABLE([asm], AS_HELP_STRING([--disable-asm],
    [disable compiling with ASM @<:@default=check@:>@]))
AC_ARG_ENABLE([rasterizer], AS_HELP_STRING([--disable-rasterizer],
    [disable internal rasterizer @<:@default=enabled@:>@]))
AC_ARG_ENABLE([large-tiles], AS_HELP_STRING([--enable-large-tiles],
    [use larger tiles in the rasterizer (better performance, slightly worse quality) @<:@default=disabled@:>@]))

AS_IF([test x$enable_asm != xno], [
    AS_CASE([$host],
        [i?86-*], [
            INTEL=true
            AS=yasm
            X86=true
            BITS=32
            ASFLAGS="$ASFLAGS -DARCH_X86_64=0 -m x86" ],
        [x86_64-*|amd64-*], [
            AS=yasm
            INTEL=true
            X64=true
            BITS=64
            ASFLAGS="$ASFLAGS -DARCH_X86_64=1 -m amd64" ],
        )
    AS_CASE([$host],
        [*darwin*], [
            ASFLAGS="$ASFLAGS -f macho$BITS -DPREFIX -DHAVE_ALIGNED_STACK=1" ],
        [*linux*|*dragonfly*|*bsd*], [
            ASFLAGS="$ASFLAGS -f elf -DHAVE_ALIGNED_STACK=1" ],
        [*cygwin*|*mingw*], [
            ASFLAGS="$ASFLAGS -f win$BITS"
            AS_IF([test x$BITS = x64], [
                ASFLAGS="$ASFLAGS -DHAVE_ALIGNED_STACK=1"
            ], [
                ASFLAGS="$ASFLAGS -DHAVE_ALIGNED_STACK=0 -DPREFIX"
            ])
        ])
    AS_IF([test x$INTEL = xtrue], [
        AC_CHECK_PROG([yasm_check], [$AS], [yes])
        AS_IF([test x$yasm_check != xyes], [
            AC_MSG_WARN(yasm was not found; ASM functions are disabled.)
            AC_MSG_WARN(Install yasm for a significantly faster libass build.)
            enable_asm=no
        ], [
            AC_MSG_CHECKING([if $AS supports vpmovzxwd])
            echo "vpmovzxwd ymm0, xmm0" > conftest.asm
            AS_IF([$AS conftest.asm $ASFLAGS -o conftest.o >conftest.log 2>&1], [
                AC_MSG_RESULT([yes])
            ], [
                AC_MSG_RESULT([no])
                VER=`($AS --version || echo no assembler) 2>/dev/null | head -n 1`
                AC_MSG_WARN([yasm is too old (found $VER); ASM functions are disabled.])
                AC_MSG_WARN([Install yasm-1.2.0 or later for a significantly faster libass build.])
                enable_asm=no
            ])
            rm conftest.asm conftest.o > /dev/null 2>&1
        ])
    ])
])

AC_SUBST([ASFLAGS], ["$ASFLAGS"])
AC_SUBST([AS], ["$AS"])

AM_CONDITIONAL([ASM], [test x$enable_asm != xno])
AM_CONDITIONAL([INTEL], [test x$INTEL = xtrue])
AM_CONDITIONAL([X86], [test x$X86 = xtrue])
AM_CONDITIONAL([X64], [test x$X64 = xtrue])

AM_COND_IF([ASM],
    [AC_DEFINE(CONFIG_ASM, 1, [ASM enabled])],
    [AC_DEFINE(CONFIG_ASM, 0, [ASM enabled])]
    )

AM_CONDITIONAL([RASTERIZER], [test x$enable_rasterizer != xno])

AM_COND_IF([RASTERIZER],
    [AC_DEFINE(CONFIG_RASTERIZER, 1, [rasterizer enabled])],
    [AC_DEFINE(CONFIG_RASTERIZER, 0, [rasterizer enabled])]
    )

AM_CONDITIONAL([ENABLE_LARGE_TILES], [test x$enable_large_tiles = xyes])

AM_COND_IF([ENABLE_LARGE_TILES],
    [AC_DEFINE(CONFIG_LARGE_TILES, 1, [use large tiles])]
    [AC_DEFINE(CONFIG_LARGE_TILES, 0, [use small tiles])],
    )

PKG_CHECK_MODULES([FREETYPE], freetype2 >= 9.10.3, [
    CFLAGS="$CFLAGS $FREETYPE_CFLAGS"
    CXXFLAGS="$CFLAGS $FREETYPE_CFLAGS"
    LIBS="$LIBS $FREETYPE_LIBS"
    AC_DEFINE(CONFIG_FREETYPE, 1, [found freetype2 via pkg-config])
    ])

PKG_CHECK_MODULES([FRIBIDI], fribidi >= 0.19.0, [
    CFLAGS="$CFLAGS $FRIBIDI_CFLAGS"
    LIBS="$LIBS $FRIBIDI_LIBS"
    AC_DEFINE(CONFIG_FRIBIDI, 1, [found fribidi via pkg-config])
    ])

if test x$enable_fontconfig != xno; then
PKG_CHECK_MODULES([FONTCONFIG], fontconfig >= 2.4.2, [
    CFLAGS="$CFLAGS $FONTCONFIG_CFLAGS"
    LIBS="$LIBS $FONTCONFIG_LIBS"
    AC_DEFINE(CONFIG_FONTCONFIG, 1, [found fontconfig via pkg-config])
	fontconfig=true
    ], [fontconfig=false])
fi

if test x$enable_coretext != xno; then
OLDLIBS="$LIBS"
# Linking to CoreText directly only works from Mountain Lion and iOS6. In
# earlier OS releases CoreText was part of the ApplicationServices umbrella
# framework.
LIBS="$LIBS -framework CoreText -framework CoreFoundation -framework CoreGraphics"
AC_MSG_CHECKING([for CORETEXT])
AC_LINK_IFELSE([
  AC_LANG_PROGRAM(
    [[#include <CoreText/CoreText.h>]],
    [[CTFontCreateWithFontDescriptor(NULL, 0.0, NULL);]],)
  ], [
    AC_DEFINE(CONFIG_CORETEXT, 1, [found CoreText in System library])
    coretext=true
    AC_MSG_RESULT([yes])
  ], [
    LIBS="$OLDLIBS"
    coretext=false
    AC_MSG_RESULT([no])
  ])
fi
AM_CONDITIONAL([CORETEXT], [test x$coretext = xtrue])



if test x$enable_directwrite != xno; then
# Linking to DirectWrite directly only works from Windows
AC_MSG_CHECKING([for DIRECTWRITE])
AC_LINK_IFELSE([
  AC_LANG_PROGRAM(
    [[#include <windows.h>]],
    [[;]],)
  ], [
    AC_DEFINE(CONFIG_DIRECTWRITE, 1, [found DirectWrite])
    directwrite=true
    AC_MSG_RESULT([yes])
  ], [
    directwrite=false
    AC_MSG_RESULT([no])
  ])
fi
AM_CONDITIONAL([DIRECTWRITE], [test x$directwrite = xtrue])

if test x$enable_harfbuzz != xno; then
PKG_CHECK_MODULES([HARFBUZZ], harfbuzz >= 0.9.5, [
    CFLAGS="$CFLAGS $HARFBUZZ_CFLAGS"
    LIBS="$LIBS $HARFBUZZ_LIBS"
    AC_DEFINE(CONFIG_HARFBUZZ, 1, [found harfbuzz-ng via pkg-config])
	harfbuzz=true
    ], [harfbuzz=false])
fi

if test x$enable_enca != xno; then
PKG_CHECK_MODULES([ENCA], enca, [
    CFLAGS="$CFLAGS $ENCA_CFLAGS"
    LIBS="$LIBS $ENCA_LIBS"
    AC_DEFINE(CONFIG_ENCA, 1, [found enca via pkg-config])
	enca=true
	], [enca=false])
fi

libpng=false
if test x$enable_test = xyes; then
PKG_CHECK_MODULES([LIBPNG], libpng >= 1.2.0, [
    CFLAGS="$CFLAGS $LIBPNG_CFLAGS"
    AC_DEFINE(CONFIG_LIBPNG, 1, [found libpng via pkg-config])
    libpng=true])
fi

AM_CONDITIONAL([HAVE_LIBPNG], [test x$libpng = xtrue])

AM_CONDITIONAL([ENABLE_PROFILE], [test x$enable_profile = xyes])

# add libraries/packages to pkg-config for static linking
pkg_libs="-lm"
pkg_requires="freetype2 >= 9.10.3"
pkg_requires="fribidi >= 0.19.0, ${pkg_requires}"
if test x$enca = xtrue; then
    pkg_requires="enca, ${pkg_requires}"
fi
if test x$fontconfig = xtrue; then
    pkg_requires="fontconfig >= 2.4.2, ${pkg_requires}"
fi
if test x$harfbuzz = xtrue; then
    pkg_requires="harfbuzz >= 0.9.5, ${pkg_requires}"
fi
if test x$directwrite = xtrue; then
    pkg_libs="${pkg_libs} -ldwrite"
fi

if test x$enable_require_system_font_provider != xno &&
   test x$fontconfig != xtrue &&
   test x$directwrite != xtrue &&
   test x$coretext != xtrue
then
    AC_MSG_ERROR([\
Either DirectWrite (on Windows), CoreText (on OSX), or Fontconfig\
(Linux, other) is required. If you really want to compile without\
a system font provider, add --disable-require-system-font-provider])
fi

AC_SUBST([PKG_LIBS_DEFAULT], [$(test x$enable_shared = xno && echo ${pkg_libs})])
AC_SUBST([PKG_REQUIRES_DEFAULT], [$(test x$enable_shared = xno && echo ${pkg_requires})])
AC_SUBST([PKG_LIBS_PRIVATE], [$(test x$enable_shared = xno || echo ${pkg_libs})])
AC_SUBST([PKG_REQUIRES_PRIVATE], [$(test x$enable_shared = xno || echo ${pkg_requires})])

# Setup output beautifier.
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])

AC_CONFIG_FILES([Makefile libass/Makefile test/Makefile profile/Makefile libass.pc])
AC_OUTPUT
