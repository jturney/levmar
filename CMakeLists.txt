cmake_minimum_required(VERSION 3.0)
project(levmar LANGUAGES C)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)

################################### Options ####################################
include(Psi4Macros)
option_with_default(CMAKE_BUILD_TYPE "Build type" Release)
option_with_print(BUILD_SHARED_LIBS "Build final library as shared, not static" OFF)
option_with_default(BUILD_FPIC "Libraries will be compiled with position independent code" ON)
if(${BUILD_SHARED_LIBS} AND NOT ${BUILD_FPIC})
    message(FATAL_ERROR "BUILD_SHARED_LIBS ON and BUILD_FPIC OFF are incompatible, as shared library requires position independent code")
endif()
option_with_print(ENABLE_GENERIC "Enable mostly static linking in shared library" OFF)
option_with_flags(ENABLE_XHOST "Enable processor-specific optimization" ON
                    "-xHost" "-march=native")

######################### Process & Validate Options ###########################
include(autocmake_safeguards)
include(autocmake_static_library)

################################# Main Project #################################
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(PN ${PROJECT_NAME})

set(LM_DBL_PREC 1)
set(HAVE_LAPACK 1)

# For add_library STATIC/SHARED is governed by BUILD_SHARED_LIBS
add_library(levmar 
    src/lm.c
    src/Axb.c
    src/misc.c
    src/lmlec.c
    src/lmbc.c
    src/lmblec.c
    src/lmbleic.c
    src/misc.h
    src/compiler.h
    )

configure_file(
    ${CMAKE_SOURCE_DIR}/src/levmar.h.in
    ${CMAKE_BINARY_DIR}/src/levmar.h
    )

include_directories(
    ${CMAKE_SOURCE_DIR}/src
    ${CMAKE_BINARY_DIR}/src
    )

set_target_properties(levmar PROPERTIES POSITION_INDEPENDENT_CODE ${BUILD_FPIC})
if (${BUILD_SHARED_LIBS})
    target_link_libraries(levmar PRIVATE ${LAPACK_LIBRARIES} ${BLAS_LIBRARIES} ${LIBC_INTERJECT})
endif()

# <<<  Install  >>>
install(FILES 
            ${CMAKE_BINARY_DIR}/src/levmar.h
        DESTINATION
            ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_INCLUDEDIR}/levmar
        )
install(TARGETS
            levmar
        EXPORT
            "${PN}Targets"
        ARCHIVE DESTINATION
            ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION
            ${CMAKE_INSTALL_LIBDIR}
        )

# <<<  Export Config  >>>
set(CMAKECONFIG_INSTALL_DIR "${CMAKE_INSTALL_DATADIR}/cmake/${PN}")
configure_package_config_file(${PN}Config.cmake.in
                              "${CMAKE_CURRENT_BINARY_DIR}/${PN}Config.cmake"
                              INSTALL_DESTINATION ${CMAKECONFIG_INSTALL_DIR})
write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/${PN}ConfigVersion.cmake
                                 VERSION 1.2
                                 COMPATIBILITY SameMajorVersion)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PN}Config.cmake
              ${CMAKE_CURRENT_BINARY_DIR}/${PN}ConfigVersion.cmake
        DESTINATION ${CMAKECONFIG_INSTALL_DIR})
install(EXPORT "${PN}Targets"
        NAMESPACE "${PN}::"
        DESTINATION ${CMAKECONFIG_INSTALL_DIR})

