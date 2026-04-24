cmake_minimum_required(VERSION 3.20)

# Use the arm-none-eabi-gcc compiler (provide a toolchain file to use something else like clang)
set(CMAKE_TOOLCHAIN_FILE cmake/stm32-toolchain.cmake)

project(stm32_project C ASM)

# Compiler flags
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

# set(COMMON_FLAGS "-Wall -Wextra -Wpedantic -fdata-sections -ffunction-sections")
# set(CMAKE_C_FLAGS "${COMMON_FLAGS}")
# set(CMAKE_C_FLAGS_DEBUG "-Og -g3")
# set(CMAKE_C_FLAGS_RELEASE "-O3")

# Defines
add_compile_definitions(
    STM32F103xE
    USE_HAL_DRIVER
)

# Include directories
include_directories(
    ${CMAKE_SOURCE_DIR}/src
    ${CMAKE_SOURCE_DIR}/Drivers/STM32F1xx_HAL_Driver/Inc
    ${CMAKE_SOURCE_DIR}/Drivers/CMSIS/Device/ST/STM32F1xx/Include
    ${CMAKE_SOURCE_DIR}/Drivers/CMSIS/Include
)

# HAL source files
file(GLOB_RECURSE HAL_SOURCES 
    "${CMAKE_SOURCE_DIR}/Drivers/STM32F1xx_HAL_Driver/Src/*.c"
)

# Remove template file
list(FILTER HAL_SOURCES EXCLUDE REGEX ".*template\\.c$")

# Project sources
set(PROJECT_SOURCES
    src/main.c
    src/system_stm32f1xx.c
    src/stm32f1xx_it.c
    src/stm32f1xx_hal_msp.c
    startup/startup_stm32f103xe.s
)

# Create executable
add_executable(${PROJECT_NAME}.elf
    ${PROJECT_SOURCES}
    ${HAL_SOURCES}
)

# Linker script
set(LINKER_SCRIPT ${CMAKE_SOURCE_DIR}/linker/STM32F103VETx_FLASH.ld)

# Linker flags
target_link_options(${PROJECT_NAME}.elf PRIVATE
    -T${LINKER_SCRIPT}
    -Wl,-Map=${PROJECT_NAME}.map,--cref
    -Wl,--gc-sections
    -specs=nano.specs
    -specs=nosys.specs
    -lc
    -lm
)

# Post build commands
add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O ihex ${PROJECT_NAME}.elf ${PROJECT_NAME}.hex
    COMMAND ${CMAKE_OBJCOPY} -O binary ${PROJECT_NAME}.elf ${PROJECT_NAME}.bin
    COMMAND ${CMAKE_SIZE} ${PROJECT_NAME}.elf
    COMMENT "Building HEX and BIN files"
)

# OpenOCD flash target
add_custom_target(flash
    COMMAND openocd 
        -f ${CMAKE_SOURCE_DIR}/openocd.cfg
        -c "program ${PROJECT_NAME}.elf verify reset exit"
    DEPENDS ${PROJECT_NAME}.elf
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    COMMENT "Flashing ${PROJECT_NAME}.elf using OpenOCD"
)

# Debug server target (run OpenOCD without flashing)
add_custom_target(openocd
    COMMAND openocd -f ${CMAKE_SOURCE_DIR}/openocd.cfg
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    COMMENT "Starting OpenOCD debug server"
)

# Erase flash target
add_custom_target(erase
    COMMAND openocd 
        -f ${CMAKE_SOURCE_DIR}/openocd.cfg
        -c "init; reset halt; stm32f1x mass_erase 0; exit"
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    COMMENT "Erasing STM32 flash"
)

# OpenOCD flash target
# OpenOCD flash target (using flash write_image)
add_custom_target(flash_openocd
    COMMAND openocd 
        -f ${CMAKE_SOURCE_DIR}/openocd.cfg
        -c "transport select hla_swd"
        -c "init"
        -c "reset halt"
        -c "flash write_image erase ${PROJECT_NAME}.elf"
        -c "reset run"
        -c "shutdown"
    DEPENDS ${PROJECT_NAME}.elf
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    COMMENT "Flashing ${PROJECT_NAME}.elf using OpenOCD"
)