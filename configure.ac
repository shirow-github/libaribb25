#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.
AC_PREREQ([2.69])
AC_INIT([aribb25], [0.2.8], [kazhiro@marumo.ne.jp])
AM_INIT_AUTOMAKE([foreign subdir-objects])
AC_CONFIG_MACRO_DIR([m4])

AC_USE_SYSTEM_EXTENSIONS

AC_PROG_CC
AC_PROG_CC_STDC
AC_PROG_INSTALL
AC_PROG_LN_S
AC_PROG_MAKE_SET
LT_INIT([win32-dll shared disable-static])

AC_DEFINE([WIN32_LEAN_AND_MEAN],, [Define to limit the scope of <windows.h>.])

check_macosx_sdk()
{
   if test -z "${OSX_VERSION}"
   then
      OSX_VERSION=`xcrun --show-sdk-version`
      echo "OSX_VERSION not specified, assuming $OSX_VERSION"
   fi

   if test -z "$SDKROOT"
   then
      SDKROOT=`xcode-select -print-path`/Platforms/MacOSX.platform/Developer/SDKs/MacOSX$OSX_VERSION.sdk
      echo "SDKROOT not specified, assuming $SDKROOT"
   fi

   if test ! -d "${SDKROOT}"
   then
      SDKROOT_NOT_FOUND=`xcode-select -print-path`/Platforms/MacOSX.platform/Developer/SDKs/MacOSX$OSX_VERSION.sdk
      SDKROOT=`xcode-select -print-path`/SDKs/MacOSX$OSX_VERSION.sdk
      echo "SDKROOT not found at $SDKROOT_NOT_FOUND, trying $SDKROOT"
   fi

   if test ! -d "${SDKROOT}"
   then
      SDKROOT_NOT_FOUND="$SDKROOT"
      SDKROOT=`xcrun --show-sdk-path`
      echo "SDKROOT not found at $SDKROOT_NOT_FOUND, trying $SDKROOT"
   fi

   if test ! -d "${SDKROOT}"
   then
      echo "*** ${SDKROOT} does not exist, please install required SDK, or set SDKROOT manually. ***"
      exit 1
   fi
}

case "${host_os}" in
  *mingw32* | *cygwin*)
    SYS=mingw32
    ;;
  darwin*)
    SYS=macosx
    check_macosx_sdk
    ;;
esac

# Checks for libraries.
AS_IF([test "$SYS" = mingw32],[
  LDFLAGS="-no-undefined -lwinscard"
  AC_SUBST(PC_LIBS, "-lwinscard")
],[test "$SYS" = macosx],[
  AC_MSG_CHECKING([for PCSC.framework])
  gt_save_LIBS=$LIBS
  LIBS="$LIBS -Wl,-framework -Wl,PCSC"
  AC_TRY_LINK([#include <PCSC/winscard.h>],
    [SCardUnload()],
  AC_MSG_RESULT([yes]),
  AC_MSG_RESULT([no]))
  LIBS="$gt_save_LIBS"
  AC_SUBST(pcsclite_CFLAGS, ["-I$SDKROOT/System/Library/Frameworks/PCSC.framework/Headers"])
  AC_SUBST(PCSC_LIBS, ["-Wl,-framework -Wl,PCSC"])
],[
  PKG_CHECK_MODULES(pcsclite, "libpcsclite", [
    AC_DEFINE(HAVE_PCSC, 1, "have libpcsclite")
    AC_SUBST(PC_REQUIRE, "libpcsclite")
    AC_SUBST(PC_LIBS, "-lpcsclite")
  ], [AC_MSG_ERROR([${pcsclite_PKG_ERRORS}.])])
])

AC_CONFIG_HEADERS([src/config.h])
AC_CONFIG_FILES([Makefile src/aribb25.pc])
AC_OUTPUT
