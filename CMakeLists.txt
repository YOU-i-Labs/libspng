cmake_minimum_required(VERSION 3.0)

include(cmake/Hunter/HunterGate.cmake)
HunterGate(
    URL "https://api.github.com/repos/You-i-Labs/Hunter/tarball/v0.23.190-youi4"
    HTTPHEADER "Authorization: token ${GITHUB_ACCESS_TOKEN}"
    SHA1 "0a6a086585a324e047455b8576a5459cedd8b96c"
)

project(spng C)

set(SPNG_MAJOR 0)
set(SPNG_MINOR 5)
set(SPNG_REVISION 0)
set(SPNG_VERSION ${SPNG_MAJOR}.${SPNG_MINOR}.${SPNG_REVISION})

option(ENABLE_OPT "Enable architecture-specific optimizations" ON)
option(SPNG_SHARED "Build shared lib" ON)
option(SPNG_STATIC "Build static lib" OFF)
option(INSTALL_TESTS "Install test files" ON)

include(GNUInstallDirs)

hunter_add_package(ZLIB)
find_package(ZLIB CONFIG REQUIRED)

include_directories(${ZLIB_INCLUDE_DIRS})

set(spng_SOURCES spng.c)

if(NOT ENABLE_OPT)
    add_definitions( -DSPNG_DISABLE_OPT=1 )
endif()

if(SPNG_SHARED)
	add_library(spng SHARED ${spng_SOURCES})
endif()

if(SPNG_STATIC)
	add_library(spng STATIC ${spng_SOURCES})
endif()

if (SPNG_SHARED AND SPNG_STATIC)
    message(FATAL_ERROR "Cannot build with both SPNG_SHARED and SPNG_STATIC enabled.")
endif()

target_link_libraries(spng PRIVATE ZLIB::zlib)

if(NOT CMAKE_HOST_WIN32 OR CYGWIN OR MINGW)
    set(prefix      ${CMAKE_INSTALL_PREFIX})
    set(exec_prefix ${CMAKE_INSTALL_PREFIX})
    set(libdir      ${CMAKE_INSTALL_FULL_LIBDIR})
    set(includedir  ${CMAKE_INSTALL_FULL_INCLUDEDIR})
    set(LIBS        "-lz -lm")

    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/tests/libspng.pc.in ${CMAKE_CURRENT_BINARY_DIR}/libspng.pc @ONLY)

    if (INSTALL_TESTS)
        install(FILES ${CMAKE_CURRENT_BINARY_DIR}/tests/libspng.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
    endif()
endif()

### Installation (https://github.com/forexample/package-example) {

set(config_install_dir "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(NAMESPACE "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)

configure_package_config_file(
    "cmake/template/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(TARGETS spng
    EXPORT "${TARGETS_EXPORT_NAME}"
    DESTINATION lib)

install(FILES spng.h
    DESTINATION include)

install(
    FILES "${project_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    NAMESPACE "${NAMESPACE}"
    DESTINATION "${config_install_dir}"
)

### }
