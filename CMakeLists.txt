# Generated Cmake Pico project file

cmake_minimum_required(VERSION 3.13)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(FREERTOS_KERNEL_PATH ~/git/FreeRTOS-Kernel) # TODO: update?
set(PICO_SDK_PATH ~/git/pico-sdk)

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
# ====================================================================================
set(PICO_BOARD pico2 CACHE STRING "Board type")
set(PICO_PLATFORM rp2350-arm-s)

# Pull in Raspberry Pi Pico SDK (must be before project)
include(pico_sdk_import.cmake)

project(pico2_freertos_template C CXX ASM)

# Initialise the Raspberry Pi Pico SDK
pico_sdk_init()

# Pull in FreeRTOS
if (NOT FREERTOS_KERNEL_PATH AND NOT DEFINED ENV{FREERTOS_KERNEL_PATH})
    message(FATAL_ERROR "FREERTOS_KERNEL_PATH must be defined!")
endif()
include(FreeRTOS_Kernel_import.cmake)


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

