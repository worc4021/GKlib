cmake_minimum_required(VERSION 3.5)
project(GKlib C)

option(BUILD_SHARED_LIBS "Build shared libraries (.dll/.so) instead of static ones (.lib/.a)" OFF)

enable_testing()
include(CTest)
include(GNUInstallDirs)

get_filename_component(abs "." ABSOLUTE)
set(GKLIB_PATH ${abs})
unset(abs)
include(GKlibSystem.cmake)

include_directories(".")
if(WIN32)
  include_directories("win32")
  file(GLOB win32_sources RELATIVE "win32" "*.c")
  install(FILES win32/adapt.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/GKlib/win32)
else(WIN32)
  set(win32_sources, "")
endif(WIN32)

if (BUILD_SHARED_LIBS)
  add_library(GKlib SHARED ${GKlib_sources} ${win32_sources})
else()
  add_library(GKlib STATIC ${GKlib_sources} ${win32_sources})
endif()
target_include_directories(GKlib 
  PUBLIC 
    $<BUILD_INTERFACE:${GKLIB_PATH}> 
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/GKlib>)

set_target_properties(GKlib PROPERTIES
    POSITION_INDEPENDENT_CODE ON
    MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

if(UNIX)
  target_link_libraries(GKlib m)
endif(UNIX)


add_subdirectory(test)

install(TARGETS GKlib
        EXPORT GKlibTargets
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/${CMAKE_BUILD_TYPE}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}/${CMAKE_BUILD_TYPE}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}/${CMAKE_BUILD_TYPE})
install(FILES ${GKlib_includes} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/GKlib)
install(EXPORT GKlibTargets
        FILE GKlibTargets.cmake
        NAMESPACE GKlib::
        DESTINATION cmake)