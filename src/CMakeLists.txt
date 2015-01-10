cmake_minimum_required(VERSION 2.8)
# TODO headers as dependencies
# TODO shared/static switch CMake-style
set(libOpenVG_SOURCES 
    shExtensions.c
    shArrays.c
    shVectors.c
    shPath.c
    shImage.c
    shPaint.c
    shGeometry.c
    shPipeline.c
    shParams.c
    shContext.c
    shVgu.c
)
set(libOpenVG_HEADERS
    shDefs.h
    shExtensions.h
    shArrayBase.h
    shArrays.h
    shVectors.h
    shPath.h
    shImage.h
    shPaint.h
    shGeometry.h
    shContext.h
)
add_definitions(-std=c99 -pedantic -DHAVE_CONFIG_H)
include_directories(${OPENGL_INCLUDE_DIR})
# TODO what if OpenGL library has nonstandard location?
add_library(OpenVG SHARED ${libOpenVG_SOURCES})
target_link_libraries(OpenVG ${OPENGL_LIBRARIES} -lm)
SET_TARGET_PROPERTIES( OpenVG PROPERTIES COMPILE_FLAGS -fPIC)

install(TARGETS OpenVG DESTINATION lib)
