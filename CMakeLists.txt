cmake_minimum_required(VERSION 3.12)

set(FREERTOS_KERNEL_PATH ~/git/FreeRTOS-Kernel) # TODO: update?
set(PICO_SDK_PATH ~/git/pico-sdk)
set(PICO_BOARD pico2)
set(PICO_PLATFORM rp2350-arm-s)

# Pull in SDK (must be before project)
include(pico_sdk_import.cmake)

project(pico_freertos C CXX ASM)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

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
# ====================================================================================

# Initialize the SDK
pico_sdk_init()

add_compile_options(-Wall
        -Wno-format          # int != int32_t as far as the compiler is concerned because gcc has int32_t as long int
        -Wno-unused-function # we have some for the docs that aren't called
        )
if (CMAKE_C_COMPILER_ID STREQUAL "GNU")
    add_compile_options(-Wno-maybe-uninitialized)
endif()

if (NOT FREERTOS_KERNEL_PATH AND NOT DEFINED ENV{FREERTOS_KERNEL_PATH})
    message("Skipping FreeRTOS examples as FREERTOS_KERNEL_PATH not defined")
    return()
endif()

include(FreeRTOS_Kernel_import.cmake)

# add_subdirectory(hello_freertos)

# Example running FreeRTOS on 1 core
add_executable(hello_freertos_one_core
    hello_freertos.c
    )
target_include_directories(hello_freertos_one_core PRIVATE
    ${CMAKE_CURRENT_LIST_DIR}
    )
# Linking to FreeRTOS-Kernel-Heap4 means we use a dynamic heap for allocations
target_link_libraries(hello_freertos_one_core PRIVATE
    pico_async_context_freertos
    FreeRTOS-Kernel-Heap4
    pico_stdlib
    )
# Set the nunber of cores to 1.
# This defaults to 2 in FreeRTOSConfig_examples_common.h if not defined in here
target_compile_definitions(hello_freertos_one_core PRIVATE
    configNUMBER_OF_CORES=1
    )
if(PICO_CYW43_SUPPORTED)
    # For led support on pico_w
    target_link_libraries(hello_freertos_one_core PRIVATE
        pico_cyw43_arch_none
        )
endif()
pico_add_extra_outputs(hello_freertos_one_core)

# Example running FreeRTOS on 2 cores
add_executable(hello_freertos_two_cores
    hello_freertos.c
    )
target_include_directories(hello_freertos_two_cores PRIVATE
    ${CMAKE_CURRENT_LIST_DIR}
    )
# Linking to FreeRTOS-Kernel-Heap4 to use a dynamic heap for allocations
target_link_libraries(hello_freertos_two_cores PRIVATE
    pico_async_context_freertos
    FreeRTOS-Kernel-Heap4
    pico_stdlib
    )
if(PICO_CYW43_SUPPORTED)
    # For led support on pico_w
    target_link_libraries(hello_freertos_two_cores PRIVATE
        pico_cyw43_arch_none
        )
endif()
pico_add_extra_outputs(hello_freertos_two_cores)

# Example running FreeRTOS on 2 cores with static RAM allocation
add_executable(hello_freertos_static_allocation
    hello_freertos.c
    )
target_include_directories(hello_freertos_static_allocation PRIVATE
    ${CMAKE_CURRENT_LIST_DIR}
    )
# Linking to FreeRTOS-Kernel-Static to use static memory instead of a dynamic heap for allocations
target_link_libraries(hello_freertos_static_allocation PRIVATE
    pico_async_context_freertos
    FreeRTOS-Kernel-Static
    pico_stdlib
    )
# Change the configuration to just use static RAM allocation
# If configSUPPORT_DYNAMIC_ALLOCATION is left undefined it will default to 1
# If configSUPPORT_STATIC_ALLOCATION is left undefined it will default to 0
target_compile_definitions(hello_freertos_static_allocation PRIVATE
    configSUPPORT_STATIC_ALLOCATION=1
    configSUPPORT_DYNAMIC_ALLOCATION=0
    )
if(PICO_CYW43_SUPPORTED)
    # For led support on pico_w
    target_link_libraries(hello_freertos_static_allocation PRIVATE
        pico_cyw43_arch_none
        )
endif()
pico_add_extra_outputs(hello_freertos_static_allocation)

# Enable usb output, disable uart output
pico_enable_stdio_usb(hello_freertos_one_core 1)
pico_enable_stdio_uart(hello_freertos_one_core 0)
