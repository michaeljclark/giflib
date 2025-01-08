cmake_minimum_required(VERSION 3.14)

project(giflib)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

option(GIF_BUILD_SHARED "Build shared library" ON)
option(GIF_BUILD_STATIC "Build static library" ON)
option(GIF_BUILD_EXAMPLES "Build examples" ON)
option(GIF_INSTALL_LIBS "Install libs" ON)
option(GIF_INSTALL_EXAMPLES "Install exmaples" ON)

include(CheckCSourceCompiles)
include(CheckLibraryExists)
include(CheckCCompilerFlag)

macro(add_compiler_flag)
   set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${ARGN}")
endmacro(add_compiler_flag)

check_library_exists(m sqrtf "" has_libm)
check_c_source_compiles("#include <stdlib.h>
int main(void) { return !!reallocarray(NULL, 1, 1); }" has_reallocarray)

if(has_libm)
    set(EXTRA_LIBS ${EXTRA_LIBS} m)
endif()

file(READ include/gif_lib.h gif_lib_h_src)
string(REGEX MATCH "#define GIFLIB_MAJOR [0-9]+" gif_v_major ${gif_lib_h_src})
string(REGEX MATCH "#define GIFLIB_MINOR [0-9]+" gif_v_minor ${gif_lib_h_src})
string(REGEX MATCH "#define GIFLIB_RELEASE [0-9]+" gif_v_release ${gif_lib_h_src})
string(REGEX REPLACE "#define GIFLIB_MAJOR ([0-9]+)" "\\1" GIF_MAJOR ${gif_v_major})
string(REGEX REPLACE "#define GIFLIB_MINOR ([0-9]+)" "\\1" GIF_MINOR ${gif_v_minor})
string(REGEX REPLACE "#define GIFLIB_RELEASE ([0-9]+)" "\\1" GIF_RELEASE ${gif_v_release})
set(GIF_VERSION "${GIF_MAJOR}.${GIF_MINOR}.${GIF_RELEASE}")

#
# supress some worrisome gcc warnings
#

# examples/giftext.c:388:34: warning: ‘__builtin___snprintf_chk’ output
# truncated before the last format character [-Wformat-truncation=]
#
check_c_compiler_flag("-Wno-format-truncation" has_no_fmt_truncation)
if(has_no_fmt_truncation)
	add_compiler_flag(-Wno-format-truncation)
endif()

# examples/giftool.c:70:11: warning: ‘%s’ directive argument is null
# [-Wformat-overflow=] inlined from ‘getbool’
#
check_c_compiler_flag("-Wno-format-overflow" has_no_fmt_overflow)
if(has_no_fmt_overflow)
	add_compiler_flag(-Wno-format-overflow)
endif()

#
# giflib library
#

set(gif_source
	src/dgif_lib.c
	src/egif_lib.c
	src/gif_err.c
	src/gif_font.c
	src/gif_hash.c
	src/gifalloc.c)

set(gif_private_headers
	include/gif_hash.h
	include/gif_lib_private.h)

if(has_reallocarray)
	add_definitions(-DHAVE_REALLOCARRAY)
else()
	list(APPEND gif_source src/openbsd-reallocarray.c)
endif()

include_directories(include)

if(GIF_BUILD_SHARED)
	add_library(gif SHARED)
	target_sources(gif PRIVATE ${gif_source} ${gif_private_headers}
		INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include/gif_lib.h)
	target_compile_definitions(gif PRIVATE GIF_EXPORT)
	set_target_properties(gif
		PROPERTIES SOVERSION ${GIF_MAJOR} VERSION ${GIF_VERSION})
	list(APPEND GIF_LIB_TARGETS gif)
endif()

if(GIF_BUILD_STATIC)
	add_library(gif-static STATIC)
	target_sources(gif-static PRIVATE ${gif_source})
	list(APPEND GIF_LIB_TARGETS gif-static)
endif()

if(GIF_INSTALL_LIBS)
	install(TARGETS ${GIF_LIB_TARGETS})
endif()

if(TARGET gif-static)
	set(GIF_LIBRARY gif-static)
elseif(TARGET gif)
	set(GIF_LIBRARY gif)
	add_compile_definitions(GIF_IMPORT)
else()
	message(FATAL_ERROR "GIF_BUILD_SHARED or GIF_BUILD_STATIC must be set")
endif()

#
# giflib examples
#

set(gifcommon_source
	examples/qprintf.c
	examples/quantize.c
	examples/getarg.c)

if(WIN32)
	list(APPEND gifcommon_source examples/ya_getopt.c)
endif()

set(gifcommon_headers
	examples/getarg.h)

set(gif_examples gif2rgb gifbg gifbuild gifclrmp gifcolor gifecho
	giffilter giffix gifhisto gifinto gifsponge giftext giftool gifwedge)

set(gif_installable_examples gif2rgb gifbuild giffix giftext giftool gifclrmp)

if(GIF_BUILD_EXAMPLES)
	add_library(gifcommon OBJECT)
	target_sources(gifcommon PRIVATE ${gifcommon_source} ${gifcommon_headers})
	foreach(example ${gif_examples})
		add_executable(${example} examples/${example}.c)
		target_link_libraries(${example} ${EXTRA_LIBS} gifcommon ${GIF_LIBRARY})
	endforeach()
endif()

if(GIF_INSTALL_EXAMPLES)
	install(TARGETS ${gif_installable_examples})
endif()
