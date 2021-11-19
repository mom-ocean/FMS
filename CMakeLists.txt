cmake_minimum_required(VERSION 3.6)
project(FMS C Fortran)

# make sure that the default is a RELWITHDEBINFO
if (NOT CMAKE_BUILD_TYPE)
  set (CMAKE_BUILD_TYPE RELWITHDEBINFO CACHE STRING
      "Choose the type of build, options are: Debug Release Relwithdebinfo."
      FORCE)
endif()
message("Build type: " ${CMAKE_BUILD_TYPE})

find_package(MPI)
if(NOT MPI_Fortran_FOUND)
    message(FATAL_ERROR "Could not find Fortran MPI.  Please set MPI_Fortran_COMPILER to point to the mpifort wrapper.")
endif()

include_directories(${MPI_Fortran_INCLUDE_PATH})
add_compile_options(${MPI_Fortran_COMPILE_FLAGS})

message("Using Fortran: ${CMAKE_Fortran_COMPILER_ID} ${CMAKE_Fortran_COMPILER} and C: ${CMAKE_C_COMPILER_ID} ${CMAKE_C_COMPILER}")

if(CMAKE_Fortran_COMPILER_ID STREQUAL "Intel")
  set(CMAKE_Fortran_FLAGS                "${CMAKE_Fortran_FLAGS} -fno-alias -stack_temps -safe-cray-ptr -ftz -i_dynamic -assume byterecl -i4 -r8 -nowarn -sox -traceback")
  set(CMAKE_Fortran_FLAGS_RELEASE        "${CMAKE_Fortran_FLAGS_RELEASE} -O2 -fp-model source")
  set(CMAKE_Fortran_FLAGS_RELWITHDEBINFO "${CMAKE_Fortran_FLAGS_RELWITHDEBINFO} -g")
  set(CMAKE_Fortran_FLAGS_DEBUG          "${CMAKE_Fortran_FLAGS_DEBUG} -g -O0 -check -check noarg_temp_created -check nopointer -warn -warn noerrors -fpe0 -ftrapuv")
elseif(CMAKE_Fortran_COMPILER_ID STREQUAL "GNU")
  set(CMAKE_Fortran_FLAGS                "${CMAKE_Fortran_FLAGS} -fcray-pointer -fdefault-real-8 -fdefault-double-8 -Waliasing -ffree-line-length-none -fno-range-check")
  set(CMAKE_Fortran_FLAGS_RELEASE        "${CMAKE_Fortran_FLAGS_RELEASE} -O2 -fbounds-check")
  set(CMAKE_Fortran_FLAGS_RELWITHDEBINFO "${CMAKE_Fortran_FLAGS_RELWITHDEBINFO} -g")
  set(CMAKE_Fortran_FLAGS_DEBUG          "${CMAKE_Fortran_FLAGS_DEBUG} -O -g -Wall -fcheck=bounds -ffpe-trap=invalid,zero,overflow -ftrapv -fimplicit-none")
else ()
  message ("Unknown FORTRAN compiler default flags only...")
endif()

execute_process(COMMAND nf-config --includedir OUTPUT_VARIABLE NETCDF_INCDIR OUTPUT_STRIP_TRAILING_WHITESPACE)
execute_process(COMMAND nf-config --flibs OUTPUT_VARIABLE NETCDF_FLIBS OUTPUT_STRIP_TRAILING_WHITESPACE)
set(CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} -I${NETCDF_INCDIR}")

if(CMAKE_C_COMPILER_ID STREQUAL "Intel")
  set(CMAKE_C_FLAGS         "${CMAKE_C_FLAGS} -sox -traceback") 
  set(CMAKE_C_FLAGS_DEBUG   "${CMAKE_C_FLAGS_DEBUG} -O0 -debug")
  set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O2 -debug minimal")
elseif(CMAKE_C_COMPILER_ID STREQUAL "GNU")
  set(CMAKE_C_FLAGS         "${CMAKE_C_FLAGS} -frecord-gcc-switches")
  set(CMAKE_C_FLAGS_DEBUG   "${CMAKE_C_FLAGS_DEBUG} -O0")
  set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O2")
else ()
  message ("Unknown C compiler default flags only...")
endif()

string(TOUPPER ${CMAKE_BUILD_TYPE} BUILD_TYPE)
message("Using Fortran flags: ${CMAKE_Fortran_FLAGS} ${CMAKE_Fortran_FLAGS_${BUILD_TYPE}}")
message("Using C flags: ${CMAKE_C_FLAGS} ${CMAKE_C_FLAGS_${BUILD_TYPE}}")

add_definitions(-Duse_libMPI -Duse_netCDF -Duse_LARGEFILE -DSPMD -D__IFC)

set(FMS_DIR "${CMAKE_CURRENT_SOURCE_DIR}")

# FMS static library
file(GLOB_RECURSE FMS_SOURCE LIST_DIRECTORIES false
  ${FMS_DIR}/*.[fF]90 ${FMS_DIR}/*.c)

add_library(FMS ${FMS_SOURCE})

target_include_directories(FMS PRIVATE
  /usr/include ${FMS_DIR}/include ${FMS_DIR}/mosaic ${FMS_DIR}/drifters ${FMS_DIR}/fms ${FMS_DIR}/mpp/include)
