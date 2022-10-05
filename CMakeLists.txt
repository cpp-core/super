cmake_minimum_required (VERSION 3.24 FATAL_ERROR)
project (super VERSION 0.1 LANGUAGES C CXX)

# Make including this file idempotent
#
include_guard()

# Set the root project directories.
#
set(CPP_CORE_SOURCE_DIR ${PROJECT_SOURCE_DIR})
set(CPP_CORE_BINARY_DIR ${PROJECT_BINARY_DIR})

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
set(CPP_CORE_COMPONENT_LIST "pp;mp;tuple")
option(CPP_CORE_COMPONENTS "Build these components (and dependencies as needed)" "pp")

foreach(ITEM ${CPP_CORE_COMPONENTS})
  if (NOT (${ITEM} IN_LIST CPP_CORE_COMPONENT_LIST))
    message(FATAL_ERROR "Unknown component '${ITEM}' specified")
  endif()
endforeach()

# Output configuration
#
message("-- super: Install prefix: ${CMAKE_INSTALL_PREFIX}")

foreach(ITEM ${CPP_CORE_COMPONENTS})
  message("-- super: Building ${ITEM}")
endforeach()

# Add the components
#
enable_testing()

foreach(ITEM ${CPP_CORE_COMPONENTS})
  cmake_language(CALL add_${ITEM})
endforeach()
