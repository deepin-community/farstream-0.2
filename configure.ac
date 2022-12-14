AC_PREREQ(2.60)

dnl please read gstreamer/docs/random/autotools before changing this file

dnl initialize autoconf
dnl releases only do -Wall, cvs and prerelease does -Werror too
dnl use a three digit version number for releases, and four for cvs/prerelease
AC_INIT(Farstream, 0.2.9,
    https://bugs.freedesktop.org/enter_bug.cgi?product=Farstream,
    farstream)

AG_GST_INIT

dnl initialize automake
AM_INIT_AUTOMAKE([-Wno-portability subdir-objects])

dnl define PACKAGE_VERSION_* variables
AS_VERSION

dnl check if this is a release version
AS_NANO(FS_GIT="no", FS_GIT="yes")

dnl can autoconf find the source ?
AC_CONFIG_SRCDIR([farstream/fs-conference.c])

dnl define the output header for config
AC_CONFIG_HEADERS([config.h])

dnl sets host_* variables
AC_CANONICAL_HOST

dnl use pretty build output with automake >= 1.11
m4_ifdef([AM_SILENT_RULES],[AM_SILENT_RULES([yes])],
  [AM_DEFAULT_VERBOSITY=1
   AC_SUBST(AM_DEFAULT_VERBOSITY)])

dnl our libraries and install dirs use major.minor as a version
FS_APIVERSION=0.2
FS_API_VERSION=0_2
AC_SUBST(FS_APIVERSION)
AC_SUBST(FS_API_VERSION)
AC_DEFINE_UNQUOTED(FS_APIVERSION, "$FS_APIVERSION",
  [Farstream x.y API version])
AC_DEFINE_UNQUOTED(FS_API_VERSION, "$FS_APIVERSION",
  [Farstream x_y API version])

GST_API_VERSION=1.0
AC_SUBST(GST_API_VERSION)
AC_DEFINE_UNQUOTED(GST_API_VERSION, "$GST_API_VERSION",
  [GStreamer major.minor version])


dnl CURRENT, REVISION, AGE
dnl - library source changed -> increment REVISION
dnl - interfaces added/removed/changed -> increment CURRENT, REVISION = 0
dnl - interfaces added -> increment AGE
dnl - interfaces removed -> AGE = 0
dnl sets FS_LT_LDFLAGS
AS_LIBTOOL(FS, 6, 1, 1)

dnl FIXME: this macro doesn't actually work;
dnl the generated libtool script has no support for the listed tags.
dnl So this needs to be fixed first if we want to use this
dnl AS_LIBTOOL_TAGS

AM_PROG_LIBTOOL

dnl *** required versions of GStreamer stuff ***
GST_REQ=1.4
GSTPB_REQ=1.4

dnl *** autotools stuff ****

dnl allow for different autotools
AS_AUTOTOOLS_ALTERNATE

dnl Add parameters for aclocal
AC_SUBST(ACLOCAL_AMFLAGS, "-I common/m4")

dnl *** check for arguments to configure ***

AG_GST_ARG_DEBUG
AG_GST_ARG_PROFILING
AG_GST_ARG_VALGRIND
AG_GST_ARG_GCOV

AG_GST_ARG_WITH_PKG_CONFIG_PATH
AG_GST_ARG_WITH_PACKAGE_NAME
AG_GST_ARG_WITH_PACKAGE_ORIGIN

dnl let distro override plugin install helper path
AC_ARG_WITH(install-plugins-helper,
  AC_HELP_STRING([--with-install-plugins-helper],
    [specify path of helper script to call to install plugins]),
  [
    case "${withval}" in
      yes) AC_MSG_ERROR(bad value ${withval} for --with-install-plugins-helper) ;;
      no)  AC_MSG_ERROR(bad value ${withval} for --with-install-plugins-helper) ;;
      *)   GST_INSTALL_PLUGINS_HELPER="${withval}" ;;
    esac
  ],
  [
    dnl Default value
    AS_AC_EXPAND(GST_INSTALL_PLUGINS_HELPER,${libexecdir}/gst-install-plugins-helper)
  ]
)
AC_MSG_NOTICE(Using $GST_INSTALL_PLUGINS_HELPER as plugin install helper)
AC_DEFINE_UNQUOTED(GST_INSTALL_PLUGINS_HELPER, "$GST_INSTALL_PLUGINS_HELPER",
    [plugin install helper script])
AC_SUBST(GST_INSTALL_PLUGINS_HELPER)

dnl these are all the gst plug-ins, compilable without additional libs
FS_PLUGINS_ALL=" \
	fsrawconference \
	fsrtpconference \
 	fsvideoanyrate \
 	fsrtpxdata \
	"
AC_SUBST(FS_PLUGINS_ALL)

FS_PLUGINS_SELECTED=""

AC_ARG_WITH(plugins,
    AC_HELP_STRING([--with-plugins],
      [comma-separated list of plug-ins to compile]),
    [for i in `echo $withval | tr , ' '`; do
        if echo $FS_PLUGINS_ALL | grep $i > /dev/null
        then
            FS_PLUGINS_SELECTED="$FS_PLUGINS_SELECTED $i"
        else
            echo "plug-in $i not recognized, ignoring..."
        fi
    done],
    [FS_PLUGINS_SELECTED=$FS_PLUGINS_ALL])

AC_SUBST(FS_PLUGINS_SELECTED)

dnl *** path for our local plugins ***


dnl these are all the transmitter plug-ins
FS_TRANSMITTER_PLUGINS_ALL=" \
	rawudp \
	multicast \
	nice \
	shm
	"
AC_SUBST(FS_TRANSMITTER_PLUGINS_ALL)

FS_TRANSMITTER_PLUGINS_SELECTED=""

TRANSMITTERS_DEFAULT=yes

AC_ARG_WITH(transmitter-plugins,
    AC_HELP_STRING([--with-transmitter-plugins],
      [comma-separated list of plug-ins to compile]),
    [for i in `echo $withval | tr , ' '`; do
        if echo $FS_TRANSMITTER_PLUGINS_ALL | grep $i > /dev/null
        then
            FS_TRANSMITTER_PLUGINS_SELECTED="$FS_TRANSMITTER_PLUGINS_SELECTED $i"
        else
            echo "plug-in $i not recognized, ignoring..."
        fi
    done
    TRANSMITTERS_DEFAULT=no],
    [FS_TRANSMITTER_PLUGINS_SELECTED=$FS_TRANSMITTER_PLUGINS_ALL])


NICE_REQUIRED=0.1.8

PKG_CHECK_MODULES(NICE, \
       nice >= $NICE_REQUIRED,, AC_MSG_ERROR([Need libnice >= $NICE_REQUIRED]))
AC_SUBST(NICE_CFLAGS)
AC_SUBST(NICE_LIBS)

AC_SUBST(FS_TRANSMITTER_PLUGINS_SELECTED)

dnl set the plugindir where plugins should be installed
AS_AC_EXPAND(FS_PLUGIN_PATH, ${libdir}/farstream-$FS_APIVERSION)
AC_SUBST(FS_PLUGIN_PATH)
AC_DEFINE_UNQUOTED(FS_PLUGIN_PATH, "${FS_PLUGIN_PATH}", [The path were plugins are installed and search by default])


dnl *** checks for platform ***

dnl * hardware/architecture *

AG_GST_ARCH

dnl *** checks for programs ***

dnl find a compiler
AC_PROG_CC

AC_PATH_PROG(VALGRIND_PATH, valgrind, no)
AM_CONDITIONAL(HAVE_VALGRIND, test ! "x$VALGRIND_PATH" = "xno")

dnl check for gobject-introspection
GOBJECT_INTROSPECTION_CHECK([0.10.1])

dnl check for documentation tools
GTK_DOC_CHECK([1.18])
AG_GST_PLUGIN_DOCS([1.8],[2.1])

dnl Make automake happy with per-target flags
AM_PROG_CC_C_O

dnl *** checks for libraries ***

dnl For interactive UNIX (a Sun thing)
dnl FIXME: this adds -lcposix to LIBS, but I doubt we use LIBS
AC_ISC_POSIX

dnl *** checks for header files ***

dnl check if we have ANSI C header files
AC_HEADER_STDC

dnl used in gst/ffmpegcolorspace/mem.c
dnl FIXME: could be fixed by redefining av_malloc and av_free to GLib's
AC_CHECK_HEADERS([malloc.h])

dnl *** checks for types/defines ***

dnl *** checks for structures ***

AC_CHECK_MEMBER([struct ip_mreqn.imr_ifindex],
	        [AC_DEFINE([HAVE_IP_MREQN], [1], [Have the struct ip_mreqn])],
		[],
		[
#include <sys/socket.h>
#include <netinet/in.h>
])


dnl *** checks for compiler characteristics ***

dnl *** checks for library functions ***

dnl *** checks for dependency libraries ***

dnl GLib is required
AG_GST_GLIB_CHECK([2.40])
AC_DEFINE(GLIB_VERSION_MIN_REQUIRED, GLIB_VERSION_2_40, [Ignore post 2.40 deprecations])
AC_DEFINE(GLIB_VERSION_MAX_ALLOWED, GLIB_VERSION_2_40, [Prevent post 2.40 APIs])


PKG_CHECK_MODULES(GIO_UNIX, gio-unix-2.0 >= 2.40, [HAVE_GIO_UNIX=true],[HAVE_GIO_UNIX=false])
AC_SUBST(GIO_UNIX_CFLAGS)
AC_SUBST(GIO_UNIX_LIBS)
AM_CONDITIONAL(HAVE_GIO_UNIX, test "x$HAVE_GIO_UNIX" = "xyes")

dnl checks for gstreamer
dnl uninstalled is selected preferentially -- see pkg-config(1)
AG_GST_CHECK_GST($GST_API_VERSION, [$GST_REQ])
AG_GST_CHECK_GST_BASE($GST_API_VERSION, [$GST_REQ])
AG_GST_CHECK_GST_CHECK($GST_API_VERSION, [$GST_REQ], no)
AG_GST_CHECK_GST_PLUGINS_BASE($GST_API_VERSION, [$GSTPB_REQ])
AM_CONDITIONAL(HAVE_GST_CHECK, test "x$HAVE_GST_CHECK" = "xyes")

dnl Check for documentation xrefs
GLIB_PREFIX="`$PKG_CONFIG --variable=prefix glib-2.0`"
FS_PREFIX="`$PKG_CONFIG --variable=prefix farstream-$FS_APIVERSION`"
AC_SUBST(GLIB_PREFIX)
AC_SUBST(FS_PREFIX)

dnl *** set variables based on configure arguments ***

dnl set license and copyright notice
FS_LICENSE="LGPL"
AC_DEFINE_UNQUOTED(FS_LICENSE, "$FS_LICENSE", [Farstream license])
AC_SUBST(FS_LICENSE)

dnl set location of plugin directory
AG_GST_SET_PLUGINDIR

dnl define an ERROR_CFLAGS Makefile variable
AG_GST_SET_ERROR_CFLAGS($FS_GIT)

dnl define correct level for debugging messages
AG_GST_SET_LEVEL_DEFAULT($FS_GIT)

AC_CHECK_FUNCS(getifaddrs)

dnl *** finalize CFLAGS, LDFLAGS, LIBS

dnl Overview:
dnl FS_OPTION_CFLAGS:  common flags for profiling, debugging, errors, ...
dnl GST_*:              flags shared by built objects to link against GStreamer
dnl FS_INTERNAL_CFLAGS: to link internally against the farstream libs
dnl                          (compare to other modules) or for i18n
dnl FS_ALL_LDFLAGS:    linker flags shared by all
dnl FS_LIB_LDFLAGS:    additional linker flags for all libaries
dnl FS_LT_LDFLAGS:     library versioning of our libraries
dnl FS_PLUGIN_LDFLAGS: flags to be used for all plugins

dnl FS_OPTION_CFLAGS
if test "x$USE_DEBUG" = xyes; then
   PROFILE_CFLAGS="-g"
fi
AC_SUBST(PROFILE_CFLAGS)

dnl every flag in FS_OPTION_CFLAGS can be overridden at make time
FS_OPTION_CFLAGS="\$(WARNING_CFLAGS) \$(ERROR_CFLAGS) \$(DEBUG_CFLAGS) \$(PROFILE_CFLAGS) \$(GCOV_CFLAGS) \$(OPT_CFLAGS) -Wno-error=deprecated-declarations "
AC_SUBST(FS_OPTION_CFLAGS)

dnl our libraries need to be versioned correctly
AC_SUBST(FS_LT_LDFLAGS)

dnl FS_INTERNAL_CFLAGS
dnl prefer internal headers to already installed ones
dnl also add builddir include for enumtypes
FS_INTERNAL_CFLAGS="-I\$(top_srcdir) -I\$(top_builddir)"
AC_SUBST(FS_INTERNAL_CFLAGS)

dnl FIXME: do we want to rename to GST_ALL_* ?
dnl add FS_OPTION_CFLAGS, but overridable
FS_CFLAGS="$GST_CFLAGS \$(FS_OPTION_CFLAGS)"
AC_SUBST(FS_CFLAGS)

dnl add GCOV libs because libtool strips -fprofile-arcs -ftest-coverage
FS_LIBS="$GST_LIBS \$(GCOV_LIBS)"
AC_SUBST(FS_LIBS)

dnl LDFLAGS really should only contain flags, not libs - they get added before
dnl whatevertarget_LIBS and -L flags here affect the rest of the linking
FS_ALL_LDFLAGS="-no-undefined"
AC_SUBST(FS_ALL_LDFLAGS)

dnl FS_LIB_LDFLAGS
dnl linker flags shared by all libraries
dnl LDFLAGS modifier defining exported symbols from built libraries
FS_LIB_LDFLAGS="-export-symbols-regex ^fs_.*"
AC_SUBST(FS_LIB_LDFLAGS)

dnl this really should only contain flags, not libs - they get added before
dnl whatevertarget_LIBS and -L flags here affect the rest of the linking
FS_PLUGIN_LDFLAGS="-module -avoid-version -export-symbols-regex '^[_]*fs_.*plugin.*' $FS_ALL_LDFLAGS"
AC_SUBST(FS_PLUGIN_LDFLAGS)

dnl this really should only contain flags, not libs - they get added before
dnl whatevertarget_LIBS and -L flags here affect the rest of the linking
GST_PLUGIN_LDFLAGS="-module -avoid-version -export-symbols-regex '^[_]*gst_plugin_.*' $FS_ALL_LDFLAGS"
AC_SUBST(GST_PLUGIN_LDFLAGS)

AC_ARG_ENABLE([gupnp],
	AC_HELP_STRING([--disable-gupnp], [Disable GUPnP IGD support]),
	[case "${enableval}" in
	    yes) WANT_GUPNP=yes ;;
	    no)  WANT_GUPNP=no ;;
	    *) AC_MSG_ERROR(bad value ${enableval} for --enable-gupnp) ;;
        esac],
	WANT_GUPNP=test)

HAVE_GUPNP=no
if test "x$WANT_GUPNP" != "xno"; then
   PKG_CHECK_MODULES(GUPNP, [ gupnp-igd-1.0 >= 0.2 ],
    [ HAVE_GUPNP=yes ],
    [ HAVE_GUPNP=no ])
fi
if test "x$WANT_GUPNP" = "xyes" && test "x$HAVE_GUPNP" = "xno"; then
   AC_ERROR([Requested GUPnP IGD, but it is not available])
fi

if test "x$HAVE_GUPNP" = "xyes"; then
   AC_DEFINE(HAVE_GUPNP,,[Have the GUPnP IGD library])
fi

dnl build static plugins or not
AC_MSG_CHECKING([whether to build static plugins or not])
AC_ARG_ENABLE(
  static-plugins,
  AC_HELP_STRING(
    [--enable-static-plugins],
    [build static plugins @<:@default=no@:>@]),
  [AS_CASE(
    [$enableval], [no], [], [yes], [],
    [AC_MSG_ERROR([bad value "$enableval" for --enable-static-plugins])])],
  [enable_static_plugins=no])
AC_MSG_RESULT([$enable_static_plugins])
if test "x$enable_static_plugins" = xyes; then
  AS_IF([test x$enable_static = xno], [AC_MSG_ERROR([--enable-static-plugins requires --enable-static to work])])
  AC_DEFINE(GST_PLUGIN_BUILD_STATIC, 1,
    [Define if static plugins should be built])
  PLUGIN_LIBTOOLFLAGS=""
else
  PLUGIN_LIBTOOLFLAGS="--tag=disable-static"
fi
AC_SUBST(PLUGIN_LIBTOOLFLAGS)
AM_CONDITIONAL(GST_PLUGIN_BUILD_STATIC, test "x$enable_static_plugins" = "xyes")

dnl *** output files ***

AC_CONFIG_FILES(
Makefile
farstream.pc
farstream-uninstalled.pc
common/Makefile
common/m4/Makefile
common-modified/Makefile
gst/Makefile
gst/fsrawconference/Makefile
gst/fsrtpconference/Makefile
gst/fsvideoanyrate/Makefile
gst/fsrtpxdata/Makefile
farstream/Makefile
transmitters/Makefile
transmitters/rawudp/Makefile
transmitters/multicast/Makefile
transmitters/nice/Makefile
transmitters/shm/Makefile
dnl pkgconfig/Makefile
dnl pkgconfig/farstream.pc
dnl pkgconfig/farstream-uninstalled.pc
tests/Makefile
tests/check/Makefile
tests/rtp/Makefile
examples/Makefile
examples/gui/Makefile
examples/commandline/Makefile
docs/Makefile
docs/libs/Makefile
docs/plugins/Makefile
docs/version.entities
dnl docs/plugins/Makefile
)
AC_OUTPUT
