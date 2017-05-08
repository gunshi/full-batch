cmake_minimum_required(VERSION 2.8)
project( main )
include(ExternalProject)

if(MSVC)
  if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
    string(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
  else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
  endif()
elseif(CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -pedantic")
endif()

set(HDRS
  includes/DLoopDetector.h         includes/TemplatedLoopDetector.h)

set(DEPENDENCY_DIR ${CMAKE_CURRENT_BINARY_DIR}/dependencies)
set(DEPENDENCY_INSTALL_DIR ${DEPENDENCY_DIR}/install)

macro(GetDependency name other_dependency)
  find_package(${name} QUIET 
    PATHS ${DEPENDENCY_INSTALL_DIR})
include_directories(${${name}_INCLUDE_DIRS})
    add_custom_target(${name})
endmacro(GetDependency)

GetDependency(DLib "")
GetDependency(DBoW2 DLib)
add_custom_target(Dependencies ${CMAKE_COMMAND} ${CMAKE_SOURCE_DIR} DEPENDS DBoW2 DLib)


# directories
set (INCLUDE_DIR includes)

# use sse3 instruction set
SET(CMAKE_CXX_FLAGS "-msse3")

# include directory
include_directories(${INCLUDE_DIR} ${OpenCV_INCLUDE_DIRS})

# Sources
FILE(GLOB SRC_FILES "src/*.cpp")

# make release version
# set(CMAKE_BUILD_TYPE Release)

# main executable
add_executable(main ${SRC_FILES} main.cpp)

find_package( OpenCV REQUIRED )





# Find Eigen3
FIND_PACKAGE(Eigen3 REQUIRED)
INCLUDE_DIRECTORIES( ${EIGEN3_INCLUDE_DIR})







FIND_PACKAGE(GTSAM REQUIRED)
IF (GTSAM_FOUND)
  SET (HAVE_GTSAM 1)
  INCLUDE_DIRECTORIES(${GTSAM_INCLUDE_DIR})
  MESSAGE(STATUS "Compiling with gtsam")
  MESSAGE(STATUS ${GTSAM_LIBS})
ENDIF()


find_library(GTSAM_LIBRARY gtsam HINTS /usr/local/lib)
MESSAGE(STATUS ${GTSAM_LIBRARY})
target_link_libraries(main ${GTSAM_LIBRARY})

find_library(BOOST_SYSTEM_LIBRARY boost_system HINTS /usr/lib/x86_64-linux-gnu)
MESSAGE(STATUS ${BOOST_SYSTEM_LIBRARY})
target_link_libraries(main ${BOOST_SYSTEM_LIBRARY})







target_link_libraries( main ${OpenCV_LIBS} ${DLib_LIBS} ${DBoW2_LIBS} gtsam)

