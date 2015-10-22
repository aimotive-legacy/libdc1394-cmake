cmake_minimum_required(VERSION 2.8.12)
project(libdc1394 C)

include(CMakePrintHelpers)

set(CMAKE_DEBUG_POSTFIX d)

if(NOT EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/libdc1394)
    find_package(Git QUIET REQUIRED)
    execute_process(
        COMMAND ${GIT_EXECUTABLE}
            clone --depth 1 git://git.code.sf.net/p/libdc1394/code ${CMAKE_CURRENT_SOURCE_DIR}/libdc1394
        RESULT_VARIABLE r)
    if(r)
        message(FATAL_ERROR "git-cloning libdc1394 failed.")
    endif()
endif()

set(repo ${CMAKE_CURRENT_SOURCE_DIR}/libdc1394/libdc1394)


include(CheckIncludeFile)
include(CheckCSourceCompiles)

set(HAVE_LIBRAW1394 0) #used in c am
set(HAVE_LIBSDL 0) #used in am
set(HAVE_LIBUSB 0) # used in c, am
if(CMAKE_SYSTEM_NAME MATCHES "Linux")
	set(HAVE_LINUX 1) # used in c am
endif()
if(APPLE AND NOT IOS)
	set(HAVE_MACOSX 1) # used in c am
endif()
check_include_file(netinet/in.h HAVE_NETINET_IN_H) # used in c
find_package(OpenGL QUIET)
if(OPENGL_FOUND)
	set(HAVE_OPENGL 1) # used in am
endif()
check_include_file(sys/mman.h HAVE_SYS_MMAN_H) #used in c
if(WIN32)
	set(HAVE_WINDOWS 1) # used in c am
endif()
set(PACKAGE "libdc1394") # used in am
set(C99_RESTRICT "")
foreach(r restrict __restrict _Restrict __restrict__ __RESTRICT)
	check_c_source_compiles("int main() {int*${r} p;return 0;}" ${r}_IS_C99_RESTRICT)
	if(${r}_IS_C99_RESTRICT)
		set(C99_RESTRICT ${r})
		break()
	endif()
endforeach()
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/config.h "")
# need to define size_t if you compiler does not support it: Define to `unsigned int' if <sys/types.h> does not define.
foreach(v
	HAVE_LIBRAW1394
	HAVE_LIBSDL
	HAVE_LIBUSB
	HAVE_LINUX
	HAVE_MACOSX
	HAVE_NETINET_IN_H
	HAVE_OPENGL
	HAVE_SYS_MMAN_H
	HAVE_WINDOWS
)
	if(${v})
		file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/config.h "#define ${v} 1\n")
	else()
		file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/config.h "/* #undef ${v} */\n")
	endif()
endforeach()
file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/config.h "#define PACKAGE \"${PACKAGE}\"\n")
if(C99_RESTRICT STREQUAL "restrict")
	file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/config.h "/* #undef restrict */\n")
else()
	file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/config.h "#define restrict ${C99_RESTRICT}\n")
endif()

check_include_file(linux/videodev.h HAVE_VIDEODEV) # used in am

# add lm

option(MAKE_EXAMPLES "build example programs" ON)

#    Makefile \
#    dc1394/Makefile \
#    dc1394/linux/Makefile \
#    dc1394/juju/Makefile \
#    dc1394/macosx/Makefile \
#    dc1394/windows/Makefile \
#    dc1394/usb/Makefile \
#    dc1394/vendor/Makefile \
#    examples/Makefile \


set(LINUXMSG "Disabled (Linux not detected)")
set(RAW1394MSG "Disabled (libraw1394 not found)")
set(MACOSXMSG "Disabled (Mac OS not detected)")
set(WINDOWSMSG "Disabled (Windows not detected)")
set(USBMSG "Disabled (libusb-1.0 not found)")

if(HAVE_LINUX)
	if(HAVE_LIBRAW1394)
		set(RAW1394MSG Enabled)
	endif()
	set(LINUXMSG Enabled)
endif()

if(HAVE_MACOSX)
  set(MACOSXMSG Enabled)
endif()

if(HAVE_WINDOWS)
	set(WINDOWSMSG Enabled)
endif()

if(HAVE_LIBUSB)
	set(USBMSG Enabled)
endif()

set(EXAMPLESMSG No)
set(SDLEXAMPLESMSG No)
set(XVEXAMPLESMSG No)
set(VIDEXAMPLESMSG No)

if(MAKE_EXAMPLES)
	set(EXAMPLESMSG Yes)
	if(HAVE_OPENGL)
    	if(HAVE_LIBSDL)
    		set(SDLEXAMPLESMSG Yes)
    	endif()
    endif()
	if(HAVE_XV)
    	set(XVEXAMPLESMSG Yes)
    endif()
    if(HAVE_VIDEODEV)
    	set(VIDEXAMPLESMSG Yes)
    endif()
endif()

message(STATUS "Linux support:                    ${LINUXMSG}")
if(LINUXMSG STREQUAL "Enabled")
	message(STATUS "Legacy drivers (libraw1394):    ${RAW1394MSG}")
endif()
message(STATUS "Mac OS X support:                 ${MACOSXMSG}")
message(STATUS "Windows support:                  ${WINDOWSMSG}")
message(STATUS "IIDC-over-USB support:            ${USBMSG}")
message(STATUS "Build examples:                   ${EXAMPLESMSG}")
if(EXAMPLESMSG)
	message(STATUS "Build SDL/OpenGL examples:        ${SDLEXAMPLESMSG}")
	message(STATUS "Build XV examples:                ${XVEXAMPLESMSG}")
	message(STATUS "Build V4L examples:               ${VIDEXAMPLESMSG}")
endif()

# Makefile.am
# process this file with automake to create a Makefile.in

add_definitions(-DHAVE_CONFIG)
add_subdirectory(dc1394)
if(MAKE_EXAMPLES)
	add_subdirectory(examples)
endif()

