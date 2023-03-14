# Declare minimum CMake version
cmake_minimum_required(VERSION 3.16)

# Declare project
project(sofa LANGUAGES C VERSION 20210512)

# Set output directories
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin)

# Set source and header file directories
set(SOFA_SOURCE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(SOFA_HEADER_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include)

# Set installation directories
include(GNUInstallDirs)
set(SOFA_INCLUDE_INSTALL_DIR "${CMAKE_INSTALL_INCLUDEDIR}")
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
    sofa
    URL ${SOFA_URL}
)
FetchContent_MakeAvailable(sofa)

# Glob source and header files
file(GLOB_RECURSE SOFA_FETCH_SOURCE_FILES ${sofa_SOURCE_DIR}/20210512/c/src/*.c)
file(GLOB_RECURSE SOFA_FETCH_HEADER_FILES ${sofa_SOURCE_DIR}/20210512/c/src/*.h)

# Copy source and header files
file(COPY ${SOFA_FETCH_SOURCE_FILES} DESTINATION ${SOFA_SOURCE_DIRECTORY})
file(COPY ${SOFA_FETCH_HEADER_FILES} DESTINATION ${SOFA_HEADER_DIRECTORY})

# Glob source and header files
file(GLOB_RECURSE SOFA_SOURCE_FILES ${SOFA_SOURCE_DIRECTORY}/*.c)
file(GLOB_RECURSE SOFA_HEADER_FILES ${SOFA_HEADER_DIRECTORY}/*.h)

# Set compiler flags
set_source_files_properties(${SOFA_SOURCE_FILES} PROPERTIES COMPILE_FLAGS "-pedantic -Wall -O")

# Compile SOFA
add_library(sofa ${SOFA_SOURCE_FILES} ${SOFA_HEADER_FILES})
target_include_directories(sofa PUBLIC $<BUILD_INTERFACE:${SOFA_HEADER_DIRECTORY}>
                                       $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
                                       $<INSTALL_INTERFACE:${SOFA_INCLUDE_INSTALL_DIR}>
                                PRIVATE ${SOFA_HEADER_DIRECTORY}
)
set_target_properties(sofa PROPERTIES POSITION_INDEPENDENT_CODE ON)

# Compile test
add_executable(sofa_test ${SOFA_SOURCE_DIRECTORY}/t_sofa_c.c)
if(MSVC)
    target_link_libraries(sofa_test PRIVATE sofa)
else()
    target_link_libraries(sofa_test PRIVATE sofa m)
endif()

# Install SOFA
install(TARGETS sofa
        RUNTIME DESTINATION ${SOFA_RUNTIME_INSTALL_DIR}
        LIBRARY DESTINATION ${SOFA_LIBRARY_INSTALL_DIR}
        ARCHIVE DESTINATION ${SOFA_ARCHIVE_INSTALL_DIR}
)
install(FILES ${SOFA_HEADER_FILES} DESTINATION ${SOFA_INCLUDE_INSTALL_DIR})
