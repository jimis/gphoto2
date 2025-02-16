dnl Process this file with autoreconf to produce a configure script.
AC_PREREQ([2.69])

dnl Next stable release must be 2.5.29 or later
AC_INIT([gphoto2 command line interface],
        [2.5.28.1],
        [gphoto-devel@lists.sourceforge.net],
        [gphoto2])

AC_CONFIG_SRCDIR([gphoto2/main.c])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_MACRO_DIR([auto-m4])
AC_CONFIG_AUX_DIR([auto-aux])
AM_INIT_AUTOMAKE([
  -Wall
  -Werror
  gnu
  1.14
  dist-bzip2
  dist-xz
  check-news
])

GP_CHECK_SHELL_ENVIRONMENT
GP_CONFIG_MSG([Build])
GP_CONFIG_MSG([Source code location], [${srcdir}])
AM_CPPFLAGS=""
AM_LDFLAGS=""


dnl ---------------------------------------------------------------------------
dnl Some programs we need
dnl ---------------------------------------------------------------------------
AC_PROG_CC
AC_PROG_CPP
AM_PROG_CC_C_O
AC_PROG_INSTALL
AC_SYS_LARGEFILE

dnl Check for and set up libtool
dnl
dnl We use libtool to allow linking against uninstalled *.la libraries
dnl in gphoto-suite.
LT_PREREQ([2.4.2])
LT_INIT([
  disable-static
  dlopen
  win32-dll
])

GP_CONFIG_MSG([Compiler], [${CC}])
AC_DEFINE_UNQUOTED([HAVE_CC], ["$CC"], [The C compiler we are using])

dnl AC_STRUCT_TIMEZONE


GP_CONFIG_MSG([Features])


# ----------------------------------------------------------------------
# References to external resources (web sites, mailing listes, etc.)
# ----------------------------------------------------------------------

GP_REFERENCES()dnl


dnl ---------------------------------------------------------------------------
dnl i18n support
dnl ---------------------------------------------------------------------------
GP_GETTEXT_SETUP([GETTEXT_PACKAGE_GPHOTO2],
                 [gphoto2],
                 [po])
AM_PO_SUBDIRS()
AM_GNU_GETTEXT_VERSION([0.19.1])
AM_GNU_GETTEXT([external])
AM_ICONV()

GP_CONFIG_MSG([Use translations],
              [${USE_NLS}])

dnl GP_GETTEXT_HACK([],[Lutz Müller and others],[${MAIL_GPHOTO_TRANSLATION}])


dnl ---------------------------------------------------------------------------
dnl Check for libgphoto2
dnl ---------------------------------------------------------------------------
PKG_CHECK_MODULES([LIBGPHOTO2], [libgphoto2 >= 2.5.17], [dnl
], [dnl
AC_MSG_ERROR([
pkg-config cannot find libgphoto2, but gphoto2 requires libgphoto2 to build
])
])
GP_SHOW_MODULE_VARS([LIBGPHOTO2])


AC_CHECK_FUNCS([strptime])

AC_CHECK_FUNCS([signal pause sleep])


AC_HEADER_STDC
AC_CHECK_HEADERS([fcntl.h process.h signal.h sys/time.h sys/wait.h])

AC_CHECK_FUNCS([spawnve])

AC_CHECK_LIB([m], [floor])


dnl ---------------------------------------------------------------------------
dnl pthread: The gphoto2 shell (--shell) keeps up a connection to the
dnl          camera. Some cameras need to be sent keep-alive-messages
dnl          in order to prevent a shut-down. This is what we need
dnl          pthread for.
dnl ---------------------------------------------------------------------------
pthread_msg=no
try_pthread=:
AC_ARG_WITH([pthread],
            [AS_HELP_STRING([--without-pthread],
                            [Do not use pthread])],
            [AS_VAR_IF([withval], [no], [try_pthread=false])])
AS_IF([$try_pthread], [dnl
    AC_CHECK_HEADER([pthread.h], [dnl
        AC_CHECK_LIB([pthread], [pthread_exit], [dnl
            AC_DEFINE([HAVE_PTHREAD], [1], [Define if we use pthread.h.])
            pthread_msg="yes (-lpthread)"
            PTHREAD_LIBS="-lpthread"
        ], [dnl
            AC_CHECK_LIB([c_r], [pthread_create], [dnl
                AC_DEFINE([HAVE_PTHREAD], [1], [Define if we use pthread.h.])
                pthread_msg="yes (-pthread)"
                PTHREAD_LIBS="-pthread"
            ])
        ])
    ])
])
AC_SUBST([PTHREAD_LIBS])
GP_CONFIG_MSG([pthread support], [$pthread_msg])


dnl ---------------------------------------------------------------------------
dnl CDK: If you would like to interactively access the camera's configuration
dnl      using gphoto2, the command-line frontend, you need CDK. For pure
dnl      command-line access to the config, you don't need CDK, though.
dnl      Old versions of CDK don't include fscale, therefore check for it here.
dnl ---------------------------------------------------------------------------
cdk_msg="no (http://dickey.his.com/cdk/cdk.html)"
try_cdk=:
have_cdk=false
AC_ARG_WITH([cdk],
        [AS_HELP_STRING([--without-cdk], [Do not use cdk])],
        [AS_VAR_IF([withval], [no], [dnl
             try_cdk=false
             cdk_msg="no (not requested)"
         ])
])
cdk_prefix=$ac_default_prefix
AC_ARG_WITH([cdk-prefix],
            [AS_HELP_STRING([--with-cdk-prefix=PREFIX], [Location of cdk])],
            [cdk_prefix="$withval"])
AS_IF([$try_cdk], [
    CPPFLAGS_save="$CPPFLAGS"
    CPPFLAGS="-I$cdk_prefix/include $CPPFLAGS"
    AC_CHECK_HEADERS([cdk/cdk.h cdk.h], [dnl
        have_cdk=:
        AC_PATH_PROG([CDK_CONFIG], [cdk5-config])
        AS_IF([test "x${CDK_CONFIG}" != x], [dnl
            CDK_CFLAGS=`$CDK_CONFIG --cflags`
            CDK_LIBS=`$CDK_CONFIG --libs`
        ], [dnl
            CDK_CFLAGS="-I$cdk_prefix/include"
            CDK_LIBS="-L$cdk_prefix/lib -lcdk -lncurses"
        ])
        AC_DEFINE([HAVE_CDK], [1], [whether we have CDK])
        LDFLAGS_save="$LDFLAGS"
        LDFLAGS="$CDK_LIBS"
        AC_CHECK_LIB([cdk], [newCDKFScale], [cdk_msg="yes"], [dnl
            AC_DEFINE([HAVE_CDK_20010421], [1], [whether we have old CDK])
            cdk_msg="yes (but without fscale)"
        ])
        LDFLAGS="$LDFLAGS_save"
    ])
    CPPFLAGS="$CPPFLAGS_save"
])
AM_CONDITIONAL([HAVE_CDK], [$have_cdk])
AC_SUBST([CDK_CFLAGS])
AC_SUBST([CDK_LIBS])
GP_CONFIG_MSG([Interactive config support], [$cdk_msg])
GP_SHOW_MODULE_VARS([CDK])

dnl We emulate signals in Windows
dnl This is taken from
dnl     https://github.com/OpenVPN/openvpn/blob/master/configure.ac
AC_CHECK_DECLS(
	[SIGHUP],
	,
	[AC_DEFINE([SIGHUP], [1], [SIGHUP replacement])],
	[[
		#ifdef HAVE_SIGNAL_H
		#include <signal.h>
		#endif
	]]
)
AC_CHECK_DECLS(
	[SIGINT],
	,
	[AC_DEFINE([SIGINT], [2], [SIGINT replacement])],
	[[
		#ifdef HAVE_SIGNAL_H
		#include <signal.h>
		#endif
	]]
)
AC_CHECK_DECLS(
	[SIGUSR1],
	,
	[AC_DEFINE([SIGUSR1], [10], [SIGUSR1 replacement])],
	[[
		#ifdef HAVE_SIGNAL_H
		#include <signal.h>
		#endif
	]]
)
AC_CHECK_DECLS(
	[SIGUSR2],
	,
	[AC_DEFINE([SIGUSR2], [12], [SIGUSR2 replacement])],
	[[
		#ifdef HAVE_SIGNAL_H
		#include <signal.h>
		#endif
	]]
)
AC_CHECK_DECLS(
	[SIGTERM],
	,
	[AC_DEFINE([SIGTERM], [15], [SIGTERM replacement])],
	[[
		#ifdef HAVE_SIGNAL_H
		#include <signal.h>
		#endif
	]]
)

dnl ---------------------------------------------------------------------------
dnl libjpeg: Right now, libjpeg is only used to interpret JPEG previews in
dnl          order to feed them to aalib (see below).
dnl ---------------------------------------------------------------------------
jpeg_msg=no
try_jpeg=:
have_jpeg=false
AC_ARG_WITH([jpeg],
            [AS_HELP_STRING([--without-jpeg], [Do not use jpeg])],
            [AS_VAR_IF([withval], [no], [dnl
               try_jpeg=false
              jpeg_msg="no (not requested)"
             ])
])
jpeg_prefix=$ac_default_prefix
AC_ARG_WITH([jpeg-prefix],
            [AS_HELP_STRING([--with-jpeg-prefix=PREFIX], [Location of jpeg])],
            [jpeg_prefix="$withval"])
AS_IF([$try_jpeg], [dnl
    CPPFLAGS_save="$CPPFLAGS"
    CPPFLAGS="-I$jpeg_prefix/include"
    AC_CHECK_HEADER([jpeglib.h], [dnl
        AC_CHECK_LIB([jpeg], [jpeg_read_scanlines], [dnl
            have_jpeg=:
            jpeg_msg="yes"
            AC_DEFINE([HAVE_JPEG],1,[whether we use libjpeg])
            JPEG_LIBS="-ljpeg"
            JPEG_CFLAGS=$CPPFLAGS
        ], [dnl
            LDFLAGS_save="$LDFLAGS"
            LDFLAGS="-L$jpeg_prefix/lib"
            AC_CHECK_LIB([jpeg], [jpeg_stdio_src], [dnl
                have_jpeg=:
                jpeg_msg="yes (in '$jpeg_prefix')"
                AC_DEFINE([HAVE_JPEG], [1], [whether we use libjpeg])
                JPEG_LIBS="-L$jpeg_prefix/lib -ljpeg"
                JPEG_CFLAGS=$CPPFLAGS
            ], [dnl
                jpeg_msg="no (could not link)"
            ])
            LDFLAGS=$LDFLAGS_save
        ])
    ])
    CPPFLAGS="$CPPFLAGS_save"
])
AM_CONDITIONAL([HAVE_JPEG], [$have_jpeg])
AC_SUBST([JPEG_LIBS])
AC_SUBST([JPEG_CFLAGS])
GP_CONFIG_MSG([JPEG support], [$jpeg_msg])
GP_SHOW_MODULE_VARS([JPEG])


dnl ---------------------------------------------------------------------------
dnl libexif: The CameraFilesystem can use libexif for extracting thumbnails
dnl          out of EXIF data. Similarly, it can extract the mtime of
dnl          a file.
dnl          libexif is available from
dnl          http://www.sourceforge.net/projects/libexif
dnl ---------------------------------------------------------------------------
dnl
dnl 0.3.3 oldest supported version
dnl 0.5.3 introduces exif-ifd.h which we use
dnl 0.6.9 introduces exif-log.h which we use
PKG_CHECK_MODULES([LIBEXIF], [libexif >= 0.6.9], [dnl
    AC_DEFINE([HAVE_LIBEXIF], [1], [whether building with libexif])
], [dnl
    AC_MSG_WARN([building gphoto2 *without* libexif])
])
GP_SHOW_MODULE_VARS([LIBEXIF])


dnl ---------------------------------------------------------------------------
dnl aalib: gphoto2 (the frontend) has been designed as a command-line tool.
dnl        However, live previews make only sense when you can actually
dnl        see the preview. This is where aalib fits in - it gives you the
dnl        possibility to view live previews on any console.
dnl ---------------------------------------------------------------------------
aa_msg="no (http://www.sourceforge.net/projects/aa-project)"
try_aa=:
have_aa=false
AC_ARG_WITH([aalib],
            [AS_HELP_STRING([--with-aalib],
                            [Do use aalib for preview ASCII rendering])],
            [AS_VAR_IF([withval], [no], [dnl
                 try_aa=false
                 aa_msg="no (not requested)"
             ])
])
AS_IF([$try_aa], [dnl
    AC_CHECK_HEADER([aalib.h], [dnl
        have_aa=:
        aa_msg=yes
        AC_DEFINE([HAVE_AA], [1], [whether we use aalib])
        AC_PATH_PROG([AALIB_CONFIG], [aalib-config])
        AS_IF([test "x${AALIB_CONFIG}" != x], [dnl
            AA_CFLAGS=`$AALIB_CONFIG --cflags`
            AA_LIBS=`$AALIB_CONFIG --libs`
        ], [dnl
            AA_LIBS="-laa"
        ])
    ])
])
AM_CONDITIONAL([HAVE_AA], [$have_aa])
AC_SUBST([AA_LIBS])
AC_SUBST([AA_CFLAGS])
GP_CONFIG_MSG([Text preview support], [$aa_msg])
GP_SHOW_MODULE_VARS([AA])


dnl ---------------------------------------------------------------------------
dnl popt.h: Simplifies handling of command-line options enormously.
dnl ---------------------------------------------------------------------------
dnl
dnl POPT_TABLEEND is only defined starting with popt 1.6.1
PKG_CHECK_MODULES([POPT], [popt >= 1.6.1], [dnl
], [dnl
AC_MSG_ERROR([
pkg-config could not find popt.pc >= 1.6.1 which gphoto2 requires to build
])
])
GP_SHOW_MODULE_VARS([POPT])


dnl ---------------------------------------------------------------------------
dnl readline: Readline is pretty cool - it allows you to use <tab> to complete
dnl           commands in the gphoto2 shell or to use <up> or <down> to
dnl           navigate through the history.
dnl ---------------------------------------------------------------------------
rl_msg="no (http://cnswww.cns.cwru.edu/~chet/readline/rltop.html)"
try_readline=:
have_readline=false
AC_ARG_WITH([readline],
            [AS_HELP_STRING([--without-readline], [Do not use readline])],
            [AS_VAR_IF([withval], [no], [dnl
                 try_readline=false
                 rl_msg="no (not requested)"
             ])
])
AS_IF([$try_readline], [dnl
    AC_CHECK_HEADER([readline/readline.h], [dnl
        AC_CHECK_LIB([readline], [rl_completion_append_character], [dnl
            AC_CHECK_LIB([readline], [rl_completion_matches], [dnl
                AC_CHECK_LIB([readline], [rl_copy_text], [dnl
                    have_readline=:
                    rl_msg="yes (doesn't require explicit -lncurses)"
                    RL_LIBS="-lreadline"
                    RL_CFLAGS=""
                    AC_DEFINE([HAVE_RL], [1], [whether we use readline])
                ], [dnl
                    rl_msg="no (available version too old or not GNU readline)"
                ])
            ])
        ], [dnl
            LDFLAGS_save="$LDFLAGS"
            LDFLAGS="-lncurses"
            AC_CHECK_LIB([readline], [rl_completion_matches], [dnl
                have_readline=:
                rl_msg="yes (requires explicit -lncurses)"
                RL_LIBS="-lreadline -lncurses"
                RL_CFLAGS=""
                AC_DEFINE([HAVE_RL], [1], [whether we use readline])
            ], [dnl
                rl_msg="no (couldn't link against it or version too old)"
            ])
            LDFLAGS="$LDFLAGS_save"
        ])
    ])
])
AM_CONDITIONAL([HAVE_RL], [$have_readline])
AC_SUBST([RL_LIBS])
AC_SUBST([RL_CFLAGS])
GP_SHOW_MODULE_VARS([RL])
GP_CONFIG_MSG([Readline support], [$rl_msg])


dnl ---------------------------------------------------------------------------
dnl Turn on all warnings when using gcc
dnl ---------------------------------------------------------------------------
AS_VAR_IF([GCC], [yes], [dnl
    CFLAGS="$CFLAGS -Wall"
    CFLAGS="$CFLAGS -Wmissing-declarations -Wmissing-prototypes"
    # FIXME: Check for --enable-debug here
    CFLAGS="$CFLAGS -g"
    LDFLAGS="$LDFLAGS -g"
])

AM_CPPFLAGS="$AM_CPPFLAGS -D_GPHOTO2_INTERNAL_CODE"
AC_SUBST([AM_CPPFLAGS])
AC_SUBST([AM_LDFLAGS])


# ---------------------------------------------------------------------------
# Create output files
# ---------------------------------------------------------------------------
AC_CONFIG_FILES([
  po/Makefile.in
  Makefile
  gphoto2/Makefile
  gphoto-m4/Makefile
  doc/Makefile
  contrib/Makefile
  packaging/Makefile
  packaging/rpm/Makefile
  packaging/rpm/package.spec
  tests/data/Makefile
  tests/staging/subdir1/Makefile
  tests/staging/subdir2/Makefile
  tests/staging/Makefile
  tests/Makefile
])
AC_OUTPUT()dnl

# Status output
GP_CONFIG_OUTPUT

dnl Local Variables:
dnl indent-tabs-mode: nil
dnl End:
