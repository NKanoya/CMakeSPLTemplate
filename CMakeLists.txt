cmake_minimum_required(VERSION 3.22)

set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_PROCESSOR cortex-m3)

project(stm32_template C ASM)

set(MCU_FLAGS -mcpu=cortex-m3 -mthumb)

# ===== OpenOCD 安装目录 / OpenOCD Installation Directory =====
#
# 请将此路径修改为你本地 OpenOCD 的安装目录
# / Please change this path to your local OpenOCD installation directory
# / e.g. on Windows: "D:/Program/OpenOCD"
#
set(OPENOCD_DIR "D:/Program/OpenOCD")

set(OPENOCD "${OPENOCD_DIR}/bin/openocd.exe")
set(OPENOCD_SCRIPTS "${OPENOCD_DIR}/scripts")

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
