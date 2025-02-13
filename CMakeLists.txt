########################################################
#  
#  This is a CMake configuration file.
#  To use it you need CMake which can be 
#  downloaded from here: 
#    http://www.cmake.org/cmake/resources/software.html
#
#########################################################

cmake_minimum_required( VERSION 3.0 ) 

project( Sigil )

# Need CMAKE 3.18+ to make full use of new FindPython3
# with the Development.Embed submodule.
set( RECENT_CMAKE_VER "3.18.1" )
if ( NOT DEFINED TRY_NEWER_FINDPYTHON3 )
    set ( TRY_NEWER_FINDPYTHON3 0 )
endif()

if (${CMAKE_VERSION} VERSION_GREATER_EQUAL ${RECENT_CMAKE_VER} AND ${TRY_NEWER_FINDPYTHON3})
    set ( USE_NEWER_FINDPYTHON3 1 )
else()
    set ( USE_NEWER_FINDPYTHON3 0 )
endif()

set( CMAKE_DEBUG_POSTFIX "d" )
# Avoid the whole Release/Debug folder creation with Visual Studio
if (MSVC_IDE)
   set( CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG "${PROJECT_BINARY_DIR}/bin")
   set( CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE "${PROJECT_BINARY_DIR}/bin")
   set( CMAKE_LIBRARY_OUTPUT_DIRECTORY_DEBUG "${PROJECT_BINARY_DIR}/lib")
   set( CMAKE_LIBRARY_OUTPUT_DIRECTORY_RELEASE "${PROJECT_BINARY_DIR}/lib")
   set( CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG "${PROJECT_BINARY_DIR}/lib")
   set( CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE "${PROJECT_BINARY_DIR}/lib")
endif()
set( CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin )
set( CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib )
set( CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib )
set( PACKAGE_DIRECTORY ${PROJECT_BINARY_DIR}/package )
set( CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake_extras" )

set( SIGIL_MAJOR_VERSION 1 )
set( SIGIL_MINOR_VERSION 7 )
set( SIGIL_REVISION_VERSION 0 )
set( SIGIL_FULL_VERSION ${SIGIL_MAJOR_VERSION}.${SIGIL_MINOR_VERSION}.${SIGIL_REVISION_VERSION} )

if( UNIX AND NOT APPLE )
   include(GNUInstallDirs)

   # Make sure the correct rpath is set for the sigil executable
   # on Linux -- for both the build and the installed versions.
   # Needed to make sure the installed shared hunspell and gumbo
   # libraries are found/used at runtime.

   # use, i.e. don't skip the full RPATH for the build tree
   SET(CMAKE_SKIP_BUILD_RPATH  FALSE)

   # when building, don't use the install RPATH already
   # (but later on when installing)
   SET(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)

   SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_FULL_LIBDIR}/sigil")

   # add the automatically determined parts of the RPATH
   # which point to directories outside the build tree to the install RPATH.
   # This will ensure the Qt5 lib directory is always included after install.
   SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
endif()

# Check if platform is 64 bit
if( NOT APPLE )
   if( CMAKE_SIZEOF_VOID_P EQUAL 4 )
     set( 64_BIT_PLATFORM 0 )
   else()
     set( 64_BIT_PLATFORM 1 )
   endif()
endif()

# Profiler configuration for GCC
if( CMAKE_COMPILER_IS_GNUCXX  )
    set( CMAKE_BUILD_TYPE profiling )
    set( CMAKE_CXX_FLAGS_PROFILING "-O2 -DNDEBUG -pg -g -fno-omit-frame-pointer -fno-inline-functions -fno-inline-functions-called-once -fno-optimize-sibling-calls" )

    # After creating the new build type,
    # we clear the default back to empty
    set( CMAKE_BUILD_TYPE  )
endif()

if (USE_SYSTEM_LIBS)
    MESSAGE(STATUS "Trying to use System Libraries...")
    find_package( ZLIB )
    find_package( PkgConfig )
    find_package( MiniZip )
    pkg_check_modules( HUNSPELL hunspell )
    if ( NOT APPLE )
        pkg_check_modules( PCRE libpcre16 )
    endif()
endif()

message(STATUS "CMake version in use: ${CMAKE_VERSION}")

if (${USE_NEWER_FINDPYTHON3})
    message(STATUS "Using newer findpython3 cmake module")
    if ( APPLE )
        find_package(Python3 3.8 COMPONENTS Interpreter Development)
    endif()
    if ( WIN32 )
        find_package(Python3 3.5 COMPONENTS Interpreter Development)
    endif()
    if( UNIX AND NOT APPLE )
        find_package(Python3 3.4 COMPONENTS Interpreter Development)
    endif()
else()
    message(STATUS "Using older findpython cmake module")
    if ( APPLE )
        find_package(PythonInterp 3.8)
        find_package (PythonLibs 3.8)
    endif()
    if ( WIN32 )
        find_package(PythonInterp 3.5)
        find_package (PythonLibs 3.5)
    endif()
    if( UNIX AND NOT APPLE )
        find_package(PythonInterp 3.4)
        find_package (PythonLibs 3.4)
    endif()
endif()

# gumbo-parser it is our main xhtml/html5 parser.
# We have an internal version because it diverges from Google's and GitHub's
# versions and neither want's our epub specific changes.
add_subdirectory( internal/gumbo )

add_subdirectory( 3rdparty/ )
add_subdirectory( src/ )
