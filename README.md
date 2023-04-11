# SOFA
Unofficial repository for the ANSI C and Fortran versions of the [Standards of Fundamental Astronomy (SOFA)](https://www.iausofa.org/index.html) library.

The CMake configuration automatically downloads the SOFA sources from the official website and compiles the libraries.

## Getting Started

### Dependencies
* CMake
* gcc
* gfortran

### Compilation
The project can be configured by:
```
cmake -S . -B build
```
and compiled by:
```
cmake --build build
```

### Installation
The project can be installed by running the install command:
```
cmake --install build
```
Note: elevated privileges may be required for installation.

## Authors
* Max Hallgarten La Casta (m.hallgarten-la-casta21@imperial.ac.uk)

## License
This project is licensed under the MIT License - see the `LICENSE` file for more details.