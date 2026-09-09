cmake_minimum_required(VERSION 3.16)
project(CorbaPlugin)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING
    "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
    FORCE)
endif()

if(NOT Choreonoid_FOUND)
  file(GLOB choreonoid_search_path
    "c:/choreonoid*/share/choreonoid/cmake"
    "c:/choreonoid*/*/share/choreonoid/cmake")
endif()
if(choreonoid_search_path)
  find_package(Choreonoid REQUIRED HINTS ${choreonoid_search_path})
else()
  find_package(Choreonoid REQUIRED)
endif()
  

  
if(CHOREONOID_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD ${CHOREONOID_CXX_STANDARD})
  set(CMAKE_CXX_EXTENSIONS OFF)
else()
  add_compile_options(-std=c++14)
endif()

if(MSVC)
  if(${MSVC_VERSION} GREATER_EQUAL 1910)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /Zc:__cplusplus")
  endif()
endif()

include_directories(${CHOREONOID_INCLUDE_DIRS})
link_directories(${CHOREONOID_LIBRARY_DIRS})
set_property(DIRECTORY APPEND PROPERTY COMPILE_DEFINITIONS ${CHOREONOID_COMPILE_DEFINITIONS})


# Note that the OMNIORB_XXX varialbes cannot be used because
# the variables collide with those of OpenRTM-aist configuration.
find_package(PkgConfig REQUIRED)
if(UNIX)
  if(NOT CHOREONOID_OMNIORB_DIR)
    # The required package can be installed by the following command in Ubuntu.
    # sudo apt install libomniorb4-dev libcos4-dev omniidl omniorb-nameserver
    pkg_check_modules(CHOREONOID_OMNIORB omniDynamic4)
    if(CHOREONOID_OMNIORB_FOUND)
set(CHOREONOID_OMNIORB_DIR ${CHOREONOID_OMNIORB_PREFIX})
    endif()
  else()
    set(CHOREONOID_OMNIORB_FOUND TRUE)
    set(CHOREONOID_OMNIORB_INCLUDE_DIRS ${CHOREONOID_OMNIORB_DIR}/include)
    set(CHOREONOID_OMNIORB_LIBRARY_DIRS ${CHOREONOID_OMNIORB_DIR}/lib)
  endif()
elseif(MSVC)
  if(NOT CHOREONOID_OMNIORB_DIR)
    if(NOT $ENV{OMNI_ROOT} STREQUAL "")
set(CHOREONOID_OMNIORB_DIR $ENV{OMNI_ROOT})
    endif()
  endif()
  if(CHOREONOID_OMNIORB_DIR)
    set(CHOREONOID_OMNIORB_FOUND TRUE)
    set(CHOREONOID_OMNIORB_INCLUDE_DIRS "${CHOREONOID_OMNIORB_DIR}/include")
    set(CHOREONOID_OMNIORB_LIBRARY_DIRS "${CHOREONOID_OMNIORB_DIR}/lib/x86_win32")
    set(CHOREONOID_OMNIORB_BINARY_DIR "${CHOREONOID_OMNIORB_DIR}/bin/x86_win32")
    set(CHOREONOID_OMNIORB_CFLAGS -D__WIN32__ -D__x86__ )

    file(GLOB libomniorb RELATIVE ${CHOREONOID_OMNIORB_LIBRARY_DIRS} "${CHOREONOID_OMNIORB_LIBRARY_DIRS}/omniORB???_rt.lib")
    get_filename_component(libomniorb ${libomniorb} NAME_WE)

    file(GLOB libomnithread RELATIVE ${CHOREONOID_OMNIORB_LIBRARY_DIRS} "${CHOREONOID_OMNIORB_LIBRARY_DIRS}/omnithread??_rt.lib")
    get_filename_component(libomnithread ${libomnithread} NAME_WE)

    file(GLOB libomnidynamic RELATIVE ${CHOREONOID_OMNIORB_LIBRARY_DIRS} "${CHOREONOID_OMNIORB_LIBRARY_DIRS}/omniDynamic???_rt.lib")
    get_filename_component(libomnidynamic ${libomnidynamic} NAME_WE)

    set(CHOREONOID_OMNIORB_LIBRARIES_RELEASE ${libomniorb} ${libomnithread} ${libomnidynamic})
    foreach(library ${CHOREONOID_OMNIORB_LIBRARIES_RELEASE})
list(APPEND CHOREONOID_OMNIORB_LIBRARIES optimized ${library} debug ${library}d )
    endforeach()

    file(GLOB libomniorb RELATIVE ${CHOREONOID_OMNIORB_BINARY_DIR} "${CHOREONOID_OMNIORB_BINARY_DIR}/omniORB*_rt.dll")
    get_filename_component(libomniorb ${libomniorb} NAME_WE)

    file(GLOB libomnithread RELATIVE ${CHOREONOID_OMNIORB_BINARY_DIR} "${CHOREONOID_OMNIORB_BINARY_DIR}/omnithread*_rt.dll")
    get_filename_component(libomnithread ${libomnithread} NAME_WE)

    file(GLOB libomnidynamic RELATIVE ${CHOREONOID_OMNIORB_BINARY_DIR} "${CHOREONOID_OMNIORB_BINARY_DIR}/omniDynamic*_rt.dll")
    get_filename_component(libomnidynamic ${libomnidynamic} NAME_WE)

    set(CHOREONOID_OMNIORB_DLL_BASES ${libomniorb} ${libomnithread} ${libomnidynamic})

    if(INSTALL_RUNTIME_DEPENDENCIES)
foreach(library ${CHOREONOID_OMNIORB_DLL_BASES})
  install(PROGRAMS "${CHOREONOID_OMNIORB_BINARY_DIR}/${library}.dll" DESTINATION bin CONFIGURATIONS Release RelWithDebInfo MinSizeRel)
  install(PROGRAMS "${CHOREONOID_OMNIORB_BINARY_DIR}/${library}d.dll" DESTINATION bin CONFIGURATIONS Debug)
endforeach()
    endif()
  endif()
endif()

include_directories(${PROJECT_SOURCE_DIR})
include_directories(${PROJECT_BINARY_DIR}/include)
include_directories(${CHOREONOID_OMNIORB_INCLUDE_DIRS})
link_directories(${CHOREONOID_OMNIORB_LIBRARY_DIRS})
list(APPEND CHOREONOID_SDK_INCLUDE_DIRS ${CHOREONOID_OMNIORB_INCLUDE_DIRS})
list(APPEND CHOREONOID_SDK_LIBRARY_DIRS ${CHOREONOID_OMNIORB_LIBRARY_DIRS})
add_definitions(${CHOREONOID_OMNIORB_CFLAGS})

set(CHOREONOID_OMNIORB_DIR ${CHOREONOID_OMNIORB_DIR} CACHE PATH "The top directory of omniORB")
set(CHOREONOID_OMNIORB_CFLAGS ${CHOREONOID_OMNIORB_CFLAGS} CACHE STRING "Compile flags for omniORB")

if(NOT CHOREONOID_OMNIORB_FOUND)
  message(FATAL_ERROR "CORBA-related modules require the omniORB library but the library is not found.")
endif()

if(ENABLE_PYTHON)
  install(DIRECTORY ${PROJECT_BINARY_DIR}/${CHOREONOID_PYTHON_SUBDIR} DESTINATION ${CHOREONOID_PLUGIN_SUBDIR}
    FILES_MATCHING PATTERN "*.py")
endif()

set(ENABLE_CORBA ON)
set(ENABLE_GUI ON)
include(cmake/ChoreonoidCorbaBuildFunctions.cmake)
find_package(fmt REQUIRED)

add_subdirectory(src)
