cmake_minimum_required(VERSION 3.14...3.22)

project(Viewer LANGUAGES CXX)

# --- Import tools ----

include(../cmake/tools.cmake)

# ---- Dependencies ----

include(../cmake/CPM.cmake)

CPMAddPackage(NAME Frank SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/..)

# ---- Create standalone executable ----

file(GLOB sources CONFIGURE_DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/source/*.cpp)

add_executable(Viewer ${sources})

set_target_properties(Viewer PROPERTIES CXX_STANDARD 20 OUTPUT_NAME "Viewer")

target_link_libraries(Viewer Frank::Frank)
