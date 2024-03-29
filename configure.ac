dnl Process this file with autoconf to produce a configure script.
dnl This config.in requires autoconf 2.5 or greater.

AC_PREREQ([2.68])
AC_INIT([qpdf],[9.0.1])

AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_FILES([autoconf.mk])
AC_CONFIG_FILES([manual/html.xsl manual/print.xsl])
AC_CONFIG_FILES([libqpdf.pc])
AC_CONFIG_FILES([libqpdf.map])
AC_CONFIG_HEADERS([libqpdf/qpdf/qpdf-config.h])

AC_ARG_ENABLE(check-autofiles,
   AS_HELP_STRING([--enable-check-autofiles],
		  [if specified, verify checksums on automatically generated files (default=yes); package maintainers may want to disable this]),
   [if test "$enableval" = "no"; then
      CHECK_AUTOFILES=0
    else
      CHECK_AUTOFILES=1
    fi],
   [CHECK_AUTOFILES=1])

# Check to see if automatically generated files are outdated and if we
# can update them.
AC_CHECK_PROG(AUTOCONF,autoconf,1,0)
AC_CHECK_PROG(AUTOHEADER,autoheader,1,0)
AC_CHECK_PROG(ACLOCAL,aclocal,1,0)
AC_CHECK_PROG(SHA256SUM,sha256sum,1,0)
if test "$CHECK_AUTOFILES$AUTOCONF$AUTOHEADER$ACLOCAL$SHA256SUM" = "11111"; then
   if ! sha256sum -c autofiles.sums; then
       AC_MSG_ERROR(autofiles are autodated; rerun autogen.sh)
   fi
fi

AC_PROG_CC
AC_PROG_CC_C99
AC_PROG_CXX
AC_HEADER_STDC
LT_INIT([win32-dll])

# * If any interfaces have been removed or changed, or if any private
#   member variables or virtual functions have been added to any
#   class, we are not binary compatible. Increment LT_CURRENT, and set
#   LT_AGE and LT_REVISION to 0.
#
# * Otherwise, if any interfaces have been added since the last public
#   release, then increment LT_CURRENT and LT_AGE, and set LT_REVISION
#   to 0.
#
# * Otherwise, increment LT_REVISION

# LT = libtool
LT_CURRENT=26
LT_AGE=0
LT_REVISION=1
AC_SUBST(LT_CURRENT)
AC_SUBST(LT_REVISION)
AC_SUBST(LT_AGE)
LT_SONAME=$(expr $LT_CURRENT - $LT_AGE)
AC_SUBST(LT_SONAME)

AC_MSG_CHECKING(for -fvisibility=hidden)
try_flags=-fvisibility=hidden
oCXXFLAGS=$CXXFLAGS
CXXFLAGS="$CXXFLAGS $try_flags"
AC_LANG_PUSH([C++])
AC_LINK_IFELSE([AC_LANG_PROGRAM(
    [[class X
{
  public:
    __attribute__ ((visibility ("default")))
    X() {}
    __attribute__ ((visibility ("default")))
    void f() {}
};
   ]],[[X x; x.f();]])],
   [qpdf_VISIBILITY_HIDDEN=1],
   [qpdf_VISIBILITY_HIDDEN=0])
AC_LANG_POP
if test "$qpdf_VISIBILITY_HIDDEN" = "0"; then
   AC_MSG_RESULT(no)
   CXXFLAGS=$oCXXFLAGS
else
   AC_MSG_RESULT(yes)
   CFLAGS="$CFLAGS $try_flags"
fi

AC_MSG_CHECKING(whether to use insecure random numbers)
AC_ARG_ENABLE(insecure-random,
   AS_HELP_STRING([--enable-insecure-random],
                  [whether to use stdlib's random number generator (default is no)]),
   [if test "$enableval" = "yes"; then
      qpdf_INSECURE_RANDOM=1;
    else
      qpdf_INSECURE_RANDOM=0;
    fi], [qpdf_INSECURE_RANDOM=0])
if test "$qpdf_INSECURE_RANDOM" = "1"; then
   AC_MSG_RESULT(yes)
   AC_DEFINE([USE_INSECURE_RANDOM], [1], [Whether to use insecure random numbers])
else
   AC_MSG_RESULT(no)
fi

AC_MSG_CHECKING(whether to use OS-provided secure random numbers)
AC_ARG_ENABLE(os-secure-random,
   AS_HELP_STRING([--enable-os-secure-random],
                  [whether to try to use OS-provided secure random numbers (default is yes)]),
   [if test "$enableval" = "yes"; then
      qpdf_OS_SECURE_RANDOM=1;
    else
      qpdf_OS_SECURE_RANDOM=0;
    fi], [qpdf_OS_SECURE_RANDOM=1])
if test "$qpdf_OS_SECURE_RANDOM" = "1"; then
   AC_MSG_RESULT(yes)
else
   AC_MSG_RESULT(no)
   AC_DEFINE([SKIP_OS_SECURE_RANDOM], [1], [Whether to suppres use of OS-provided secure random numbers])
fi

AX_RANDOM_DEVICE

AC_MSG_CHECKING(whether to avoid Windows HANDLE type)
AC_ARG_ENABLE(avoid-windows-handle,
   AS_HELP_STRING([--enable-avoid-windows-handle],
                  [whether to avoid use of HANDLE, useful for some embedded Windows builds (default is no)]),
   [if test "$enableval" = "yes"; then
      qpdf_AVOID_HANDLE=1;
    else
      qpdf_AVOID_HANDLE=0;
    fi], [qpdf_AVOID_HANDLE=0])
if test "$qpdf_AVOID_HANDLE" = "1"; then
   AC_MSG_RESULT(yes)
   AC_DEFINE([AVOID_WINDOWS_HANDLE], [1], [Whether to avoid use of HANDLE in Windows])
else
   AC_MSG_RESULT(no)
fi

USE_EXTERNAL_LIBS=0
AC_MSG_CHECKING(for whether to use external libraries distribution)
AC_ARG_ENABLE(external-libs,
   AS_HELP_STRING([--enable-external-libs],
		  [whether to use external libraries distribution]),
   [if test "$enableval" = "yes"; then
      USE_EXTERNAL_LIBS=1;
    else
      USE_EXTERNAL_LIBS=0;
    fi], [BUILD_INTERNAL_LIBS=0])
if test "$BUILD_INTERNAL_LIBS" = "0"; then
   AC_MSG_RESULT(no)
else
   AC_MSG_RESULT(yes)
fi

AC_CHECK_SIZEOF([size_t])
if test "$ac_cv_sizeof_size_t" = "4"; then
   IS_32BIT=1
   WINDOWS_WORDSIZE=32
else
   IS_32BIT=0
   WINDOWS_WORDSIZE=64
fi
AC_SUBST(IS_32BIT)
AC_SUBST(WINDOWS_WORDSIZE)

PKG_PROG_PKG_CONFIG
if test "$BUILD_INTERNAL_LIBS" = "0"; then
    if test "$PKG_CONFIG" != ""; then
       PKG_CHECK_MODULES([pc_zlib], [zlib],
         [CFLAGS="$CFLAGS $pc_zlib_CFLAGS"
          CXXFLAGS="$CXXFLAGS $pc_zlib_CXXFLAGS"
          LIBS="$LIBS $pc_zlib_LIBS"
         ], [:])
       PKG_CHECK_MODULES([pc_libjpeg], [libjpeg],
         [CFLAGS="$CFLAGS $pc_libjpeg_CFLAGS"
          CXXFLAGS="$CXXFLAGS $pc_libjpeg_CXXFLAGS"
          LIBS="$LIBS $pc_libjpeg_LIBS"
         ],[:])
    fi
    if test "$pc_zlib_LIBS" = ""; then
        AC_CHECK_HEADER(zlib.h,,[MISSING_ZLIB_H=1; MISSING_ANY=1])
        AC_SEARCH_LIBS(deflate,z zlib,,[MISSING_ZLIB=1; MISSING_ANY=1])
    fi
    if test "$pc_libjpeg_LIBS" = ""; then
        AC_CHECK_HEADER(jpeglib.h,,[MISSING_JPEG_H=1; MISSING_ANY=1])
        AC_SEARCH_LIBS(jpeg_destroy,jpeg,,[MISSING_JPEG=1; MISSING_ANY=1])
    fi
    if test "$LIBS" != ""; then
        nLIBS=""
        for x in $LIBS; do
            if echo $x | grep -q '^-L'; then
                LDFLAGS="$LDFLAGS $x"
            else
                nLIBS="$nLIBS $x"
            fi
            LIBS=$nLIBS
        done
    fi
fi

qpdf_USE_WMAIN=0
AC_LANG_PUSH([C++])
AC_SUBST(WINDOWS_WMAIN_COMPILE)
AC_SUBST(WINDOWS_WMAIN_LINK)
for i in 0 1; do
  if test "$qpdf_USE_WMAIN" = "0"; then
    oLDFLAGS="$LDFLAGS"
    if test "$i" = "1"; then
      nLDFLAGS="-municode"
      LDFLAGS="$LDFLAGS $nLDFLAGS"
      msg="checking for wmain with $nLDFLAGS"
    else
      nLDFLAGS=
      msg="checking for wmain"
    fi
    AC_MSG_CHECKING($msg)
    AC_LINK_IFELSE([AC_LANG_SOURCE(
      [[#include <windows.h>
       #include <string.h>
       #include <stdio.h>
       extern "C"
       int wmain(int argc, wchar_t* argv[])
       {
           size_t x = wcslen(argv[0]);
           return 0;
       }
       ]])],
       [qpdf_USE_WMAIN=1],
       [qpdf_USE_WMAIN=0])
    LDFLAGS="$oLDFLAGS"
    if test "$qpdf_USE_WMAIN" = "1"; then
      AC_MSG_RESULT(yes)
      WINDOWS_WMAIN_COMPILE="-DWINDOWS_WMAIN $nLDFLAGS"
      WINDOWS_WMAIN_LINK="$nLDFLAGS"
    else
      AC_MSG_RESULT(no)
    fi
  fi
done
AC_LANG_POP

if test "x$qpdf_OS_SECURE_RANDOM" = "x1"; then
  OLIBS=$LIBS
  LIBS="$LIBS Advapi32.lib"
  AC_MSG_CHECKING(for Advapi32 library)
  AC_LINK_IFELSE([AC_LANG_PROGRAM(
    [[#pragma comment(lib, "crypt32.lib")
     #include <windows.h>
     #include <wincrypt.h>
     HCRYPTPROV cp;]],
     [CryptAcquireContext(&cp, NULL, NULL, PROV_RSA_FULL, 0);]
     )],
     [AC_MSG_RESULT(yes)
      LIBS="$OLIBS -lAdvapi32"],
     [AC_MSG_RESULT(no)
      LIBS=$OLIBS])
fi

QPDF_LARGE_FILE_TEST_PATH=
AC_SUBST(QPDF_LARGE_FILE_TEST_PATH)
AC_ARG_WITH(large-file-test-path,
   AS_HELP_STRING([--with-large-file-test-path=path],
		  [To enable testing of files > 4GB, give the path to a directory with at least 11 GB free.  The test suite will write temporary files to this directory.  Alternatively, just set the QPDF_LARGE_FILE_TEST_PATH environment variable to the path before running the test suite.]),
   [QPDF_LARGE_FILE_TEST_PATH=$withval],
   [QPDF_LARGE_FILE_TEST_PATH=])

AC_SYS_LARGEFILE
AC_FUNC_FSEEKO
AC_CHECK_FUNCS([fseeko64])
AC_TYPE_UINT16_T
AC_TYPE_UINT32_T

AC_CHECK_FUNCS(random)

# Check if LD supports linker scripts, and define conditional
# HAVE_LD_VERSION_SCRIPT if so.  This functionality is currently
# constrained to compilers using GNU ld on ELF systems or systems
# which provide an adequate emulation thereof.
AC_ARG_ENABLE([ld-version-script],
  AS_HELP_STRING([--enable-ld-version-script],
    [enable linker version script (default is enabled)]),
    [have_ld_version_script=$enableval], [have_ld_version_script=yes])
if test "$have_ld_version_script" != no; then
  AC_MSG_CHECKING([if LD -Wl,--version-script works])
  save_LDFLAGS="$LDFLAGS"
  LDFLAGS="$LDFLAGS -Wl,--version-script=conftest.map"
  cat > conftest.map <<EOF
VERS_1 {
        global: sym;
};

VERS_2 {
        global: sym;
} VERS_1;
EOF
  AC_LINK_IFELSE([AC_LANG_PROGRAM([], [])],
                 [have_ld_version_script=yes], [have_ld_version_script=no])
  rm -f conftest.map
  LDFLAGS="$save_LDFLAGS"
  AC_MSG_RESULT($have_ld_version_script)
fi
if test "$have_ld_version_script" = "yes"; then
  HAVE_LD_VERSION_SCRIPT=1
else
  HAVE_LD_VERSION_SCRIPT=0
fi
AC_SUBST(HAVE_LD_VERSION_SCRIPT)

make_okay=0
for make_prog in make gmake; do
  this_make_okay=0
  AC_MSG_CHECKING(for gnu make >= 3.81 ($make_prog))
  if $make_prog --version >/dev/null 2>&1; then
    v=`$make_prog --version | grep 'GNU Make' | sed -e 's/.*Make //'`
    maj=`echo $v | cut -d. -f 1`
    min=`echo $v | cut -d. -f 2`
    if test $maj -gt 3 -o '(' $maj -eq 3 -a $min -ge 81 ')'; then
       this_make_okay=1
       make_okay=1
    fi
  fi
  if test "$this_make_okay" = "1"; then
    AC_MSG_RESULT(yes)
  else
    AC_MSG_RESULT(no)
  fi
done

if test "$make_okay" = "0"; then
  dnl Don't set MISSING_ANY=1 -- maybe user calls make something else
  MISSING_MAKE_381=1
  ISSUE_WARNINGS=1
fi

AC_SUBST(GENDEPS)
GENDEPS=0
AC_MSG_CHECKING(for whether $CC supports -MD -MF x.dep -MP)
oCFLAGS=$CFLAGS
rm -f x.dep
CFLAGS="$CFLAGS -MD -MF x.dep -MP"
AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
                    [[#include <stdio.h>]], [[FILE* a = stdout]]
                  )],
                  [qpdf_DEPFLAGS=yes],
                  [qpdf_DEPFLAGS=no])
CFLAGS=$oCFLAGS
if test "$qpdf_DEPFLAGS" = "yes"; then
   if ! grep stdio.h x.dep >/dev/null 2>&1; then
      qpdf_DEPFLAGS=no
   fi
fi
rm -f x.dep
if test "$qpdf_DEPFLAGS" = "yes"; then
   AC_MSG_RESULT(yes)
   GENDEPS=1
else
   AC_MSG_RESULT(no)
fi

AC_MSG_CHECKING(which build rules to use)
AC_SUBST(BUILDRULES)
AC_ARG_WITH(buildrules,
   AS_HELP_STRING([--with-buildrules=rules],
		  [which build rules to use; see README.md]),
   [BUILDRULES=$withval],
   [BUILDRULES=libtool])
AC_MSG_RESULT($BUILDRULES)

AC_SUBST(WFLAGS)
AC_SUBST(CXXWFLAGS)
qpdf_USE_EXTRA_WARNINGS=0
if test "$BUILDRULES" = "msvc"; then
   dnl /w14996 makes warning 4996 a level 1 warning.  This warning
   dnl reports on functions that Microsoft considers unsafe or
   dnl deprecated.  Removing these warnings helps people who want to
   dnl write applications based on qpdf that can be Windows 8
   dnl certified.
   try_flags="-w14996"
else
   try_flags="-Wall"
fi
AC_MSG_CHECKING(for whether $CC supports $try_flags)
oCFLAGS=$CFLAGS
CFLAGS="$CFLAGS $try_flags"
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[]], [[int a = 1; int b = a; a = b;]])],
                  [qpdf_USE_EXTRA_WARNINGS=1],[qpdf_USE_EXTRA_WARNINGS=0])
CFLAGS=$oCFLAGS
if test "$qpdf_USE_EXTRA_WARNINGS" = "1"; then
   AC_MSG_RESULT(yes)
   WFLAGS="$try_flags"
else
   AC_MSG_RESULT(no)
fi
if test "$BUILDRULES" != "msvc"; then
   qpdf_USE_EXTRA_WARNINGS=0
   try_flags="-Wold-style-cast"
   AC_MSG_CHECKING(for whether $CXX supports $try_flags)
   oCXXFLAGS=$CXXFLAGS
   CXXFLAGS="$CXXFLAGS $try_flags"
   AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[]], [[int a = 1; int b = a; a = b;]])],
                     [qpdf_USE_EXTRA_WARNINGS=1],[qpdf_USE_EXTRA_WARNINGS=0])
   CXXFLAGS=$oCXXFLAGS
   if test "$qpdf_USE_EXTRA_WARNINGS" = "1"; then
      AC_MSG_RESULT(yes)
      CXXWFLAGS="$try_flags"
   else
      AC_MSG_RESULT(no)
   fi
fi

if test "$BUILDRULES" = "msvc"; then
   try_flags=-FS
   AC_MSG_CHECKING(for whether $CC supports $try_flags)
   oCFLAGS=$CFLAGS
   CFLAGS="$CFLAGS $try_flags"
   AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[]], [[int a = 1; int b = a; a = b;]])],
                     [qpdf_USE_FS=1],[qpdf_USE_FS=0])
   if test "$qpdf_USE_FS" = "1"; then
      AC_MSG_RESULT(yes)
      CXXFLAGS="$CXXFLAGS $try_flags"
   else
      AC_MSG_RESULT(no)
      CFLAGS=$oCFLAGS
   fi
fi

if test "$BUILDRULES" = "msvc"; then
   try_flags="-WX"
else
   try_flags="-Werror"
fi
AC_MSG_CHECKING(for whether to use $try_flags)
AC_ARG_ENABLE(werror,
   AS_HELP_STRING([--enable-werror],
                  [whether to treat warnings as errors (default is no)]),
   [if test "$enableval" = "yes"; then
      qpdf_USE_WERROR=1;
    else
      qpdf_USE_WERROR=0;
    fi], [qpdf_USE_WERROR=0])
if test "$qpdf_USE_WERROR" = "1"; then
   AC_MSG_RESULT(yes)
   WFLAGS="$WFLAGS $try_flags"
else
   AC_MSG_RESULT(no)
fi

if test "$BUILDRULES" = "msvc"; then
   try_flags="-W3"
else
   try_flags="-Wconversion -Wsign-conversion"
fi
AC_MSG_CHECKING(for whether to use $try_flags)
AC_ARG_ENABLE(int-warnings,
   AS_HELP_STRING([--enable-int-warnings],
                  [whether to turn on integer type warnings (default is yes)]),
   [if test "$enableval" = "yes"; then
      qpdf_INT_WARNINGS=1;
    else
      qpdf_INT_WARNINGS=0;
    fi], [qpdf_INT_WARNINGS=1])
if test "$qpdf_INT_WARNINGS" = "1"; then
   AC_MSG_RESULT(yes)
   WFLAGS="$WFLAGS $try_flags"
else
   AC_MSG_RESULT(no)
fi

AC_SUBST(QPDF_SKIP_TEST_COMPARE_IMAGES)
AC_ARG_ENABLE(test-compare-images,
   AS_HELP_STRING([--enable-test-compare-images],
		  [whether to compare images in test suite; disabled by default, enabling requires ghostscript and tiffcmp to be available]),
   [if test "$enableval" = "no"; then
      QPDF_SKIP_TEST_COMPARE_IMAGES=1
    else
      QPDF_SKIP_TEST_COMPARE_IMAGES=0
    fi],
   [QPDF_SKIP_TEST_COMPARE_IMAGES=1])

AC_SUBST(SHOW_FAILED_TEST_OUTPUT)
AC_ARG_ENABLE(show-failed-test-output,
   AS_HELP_STRING([--enable-show-failed-test-output],
		  [if specified, write failed test output to the console; useful for building on build servers where you can't easily open the test output files]),
   [if test "$enableval" = "no"; then
      SHOW_FAILED_TEST_OUTPUT=0
    else
      SHOW_FAILED_TEST_OUTPUT=1
    fi],
   [SHOW_FAILED_TEST_OUTPUT=0])

AC_ARG_WITH(docbook-xsl,
   AS_HELP_STRING([--with-docbook-xsl=DIR],
		  [location of docbook 4.x xml stylesheets]),
   [DOCBOOK_XSL=$withval],
   [DOCBOOK_XSL=/usr/share/xml/docbook/stylesheet/nwalsh])

DOCBOOK_XHTML=
AC_SUBST(DOCBOOK_XHTML)
AC_MSG_CHECKING(for xml to xhtml docbook stylesheets)
if test -f "$DOCBOOK_XSL/xhtml/docbook.xsl"; then
   DOCBOOK_XHTML="$DOCBOOK_XSL/xhtml/docbook.xsl"
   AC_MSG_RESULT($DOCBOOK_XHTML)
else
   AC_MSG_RESULT(no)
fi
DOCBOOK_FO=
AC_SUBST(DOCBOOK_FO)
AC_MSG_CHECKING(for xml to fo docbook stylesheets)
if test -f "$DOCBOOK_XSL/fo/docbook.xsl"; then
   DOCBOOK_FO="$DOCBOOK_XSL/fo/docbook.xsl"
   AC_MSG_RESULT($DOCBOOK_FO)
else
   AC_MSG_RESULT(no)
fi

DOCBOOKX_DTD=
AC_SUBST(DOCBOOKX_DTD)
AC_ARG_WITH(docbookx-dtd,
   AS_HELP_STRING([--with-docbookx-dtd=FILE],
		  [location of docbook 4.x xml DTD]),
   [DOCBOOKX_DTD=$withval],
   [DOCBOOKX_DTD=/usr/share/xml/docbook/schema/dtd/4/docbookx.dtd])
AC_MSG_CHECKING(for docbook 4.x xml DTD)
if test -f "$DOCBOOKX_DTD"; then
   AC_MSG_RESULT($DOCBOOKX_DTD)
else
   AC_MSG_RESULT(no)
fi

AC_CHECK_PROG(FOP,fop,fop,[])
AC_CHECK_PROG(XSLTPROC,xsltproc,xsltproc,[])
AC_CHECK_PROG(XMLLINT,xmllint,xmllint,[])

AC_ARG_ENABLE(doc-maintenance,
   AS_HELP_STRING([--enable-doc-maintenance],
		  [if set, enables all documentation options]),
   [if test "$enableval" = "yes"; then
      doc_default=1;
    else
      doc_default=0;
    fi],
   [doc_default=0])

BUILD_HTML=0
AC_SUBST(BUILD_HTML)
AC_ARG_ENABLE(html-doc,
   AS_HELP_STRING([--enable-html-doc],
		  [whether to build HTML documents]),
   [if test "$enableval" = "yes"; then
      BUILD_HTML=1;
    else
      BUILD_HTML=0;
    fi],
   [BUILD_HTML=$doc_default])
BUILD_PDF=0
AC_SUBST(BUILD_PDF)
AC_ARG_ENABLE(pdf-doc,
   AS_HELP_STRING([--enable-pdf-doc],
		  [whether to build PDF documents]),
   [if test "$enableval" = "yes"; then
      BUILD_PDF=1;
    else
      BUILD_PDF=0;
    fi],
   [BUILD_PDF=$doc_default])
VALIDATE_DOC=0
AC_SUBST(VALIDATE_DOC)
AC_ARG_ENABLE(validate-doc,
   AS_HELP_STRING([--enable-validate-doc],
		  [whether to validate xml document source]),
   [if test "$enableval" = "yes"; then
      VALIDATE_DOC=1;
    else
      VALIDATE_DOC=0;
    fi],
   [VALIDATE_DOC=$doc_default])

AC_SUBST(OSS_FUZZ)
AC_ARG_ENABLE(oss-fuzz,
   AS_HELP_STRING([--enable-oss-fuzz],
		  [if set, build static fuzzers for oss-fuzz]),
   [if test "$enableval" = "yes"; then
      OSS_FUZZ=1;
    else
      OSS_FUZZ=0;
    fi],
   [OSS_FUZZ=0])

if test "$VALIDATE_DOC" = "1"; then
   if test "$XMLLINT" = ""; then
      MISSING_XMLLINT=1
      MISSING_ANY=1
   fi
fi
if test "$BUILD_HTML" = "1"; then
   if test "$XSLTPROC" = ""; then
      MISSING_XSLTPROC=1
      MISSING_ANY=1
   fi
   if test "$DOCBOOK_XHTML" = ""; then
      MISSING_DOCBOOK_XHTML=1
      MISSING_ANY=1
   fi
fi
if test "$BUILD_PDF" = "1"; then
   if test "$XSLTPROC" = ""; then
      MISSING_XSLTPROC=1
      MISSING_ANY=1
   fi
   if test "$DOCBOOK_FO" = ""; then
      MISSING_DOCBOOK_FO=1
      MISSING_ANY=1
   fi
   if test "$FOP" = ""; then
      MISSING_FOP=1
      MISSING_ANY=1
   fi
fi


if test "$MISSING_ANY" = "1"; then
  ISSUE_WARNINGS=1
fi
if test "$ISSUE_WARNINGS" = "1"; then
  echo ""
  echo ""
fi

if test "$MISSING_MAKE_381" = "1"; then
  AC_MSG_WARN(gnu make >= 3.81 is required)
fi

if test "$MISSING_ZLIB_H" = "1"; then
  AC_MSG_WARN(unable to find required header zlib.h)
fi

if test "$MISSING_ZLIB" = "1"; then
  AC_MSG_WARN(unable to find required library z (or zlib))
fi

if test "$MISSING_JPEG_H" = "1"; then
  AC_MSG_WARN(unable to find required header jpeglib.h)
fi

if test "$MISSING_JPEG" = "1"; then
  AC_MSG_WARN(unable to find required library jpeg)
fi

if test "$MISSING_DOCBOOK_FO" = "1"; then
  AC_MSG_WARN(docbook fo stylesheets are required to build PDF documentation)
fi

if test "$MISSING_DOCBOOK_XHTML" = "1"; then
  AC_MSG_WARN(docbook xhmtl stylesheets are required to build HTML documentation)
fi

if test "$MISSING_FOP" = "1"; then
  AC_MSG_WARN(apache fop is required to build PDF documentation)
fi

if test "$MISSING_XMLLINT" = "1"; then
  AC_MSG_WARN(xmllint is required to validate documentation)
fi

if test "$MISSING_XSLTPROC" = "1"; then
  AC_MSG_WARN(xsltproc is required to build documentation)
fi

if test "$ISSUE_WARNINGS" = "1"; then
  echo ""
  echo ""
fi

if test "$MISSING_ANY" = "1"; then
  AC_MSG_ERROR(some required prerequisites were not found)
fi

# Do this last so it doesn't interfere with other tests.
if test "$USE_EXTERNAL_LIBS" = "1"; then
   # Don't actually check for the presence of this -- we document that
   # the user can run this and then edit autoconf.mk if they have too
   # much trouble getting it to work with a different compiler.
   CPPFLAGS="$CPPFLAGS -Iexternal-libs/include"
   LDFLAGS="$LDFLAGS -Lexternal-libs/lib-$BUILDRULES$WINDOWS_WORDSIZE"
   LIBS="$LIBS -lz -ljpeg"
fi

AC_OUTPUT
