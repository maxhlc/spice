# Declare minimum CMake version
cmake_minimum_required(VERSION 3.16)

# Declare project
project(spice LANGUAGES Fortran VERSION 0067)

# Set output directories
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin)

# Set download name
set(SPICE_DOWNLOAD ${PROJECT_NAME}-download)

# Set source and header file directories
set(SPICE_SOURCE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/src)

# Set installation directories
include(GNUInstallDirs)
set(SPICE_RUNTIME_INSTALL_DIR "${CMAKE_INSTALL_BINDIR}")
set(SPICE_LIBRARY_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}")
set(SPICE_ARCHIVE_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}")

# Set SPICE URL based on operating system
if(UNIX)
    # Set Linux version
    set(SPICE_URL https://naif.jpl.nasa.gov/pub/naif/toolkit//FORTRAN/PC_Linux_gfortran_64bit/packages/toolkit.tar.Z)
elseif(APPLE)
    # Set Apple version
    set(SPICE_URL https://naif.jpl.nasa.gov/pub/naif/toolkit//FORTRAN/MacIntel_OSX_gfortran_64bit/packages/toolkit.tar.Z)
else()
    # Set Windows (CYGWIN) version
    # Note: untested
    set(SPICE_URL https://naif.jpl.nasa.gov/pub/naif/toolkit//FORTRAN/PC_Cygwin_gfortran_64bit/packages/toolkit.tar.gz)
endif()

# Add module to download source code
include(FetchContent)

# Show download progress
set(FETCHCONTENT_QUIET FALSE)

# Download source code
FetchContent_Declare(
    ${SPICE_DOWNLOAD}
    URL ${SPICE_URL}
)
FetchContent_MakeAvailable(${SPICE_DOWNLOAD})

# Glob source files
file(GLOB_RECURSE SPICE_SOURCE_FILES ${${SPICE_DOWNLOAD}_SOURCE_DIR}/src/spicelib/*.f)

# Set compiler flags
set(CMAKE_Fortran_FLAGS                "-m64 -fPIC -std=legacy -Wno-character-truncation")
set(CMAKE_Fortran_FLAGS_DEBUG          "-g")
set(CMAKE_Fortran_FLAGS_RELEASE        "")
set(CMAKE_Fortran_FLAGS_RELWITHDEBINFO "${CMAKE_Fortran_FLAGS_DEBUG} ${CMAKE_Fortran_FLAGS_RELEASE}")
set(CMAKE_Fortran_FLAGS_MINSIZEREL     "")

# Compile SPICE
add_library(${PROJECT_NAME} ${SPICE_SOURCE_FILES})
set_target_properties(${PROJECT_NAME} PROPERTIES POSITION_INDEPENDENT_CODE ON)

# Create and install the configuration files
include(CMakePackageConfigHelpers)
write_basic_package_version_file("${PROJECT_NAME}ConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)
configure_package_config_file(
    "${PROJECT_SOURCE_DIR}/cmake/config.cmake.in"
    "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/cmake/${PROJECT_NAME}
    PATH_VARS CMAKE_INSTALL_INCLUDEDIR
)
install(FILES "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/cmake/${PROJECT_NAME}
)

# Install SPICE
install(TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}Targets
    RUNTIME DESTINATION ${SPICE_RUNTIME_INSTALL_DIR}
    LIBRARY DESTINATION ${SPICE_LIBRARY_INSTALL_DIR}
    ARCHIVE DESTINATION ${SPICE_ARCHIVE_INSTALL_DIR}
)

# Install SPICE targets
install(EXPORT ${PROJECT_NAME}Targets
    FILE ${PROJECT_NAME}Targets.cmake
    NAMESPACE ${PROJECT_NAME}::
    DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/cmake/${PROJECT_NAME}
)