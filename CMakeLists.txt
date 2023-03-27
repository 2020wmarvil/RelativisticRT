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

# collect shaders
set(SHADER_DIR_GLSL "Data/shaders/glsl/computeraytracing")
file(GLOB SHADERS_GLSL "${SHADER_DIR_GLSL}/*.vert" "${SHADER_DIR_GLSL}/*.frag" "${SHADER_DIR_GLSL}/*.comp" "${SHADER_DIR_GLSL}/*.geom" "${SHADER_DIR_GLSL}/*.tesc" "${SHADER_DIR_GLSL}/*.tese" "${SHADER_DIR_GLSL}/*.mesh" "${SHADER_DIR_GLSL}/*.task" "${SHADER_DIR_GLSL}/*.rgen" "${SHADER_DIR_GLSL}/*.rchit" "${SHADER_DIR_GLSL}/*.rmiss" "${SHADER_DIR_GLSL}/*.rcall")
source_group("Shaders\\GLSL" FILES ${SHADERS_GLSL})

# create executable
add_executable(${PROJECT_NAME} WIN32 ${RRT_SOURCES} ${RRT_HEADERS} ${SHADERS_GLSL})
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
