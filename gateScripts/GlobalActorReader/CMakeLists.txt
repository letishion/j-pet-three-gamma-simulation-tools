cmake_minimum_required(VERSION 2.6)

#MANDATORY: Before first cmake please insert here your path to ROOT installation directory
#set(ROOT_INSTALLATION_DIRECTORY "/home/julia/root2/build")

################################################################################
# Handling of thisroot.sh
################################################################################

#message(STATUS "Your ROOT installation directory is: ${ROOT_INSTALLATION_DIRECTORY}")

#if(EXISTS ${ROOT_INSTALLATION_DIRECTORY}/bin/thisroot.sh)
  #message(STATUS "Status of thisroot.sh file : EXIST")
#else()
 #message(FATAL_ERROR "Status of thisroot.sh file :  DOES NOT EXIST - is your path to ROOT installation directory is correct? Please check a variable ROOT_INSTALLATION_DIRECTORY in this CMakeList.txt file.")
#endif()

#message(STATUS "Calling thisroot.sh")
#execute_process(COMMAND source ${ROOT_INSTALLATION_DIRECTORY}/bin/thisroot.sh)

################################################################################
# Project configuration
################################################################################

set(projectName GAR)
set(projectBinary ${projectName}.x)

project(${projectName} CXX) # using only C++

add_definitions(-std=c++11 -Wall) # using C++11 standard

################################################################################
# Handling of ROOT 
################################################################################

# first, try the old-school ROOTSYS variable
set(root_prefix $ENV{ROOTSYS})
if(root_prefix)
  list(APPEND CMAKE_MODULE_PATH ${root_prefix}/etc/cmake/)  
endif()

# try also a search based on root-config
execute_process(COMMAND root-config --etcdir OUTPUT_VARIABLE ROOT_ETCDIR OUTPUT_STRIP_TRAILING_WHITESPACE)
list(APPEND CMAKE_MODULE_PATH ${ROOT_ETCDIR}/cmake)

find_package(ROOT QUIET COMPONENTS TreePlayer)

if(ROOT_FOUND)
  message(STATUS "Found ROOT version: ${ROOT_VERSION} (${ROOT_CONFIG_EXECUTABLE})")
else()
  # if we failed with CMake based find_package, use fallback
  # root-config version from cmake/fallback/
  list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/fallback)
  message(WARNING "No CMake modules from a ROOT installation found. Trying fallback mode.")
  find_package(ROOT 5 REQUIRED)
endif()

if(ROOT_USE_FILE)
  include(${ROOT_USE_FILE})
endif()

################################################################################
# Sources and linking 
################################################################################
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)  #to change the LD_LIBRARY_PATH and accept pcm

include_directories(${CMAKE_CURRENT_SOURCE_DIR})
ROOT_GENERATE_DICTIONARY(EventDict Event.h LINKDEF EventLinkDef.h)
#---Create a shared library with geneated dictionary
add_library(Event SHARED Event.cxx EventDict.cxx)
target_link_libraries(Event ${ROOT_LIBRARIES})


file(GLOB HEADERS *.hh)
file(GLOB SOURCES *.cc)

add_executable(GAR ${SOURCES} Event.cxx)
target_link_libraries(GAR ${ROOT_LIBRARIES} Event)
