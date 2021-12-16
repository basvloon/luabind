# Build for LuaBind
# Ryan Pavlik <rpavlik@iastate.edu>
# http://academic.cleardefinition.com/
# Iowa State University HCI Graduate Program/VRAC
# Update build script by Dennis Luxen, Project OSRM

cmake_minimum_required(VERSION 3.0)
project(LuaBind)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# Check the release mode
if(NOT CMAKE_BUILD_TYPE MATCHES Debug)
	set(CMAKE_BUILD_TYPE Release)
endif(NOT CMAKE_BUILD_TYPE MATCHES Debug)
if(CMAKE_BUILD_TYPE MATCHES Debug)
    message(STATUS "Configuring luabind in debug mode")
    message(STATUS "Enabling unit tests")
endif(CMAKE_BUILD_TYPE MATCHES Debug)
if(CMAKE_BUILD_TYPE MATCHES Release)
    message(STATUS "Configuring luabind in release mode")
    message(STATUS "Disabling unit tests")
	set(BUILD_TESTING 0)
endif(CMAKE_BUILD_TYPE MATCHES Release)

add_definitions( "-Wall" )
# Configuring other platform dependencies
if(APPLE)
  set(CMAKE_OSX_ARCHITECTURES "x86_64")
  message(STATUS "Set Architecture to x64 on OS X")
  exec_program(uname ARGS -v  OUTPUT_VARIABLE DARWIN_VERSION)
  string(REGEX MATCH "[0-9]+" DARWIN_VERSION ${DARWIN_VERSION})
  if(DARWIN_VERSION GREATER 12 AND NOT OSXLIBSTD)
    message(STATUS "Activating -std=c++11 flag for >= OS X 10.9")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
  endif()
  if(OSXLIBSTD)
    message(STATUS "linking against ${OSXLIBSTD}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=${OSXLIBSTD}")
  elseif(DARWIN_VERSION GREATER 12)
    message(STATUS "linking against libc++")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
  endif()

endif()

set(CPACK_PACKAGE_VERSION_MAJOR "0")
set(CPACK_PACKAGE_VERSION_MINOR "9")
set(CPACK_PACKAGE_VERSION_PATCH "1")
set(CPACK_PACKAGE_VERSION "${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# first try finding lua with exact
# depending on cmake version, this may or may not match patch versions
find_package(Lua 5.2 EXACT)
if(LUA_FOUND)
  message ( STATUS "Found lua 5.2")
else()
  find_package(Lua 5.1 EXACT)
  if (LUA_FOUND)
    message ( STATUS "Found lua 5.1")
  else()
    # fallback to lua version without exact
    # this cmake version will force patch versions
    FIND_PACKAGE(Lua 5.2)
    if(LUA_FOUND)
        message ( STATUS "Found lua 5.2")
    else()
        FIND_PACKAGE(Lua 5.1)
        if(LUA_FOUND)
            message ( STATUS "Found lua 5.1")
        else()
            message(FATAL_ERROR "Lua 5.1 or 5.2 was not found.")
        endif()
    endif()
  endif()
endif()

if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
	# We are the top-level project
	include(CTest)
	option(LUABIND_INSTALL "Install the LuaBind library and headers" ON)
	option(LUABIND_BUILD_DOCS "Build documentation files" OFF)
	option(LUABIND_BUILD_SHARED "Build luabind as a shared library?" OFF)
endif()

if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
	# Requiring C++11
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
endif()

if(MSVC)
	# Requiring C++11
#	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /std:c++11")
#	set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /std:c++11")
endif()

set(BUILD_SHARED_LIBS ${LUABIND_BUILD_SHARED})
if(BUILD_SHARED_LIBS)
	add_definitions(-DLUABIND_DYNAMIC_LINK)
endif()

include_directories(
	"${CMAKE_CURRENT_SOURCE_DIR}"
	${LUA_INCLUDE_DIR}
)

add_subdirectory(src)

if(BUILD_TESTING)
	add_subdirectory(test)
endif()

if(LUABIND_BUILD_DOCS)
	add_subdirectory(doc)
endif()

