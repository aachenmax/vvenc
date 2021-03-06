# Top level CMakeLists.txt
#
# minimum required cmake version
cmake_minimum_required( VERSION 3.12.0 FATAL_ERROR )

# set cmake policy
if( NOT CMAKE_VERSION VERSION_LESS 3.12.0 )
  # Use latest policy
  cmake_policy( VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION} )
endif()

# project name
project( vvenc VERSION 0.2.0.0 )

if( NOT CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR )
  # enable sse4.1 build for all source files for gcc and clang
  if( UNIX OR MINGW )
    add_compile_options( "-msse4.1" )
  endif()
  
  # set exception handling
  if( MSVC )
    add_compile_options( "/EHsc" )
  endif()

  # vvenc embedded by superproject, always include source/Lib/vvenc  as first component
  list( PREPEND ${PROJECT_NAME}_ADD_SUBDIRECTORIES "source/Lib/vvenc" )
  list( REMOVE_DUPLICATES ${PROJECT_NAME}_ADD_SUBDIRECTORIES )
  message( STATUS "${CMAKE_CURRENT_SOURCE_DIR}: ${PROJECT_NAME} embedded, subdirectories to be added: ${${PROJECT_NAME}_ADD_SUBDIRECTORIES}" )
  # add subdirectories the superproject asked for
  foreach( subdir IN LISTS ${PROJECT_NAME}_ADD_SUBDIRECTORIES )
    add_subdirectory( ${subdir} )
  endforeach()
  return()
endif()

# enable install target
set( VVENC_ENABLE_INSTALL ON CACHE BOOL "Enable or disable install target" )

# enable postfix
set( VVENC_ENABLE_BUILD_TYPE_POSTFIX OFF CACHE BOOL "Enable or disable build type postfix for apps and libs" )

if( VVENC_ENABLE_BUILD_TYPE_POSTFIX )
  if( BUILD_SHARED_LIBS )
    # set postfixes for shared libraries
    set( CMAKE_RELEASE_POSTFIX        "-s"   CACHE STRING "Set release library postfix" )
    set( CMAKE_DEBUG_POSTFIX          "-ds"  CACHE STRING "Set debug library postfix" )
    set( CMAKE_RELWITHDEBINFO_POSTFIX "-rds" CACHE STRING "Set relwithdebinfo library postfix" )
    set( CMAKE_MINSIZEREL_POSTFIX     "-mrs" CACHE STRING "Set minsizerel library postfix" )
  else()
    # set postfixes for static libraries
    set( CMAKE_RELEASE_POSTFIX        ""     CACHE STRING "Set release library postfix" )
    set( CMAKE_DEBUG_POSTFIX          "-d"   CACHE STRING "Set debug library postfix" )
    set( CMAKE_RELWITHDEBINFO_POSTFIX "-rd"  CACHE STRING "Set relwithdebinfo library postfix" )
    set( CMAKE_MINSIZEREL_POSTFIX     "-mr"  CACHE STRING "Set minsizerel library postfix" )
  endif()
endif()

# set VVENC_OUTPUT_DIR_POSTFIX
if( BUILD_SHARED_LIBS )
  set( VVENC_OUTPUT_DIR_POSTFIX shared )
else()
  set( VVENC_OUTPUT_DIR_POSTFIX static )
endif()

set( VVENC_ENABLE_TRACING OFF CACHE BOOL "Set ENABLE_TRACING=1 as a compiler flag" )

# Using CMake's default library name convention which is the same for all configurations.
set( CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG          "${CMAKE_SOURCE_DIR}/lib/debug-${VVENC_OUTPUT_DIR_POSTFIX}" )
set( CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE        "${CMAKE_SOURCE_DIR}/lib/release-${VVENC_OUTPUT_DIR_POSTFIX}" )
set( CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELWITHDEBINFO "${CMAKE_SOURCE_DIR}/lib/relwithdebinfo-${VVENC_OUTPUT_DIR_POSTFIX}" )
set( CMAKE_ARCHIVE_OUTPUT_DIRECTORY_MINSIZEREL     "${CMAKE_SOURCE_DIR}/lib/minsizerel-${VVENC_OUTPUT_DIR_POSTFIX}" )

set( CMAKE_LIBRARY_OUTPUT_DIRECTORY_DEBUG          "${CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG}" )
set( CMAKE_LIBRARY_OUTPUT_DIRECTORY_RELEASE        "${CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE}" )
set( CMAKE_LIBRARY_OUTPUT_DIRECTORY_RELWITHDEBINFO "${CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELWITHDEBINFO}" )
set( CMAKE_LIBRARY_OUTPUT_DIRECTORY_MINSIZEREL     "${CMAKE_ARCHIVE_OUTPUT_DIRECTORY_MINSIZEREL}" )    

set( CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG          "${CMAKE_SOURCE_DIR}/bin/debug-${VVENC_OUTPUT_DIR_POSTFIX}" )
set( CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE        "${CMAKE_SOURCE_DIR}/bin/release-${VVENC_OUTPUT_DIR_POSTFIX}" )
set( CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO "${CMAKE_SOURCE_DIR}/bin/relwithdebinfo-${VVENC_OUTPUT_DIR_POSTFIX}" )
set( CMAKE_RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL     "${CMAKE_SOURCE_DIR}/bin/minsizerel-${VVENC_OUTPUT_DIR_POSTFIX}" )


# set default CMAKE_BUILD_TYPE to Release if not set
if( NOT CMAKE_BUILD_TYPE )
  set( CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel." FORCE )
endif()

# address sanitizer
if( CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR
    CMAKE_CXX_COMPILER_ID STREQUAL "Clang" )
  set( VVENC_USE_ADDRESS_SANITIZER OFF CACHE BOOL "Compiles with -sanitize=address and links to libasan" )
endif()

#set( ENABLE_LINK_TIME_OPT ON       CACHE BOOL   "Enable link time optimization for release and profile builds" )
#set( OPT_TARGET_ARCH      "native" CACHE STRING "Create code and optimize for this architecture (default native)" )

# set c++11
set( CMAKE_CXX_STANDARD 14 )
set( CMAKE_CXX_STANDARD_REQUIRED ON )

# compile everything position independent (even static libraries)
set( CMAKE_POSITION_INDEPENDENT_CODE TRUE )

# set verbose compile options
#set( CMAKE_VERBOSE_MAKEFILE ON )

# use folders in IDEs for projects (e.g. lib sample app test)
set_property( GLOBAL PROPERTY USE_FOLDERS ON )

set( CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/modules" )
message( STATUS "CMAKE_MODULE_PATH: updating module path to: ${CMAKE_MODULE_PATH}" )

# Enable multithreading
find_package( Threads REQUIRED )

# enable sse4.1 build for all source files for gcc and clang
if( UNIX OR MINGW )
  add_compile_options( "-msse4.1" )
endif()

# set _WIN32_WINNT
if( WIN32 )
  # set _WIN32_WINT version global
  add_definitions( -D_WIN32_WINNT=0x0600 )
endif()

# enable parallel build for Visual Studio
if( MSVC )
  # add compile options
  add_compile_options( "/MP" )
  add_compile_options( "/EHsc" )
endif()

if( VVENC_ENABLE_TRACING )
  add_definitions( -DENABLE_TRACING=1 )
endif()

# set address sanitizer compiler arguments
if( VVENC_USE_ADDRESS_SANITIZER )
   # add compile options
   add_compile_options( "-fsanitize=address" )
   add_compile_options( "-fno-omit-frame-pointer" )
   add_compile_options( "-fsanitize=undefined" )
   add_compile_options( "-fsanitize=leak" )
   add_link_options( "-fsanitize=address" )
   add_link_options( "-fno-omit-frame-pointer" )
   add_link_options( "-fsanitize=undefined" )
   add_link_options( "-fsanitize=leak" )
endif()

# use ccache
find_program( CCACHE_FOUND ccache )
if( CCACHE_FOUND )
  message( STATUS "ccache found. using it." )
  set_property( GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache )
  set_property( GLOBAL PROPERTY RULE_LAUNCH_LINK ccache )
endif()

# handle rpath correctly
if( VVENC_ENABLE_INSTALL )
  if( CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT )
    set( CMAKE_INSTALL_PREFIX "${CMAKE_SOURCE_DIR}/install" CACHE PATH "Standard install prefix" FORCE )
  endif()

  # use GNU install dirs
  include( GNUInstallDirs )
  
  if( BUILD_SHARED_LIBS AND NOT WIN32 )
    set( CMAKE_SKIP_INSTALL_RPATH OFF CACHE BOOL "skip rpath" )
    if( APPLE )
      set( RPATH_BASE @loader_path )
    elseif( UNIX )
      set( RPATH_BASE $ORIGIN )
    endif()
    
    file( RELATIVE_PATH RPATH_REL_DIR ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_INSTALL_BINDIR} ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR} )
    
    set( CMAKE_INSTALL_RPATH ${RPATH_BASE} ${RPATH_BASE}/${RPATH_REL_DIR} )
    message( STATUS "CMAKE_INSTALL_RPATH=${CMAKE_INSTALL_RPATH}" )
  endif()  
endif()


add_subdirectory( "source/Lib/vvenc" )
add_subdirectory( "source/App/vvencapp" )
add_subdirectory( "source/App/vvencFFapp" )
add_subdirectory( "test/vvenclibtest" )

# enable testing with ctest
enable_testing()

# add test
add_test( NAME Test_vvenclibtest-paramerter_range COMMAND vvenclibtest 1 )
add_test( NAME Test_vvenclibtest-calling_order COMMAND vvenclibtest 2 )
add_test( NAME Test_vvenclibtest-input_params COMMAND vvenclibtest 3 )
add_test( NAME Test_vvenclibtest-output_params COMMAND vvenclibtest 4 )

add_test( NAME Test_vvencapp-tooltest COMMAND vvencapp --preset tooltest -s 80x44 -r 15 -i ../../test/data/RTn23_80x44p15_f15.yuv -f 8 -o out.vvc )
set_tests_properties( Test_vvencapp-tooltest PROPERTIES TIMEOUT 90 )
add_test( NAME Test_vvencFFapp-tooltest COMMAND vvencFFapp -c ../../cfg/randomaccess_tooltest.cfg -c ../../test/data/RTn23.cfg -f 8 -b outf.vvc )
set_tests_properties( Test_vvencFFapp-tooltest PROPERTIES TIMEOUT 60 )
add_test( NAME Test_compare_output-tooltest COMMAND ${CMAKE_COMMAND} -E compare_files out.vvc outf.vvc )

add_test( NAME Test_vvencFFapp-tooltest-Scalar COMMAND vvencFFapp -c ../../cfg/randomaccess_tooltest.cfg -c ../../test/data/RTn23.cfg -f 8 -b outf.vvc --SIMD=SCALAR )
set_tests_properties( Test_vvencFFapp-tooltest-Scalar PROPERTIES TIMEOUT 70 )
add_test( NAME Test_compare_output-tooltest-Scalar COMMAND ${CMAKE_COMMAND} -E compare_files out.vvc outf.vvc )

add_test( NAME Test_vvencFFapp-tooltesttrans COMMAND vvencFFapp -c ../../cfg/randomaccess_tooltest.cfg -c ../../test/data/RTn23.cfg -f 8 --DebugBitstream=outf.vvc --DebugPOC=3 -b out.vvc )
set_tests_properties( Test_vvencFFapp-tooltesttrans PROPERTIES TIMEOUT 20 )
add_test( NAME Test_output-tooltesttrans COMMAND ${CMAKE_COMMAND} -E compare_files out.vvc outf.vvc )

add_test( NAME Test_vvencapp-faster COMMAND vvencapp --preset faster -s 80x44 -r 15 -i ../../test/data/RTn23_80x44p15_f15.yuv -f 11 -o out.vvc )
set_tests_properties( Test_vvencapp-faster PROPERTIES TIMEOUT 30 )
add_test( NAME Test_vvencFFapp-faster COMMAND vvencFFapp -c ../../cfg/randomaccess_faster.cfg -c ../../test/data/RTn23.cfg -f 11 -b outf.vvc )
set_tests_properties( Test_vvencFFapp-faster PROPERTIES TIMEOUT 30 )
add_test( NAME Test_compare_output-faster COMMAND ${CMAKE_COMMAND} -E compare_files out.vvc outf.vvc )

add_test( NAME Test_vvencapp-fast COMMAND vvencapp --preset fast -s 80x44 -r 15 -i ../../test/data/RTn23_80x44p15_f15.yuv -f 8 -o out.vvc )
set_tests_properties( Test_vvencapp-fast PROPERTIES TIMEOUT 40 )
add_test( NAME Test_vvencFFapp-fast COMMAND vvencFFapp -c ../../cfg/randomaccess_fast.cfg -c ../../test/data/RTn23.cfg -f 8 -b outf.vvc )
set_tests_properties( Test_vvencFFapp-fast PROPERTIES TIMEOUT 30 )
add_test( NAME Test_compare_output-fast COMMAND ${CMAKE_COMMAND} -E compare_files out.vvc outf.vvc )

add_test( NAME Test_vvencFFapp-transcoding COMMAND vvencFFapp -c ../../cfg/randomaccess_fast.cfg -c ../../test/data/RTn23.cfg -f 8 --DebugBitstream=outf.vvc --DebugPOC=3 -b out.vvc )
set_tests_properties( Test_vvencFFapp-transcoding PROPERTIES TIMEOUT 20 )
add_test( NAME Test_output-transcoding COMMAND ${CMAKE_COMMAND} -E compare_files out.vvc outf.vvc )

add_test( NAME Test_vvencapp-medium COMMAND vvencapp --preset medium -s 80x44 -r 15 -i ../../test/data/RTn23_80x44p15_f15.yuv -f 5 -o out.vvc )
set_tests_properties( Test_vvencapp-medium PROPERTIES TIMEOUT 30 )
add_test( NAME Test_vvencFFapp-medium COMMAND vvencFFapp -c ../../cfg/randomaccess_medium.cfg -c ../../test/data/RTn23.cfg -f 5 -b outf.vvc )
set_tests_properties( Test_vvencFFapp-medium PROPERTIES TIMEOUT 30 )
add_test( NAME Test_compare_output-medium COMMAND ${CMAKE_COMMAND} -E compare_files out.vvc outf.vvc )

add_test( NAME Test_vvencapp-slow COMMAND vvencapp --preset slow -s 80x44 -r 15 -i ../../test/data/RTn23_80x44p15_f15.yuv -f 3 -o out.vvc )
set_tests_properties( Test_vvencapp-slow PROPERTIES TIMEOUT 90 )
add_test( NAME Test_vvencFFapp-slow COMMAND vvencFFapp -c ../../cfg/randomaccess_slow.cfg -c ../../test/data/RTn23.cfg -f 3 -b outf.vvc )
set_tests_properties( Test_vvencFFapp-slow PROPERTIES TIMEOUT 90 )
add_test( NAME Test_compare_output-slow COMMAND ${CMAKE_COMMAND} -E compare_files out.vvc outf.vvc )

add_test( NAME Test_vvencapp-medium_noqpa_0thr COMMAND vvencapp --preset medium --qpa 0 -threads 0 -s 80x44 -r 15 -i ../../test/data/RTn23_80x44p15_f15.yuv -f 5 -o out.vvc )
set_tests_properties( Test_vvencapp-medium_noqpa_0thr PROPERTIES TIMEOUT 30 )
add_test( NAME Test_vvencFFapp-medium_noqpa_0thr COMMAND vvencFFapp -c ../../cfg/randomaccess_medium.cfg -c ../../test/data/RTn23.cfg -qpa 0 --WppBitEqual=0 --NumWppThreads=0 -f 5 -b outf.vvc )
set_tests_properties( Test_vvencFFapp-medium_noqpa_0thr PROPERTIES TIMEOUT 30 )
add_test( NAME Test_compare_output-medium_noqpa_0thr COMMAND ${CMAKE_COMMAND} -E compare_files out.vvc outf.vvc )

add_test( NAME Test_remove_temp_files COMMAND ${CMAKE_COMMAND} -E remove out.vvc tout.vvc rec.yuv  )
 

if( VVENC_ENABLE_INSTALL )
  # include installer
  include( cmake/modules/vvencInstall.cmake )
endif()

