#
# Copyright (c) 2011 Marius Zwicker <marius@mlba-team.de>
#
# Permission to use, copy, modify, and distribute this software for any
# purpose with or without fee is hereby granted, provided that the above
# copyright notice and this permission notice appear in all copies.
#
# THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
# WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
# MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
# ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
# WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
# ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
# OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
#

cmake_minimum_required(VERSION 2.8.11)
project(pthread_workqueue C)

if (NOT CMAKE_BUILD_TYPE)
    set (CMAKE_BUILD_TYPE MinSizeRel)
endif ()

if (CMAKE_SYSTEM_NAME MATCHES Linux)
    set (LINUX 1)
endif ()

#///////////////////////////////////////////////////////////////////////////////
#files
file(GLOB_RECURSE headers include/*.h)
source_group(Headers FILES ${headers})

file(GLOB sources src/*.h src/*.c)

include_directories(include src)

if(WIN32)
    file(GLOB win32_sources src/windows/*.h src/windows/*.c)
    list(APPEND sources "${win32_sources}")

    add_definitions(-D_USRDLL -D_WINDLL)
endif ()

if (UNIX)
    file(GLOB posix_sources src/posix/*.h src/posix/*.c)
    list(APPEND sources "${posix_sources}")

    add_definitions(-DHAVE_ERR_H=1 -D_XOPEN_SOURCE=600 -D_GNU_SOURCE)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99 -fvisibility=hidden")
endif ()

if (LINUX)
    file(GLOB linux_sources src/linux/*.h src/linux/*.c)
    list(APPEND sources ${linux_sources})
endif ()

source_group(Sources FILES ${sources})

#///////////////////////////////////////////////////////////////////////////////
#library (static or shared)
option(STATIC_WORKQUEUE "Enable to build libpthread_workqueue as static lib" OFF)
if(STATIC_WORKQUEUE)
    message("-- building libpthread_workqueue as static lib")
    add_definitions(-DMAKE_STATIC)
    add_library(pthread_workqueue STATIC ${sources} ${headers})
else()
    add_library(pthread_workqueue SHARED ${sources} ${headers})
endif()
if(NOT WIN32)
    target_link_libraries(pthread_workqueue pthread)
endif()
if(LINUX)
    target_link_libraries(pthread_workqueue rt)
endif()

set_target_properties(pthread_workqueue PROPERTIES
    DEBUG_POSTFIX "D"
    DEFINE_SYMBOL LIBPTHREAD_WORKQUEUE_EXPORTS)

#///////////////////////////////////////////////////////////////////////////////
#tests
option(WORKQUEUE_TESTS "Enable to build tests for libpthread_workqueue" OFF)
if(WORKQUEUE_TESTS)
    message("-- Adding tests for libpthread_workqueue")
    add_subdirectory(testing)
endif()

install(TARGETS pthread_workqueue
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib)

install(FILES ${headers} DESTINATION include)

