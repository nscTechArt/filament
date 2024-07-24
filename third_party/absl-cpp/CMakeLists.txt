cmake_minimum_required(VERSION 3.19)
project(filament-absl)

find_package(Threads REQUIRED)

option(ABSL_USE_SYSTEM_INCLUDES
  "Silence warnings in Abseil headers by marking them as SYSTEM includes"
  ON)

# This needs to set properly for AbseilHelpers
list(APPEND CMAKE_MODULE_PATH
  ${CMAKE_CURRENT_LIST_DIR}/CMake
  ${CMAKE_CURRENT_LIST_DIR}/absl/copts
)

# This needs to set properly for AbseilHelpers
list(APPEND ABSL_COMMON_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR})

include(AbseilHelpers)
#include(AbseilDll)

add_subdirectory(absl)

set(TARGET filament-absl)

list(TRANSFORM ABSL_INTERNAL_DLL_FILES PREPEND "absl/")

add_library(
  ${TARGET}
  STATIC
  ${ABSL_INTERNAL_DLL_FILES}
)

#target_link_libraries(
#  ${TARGET}  
#  PRIVATE
#    ${_dll_libs}
#      ${ABSL_DEFAULT_LINKOPTS}
#  )

set_property(TARGET ${TARGET} PROPERTY LINKER_LANGUAGE "CXX")

target_compile_options(
  ${TARGET}
  PRIVATE
  ${ABSL_DEFAULT_COPTS}
)

# This is a workaround to combine our absl deps into a single static
# lib binary. To create a static lib, we need at least one source file.
# We use an empty file as source, and we export the absl signatures in
# the lib (i.e. PUBLIC in the target_link_libraries).

#add_library(${TARGET} STATIC ${CMAKE_CURRENT_SOURCE_DIR}/tnt/dummy.cpp)
#target_compile_options(${TARGET} PUBLIC -fPIC)
# Here we indicate the parts of absl we would like to use.
#target_link_libraries(${TARGET} PUBLIC absl::flat_hash_map)
#target_link_libraries(${TARGET} PRIVATE
#    absl::raw_logging_internal
#    absl::flat_hash_map)

target_include_directories(${TARGET} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/)

set_target_properties(${TARGET} PROPERTIES FOLDER ThirdParty)

install(TARGETS ${TARGET} ARCHIVE DESTINATION lib/${DIST_DIR})
