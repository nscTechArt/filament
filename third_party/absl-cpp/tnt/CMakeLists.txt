cmake_minimum_required(VERSION 3.19)
project(filament-absl)

add_subdirectory(absl)

set(TARGET filament-absl)

set(PUBLIC_HDR_DIR ${CMAKE_CURRENT_SOURCE_DIR}/..)

add_library(${TARGET} STATIC)

target_link_libraries(${TARGET} PRIVATE absl::flat_hash_map)

target_include_directories(${TARGET} PUBLIC ${PUBLIC_HDR_DIR})
set_target_properties(${TARGET} PROPERTIES FOLDER ThirdParty)

install(TARGETS ${TARGET} ARCHIVE DESTINATION lib/${DIST_DIR})
