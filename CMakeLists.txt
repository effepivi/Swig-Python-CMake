cmake_minimum_required(VERSION 3.11)
 
PROJECT(example)
SET (module example)


# Initialise some states by default ############################################
SET (BUILD_SHARED_LIBS OFF)


# Minimum CMake version ########################################################
cmake_minimum_required(VERSION 3.1 FATAL_ERROR)
set(CMAKE_CXX_STANDARD 11) # C++11
set(CMAKE_CXX_STANDARD_REQUIRED ON) # C++11 is required (I.e. not optional)
set(CMAKE_CXX_EXTENSIONS OFF) # without compiler extensions like gnu++11


# Set a default build type if none was specified
IF (NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
      message(STATUS "Setting build type to 'Release' as none was specified.")
      SET (CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)

      # Set the possible values of build type for cmake-gui
      set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
ENDIF ()


# CMake policies ###############################################################
if(${CMAKE_VERSION} VERSION_LESS "3.13.0")
    MESSAGE (WARNING "Please consider to switch to CMake 3.13.0 or newer")
endif()


if(${CMAKE_VERSION} VERSION_GREATER "3.13.0" OR ${CMAKE_VERSION} VERSION_EQUAL "3.13.0")
    cmake_policy(SET CMP0078 NEW) # SWIG
endif()


if(${CMAKE_VERSION} VERSION_GREATER "3.14.0" OR ${CMAKE_VERSION} VERSION_EQUAL "3.14.0")
    cmake_policy(SET CMP0086 NEW) # SWIG
endif()


# Find SWIG
FIND_PACKAGE(SWIG REQUIRED)
INCLUDE(${SWIG_USE_FILE})
 
 
# Include python
find_package(PythonLibs 3 REQUIRED)
find_package(PythonInterp REQUIRED)


# Where the header files are
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR})
INCLUDE_DIRECTORIES(${PYTHON_INCLUDE_DIR})


# Interface file for SWIG
CONFIGURE_FILE (${module}.i.in ${CMAKE_CURRENT_BINARY_DIR}/${module}.i)

SET_PROPERTY(
    SOURCE ${CMAKE_CURRENT_BINARY_DIR}/${module}.i
    PROPERTY CPLUSPLUS ON)

SET_PROPERTY(
    SOURCE ${CMAKE_CURRENT_BINARY_DIR}/${module}.i
    PROPERTY SWIG_MODULE_NAME ${module})


# Python script to test the wrapper
FILE(COPY test_${module}.py
		DESTINATION ${CMAKE_CURRENT_BINARY_DIR}
		FILE_PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ
  			GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)


# Add swig module

SET (SOURCE_FILES
   ${module}.h
    ${module}.cxx
    ${CMAKE_CURRENT_BINARY_DIR}/${module}.i
)

IF (${CMAKE_VERSION} VERSION_LESS "3.8.0")
    SWIG_ADD_MODULE (${module} python ${SOURCE_FILES})
ELSE (${CMAKE_VERSION} VERSION_LESS "3.8.0")
    SWIG_ADD_LIBRARY (${module} LANGUAGE python SOURCES ${SOURCE_FILES})
ENDIF (${CMAKE_VERSION} VERSION_LESS "3.8.0")

swig_link_libraries(${module} ${PYTHON_LIBRARIES})

