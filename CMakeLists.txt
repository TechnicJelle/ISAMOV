cmake_minimum_required(VERSION 3.28)

# set the output directory for built objects.
# This makes sure that the dynamic library goes into the build directory automatically.
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/$<CONFIGURATION>")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/$<CONFIGURATION>")

# prevent installing to system directories.
set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}" CACHE INTERNAL "")

# Declare the project
project(ISAMIV_old)

# Set the name of the executable
set(EXECUTABLE_NAME ${PROJECT_NAME})

add_executable(${EXECUTABLE_NAME})

find_package(OpenGL REQUIRED)
target_link_libraries(${EXECUTABLE_NAME} PRIVATE ${OPENGL_LIBRARIES})

# Glad
set(GLAD_DIR ${CMAKE_CURRENT_SOURCE_DIR}/deps/glad)
include_directories(${GLAD_DIR}/include)

set(GLAD_SOURCES
		${GLAD_DIR}/src/gl.c
)

# Dear ImGui
set(IMGUI_DIR ${CMAKE_CURRENT_SOURCE_DIR}/deps/imgui)
include_directories(${IMGUI_DIR})
include_directories(${IMGUI_DIR}/backends)

set(IMGUI_SOURCES
		${IMGUI_DIR}/imgui.cpp
		${IMGUI_DIR}/imgui_demo.cpp
		${IMGUI_DIR}/imgui_draw.cpp
		${IMGUI_DIR}/imgui_tables.cpp
		${IMGUI_DIR}/imgui_widgets.cpp
		# Specific implementations:
		${IMGUI_DIR}/backends/imgui_impl_sdl3.cpp
		${IMGUI_DIR}/backends/imgui_impl_opengl3.cpp
)

# Tex Inspect
set(TEXINSPECT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/deps/imgui_tex_inspect)
include_directories(${TEXINSPECT_DIR})
include_directories(${TEXINSPECT_DIR}/backends)
add_compile_definitions(IMGUI_IMPL_OPENGL_LOADER_GLAD2)

set(TEXINSPECT_SOURCES
		${TEXINSPECT_DIR}/imgui_tex_inspect.cpp
		# Specific implementations:
		${TEXINSPECT_DIR}/backends/tex_inspect_opengl.cpp
)

# Add your sources to the target
target_sources(${EXECUTABLE_NAME}
		PRIVATE
		src/main.cpp
		${GLAD_SOURCES}
		${IMGUI_SOURCES}
		${TEXINSPECT_SOURCES}
)

# Set C++ version
target_compile_features(${EXECUTABLE_NAME} PUBLIC cxx_std_20)
set_target_properties(${EXECUTABLE_NAME} PROPERTIES CXX_EXTENSIONS OFF)

# Configure SDL by calling its CMake file.
# we use EXCLUDE_FROM_ALL so that its install targets and configs don't
# pollute upwards into our configuration.
add_subdirectory(deps/SDL EXCLUDE_FROM_ALL)

# Link SDL to our executable. This also makes its include directory available to us.
target_link_libraries(${EXECUTABLE_NAME} PUBLIC SDL3::SDL3)

# make sure that the executable can find the SDL3 library file(s)
install(TARGETS ${EXECUTABLE_NAME} BUNDLE DESTINATION ./install)

