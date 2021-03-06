# TODO: review configure.ac, take definitions from it and define them here
cmake_minimum_required(VERSION 2.8)
project(libmpdclient) 

set(LIBMPDCLIENT_VERSION 2.7)
set(LIBMPDCLIENT_SOVERSION_MAJOR 2)
set(LIBMPDCLIENT_SOVERSION_MINOR 6)
set(LIBMPDCLIENT_SOVERSION_PATCH 0)

set(DOXYGEN_OUTPUT_DIRECTORY doc/api)

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
    ${CMAKE_CURRENT_BINARY_DIR}
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

# build options

# http://www.bluequartz.net/projects/EIM_Segmentation/SoftwareDocumentation/html/usewithcmakeproject.html
#-- Add an Option to toggle the generation of the API documentation
# option(BUILD_DOCUMENTATION "Use Doxygen to create the HTML based API documentation" OFF)
option(ENABLE_DOCUMENTATION "Enable API doc generation" ON)
if(ENABLE_DOCUMENTATION)
    FIND_PACKAGE(Doxygen)
    if (NOT DOXYGEN_FOUND)
        message(WARNING "Doxygen is needed to build the documentation. Please install it correctly")
    else()
        file(MAKE_DIRECTORY ${PROJECT_BINARY_DIR}/${DOXYGEN_OUTPUT_DIRECTORY})
        #-- Configure the Template Doxyfile for our specific project
        configure_file(doc/doxygen.conf.in 
            ${PROJECT_BINARY_DIR}/doxygen.conf @ONLY IMMEDIATE)
        #-- Add a custom target to run Doxygen when ever the project is built
        add_custom_target (Docs ALL 
            COMMAND ${DOXYGEN_EXECUTABLE} ${PROJECT_BINARY_DIR}/doxygen.conf
            SOURCES ${PROJECT_BINARY_DIR}/doxygen.conf)
        # IF you do NOT want the documentation to be generated EVERY time you build the project
        # then leave out the 'ALL' keyword from the above command.
    endif()
endif()

option(ENABLE_TCP "Enable TCP support" ON)

option(ENABLE_WERROR "Treat warnings as errors" OFF)
if (ENABLE_WERROR)
    set(WERROR_FLAGS "-Werror -pedantic-errors")
    check_c_compiler_flag("${WERROR_FLAGS}" COMPILER_SUPPORTS_WERROR_AND_PEDANTIC_ERRORS)
    if (COMPILER_SUPPORTS_WERROR_AND_PEDANTIC_ERRORS)
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${WERROR_FLAGS}")
    endif()
endif()

option(ENABLE_STD99 "Treat warnings as errors" ON)
if (ENABLE_STD99)
    set(STD99_FLAGS "-std=c99")
    check_c_compiler_flag("${STD99_FLAGS}" COMPILER_SUPPORTS_STD99)
    if (COMPILER_SUPPORTS_STD99)
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${STD99_FLAGS}")
    endif()
endif()

if (CMAKE_BUILD_TYPE STREQUAL Debug)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DNDEBUG")
endif()

# CFLAGS

# TODO: check exact behaviour of AX_CHECK_COMPILER_FLAGS
foreach(flag -Wall -W -Wextra -Wno-deprecated-declarations 
        -Wmissing-prototypes -Wshadow -Wpointer-arith -Wstrict-prototypes 
        -Wcast-qual -Wwrite-strings)
    set(FLAG_TEST_VAR "${flag}")
    string(TOUPPER "${FLAG_TEST_VAR}" FLAG_TEST_VAR)
    string(REPLACE "-" "_" FLAG_TEST_VAR "${FLAG_TEST_VAR}")
    set(FLAG_TEST_VAR "COMPILER_SUPPORTS${FLAG_TEST_VAR}")
    check_c_compiler_flag("${flag}" ${FLAG_TEST_VAR})
    if (${FLAG_TEST_VAR})
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${flag}")
    endif()
endforeach()

# Compile-time options

# no UNIX domain sockets on WIN32
if (UNIX)
    set(DEFAULT_SOCKET "/var/run/mpd/socket" CACHE PATH "Default path of the socket file" FORCE)
endif()

set(DEFAULT_HOST "localhost" CACHE STRING "Default MPD host")
set(DEFAULT_PORT 6600 CACHE STRING "Default MPD port")
string(REGEX MATCH "[0-9]+" IS_INTEGER "${DEFAULT_PORT}")
if (NOT IS_INTEGER)
    message(FATAL "DEFAULT_PORT must be an integer")
endif()

# libmpdclient.pc stuff

set(LIB_SUFFIX "" CACHE STRING "Define suffix of directory name (32/64)")
set(EXEC_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX} CACHE PATH "Installation prefix for executables and object code libraries" FORCE)
set(BIN_INSTALL_DIR ${EXEC_INSTALL_PREFIX}/bin CACHE PATH "Installation prefix for user executables" FORCE)
set(LIB_INSTALL_DIR ${EXEC_INSTALL_PREFIX}/lib${LIB_SUFFIX} CACHE PATH  "Installation prefix for object code libraries" FORCE)
set(INCLUDE_INSTALL_DIR ${CMAKE_INSTALL_PREFIX}/include CACHE PATH "Installation prefix for C header files" FORCE)

if (APPLE)
ADD_DEFINITIONS(-DMACOS -DSTBI_NO_DDS)
set(RESOURCE_PREFIX "Resources")
else (APPLE)
if (WIN32)
set (RESOURCE_PREFIX "Resources")
ADD_DEFINITIONS(-DWIN32 -DSTBI_NO_DDS)
else(WIN32)
set (RESOURCE_PREFIX "share/libmpdclient")
ADD_DEFINITIONS(-DLINUX -DSTBI_NO_DDS)
endif(WIN32)
endif(APPLE)

# TODO: run configure_file on these
# ./include/mpd/version.h.in
# ./doc/doxygen.conf.in
# ./libmpdclient.pc.in

# TODO: figure out how to generate config.h
configure_file(config.h.in ${CMAKE_CURRENT_BINARY_DIR}/config.h)

configure_file(libmpdclient.pc.in ${CMAKE_CURRENT_BINARY_DIR}/libmpdclient.pc @ONLY)
configure_file(include/mpd/version.h.in ${CMAKE_CURRENT_BINARY_DIR}/include/mpd/version.h)

# clementine wants static (can be specified with BUILD_SHARED_LIBS)
add_library(mpdclient ${src_libmpdclient_la_SOURCES})

set_target_properties(mpdclient PROPERTIES
    VERSION ${LIBMPDCLIENT_VERSION}
    SOVERSION ${LIBMPDCLIENT_SOVERSION_MAJOR}.${LIBMPDCLIENT_SOVERSION_MINOR}.${LIBMPDCLIENT_SOVERSION_PATCH}
)
