cmake_minimum_required(VERSION 3.0)

project(ShivaVG
    VERSION 0.2.1
    LANGUAGES C
    DESCRIPTION "The shivavg implementation of openvg"
)

# TODO test on Windows/MSVC
# TODO test on Windows/MinGW
# TODO separate targets for examples and install target
# TODO update INSTALL/README etc

include(CheckIncludeFiles)
include(CheckLibraryExists)
include(CheckTypeSize)

# make cache variables for install destinations
include(GNUInstallDirs)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/modules")
set(CMAKE_C_FLAGS "-std=c99")

# Processing the generator options
# TODO: can we have a master option that will turn others in UI?
# TODO: nicer display in UI? http://www.kitware.com/blog/home/post/82
option(BUILD_ALL_EXAMPLES   "Initialize build values for all examples (default=yes)]"   ON)
option(BUILD_VGU            "Build VGU example (default=yes)]"                          OFF)
option(BUILD_DASH           "Build Dashing example (default=yes)]"                      OFF)
option(BUILD_LINEAR         "Build Linear Gradient example (default=yes)]"              OFF)
option(BUILD_RADIAL         "Build Radial Gradient example (default=yes)]"              OFF)
option(BUILD_INTERPOLATE    "Build Path Interpolation example (default=yes)]"           OFF)
option(BUILD_TIGER          "Build Tiger SVG example (default=yes)]"                    OFF)
option(BUILD_IMAGE          "Build Images example (default=yes)]"                       OFF)
option(BUILD_PATTERN        "Build Pattern Paint example (default=yes)]"                OFF)
option(BUILD_BLEND          "Build Blending example (default=yes)]"                     OFF)
option(BUILD_EB             "Build Electro Body example (default=yes)]"                 OFF)
# Setting the master option sets the rest
if(BUILD_ALL_EXAMPLES)
    set(BUILD_VGU ON)
    set(BUILD_DASH ON)
    set(BUILD_LINEAR ON)
    set(BUILD_RADIAL ON)
    set(BUILD_INTERPOLATE ON)
    set(BUILD_TIGER ON)
    set(BUILD_IMAGE ON)
    set(BUILD_PATTERN ON)
    set(BUILD_BLEND ON)
    set(BUILD_EB ON)
    set(BUILD_WARP ON)
endif()
set(BUILD_EXAMPLES BUILD_VGU OR BUILD_DASH OR BUILD_LINEAR OR BUILD_RADIAL OR BUILD_INTERPOLATE OR BUILD_TIGER OR 
    BUILD_IMAGE OR BUILD_PATTERN OR BUILD_BLEND OR BUILD_EB)

# Finding packages
# TODO: EGL/OpenGLES and friends?
# TODO: learn how to specify prefixes
# TODO: try linking actual libraries like configure does
# TODO: detailed report instead of failure like configure did
# TODO: properly look for GLX on XWindow-based systems
find_package(OpenGL REQUIRED)
if(BUILD_EXAMPLES)
    # All the examples require GLUT
    find_package(GLUT REQUIRED)
    if(BUILD_IMAGE OR BUILD_PATTERN)
        find_package(JPEG REQUIRED)
    endif()
endif()

# Platform checks
check_include_files(dlfcn.h HAVE_DLFCN_H)
check_include_files(inttypes.h HAVE_INTTYPES_H)
check_include_files(jpeglib.h HAVE_JPEGLIB_H) # TODO we can also just reuse FindJPEG's result
check_library_exists(m cos "" HAVE_LIBM) # TODO use link directories instead of "" location parameter
check_include_files(memory.h HAVE_MEMORY_H)
check_include_files(stdint.h HAVE_STDINT_H)
check_include_files(stdlib.h HAVE_STDLIB_H)
check_include_files(strings.h HAVE_STRINGS_H)
check_include_files(string.h HAVE_STRING_H)
check_include_files(sys/stat.h HAVE_SYS_STAT_H)
check_include_files(sys/types.h HAVE_SYS_TYPES_H)
check_include_files(unistd.h HAVE_UNISTD_H)
check_type_size("void*" SIZEOF_VOID_P)

# Trying to get the version of the package
if(EXISTS "${CMAKE_SOURCE_DIR}/.git")
    find_package(Git)
    if(GIT_FOUND)
        execute_process(COMMAND ${GIT_EXECUTABLE} describe WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
            OUTPUT_VARIABLE PACKAGE_VERSION RESULT_VARIABLE GIT_RESULT OUTPUT_STRIP_TRAILING_WHITESPACE ERROR_QUIET) 
        if(GIT_RESULT)
            execute_process(COMMAND ${GIT_EXECUTABLE} rev-parse -q HEAD WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
                OUTPUT_VARIABLE PACKAGE_VERSION RESULT_VARIABLE GIT_RESULT OUTPUT_STRIP_TRAILING_WHITESPACE ERROR_QUIET)
            if(GIT_RESULT)
                set(PACKAGE_VERSION "Unknown(error)")
            endif()
        endif()
    else()
        set(PACKAGE_VERSION $PROJECT_VERSION)
    endif()
else()
    set(PACKAGE_VERSION $PROJECT_VERSION)
endif()

# Names, strings etc
set(PACKAGE ${CMAKE_PROJECT_NAME})
set(PACKAGE_BUGREPORT "elfy.ua@gmail.com") # TODO replace this after Quturnix gets an official bug reporting email
set(PACKAGE_NAME ${PACKAGE})
set(PACKAGE_STRING "${PACKAGE}-${PACKAGE_VERSION}")
set(PACKAGE_TARNAME "shivavg") # TODO I don't really think we -need- that

# Now generating the config.h and installing master includes
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/config.h.in ${CMAKE_CURRENT_BINARY_DIR}/config.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/include ${CMAKE_CURRENT_SOURCE_DIR}/include/vg)
# TODO this is really ugly, replace this with some intelligent way to detect glext availability
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src)

# Report on what's going to be built
message("ShivaVG will be compiled with the following example programs\n"
        "(run (cmake-gui, ccmake or cmake -D<option>) to change selection) :\n"
        "\n"
        "  VGU                       ${BUILD_VGU}\n"
        "  Dashing                   ${BUILD_DASH}\n"
        "  Linear Gradient           ${BUILD_LINEAR}\n"
        "  Radial Gradient           ${BUILD_RADIAL}\n"
        "  Path Interpolation        ${BUILD_INTERPOLATE}\n"
        "  Tiger SVG                 ${BUILD_TIGER}\n"
        "  Images                    ${BUILD_IMAGE}\n"
        "  Pattern paint             ${BUILD_PATTERN}\n"
        "  Blending                  ${BUILD_BLEND}\n"
        "  Electro Body              ${BUILD_EB}\n")

# Pick up subdirectories 
# TODO see if include_directories work like a stack and whether we can avoid duplication
# TODO break subdir execution of somebody tries to run them directly
add_subdirectory(src)
if(BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()


install(DIRECTORY include/vg DESTINATION include)
