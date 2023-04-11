# Declare minimum CMake version
cmake_minimum_required(VERSION 3.16)

# Declare project
project(sofa VERSION 20210512)

# Add ANSI C version
add_subdirectory(c)

# Add Fortran version
add_subdirectory(fortran)