# Declare minimum CMake version
cmake_minimum_required(VERSION 3.16)

# Declare project
project(sofa VERSION 20210512)

# Add language options
option(SOFA_BUILD_C "Build the ANSI C version of SOFA" ON)
option(SOFA_BUILD_F "Build the Fortran version of SOFA" ON)

# Add ANSI C version
if(SOFA_BUILD_C)
    add_subdirectory(c)
endif(SOFA_BUILD_C)

# Add Fortran version
if(SOFA_BUILD_F)
    add_subdirectory(fortran)
endif(SOFA_BUILD_F)