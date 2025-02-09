# Adapted from: https://cmake.org/Wiki/CMakeForFortranExample
# CMake project file for HAZ Tornado GMC

cmake_minimum_required (VERSION 2.8.12)
project (haz-tornado-gmc)
enable_language (Fortran)

option(STATIC "Enable static linking for binary distribution")

# make sure that the default is a RELEASE
if (NOT CMAKE_BUILD_TYPE)
  set (CMAKE_BUILD_TYPE RELEASE CACHE STRING
      "Choose the type of build, options are: None Debug Release."
      FORCE)
endif (NOT CMAKE_BUILD_TYPE)

# default installation
get_filename_component (default_prefix ".." ABSOLUTE)
set (CMAKE_INSTALL_PREFIX ${default_prefix} CACHE STRING
      "Choose the installation directory; by default it installs in the build directory."
      FORCE)

# FFLAGS depend on the compiler
get_filename_component (Fortran_COMPILER_NAME ${CMAKE_Fortran_COMPILER} NAME)

if (Fortran_COMPILER_NAME MATCHES "gfortran.*")
  # gfortran
  # Flags required for source formatting
  set (CMAKE_Fortran_FLAGS "-ffixed-line-length-132 -fno-f2c") # -std=legacy")
  # Static compliation
  if (STATIC)
      message("--> Static build.")
      set (CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} -static")
  endif (STATIC)
  # Optimization flags
  set (CMAKE_Fortran_FLAGS_RELEASE "-O3 -ffpe-summary=none -ffast-math -funroll-loops -finline-limit=600 -fstack-arrays") # -ffpe-trap=invalid,zero,overflow,underflow,denormal 
  set (CMAKE_Fortran_FLAGS_DEBUG   "-O0 -g -std=legacy -fbacktrace -ffree-line-length-0 -fcheck=all -fcheck=bounds -finit-real=nan -Wall -ffpe-trap=invalid,zero,overflow,underflow")
elseif (Fortran_COMPILER_NAME MATCHES "ifort.*")
  # ifort (untested)
  set (CMAKE_Fortran_FLAGS_RELEASE "-extend-source 132 -warn interfaces -check bounds -threads -c -fixed -assume buffered_io -traceback -static -heap-arrays 10")
  set (CMAKE_Fortran_FLAGS_DEBUG   "-O0 -g")

endif (Fortran_COMPILER_NAME MATCHES "gfortran.*")

message ("--> Fortran compiler: " ${Fortran_COMPILER_NAME})
message ("--> CMAKE_Fortran_COMPILER full path: " ${CMAKE_Fortran_COMPILER})
message ("--> CMAKE_Fortran_FLAGS: " ${CMAKE_Fortran_FLAGS})

# build executables
file (GLOB SOURCES CONFIGURE_DEPENDS src/*.f)
set (TARGET "haz-tornado-gmc")

add_executable (${TARGET} ${SOURCES})
set_target_properties(${TARGET} PROPERTIES LINKER_LANGUAGE Fortran)
