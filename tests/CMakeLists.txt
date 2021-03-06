#------------------------------------------------------------------------------
# Modules
#------------------------------------------------------------------------------
include(CheckCXXCompilerFlag)
include(CheckCXXSymbolExists)
include(CheckIncludeFileCXX)
include(ExternalProject)

#------------------------------------------------------------------------------
# Macros and Functions
#------------------------------------------------------------------------------
function(require_cxx_include_file header)
  if (CAN_USE_STDLIB_LIBCXX AND BUILD_WITH_LIBCXX)
    set(CMAKE_REQUIRED_FLAGS "${CMAKE_REQUIRED_FLAGS} ${USE_STDLIB_LIBCXX}")
  endif ()
  if (CAN_USE_STD_CXX11)
    set(CMAKE_REQUIRED_FLAGS "${CMAKE_REQUIRED_FLAGS} ${USE_STD_CXX11}")
  endif ()
  check_include_file_cxx(${header} variable)
  if (NOT variable)
    message(FATAL_ERROR "Header <${header}> could not be found")
  endif ()
endfunction ()

function(require_cxx_symbol_exists symbol header)
  if (CAN_USE_STDLIB_LIBCXX AND BUILD_WITH_LIBCXX)
    set(CMAKE_REQUIRED_FLAGS "${CMAKE_REQUIRED_FLAGS} ${USE_STDLIB_LIBCXX}")
  endif ()
  if (CAN_USE_STD_CXX11)
    set(CMAKE_REQUIRED_FLAGS "${CMAKE_REQUIRED_FLAGS} ${USE_STD_CXX11}")
  endif ()
  check_cxx_symbol_exists(${symbol} ${header} variable)
  if (NOT variable)
    message(FATAL_ERROR "Symbol '${symbol}' could not be found")
  endif ()
endfunction ()

function(add_unit_test name file)
  set(LIBCXX $<BOOL:${CAN_USE_STDLIB_LIBCXX}>,$<BOOL:${BUILD_WITH_LIBCXX}>)
  add_executable(test-${name} ${file})
  add_test(${name} test-${name})
  add_dependencies(check test-${name})
  add_dependencies(test-${name} catch)
  target_compile_options(test-${name}
    PRIVATE
      $<$<AND:${LIBCXX}>:${USE_STDLIB_LIBCXX}>
      $<$<BOOL:${CAN_USE_STD_CXX11}>:${USE_STD_CXX11}>
      $<$<BOOL:${CAN_USE_PEDANTIC}>:${USE_PEDANTIC}>

      $<$<BOOL:${CAN_WARN_EVERYTHING}>:${WARN_EVERYTHING}>
      $<$<BOOL:${CAN_WARN_NOEXCEPT}>:${WARN_NOEXCEPT}>
      $<$<BOOL:${CAN_WARN_EXTRA}>:${WARN_EXTRA}>
      $<$<BOOL:${CAN_WARN_ALL}>:${WARN_ALL}>

      $<$<BOOL:${CAN_WARN_NO_CXX98_PEDANTIC}>:${WARN_NO_CXX98_PEDANTIC}>
      $<$<BOOL:${CAN_WARN_NO_CXX98_COMPAT}>:${WARN_NO_CXX98_COMPAT}>
      $<$<BOOL:${CAN_WARN_NO_EXIT_TIME}>:${WARN_NO_EXIT_TIME}>)
  target_include_directories(test-${name}
    SYSTEM
    PRIVATE
      ${CATCH_INCLUDE_DIR})
  target_include_directories(test-${name}
    PRIVATE
      $<TARGET_PROPERTY:platform,INTERFACE_INCLUDE_DIRECTORIES>)
endfunction ()

#------------------------------------------------------------------------------
# Compiler Environment Check
#------------------------------------------------------------------------------
set(USE_STDLIB_LIBCXX "-stdlib=libc++")
set(USE_STD_CXX11 "-std=c++11")

set(USE_PEDANTIC "-pedantic")

set(WARN_EVERYTHING "-Weverything")
set(WARN_NOEXCEPT "-Wnoexcept")
set(WARN_ONE_LINE "-WL")
set(WARN_ERROR "-Werror")
set(WARN_EXTRA "-Wextra")
set(WARN_ALL "-Wall")

set(WARN_NO_CXX98_PEDANTIC "-Wno-c++98-compat-pedantic")
set(WARN_NO_CXX98_COMPAT "-Wno-c++98-compat")
set(WARN_NO_EXIT_TIME "-Wno-exit-time-destructors")
if (MSVC)
  set(WARN_EVERYTHING "/Wall")
  set(WARN_ERROR "/Werror")
  set(WARN_ALL "/W4")
endif ()

check_cxx_compiler_flag(${USE_STDLIB_LIBCXX} CAN_USE_STDLIB_LIBCXX)
check_cxx_compiler_flag(${USE_STD_CXX11} CAN_USE_STD_CXX11)

check_cxx_compiler_flag(${USE_PEDANTIC} CAN_USE_PEDANTIC)

check_cxx_compiler_flag(${WARN_EVERYTHING} CAN_WARN_EVERYTHING)
check_cxx_compiler_flag(${WARN_EXTRA} CAN_WARN_EXTRA)
check_cxx_compiler_flag(${WARN_ALL} CAN_WARN_ALL)

check_cxx_compiler_flag(${WARN_NO_CXX98_PEDANTIC} CAN_WARN_NO_CXX98_PEDANTIC)
check_cxx_compiler_flag(${WARN_NO_CXX98_COMPAT} CAN_WARN_NO_CXX98_COMPAT)
check_cxx_compiler_flag(${WARN_NO_EXIT_TIME} CAN_WARN_NO_EXIT_TIME)

require_cxx_include_file(cstdint)

require_cxx_symbol_exists(UINT16_C cstdint)
require_cxx_symbol_exists(UINT32_C cstdint)
require_cxx_symbol_exists(UINT64_C cstdint)
require_cxx_symbol_exists(INT16_C cstdint)
require_cxx_symbol_exists(INT32_C cstdint)
require_cxx_symbol_exists(INT64_C cstdint)

#------------------------------------------------------------------------------
# Configuration
#------------------------------------------------------------------------------
ExternalProject_Add(catch
  PREFIX ${PROJECT_BINARY_DIR}/catch
  GIT_REPOSITORY git://github.com/philsquared/Catch.git
  GIT_TAG d4e5f184369ce34592bb6f89e793bdb22d1d011a
  TIMEOUT 10
  CONFIGURE_COMMAND ""
  INSTALL_COMMAND ""
  UPDATE_COMMAND ""
  BUILD_COMMAND "")

ExternalProject_Get_Property(catch source_dir)
set(CATCH_INCLUDE_DIR "${source_dir}/include" CACHE INTERNAL "Catch Framework")
add_custom_target(check COMMAND ${CMAKE_TEST_COMMAND})

add_unit_test(intrinsic "${TEST_SOURCE_DIR}/intrinsic.cpp")
add_unit_test(traits "${TEST_SOURCE_DIR}/traits.cpp")
