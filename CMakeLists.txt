cmake_minimum_required(VERSION 3.15)
project(FreeRTOS
        VERSION 10.3.0
        DESCRIPTION "FreeRTOS Kernel CMake Package"
        LANGUAGES C ASM)

include_guard(GLOBAL)

add_library(FreeRTOS_Kernel INTERFACE)
add_library(FreeRTOS::Kernel ALIAS FreeRTOS_Kernel)
target_include_directories(FreeRTOS_Kernel INTERFACE ${CMAKE_CURRENT_LIST_DIR}/include)

target_sources(FreeRTOS_Kernel INTERFACE
        ${CMAKE_CURRENT_LIST_DIR}/croutine.c
        ${CMAKE_CURRENT_LIST_DIR}/event_groups.c
        ${CMAKE_CURRENT_LIST_DIR}/list.c
        ${CMAKE_CURRENT_LIST_DIR}/queue.c
        ${CMAKE_CURRENT_LIST_DIR}/stream_buffer.c
        ${CMAKE_CURRENT_LIST_DIR}/tasks.c
        ${CMAKE_CURRENT_LIST_DIR}/timers.c
        )

if(DEFINED freertos_PORT_COMPILER)
        message(STATUS "FreeRTOS::Kernel : Using freertos_PORT_COMPILER == ${freertos_PORT_COMPILER}")
        message(STATUS "FreeRTOS::Kernel : Ignoring CMAKE_CXX_COMPILER_ID of ${CMAKE_CXX_COMPILER_ID}")
else()
        message(STATUS "FreeRTOS::Kernel : CMAKE_CXX_COMPILER_ID of ${CMAKE_CXX_COMPILER_ID}")
        if ((CMAKE_CXX_COMPILER_ID STREQUAL "ARMClang") OR (CMAKE_CXX_COMPILER_ID STREQUAL "GNU"))
                set(freertos_PORT_COMPILER "GCC")
        elseif (CMAKE_CXX_COMPILER_ID STREQUAL "IAR")
                set(freertos_PORT_COMPILER "IAR")
        elseif (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
                set(freertos_PORT_COMPILER "MSVC-MingW")
        else()
                message(FATAL_ERROR "FreeRTOS::Kernel : Unsupported CMAKE_CXX_COMPILER_ID of ${CMAKE_CXX_COMPILER_ID}")
        endif()
        message(STATUS "FreeRTOS::Kernel : Using Port ${freertos_PORT_COMPILER}")
endif()

if(DEFINED freertos_PORT_CPU)
        target_sources(FreeRTOS_Kernel INTERFACE
                ${CMAKE_CURRENT_LIST_DIR}/portable/${freertos_PORT_COMPILER}/${freertos_PORT_CPU}/port.c
                ${CMAKE_CURRENT_LIST_DIR}/portable/${freertos_PORT_COMPILER}/${freertos_PORT_CPU}/portmacro.h
                )
        target_include_directories(FreeRTOS_Kernel INTERFACE
                ${CMAKE_CURRENT_LIST_DIR}/portable/${freertos_PORT_COMPILER}/${freertos_PORT_CPU}
                )
else()
    message(FATAL_ERROR "FreeRTOS::Kernel : Port type not specified! please define freertos_PORT_CPU before including the FreeRTOS package")
endif()

if(DEFINED freertos_HEAP_TYPE)
    message(STATUS "FreeRTOS::Kernel : Using heap type ${freertos_HEAP_TYPE}")
    target_sources(FreeRTOS_Kernel INTERFACE ${CMAKE_CURRENT_LIST_DIR}/portable/MemMang/heap_${freertos_HEAP_TYPE}.c)
else()
    message(WARNING "FreeRTOS::Kernel : No heap type specified! if this is an error please define freertos_HEAP_TYPE before including FreeRTOS package")
endif()

add_library(FreeRTOS_CMSIS_V1 INTERFACE)
add_library(FreeRTOS::CMSIS ALIAS FreeRTOS_CMSIS_V1)
target_include_directories(FreeRTOS_CMSIS_V1 INTERFACE ${CMAKE_CURRENT_LIST_DIR}/CMSIS_RTOS)
target_sources(FreeRTOS_CMSIS_V1 INTERFACE ${CMAKE_CURRENT_LIST_DIR}/CMSIS_RTOS/cmsis_os.c)

add_library(FreeRTOS_CMSIS_V2 INTERFACE)
add_library(FreeRTOS::CMSISv2 ALIAS FreeRTOS_CMSIS_V2)
target_include_directories(FreeRTOS_CMSIS_V2 INTERFACE ${CMAKE_CURRENT_LIST_DIR}/CMSIS_RTOS_V2)
target_sources(FreeRTOS_CMSIS_V2 INTERFACE ${CMAKE_CURRENT_LIST_DIR}/CMSIS_RTOS_V2/cmsis_os2.c)
