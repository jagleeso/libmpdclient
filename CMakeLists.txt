# TODO: review configure.ac, take definitions from it and define them here
cmake_minimum_required(VERSION 2.8)
project(libmpdclient) 

set(mpdinclude_HEADERS
	include/mpd/async.h
	include/mpd/audio_format.h
	include/mpd/client.h
	include/mpd/capabilities.h
	include/mpd/compiler.h
	include/mpd/connection.h
	include/mpd/database.h
	include/mpd/directory.h
	include/mpd/entity.h
	include/mpd/error.h
	include/mpd/idle.h
	include/mpd/list.h
	include/mpd/mixer.h
	include/mpd/parser.h
	include/mpd/password.h
	include/mpd/player.h
	include/mpd/playlist.h
	include/mpd/protocol.h
	include/mpd/queue.h
	include/mpd/recv.h
	include/mpd/response.h
	include/mpd/send.h
	include/mpd/status.h
	include/mpd/stats.h
	include/mpd/tag.h
	include/mpd/output.h
	include/mpd/pair.h
	include/mpd/search.h
	include/mpd/song.h
	include/mpd/sticker.h
	include/mpd/settings.h
	include/mpd/message.h
	include/mpd/version.h
)

# AM_CPPFLAGS += -I$(srcdir)/include -Iinclude
include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${CMAKE_CURRENT_BINARY_DIR}/include
)

set(lib_LTLIBRARIES mpdclient)

set(src_libmpdclient_la_SOURCES
	src/async.c src/iasync.h
	src/buffer.h
	src/internal.h
	src/ierror.c src/ierror.h
	src/resolver.c src/resolver.h
	src/capabilities.c
	src/connection.c
	src/database.c
	src/directory.c
	src/rdirectory.c
	src/error.c
	src/fd_util.c src/fd_util.h
	src/output.c
	src/coutput.c
	src/entity.c
	src/idle.c
	src/iso8601.h
	src/iso8601.c
	src/list.c
	src/mixer.c
	src/parser.c
	src/password.c
	src/player.c
	src/playlist.c
	src/rplaylist.c
	src/cplaylist.c
	src/queue.c
	src/quote.c src/quote.h
	src/recv.c
	src/response.c
	src/run.c src/run.h
	src/search.c
	src/send.c src/isend.h
	src/socket.c src/socket.h
	src/song.c
	src/status.c
	src/cstatus.c
	src/stats.c
	src/cstats.c
	src/sync.c src/sync.h
	src/tag.c
	src/sticker.c
	src/settings.c
	src/message.c
	src/cmessage.c
	src/uri.h
)

list(APPEND CMAKE_EXE_LINKER_FLAGS
    -version-info @LIBMPDCLIENT_LIBTOOL_VERSION@
    -no-undefined
)

# Testing linker flags:
# http://www.mail-archive.com/cmake@cmake.org/msg37342.html
include(CheckCCompilerFlag)
function(check_linker_flag flag var) 
    set(CMAKE_REQUIRED_FLAGS ${flag})
    check_c_compiler_flag("${flag}" ${var})
endfunction(check_linker_flag)

check_linker_flag("-Wl,--version-script=${CMAKE_CURRENT_SOURCE_DIR}/libmpdclient.ld" LINKER_HAS_GNU_FLAGS)

if(LINKER_HAS_GNU_FLAGS)
    list(APPEND CMAKE_EXE_LINKER_FLAGS
        -Wl
        --version-script=${CMAKE_CURRENT_SOURCE_DIR}/libmpdclient.ld
    )
endif()
# message("CMAKE_EXE_LINKER_FLAGS == ${CMAKE_EXE_LINKER_FLAGS}")

# TODO: run configure_file on these
# ./include/mpd/version.h.in
# ./doc/doxygen.conf.in
# ./libmpdclient.pc.in

# TODO: figure out how to generate config.h
# configure_file(config.h.in ${CMAKE_CURRENT_BINARY_DIR}/config.h)

configure_file(libmpdclient.pc.in ${CMAKE_CURRENT_BINARY_DIR}/libmpdclient.pc.in)
configure_file(include/mpd/version.h.in ${CMAKE_CURRENT_BINARY_DIR}/include/mpd/version.h.in)

# clementine wants static
add_library(mpdclient SHARED ${src_libmpdclient_la_SOURCES})
