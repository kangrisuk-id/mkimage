cmake_minimum_required(VERSION 3.16)
project(mkimage VERSION 1.0.0 LANGUAGES C)

# Standard C11
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Check include headers
include(CheckIncludeFiles)
check_include_files("inttypes.h" HAVE_INTTYPES_H)
check_include_files("stdint.h" HAVE_STDINT_H)
check_include_files("stdlib.h" HAVE_STDLIB_H)
check_include_files("string.h" HAVE_STRING_H)
check_include_files("unistd.h" HAVE_UNISTD_H)

# Optional ZLIB dependency
find_package(ZLIB QUIET)
if(ZLIB_FOUND)
    set(HAVE_ZLIB 1)
endif()

# Generate config.h inside build directory
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/config.h.in"
    "${CMAKE_CURRENT_BINARY_DIR}/config.h"
    @ONLY
)

# Include directories
include_directories(
    ${CMAKE_CURRENT_BINARY_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${CMAKE_CURRENT_SOURCE_DIR}/src
)

# Compiler options
if(MSVC)
    add_compile_options(/W3 /utf-8)
else()
    add_compile_options(-Wall -Wextra -Wno-unused-parameter)
endif()

# Source files
file(GLOB MKIMAGE_SOURCES
    "src/*.c"
    "*.c"
)
list(FILTER MKIMAGE_SOURCES EXCLUDE REGEX "test_.*\\.c$")

# Target Executable
add_executable(mkimage ${MKIMAGE_SOURCES})

if(ZLIB_FOUND)
    target_link_libraries(mkimage PRIVATE ZLIB::ZLIB)
endif()

if(WIN32)
    set_target_properties(mkimage PROPERTIES OUTPUT_NAME "mkimage")
endif()

# Installation target
install(TARGETS mkimage DESTINATION bin)

if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/README.md")
    install(FILES README.md DESTINATION .)
endif()

# CPack Configuration (.zip & .tar.gz)
set(CPACK_PACKAGE_NAME "mkimage")
set(CPACK_PACKAGE_VENDOR "GRUB4DOS Team")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "GRUB4DOS EFI Image Creation Tool")
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}-${CMAKE_SYSTEM_NAME}-${CMAKE_SYSTEM_PROCESSOR}")

if(WIN32)
    set(CPACK_GENERATOR "ZIP")
else()
    set(CPACK_GENERATOR "TGZ;ZIP")
endif()

include(CPack)
