dnl Process this file with autoconf to produce a configure script.
AC_PREREQ(2.61)
AC_INIT([feng],
        [m4_esyscmd([./git-version-gen .tarball-version])],
        [lscube-devel@lscube.org])
AC_CONFIG_AUX_DIR(config)
AM_INIT_AUTOMAKE([dist-bzip2 no-dist-gzip foreign subdir-objects])
LSC_INIT

AC_CONFIG_HEADER([config.h])

AC_ARG_WITH(rtsp-port,
    AS_HELP_STRING([--with-rtsp-port], [default RTSP listening port (default=554, alternative=8554)]),,
    with_rtsp_port=554)

AC_ARG_WITH(config-file,
    AS_HELP_STRING([--with-config-file], [name for the configuration file in sysconfdir (default=feng.conf)]),,
    with_config_file="feng.conf")

AC_ARG_WITH(log-file,
    AS_HELP_STRING([--with-log-file], [name for the log file (default=feng.log)]),,
    with_log_file="feng.log")

AC_ARG_WITH(max-session,
    AS_HELP_STRING([--with-max-session], [max number of incoming sessions (default=100)]),,
    with_max_session=100)

AC_ARG_ENABLE(libav,
    AS_HELP_STRING([--enable-libav], [enable libav support (default=yes)]),,
        enable_libav="yes")

AC_ARG_ENABLE(live-streaming,
    AS_HELP_STRING([--enable-live-streaming], [enable support for live streaming (default=yes)]),,
        live_streaming="yes")

AX_CHECK_LIBRARY([LIBEV], [ev.h], [ev], [],
   [AC_MSG_ERROR([libev not found, feng requires libev])])

AC_ARG_ENABLE(trace,
    AS_HELP_STRING([--enable-trace], [enable additional informations in log [[default=no]]]),,
    enable_trace="no")

AC_ARG_ENABLE([tables],
    AS_HELP_STRING([--disable-tables], [disable big build-time tables (for embedded)]))

AS_IF([test "x$enable_tables" = "xno"], [
  AC_DEFINE([AVOID_TABLES], [1], [Define to 1 if tables should be avoided])
])

dnl Checks used by feng itself
AC_CHECK_HEADERS_ONCE([syslog.h])
AC_CHECK_FUNCS_ONCE([inet_ntop])

AC_FUNC_STRERROR_R

AC_SEARCH_LIBS([clock_gettime], [rt],
  [AC_DEFINE([HAVE_CLOCK_GETTIME], [1], [Define this if you have clock_gettime])],
  [AC_MSG_WARN([POSIX realtime features not available])])

CC_ATTRIBUTE_PACKED([AC_DEFINE([ATTR_PACKED], [__attribute__((__packed__))], [Declares a structure packed])],
  [AC_MSG_ERROR([unable to find how the compiler declares packed structures])])

CC_ATTRIBUTE_DESTRUCTOR

AX_TLS([], [
  AC_CHECK_FUNCS([pthread_key_create pthread_setspecific pthread_getspecific], [],
    [AC_MSG_ERROR([Unable to find either Thread-Local Storage support or POSIX thread-specific data management functions])])
])

AH_BOTTOM([#if !defined(NDEBUG) && defined(SUPPORT_ATTRIBUTE_DESTRUCTOR)
	   # define CLEANUP_DESTRUCTOR __attribute__((__destructor__))
	   #endif
	  ])

dnl check if we have the generic struct for net addresses the has max
dnl possible size
AC_CHECK_TYPES(struct sockaddr_storage,,, [#include <sys/socket.h>])

AX_PROG_FLEX([],
  [AC_MSG_ERROR([unable to find flex, feng needs flex to build])
])
AX_PROG_BISON([],
  [AC_MSG_ERROR([unable to find GNU bison, feng needs GNU bison to build])
])

CHECK_RAGEL_AM([src/network/ragel_range.c], [6.6])

feng_state_dir=$(eval echo ${localstatedir}/feng)
AC_SUBST(feng_state_dir)

LSC_CHECK_IPV6
LSC_CHECK_SCTP
LIBS="$LIBS $SCTP_LIBS"

dnl Set the comment marker when sctp is not enabled, this allows for
dnl the -uninstalled variant of feng.conf to use sctp if enabled.
AS_IF([test "x$have_sctp" = "xyes"],
  [AC_SUBST([ENABLE_SCTP], [""])],
  [AC_SUBST([ENABLE_SCTP], ["#"])])

AS_IF([test "x$have_sctp" = "xyes"], [
  AC_CHECK_HEADERS([linux/sockios.h])
])

dnl Check for prerequisites to enable live streaming with flux.
AS_IF([test "x$live_streaming" = "xyes"],
    [AC_SEARCH_LIBS([mq_open], [rt],
               [AC_DEFINE(LIVE_STREAMING, [1],
                          [Define this if live streaming is supported])],
               [live_streaming=no])])

AM_CONDITIONAL([LIVE_STREAMING], [test "x$live_streaming" = "xyes"])

PKG_CHECK_MODULES([GLIB], [glib-2.0 >= 2.16 gthread-2.0])
CFLAGS="$CFLAGS $GLIB_CFLAGS"
LIBS="$LIBS $GLIB_LIBS"

avformat_msg="no"
avutil_msg="no"
if test "x$enable_libav" = "xyes"; then
    PKG_CHECK_MODULES(LIBAVFORMAT,[libavformat >= 53.3.0 libavcodec],
        [AC_DEFINE(HAVE_AVFORMAT,,[Define if libavformat support is available])
         avformat_msg="yes"
         ])
         CFLAGS="$CFLAGS $LIBAVFORMAT_CFLAGS"
         LIBS="$LIBS $LIBAVFORMAT_LIBS"
        PKG_CHECK_MODULES(LIBAVUTIL,[libavutil >= 49.6.0],
        [AC_DEFINE(HAVE_AVUTIL,,[Define if libavutil support is available])
         avutil_msg="yes"],
         [avutil_msg="no, using internal"])
         CFLAGS="$CFLAGS $LIBAVUTIL_CFLAGS"
         LIBS="$LIBS $LIBAVUTIL_LIBS"
fi

AM_CONDITIONAL(FENG_LIBAV, [test "$enable_libav" = "yes"])

CC_ATTRIBUTE_UNUSED(
  [AC_DEFINE([ATTR_UNUSED], [__attribute__((__unused__))], [Define this if ATTR_UNUSED is supported])],
  [AC_DEFINE([ATTR_UNUSED], [], [Define this if ATTR_UNUSED is not supported])])

if test "$enable_trace" = "yes"; then
    AC_DEFINE(TRACE, 1,[Trace enabled])
fi

AC_ARG_VAR([XSLTPROC], [A libxslt-compatible XSLT processor command])
AS_IF([test "x$XSLTPROC" = "x"], [
  AC_CHECK_PROGS([XSLTPROC], [xsltproc])
  AS_IF([test "x$XSLTPROC" = "x"], [
    AC_MSG_ERROR([unable to find xsltproc, you need this to build feng])
  ])
])

PKG_HAVE_DEFINE_WITH_MODULES(JSON,[json])
CFLAGS="$CFLAGS $JSON_CFLAGS"
LIBS="$LIBS $JSON_LIBS"

feng_avroot_dir=$(eval echo $localstatedir/$PACKAGE_NAME/avroot|sed -e "s:/\+:/:g; s:NONE::g")

AC_SUBST(feng_avroot_dir)

feng_max_session=$(eval echo $with_max_session)
AC_SUBST(feng_max_session)

feng_rtsp_port=$(eval echo $with_rtsp_port)
AC_SUBST(feng_rtsp_port)

feng_conf_file=$(eval echo $with_config_file|sed -e "s:/\+:/:g")
AC_SUBST(feng_conf_file)


feng_basedir=$(eval echo $localstatedir|sed -e "s:/\+:/:g; s:NONE::g")

feng_logdir="${feng_basedir}/log/$PACKAGE_NAME"
AC_SUBST(feng_logdir)

feng_log_file="${feng_basedir}/log/$with_log_file"
AC_SUBST(feng_log_file)

feng_statedir="${feng_basedir}/lib/$PACKAGE_NAME"
AC_SUBST(feng_statedir)

feng_basedir="${feng_basedir}/$PACKAGE_NAME"
AC_SUBST(feng_basedir)

feng_conf_dir=$(eval echo $sysconfdir|sed -e "s:/\+:/:g; s:NONE::g")
AC_SUBST(feng_conf_dir)

feng_docs_dir=$(eval echo $datadir/doc/$PACKAGE_NAME|sed -e "s:/\+:/:g")
AC_SUBST(feng_docs_dir)

AC_DEFINE_UNQUOTED(FENG_MAX_SESSION_DEFAULT,
                    $feng_max_session,
                    [Define max number of RTSP incoming sessions for Feng])

AC_DEFINE_UNQUOTED(FENG_RTSP_PORT_DEFAULT,
                    $feng_rtsp_port,
                    [Define default RTSP listening port])

AC_DEFINE_UNQUOTED(FENG_AVROOT_DIR_DEFAULT,
                    $feng_avroot_dir,
                    [Define default directory for Feng A/V resources])

AC_DEFINE_UNQUOTED(FENG_AVROOT_DIR_DEFAULT_STR,
                    "$feng_avroot_dir",
                    [Define default directory string for Feng A/V resources])

AC_DEFINE_UNQUOTED(FENG_DEFAULT_PORT, "$feng_rtsp_port", [Define default listening port])

AC_DEFINE_UNQUOTED(FENG_CONF_FILE_DEFAULT,
                    $feng_conf_file,
                    [Define default file for Feng configuration])

AC_DEFINE_UNQUOTED(FENG_LOG_FILE_DEFAULT,
                    $feng_log_file,
                    [Define default file for Feng logger])

AC_DEFINE_UNQUOTED(FENG_LOG_FILE_DEFAULT_STR,
                    "$feng_log_file",
                    [Define default string for Feng log file])

AC_DEFINE_UNQUOTED(FENG_CONF_DIR_DEFAULT,
                    $feng_conf_dir,
                    [Define default directory for Feng configuration])

AC_DEFINE_UNQUOTED(FENG_CONF_PATH_DEFAULT_STR,
                    "$(eval echo ${feng_conf_dir}/${feng_conf_file}|sed -e s:/\+:/:g)",
                    [Define absolute path string for Feng configuration file])

AC_DEFINE_UNQUOTED(FENG_STATE_DIR,
                    $feng_state_dir,
                    [Define default dir for Certificate (pem format)])

AC_DEFINE_UNQUOTED(FENIC_STATE_DIR_STR,
                    "$feng_state_dir",
                    [Define default string dir for Certificate (pem format)])

CC_CHECK_CFLAGS_APPEND([-Werror=declaration-after-statement -Wdeclaration-after-statement],
                       [break;])

AC_CONFIG_MACRO_DIR([m4])

AC_CONFIG_FILES([Makefile])
AC_CONFIG_FILES([etc/feng.conf])
AC_CONFIG_FILES([etc/feng-uninstalled.conf])
AC_CONFIG_FILES([Doxyfile])
AC_OUTPUT

LSC_DEBUG_STATUS
AC_MSG_NOTICE([config files successfully generated!

Feng Install prefix .......... : $prefix
Feng Binary path ............. : $(eval echo $exec_prefix/bin/feng)
Feng Configuration path ...... : $feng_conf_dir
Feng Log File ................ : $feng_log_file
Feng Multimedia Files root ... : $feng_avroot_dir
Feng RTSP listening port ..... : $feng_rtsp_port

live streaming supported...... : $live_streaming
sctp support enabled ......... : $enable_sctp
avformat support enabled ..... : $avformat_msg
avutil support enabled ....... : $avutil_msg
json support enabled ......... : $with_json


 'make' will now compile Feng and 'su -c make install' will install it.
 Note: On non-Linux systems you might need to use 'gmake' instead of 'make'.
])

