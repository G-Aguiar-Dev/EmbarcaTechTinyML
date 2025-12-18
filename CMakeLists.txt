# Generated Cmake Pico project file

cmake_minimum_required(VERSION 3.13)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Initialise pico_sdk from installed location
# (note this can come from environment, CMake cache etc)

# == DO NOT EDIT THE FOLLOWING LINES for the Raspberry Pi Pico VS Code Extension to work ==
if(WIN32)
    set(USERHOME $ENV{USERPROFILE})
else()
    set(USERHOME $ENV{HOME})
endif()
set(sdkVersion 2.2.0)
set(toolchainVersion 14_2_Rel1)
set(picotoolVersion 2.2.0-a4)
set(picoVscode ${USERHOME}/.pico-sdk/cmake/pico-vscode.cmake)
if (EXISTS ${picoVscode})
    include(${picoVscode})
endif()

# Force picotool to be fetched/built if SDK version mismatch detected.
# This fixes errors like: "Incompatible picotool installation found: Requires version X, you have Y"
set(PICOTOOL_FORCE_FETCH_FROM_GIT ON CACHE BOOL "Force fetching/building picotool from git when incompatible version is installed" FORCE)
# ====================================================================================
set(PICO_BOARD pico_w CACHE STRING "Board type")

# Pull in Raspberry Pi Pico SDK (must be before project)
include(pico_sdk_import.cmake)

project(tiny_ml C CXX ASM)

# Initialise the Raspberry Pi Pico SDK
pico_sdk_init()

# Define onboard LED pin for any code that expects PICO_DEFAULT_LED_PIN (quick workaround)
add_compile_definitions(PICO_DEFAULT_LED_PIN=25)

# Prefer a fast workaround: do not build pico-tflmicro examples (they pull in board-specific LED code)
# This avoids errors about PICO_DEFAULT_LED_PIN and significantly reduces build time.
set(PICO_TFLMICRO_BUILD_EXAMPLES OFF CACHE BOOL "Disable building pico-tflmicro examples" FORCE)

# ----------------------------------------------------------------------
# Usa o port oficial do TensorFlow Lite Micro para o Pico:
#    lib/pico-tflmicro
# ----------------------------------------------------------------------
if(EXISTS "${CMAKE_CURRENT_LIST_DIR}/lib/pico-tflmicro/CMakeLists.txt")
    add_subdirectory(lib/pico-tflmicro EXCLUDE_FROM_ALL)
else()
    include(FetchContent)
    message(STATUS "lib/pico-tflmicro not found — fetching pico-tflmicro from GitHub")
    FetchContent_Declare(
        pico_tflmicro
        GIT_REPOSITORY https://github.com/raspberrypi/pico-tflmicro.git
        GIT_TAG main
    )
    # Populate source so we can control how the subdirectory is added (disable examples)
    FetchContent_Populate(pico_tflmicro)
    # Ensure examples are disabled in the pico-tflmicro build (fast)
    set(PICO_TFLMICRO_BUILD_EXAMPLES OFF CACHE BOOL "Disable building pico-tflmicro examples" FORCE)
    add_subdirectory(${pico_tflmicro_SOURCE_DIR} ${pico_tflmicro_BINARY_DIR} EXCLUDE_FROM_ALL)
endif()

# ❌ NÃO usar mais o repositório tflite-micro puro (não tem CMakeLists)
# add_subdirectory(tflite-micro)

# Add executable. Default name is the project name, version 0.1
# (ajuste o nome dos fontes se estiverem em outra pasta, ex: src/tiny_ml.c)
add_executable(tiny_ml
    tiny_ml.c
    tflm_wrapper.cpp
    tflm_wrapper.h
    wine_mlp_float.h
)

pico_set_program_name(tiny_ml "tiny_ml")
pico_set_program_version(tiny_ml "0.1")

# Modify the below lines to enable/disable output over UART/USB
pico_enable_stdio_uart(tiny_ml 0)
pico_enable_stdio_usb(tiny_ml 1)

# ----------------------------------------------------------------------
# Bibliotecas de link:
#   - pico_stdlib   : stdio, etc.
#   - pico-tflmicro : TensorFlow Lite Micro portado p/ Pico
#   - hardware_sync : usado nos exemplos com pico-tflmicro
# ----------------------------------------------------------------------
target_link_libraries(tiny_ml
    pico_stdlib
    hardware_sync
    pico-tflmicro
)

# Add the standard include files to the build
target_include_directories(tiny_ml PRIVATE
    ${CMAKE_CURRENT_LIST_DIR}
)

pico_add_extra_outputs(tiny_ml)
