# SPICE
Unofficial repository for the Fortran version of the [SPICE toolkit](https://naif.jpl.nasa.gov/naif/toolkit.html).

The CMake configuration automatically downloads the SPICE source from the official website and compiles the library.

## Getting Started

### Dependencies
* CMake
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