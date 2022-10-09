cmake_minimum_required (VERSION 3.24 FATAL_ERROR)
project (super VERSION 0.1 LANGUAGES C CXX)

# Make including this file idempotent
#
include_guard()

# CMP0077 - New behavior allows overriding option with normal variable.
# 
set(CMAKE_POLICY_DEFAULT_CMP0077 NEW)

# Set the project directories.
#
set(CPP_CORE_SOURCE_DIR ${PROJECT_SOURCE_DIR})
set(CPP_CORE_BINARY_DIR ${PROJECT_BINARY_DIR})
set(CPP_CORE_SOURCE_COMPONENT_DIR ${CPP_CORE_SOURCE_DIR}/components)
set(CPP_CORE_BINARY_COMPONENT_DIR ${CPP_CORE_BINARY_DIR}/components)

# Include the cmake helpers
#
include(components/cmake/utils/all.cmake)
include(components/cmake/recipes/all.cmake)

# compile_commands.json
#
symlink_compile_commands()

# Convenience targets for git operations
#
make_submodule_pull_target()

# Component options
#
set(CPP_CORE_INTERNAL_COMPONENT_LIST
  cc
  chrono
  codec
  fp
  mp
  ply
  pp
  strand
  stream
  string
  tuple
  util
  )

set(CPP_CORE_EXTERNAL_COMPONENT_LIST
  bzip2
  cppzmq
  cpr
  date
  fmt
  gtest
  inja
  ixwebsocket
  json
  libzmq
  spdlog
  xtensor
  zstd)

foreach(ITEM ${CPP_CORE_INTERNAL_COMPONENT_LIST})
  list(APPEND CPP_CORE_COMPONENT_LIST ${ITEM})
endforeach()

foreach(ITEM ${CPP_CORE_EXTERNAL_COMPONENT_LIST})
  list(APPEND CPP_CORE_COMPONENT_LIST ${ITEM})
endforeach()

option(CPP_CORE_COMPONENTS "Build these components (and dependencies as needed)" "pp")
option(CPP_CORE_COMPONENTS_ALL "Build all components" OFF)
option(CPP_CORE_TESTS "Test these components")
option(CPP_CORE_TEST_ALL "Test all components" OFF)

if (${CPP_CORE_COMPONENTS_ALL})
  set(CPP_CORE_COMPONENTS ${CPP_CORE_COMPONENT_LIST})
endif()

foreach(ITEM ${CPP_CORE_COMPONENTS})
  if (NOT (${ITEM} IN_LIST CPP_CORE_COMPONENT_LIST))
    message(FATAL_ERROR "Unknown component '${ITEM}' specified")
  endif()
endforeach()

if (${CPP_CORE_TEST_ALL})
  foreach(ITEM ${CPP_CORE_COMPONENTS})
    if(${ITEM} IN_LIST CPP_CORE_INTERNAL_COMPONENT_LIST)
      string(TOUPPER ${ITEM} item)
      set(${item}_TEST ON)
    endif()
  endforeach()
else()
  foreach(ITEM ${CPP_CORE_TESTS})
    string(TOUPPER ${ITEM} item)
    set(${item}_TEST ON)
  endforeach()
endif()

# Output configuration
#
message("-- super: Install prefix: ${CMAKE_INSTALL_PREFIX}")

foreach(ITEM ${CPP_CORE_COMPONENTS})
  string(TOUPPER ${ITEM} item)
  message("-- super: Building ${ITEM}, Testing: ${${item}_TEST}")
endforeach()

# Add the components
#
enable_testing()

foreach(ITEM ${CPP_CORE_COMPONENTS})
  message("-- super: Adding ${ITEM}")
  cmake_language(CALL add_${ITEM})
  message("-- super: ${ITEM} Build tests: ${BUILD_TESTS}")
endforeach()
