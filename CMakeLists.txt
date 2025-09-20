cmake_minimum_required(VERSION 3.14...3.22)

# ---- Project ----

# Note: update this to your new project's name and version
project(Frank VERSION 1.0 LANGUAGES CXX)

# ---- Include guards ----

if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
  message(
    FATAL_ERROR
      "In-source builds not allowed. Please make a new directory (called a build directory) and run CMake from there."
  )
endif()

# ---- Add dependencies via CPM ----
# see https://github.com/TheLartians/CPM.cmake for more info

include(cmake/CPM.cmake)

# PackageProject.cmake will be used to make our target installable
CPMAddPackage("gh:TheLartians/PackageProject.cmake@1.8.0")

# GLM
CPMAddPackage(
  NAME glm
  GITHUB_REPOSITORY g-truc/glm
  GIT_TAG 1.0.1
  DOWNLOAD_ONLY TRUE
)

# Assimp
CPMAddPackage(
  NAME assimp
  GITHUB_REPOSITORY assimp/assimp
  GIT_TAG v5.4.3
)

# BGFX
CPMAddPackage(
  NAME bgfx
  GITHUB_REPOSITORY bkaradzic/bgfx.cmake
  GIT_TAG v1.129.8930-495
  OPTIONS
    "BGFX_BUILD_TOOLS ON"
    "BGFX_BUILD_EXAMPLES OFF"
    "BGFX_CUSTOM_TARGETS OFF"
)

# Add the CMake package GLFW
CPMAddPackage(
  NAME GLFW
  GITHUB_REPOSITORY glfw/glfw
  GIT_TAG 3.4
  OPTIONS
    "GLFW_BUILD_TESTS OFF"
    "GLFW_BUILD_EXAMPLES OFF"
    "GLFW_BUILD_DOCS OFF"
)

# ImGui
CPMAddPackage(
  NAME imgui
  VERSION 1.91.1
  GITHUB_REPOSITORY ocornut/imgui
  DOWNLOAD_ONLY TRUE
)

# Include the imgui sources if needed
if(imgui_ADDED)
  # Ensure that backends are included, adjust as per your backend
  file(GLOB IMGUI_SOURCES ${imgui_SOURCE_DIR}/*.cpp
       ${imgui_SOURCE_DIR}/backends/imgui_impl_glfw.cpp # For GLFW
       ${imgui_SOURCE_DIR}/backends/imgui_impl_opengl3.cpp # For OpenGL
  )

  add_library(imgui INTERFACE ${IMGUI_SOURCES})
  target_include_directories(imgui INTERFACE ${imgui_SOURCE_DIR} ${imgui_SOURCE_DIR}/backends)

  set_target_properties(imgui PROPERTIES POSITION_INDEPENDENT_CODE ON)
  # target_link_libraries(imgui PRIVATE glfw)
endif()

# ---- Add source files ----

# Note: globbing sources is considered bad practice as CMake's generators may not detect new files
# automatically. Keep that in mind when changing files, or explicitly mention them here.
file(GLOB_RECURSE headers CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/include/*.hpp")
file(GLOB_RECURSE sources CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")

# ---- Create library ----

# Note: for header-only libraries change all PUBLIC flags to INTERFACE and create an interface
# target: add_library(${PROJECT_NAME} INTERFACE)
add_library(Frank ${headers} ${sources})
set_target_properties(Frank PROPERTIES CXX_STANDARD 20)

# being a cross-platform target, we enforce standards conformance on MSVC
target_compile_options(Frank PUBLIC "$<$<COMPILE_LANG_AND_ID:CXX,MSVC>:/permissive->")

# Link dependencies
target_link_libraries(Frank PRIVATE assimp bgfx glfw imgui)

target_include_directories(Frank PUBLIC
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
  $<BUILD_INTERFACE:${glm_SOURCE_DIR}>
  # $<BUILD_INTERFACE:${imgui_SOURCE_DIR}/src>
  $<INSTALL_INTERFACE:include/${PROJECT_NAME}-${PROJECT_VERSION}>
)

# ---- Create an installable target ----
# this allows users to install and find the library via `find_package()`.

# the location where the project's version header will be placed should match the project's regular
# header paths
string(TOLOWER ${PROJECT_NAME}/version.h VERSION_HEADER_LOCATION)

packageProject(
  NAME ${PROJECT_NAME}
  VERSION ${PROJECT_VERSION}
  NAMESPACE ${PROJECT_NAME}
  BINARY_DIR ${PROJECT_BINARY_DIR}
  INCLUDE_DIR ${PROJECT_SOURCE_DIR}/include
  INCLUDE_DESTINATION include/${PROJECT_NAME}-${PROJECT_VERSION}
  VERSION_HEADER "${VERSION_HEADER_LOCATION}"
  COMPATIBILITY SameMajorVersion
  DEPENDENCIES
    "assimp 5.4.3; bgfx 1.129.8930-495; glfw 3.4; glm 1.0.1; imgui 1.91.1"
)
