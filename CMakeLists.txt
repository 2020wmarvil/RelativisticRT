cmake_minimum_required(VERSION 3.20)

project(RelativisticRT)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_SUPPRESS_REGENERATION true)
set(CMAKE_SKIP_INSTALL_RULES true)
set(CMAKE_SKIP_INSTALL_ALL_DEPENDENCY true)
set(CMAKE_SKIP_PACKAGE_ALL_DEPENDENCY true)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${PROJECT_NAME})

add_subdirectory(SaschaWillems)

# glob source files
file (GLOB_RECURSE RRT_SOURCES CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/Source/*.cpp")
file (GLOB_RECURSE RRT_HEADERS CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/Source/*.h")

set (RRT_INCLUDE_DIRS "")
foreach (_headerFile ${APPLICATION_HEADERS})
	get_filename_component(_dir ${_headerFile} PATH)
	list (APPEND RRT_INCLUDE_DIRS ${_dir})
endforeach()
list (REMOVE_DUPLICATES RRT_INCLUDE_DIRS)

# create executable
add_executable(${PROJECT_NAME} WIN32 ${RRT_SOURCES} ${RRT_HEADERS})
target_link_libraries(${PROJECT_NAME} SaschaWillemsBase ${Vulkan_LIBRARY} ${WINLIBS})

# set compiler flags
if(MSVC)
 	target_compile_options(${PROJECT_NAME} PRIVATE /W4)
else()
 	target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra -Wpedantic)
endif()

# list include directories
target_include_directories(${PROJECT_NAME} PRIVATE 
	"${RRT_INCLUDE_DIRS}"
    "SaschaWillems/external"
    "SaschaWillems/external/glm"
    "SaschaWillems/external/gli"
    "SaschaWillems/external/imgui"
    "SaschaWillems/external/tinygltf"
    "SaschaWillems/external/ktx/include"
    "SaschaWillems/external/ktx/other_include"
    "SaschaWillems/base"
)

# create a shader target
find_program(GLSL_VALIDATOR glslangValidator HINTS /usr/bin /usr/local/bin $ENV{VULKAN_SDK}/Bin/ $ENV{VULKAN_SDK}/Bin32/)

set(SHADER_PATH "${PROJECT_SOURCE_DIR}/Data/shaders/glsl/computeraytracing")

file (GLOB_RECURSE SHADERS CONFIGURE_DEPENDS 
    "${SHADER_PATH}/*.vert" 
    "${SHADER_PATH}/*.frag"
    "${SHADER_PATH}/*.comp"
    )

# custom command to compile shaders
foreach(GLSL ${SHADERS})
  message(STATUS "BUILDING SHADER")
  get_filename_component(FILE_NAME ${GLSL} NAME)
  set(SPIRV "${SHADER_PATH}/${FILE_NAME}.spv")
  message(STATUS ${GLSL})
  add_custom_command(
    OUTPUT ${SPIRV}
    COMMAND ${GLSL_VALIDATOR} -V ${GLSL} -o ${SPIRV}
    DEPENDS ${GLSL})

  list(APPEND SPIRV_BINARY_FILES ${SPIRV})
endforeach(GLSL)

# create a shader target
add_custom_target(
    Shaders
    SOURCES ${SHADERS}
    DEPENDS ${SPIRV_BINARY_FILES}
    )

add_dependencies(${PROJECT_NAME} Shaders)