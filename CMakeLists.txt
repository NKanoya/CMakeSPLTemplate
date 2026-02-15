cmake_minimum_required(VERSION 3.22)

set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_PROCESSOR cortex-m3)

# ===== 项目名称 / Project Name =====
#
# 构建输出目标名称（可以根据项目需要修改）
# Build output target name (can be renamed as needed)
set(CUSTOMIZED_PROJECT_NAME stm32_template)

set(CMAKE_TOOLCHAIN_FILE "${CMAKE_CURRENT_LIST_DIR}/arm-none-eabi-toolchain.cmake" CACHE STRING "Toolchain file for ARM cross-compilation")

project(${CUSTOMIZED_PROJECT_NAME} C ASM)


set(MCU_FLAGS -mcpu=cortex-m3 -mthumb)

set(OPENOCD openocd)

include_directories(
        user/inc
        spl/inc
        core/inc
)

file(GLOB_RECURSE USER_SOURCES user/src/*.c)
file(GLOB_RECURSE SPL_SOURCES spl/src/*.c)
file(GLOB_RECURSE STARTUP_SOURCES core/src/*.c core/src/*.s)

add_executable(${PROJECT_NAME}.elf
        ${USER_SOURCES}
        ${SPL_SOURCES}
        ${STARTUP_SOURCES}
)

target_compile_definitions(${PROJECT_NAME}.elf PRIVATE
        STM32F10X_MD               # STM32F10X
        USE_STDPERIPH_DRIVER       # SPL Library
)

target_compile_options(${PROJECT_NAME}.elf PRIVATE
        ${MCU_FLAGS}
        -Wall
        -O0
        -ffunction-sections
        -fdata-sections
)

target_link_options(${PROJECT_NAME}.elf PRIVATE
        ${MCU_FLAGS}
        -Wl,--gc-sections
        -T${CMAKE_SOURCE_DIR}/STM32F103C8Tx_FLASH.ld
)

add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
        COMMAND arm-none-eabi-objcopy -O ihex ${PROJECT_NAME}.elf ${PROJECT_NAME}.hex
        COMMAND arm-none-eabi-objcopy -O binary ${PROJECT_NAME}.elf ${PROJECT_NAME}.bin
)

set(ELF_FILE "${CMAKE_BINARY_DIR}/${PROJECT_NAME}.elf")
string(REPLACE "\\" "/" ELF_FILE "${ELF_FILE}")
# burn_stm32.bat
file(WRITE "${CMAKE_BINARY_DIR}/burn_stm32.bat"
        "@echo off\n"
        "set TARGET_ELF=${ELF_FILE}\n"
        "set OPENOCD_EXE=${OPENOCD}\n"
        "echo Burning %TARGET_ELF% to STM32...\n"
        "\"%OPENOCD_EXE%\" -f interface/cmsis-dap.cfg -f target/stm32f1x.cfg -c \"program %TARGET_ELF% verify reset exit\"\n"
)
