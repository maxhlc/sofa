# Declare minimum CMake version
cmake_minimum_required(VERSION 3.16)

# Declare project
project(sofa-c LANGUAGES C VERSION 20210512)

# Set output directories
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin)

# Set download name
set(SOFA_DOWNLOAD ${PROJECT_NAME}-download)

# Set header file directories
set(SOFA_HEADER_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include)

# Set installation directories
include(GNUInstallDirs)
set(SOFA_INCLUDE_INSTALL_DIR "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}")
set(SOFA_RUNTIME_INSTALL_DIR "${CMAKE_INSTALL_BINDIR}")
set(SOFA_LIBRARY_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}")
set(SOFA_ARCHIVE_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}")

# Set SOFA URL based on operating system
if(UNIX)
    # Version with UNIX line terminations
    set(SOFA_URL http://www.iausofa.org/2021_0512_C/sofa_c-20210512.tar.gz)
elseif(WIN32)
    # Version with DOS line terminations
    set(SOFA_URL https://www.iausofa.org/2021_0512_C/sofa_c-20210512.zip)
endif()

# Add module to download source code
include(FetchContent)

# Show download progress
set(FETCHCONTENT_QUIET FALSE)

# Download source code
FetchContent_Declare(
    ${SOFA_DOWNLOAD}
    URL ${SOFA_URL}
)
FetchContent_MakeAvailable(${SOFA_DOWNLOAD})

# Set source file variables
file(GLOB_RECURSE SOFA_FETCH_SOURCE_FILES ${${SOFA_DOWNLOAD}_SOURCE_DIR}/20210512/c/src/*.c)
set(SOFA_SOURCE_FILES ${SOFA_FETCH_SOURCE_FILES})
set(SOFA_TEST_FILE ${${SOFA_DOWNLOAD}_SOURCE_DIR}/20210512/c/src/t_sofa_c.c)

# Set header file variables (header files copied to avoid including source files)
file(GLOB_RECURSE SOFA_FETCH_HEADER_FILES ${${SOFA_DOWNLOAD}_SOURCE_DIR}/20210512/c/src/*.h)
file(COPY ${SOFA_FETCH_HEADER_FILES} DESTINATION ${SOFA_HEADER_DIRECTORY})
file(GLOB_RECURSE SOFA_HEADER_FILES ${SOFA_HEADER_DIRECTORY}/*.h)

# Set compiler flags
set(CMAKE_C_FLAGS                "-pedantic -Wall -O")
set(CMAKE_C_FLAGS_DEBUG          "")
set(CMAKE_C_FLAGS_RELEASE        "")
set(CMAKE_C_FLAGS_RELWITHDEBINFO "${CMAKE_Fortran_FLAGS_DEBUG} ${CMAKE_CXX_FLAGS_RELEASE}")
set(CMAKE_C_FLAGS_MINSIZEREL     "")

# Compile SOFA
add_library(${PROJECT_NAME} ${SOFA_SOURCE_FILES} ${SOFA_HEADER_FILES})
target_include_directories(${PROJECT_NAME}
    PUBLIC $<BUILD_INTERFACE:${SOFA_HEADER_DIRECTORY}>
           $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
           $<INSTALL_INTERFACE:${SOFA_INCLUDE_INSTALL_DIR}>
    PRIVATE ${SOFA_HEADER_DIRECTORY}
)
set_target_properties(${PROJECT_NAME} PROPERTIES POSITION_INDEPENDENT_CODE ON)
set_target_properties(${PROJECT_NAME} PROPERTIES PUBLIC_HEADER "${SOFA_HEADER_FILES}")

# Compile test
add_executable(${PROJECT_NAME}_test ${SOFA_TEST_FILE})
if(MSVC)
    target_link_libraries(${PROJECT_NAME}_test PRIVATE ${PROJECT_NAME})
else()
    target_link_libraries(${PROJECT_NAME}_test PRIVATE ${PROJECT_NAME} m)
endif()

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

# Install SOFA
install(TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}Targets
    RUNTIME DESTINATION ${SOFA_RUNTIME_INSTALL_DIR}
    LIBRARY DESTINATION ${SOFA_LIBRARY_INSTALL_DIR}
    ARCHIVE DESTINATION ${SOFA_ARCHIVE_INSTALL_DIR}
    PUBLIC_HEADER DESTINATION ${SOFA_INCLUDE_INSTALL_DIR}
)

# Install SOFA targets
install(EXPORT ${PROJECT_NAME}Targets
    FILE ${PROJECT_NAME}Targets.cmake
    NAMESPACE ${PROJECT_NAME}::
    DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/cmake/${PROJECT_NAME}
)