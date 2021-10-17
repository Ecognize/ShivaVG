cmake_minimum_required(VERSION 3.0)
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

add_definitions(-DHAVE_CONFIG_H)
include_directories(${OPENGL_INCLUDE_DIR})
# TODO what if OpenGL library has nonstandard location?
add_library(OpenVG SHARED ${libOpenVG_SOURCES})
target_link_libraries(OpenVG ${OPENGL_LIBRARIES} -lm)
SET_TARGET_PROPERTIES( OpenVG PROPERTIES COMPILE_FLAGS -fPIC)

# create library
add_library(OpenVG_objlib OBJECT ${libOpenVG_SOURCES})
set_property(TARGET OpenVG_objlib PROPERTY POSITION_INDEPENDENT_CODE 1)

# create library
add_library(OpenVGStatic STATIC $<TARGET_OBJECTS:OpenVG_objlib>)
set_target_properties(OpenVGStatic PROPERTIES LIBRARY_OUTPUT_NAME OpenVG.a)

# create library
add_library(OpenVG SHARED $<TARGET_OBJECTS:OpenVG_objlib>)
target_link_libraries(OpenVG ${OPENGL_LIBRARIES} ${GLEW_LIBRARIES} -lm)

# add include directories
target_include_directories(OpenVG
    PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>"
    "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>"
)

# install the target and create export-set
install(TARGETS OpenVG OpenVGStatic
    EXPORT OpenVGTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# header files (include/VG) are installed by ../CMakeLists.txt

# generate and install export file
install(EXPORT OpenVGTargets
    FILE OpenVGTargets.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/OpenVG
)



include(CMakePackageConfigHelpers)

set(version $PROJECT_VERSION)

# generate the version file for the config file
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/OpenVGConfigVersion.cmake"
    VERSION "${version}"
    COMPATIBILITY AnyNewerVersion
)

# create config file
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
    "${CMAKE_CURRENT_BINARY_DIR}/OpenVGConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/OpenVG
    NO_CHECK_REQUIRED_COMPONENTS_MACRO
)

# install config files
install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/OpenVGConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/OpenVGConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/OpenVG
)



# TODO verify
# generate pkgconfig file
set(target1 OpenVG)
configure_file(openvg.pc.in lib/pkgconfig/openvg.pc @ONLY)
