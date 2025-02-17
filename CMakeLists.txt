project(ayatana-indicator-keyboard C CXX)
cmake_minimum_required(VERSION 3.13)

if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    SET(CMAKE_INSTALL_PREFIX "/usr" CACHE PATH "..." FORCE)
endif(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)
set(PROJECT_VERSION "22.2.1")
set(PACKAGE ${CMAKE_PROJECT_NAME})

# Options
option(ENABLE_TESTS "Enable all tests and checks" OFF)
option(ENABLE_COVERAGE "Enable coverage reports (includes enabling all tests and checks)" OFF)
option(ENABLE_WERROR "Treat all build warnings as errors" OFF)

if(ENABLE_COVERAGE)
    set(ENABLE_TESTS ON)
    set(CMAKE_BUILD_TYPE "Coverage")
else()
    set(CMAKE_BUILD_TYPE "Release")
endif()

if(ENABLE_WERROR)
    add_definitions("-Werror")
endif()

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    add_definitions("-Weverything")
else()
    add_definitions("-Wall")
endif()

# GNU standard installation directories
include (GNUInstallDirs)

# Gettext
set(GETTEXT_PACKAGE "ayatana-indicator-keyboard")
add_definitions (-DGETTEXT_PACKAGE="${GETTEXT_PACKAGE}" -DLOCALEDIR="${CMAKE_INSTALL_FULL_LOCALEDIR}")

# Check for prerequisites
find_package (PkgConfig REQUIRED)
include (CheckIncludeFile)
include (FindPkgConfig)
pkg_check_modules(SERVICE_DEPS REQUIRED glib-2.0>=2.36 gio-2.0>=2.36 libayatana-common>=0.9.3 accountsservice xkbcommon>=1.0.3 xkbregistry>=1.0.3)
pkg_check_modules(X11_DEPS REQUIRED x11>=1.6.5 libxklavier>=5.3)
include_directories (SYSTEM ${SERVICE_DEPS_INCLUDE_DIRS})

# custom targets
set (ARCHIVE_NAME ${CMAKE_PROJECT_NAME}-${PROJECT_VERSION})
add_custom_target (dist COMMAND bzr export --root=${ARCHIVE_NAME} ${CMAKE_BINARY_DIR}/${ARCHIVE_NAME}.tar.gz WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})
add_custom_target (cppcheck COMMAND cppcheck --enable=all -q --error-exitcode=2 --inline-suppr ${CMAKE_SOURCE_DIR}/src)

# Actual building
include_directories (${CMAKE_CURRENT_SOURCE_DIR}/include)
include_directories (${CMAKE_CURRENT_BINARY_DIR}/include)
add_subdirectory(src)
add_subdirectory(data)
add_subdirectory(po)

# Test and coverage
if (ENABLE_TESTS)

    include(CTest)
    enable_testing()

    if (ENABLE_COVERAGE)

        find_package(CoverageReport)

    endif()

endif()

# Display config info
message(STATUS "Install prefix: ${CMAKE_INSTALL_PREFIX}")
message(STATUS "Unit tests: ${ENABLE_TESTS}")
message(STATUS "Build with -Werror: ${ENABLE_WERROR}")
