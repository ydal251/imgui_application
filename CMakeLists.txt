cmake_minimum_required(VERSION 3.22.1)

project(imgui_application)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED on)
set(CMAKE_CXX_EXTENSIONS off)

message(STATUS "[Lib ImGui App] Build type: ${CMAKE_BUILD_TYPE}")

# ----------------------------------------------------------

set(IMGUI_DIR "${CMAKE_CURRENT_SOURCE_DIR}/third_party/imgui-1.89.7/")

set(LIB_INCLUDE_DIRECTORIES

    # ImGui
    ${IMGUI_DIR}/

    # ImGui::cpp includes
    ${IMGUI_DIR}/misc/cpp/

    # ImGui_Application includes
    ${CMAKE_CURRENT_SOURCE_DIR}/include/
)

set(LIB_INCLUDE_PRIVATE_DIRECTORIES

    # ImGui backends includes
    ${IMGUI_DIR}/backends/
)

#[[
    Exported IMGUI_APPLICATION_SHELL_MINIMAL may be used as
    path for that shell:

    if(EMSCRIPTEN)
        target_link_options(${CMAKE_PROJECT_NAME} PRIVATE
            -sWASM=1

            # ...

            --shell-file ${IMGUI_APPLICATION_SHELL_MINIMAL}
        )
    endif()
]]
set(IMGUI_APPLICATION_SHELL_MINIMAL
    ${CMAKE_CURRENT_SOURCE_DIR}/web_shell/shell_minimal.html

    CACHE INTERNAL ""
)

# ----------------------------------------------------------


set(LIB_HEADERS

    # ImGui headers
    ${IMGUI_DIR}/imconfig.h
    ${IMGUI_DIR}/imgui.h
    ${IMGUI_DIR}/imgui_internal.h

    ${IMGUI_DIR}/imstb_rectpack.h
    ${IMGUI_DIR}/imstb_textedit.h
    ${IMGUI_DIR}/imstb_truetype.h

    ${IMGUI_DIR}/misc/cpp/imgui_stdlib.h

    # ImGui backends headers
    ${IMGUI_DIR}/backends/imgui_impl_opengl3_loader.h
    ${IMGUI_DIR}/backends/imgui_impl_opengl3.h
    ${IMGUI_DIR}/backends/imgui_impl_sdl2.h

    # ImGui_Application headers
    ${CMAKE_CURRENT_SOURCE_DIR}/include/imgui_app/ImGui_Application.hpp
)

set(LIB_SOURCES

    # ImGui sources
    ${IMGUI_DIR}/imgui.cpp
    ${IMGUI_DIR}/imgui_draw.cpp
    ${IMGUI_DIR}/imgui_widgets.cpp
    ${IMGUI_DIR}/imgui_tables.cpp

    ${IMGUI_DIR}/imgui_demo.cpp

    ${IMGUI_DIR}/misc/cpp/imgui_stdlib.cpp

    # ImGui backends sources
    ${IMGUI_DIR}/backends/imgui_impl_opengl3.cpp
    ${IMGUI_DIR}/backends/imgui_impl_sdl2.cpp

    # ImGui_Application sources
    ${CMAKE_CURRENT_SOURCE_DIR}/sources/ImGui_Application.cpp
)

add_library(${PROJECT_NAME} STATIC
    ${LIB_HEADERS} ${LIB_SOURCES}
)

target_include_directories(${PROJECT_NAME} PUBLIC
    ${LIB_INCLUDE_DIRECTORIES}
)

target_include_directories(${PROJECT_NAME} PRIVATE
    ${LIB_INCLUDE_PRIVATE_DIRECTORIES}
)

target_compile_options(${PROJECT_NAME} PRIVATE
    -Wall -Wformat -Wextra -Wpedantic
)

if(EMSCRIPTEN)

    message(STATUS "[Lib ImGui App] Platform: EMSCRIPTEN")

    target_compile_definitions(${PROJECT_NAME} PRIVATE
        IMGUI_DISABLE_FILE_FUNCTIONS
    )

    target_compile_options(${PROJECT_NAME} PRIVATE
        -s USE_SDL=2
    )

else()

    message(STATUS "[Lib ImGui App] Platform: Native")

    find_package(SDL2 REQUIRED)

    target_include_directories(${PROJECT_NAME} PRIVATE
        ${SDL2_INCLUDE_DIRS}
    )

    # -ldl needed, since in 'imgui_impl_opengl3_loader.h' used `dlopen()` and `dlclose()`
    target_link_libraries(${PROJECT_NAME}
        ${SDL2_LIBRARIES} GL dl
    )

endif()
