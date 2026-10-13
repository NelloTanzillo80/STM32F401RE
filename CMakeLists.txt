cmake_minimum_required(VERSION 3.10)

set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_PROCESSOR arm)

set(CMAKE_C_COMPILER arm-none-eabi-gcc)
set(CMAKE_ASM_COMPILER arm-none-eabi-gcc)
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)

project(GPIO_IOToggle C ASM)

# Directory radice STM32CubeF4
set(CUBEF4_DIR "/home/minec/STM32CubeF4")

# Flag hardware definite come lista CMake
set(MCU_FLAGS
    -mcpu=cortex-m4
    -mthumb
    -mfpu=fpv4-sp-d16
    -mfloat-abi=hard
)

add_compile_options(
    ${MCU_FLAGS}
    -DSTM32F401xE
    -DUSE_HAL_DRIVER
    -O0 -g
)

# Include Directories
include_directories(
    Inc
    ${CUBEF4_DIR}/Drivers/STM32F4xx_HAL_Driver/Inc
    ${CUBEF4_DIR}/Drivers/CMSIS/Device/ST/STM32F4xx/Include
    ${CUBEF4_DIR}/Drivers/CMSIS/Include
    ${CUBEF4_DIR}/Drivers/BSP/STM32F4xx-Nucleo
)

# Sorgenti HAL ed esclusione template
file(GLOB HAL_SOURCES "${CUBEF4_DIR}/Drivers/STM32F4xx_HAL_Driver/Src/*.c")
list(FILTER HAL_SOURCES EXCLUDE REGEX ".*_template\\.c$")

# File Startup e Linker Script
file(GLOB STARTUP_FILE "${CUBEF4_DIR}/Drivers/CMSIS/Device/ST/STM32F4xx/Source/Templates/gcc/startup_stm32f401xe.*")
set(LINKER_FILE "${CMAKE_CURRENT_SOURCE_DIR}/STM32CubeIDE/STM32F401RETX_FLASH.ld")

# Definizione eseguibile
add_executable(${PROJECT_NAME}.elf
    Src/main.c
    Src/stm32f4xx_it.c
    Src/system_stm32f4xx.c
    ${STARTUP_FILE}
    ${HAL_SOURCES}
)

# Opzioni di linking (Opzione A: nano.specs + nosys.specs)
# Opzioni di linking pulite senza warning
target_link_options(${PROJECT_NAME}.elf PRIVATE
    ${MCU_FLAGS}
    -T${LINKER_FILE}
    -Wl,-Map=${PROJECT_NAME}.map
    --specs=nano.specs
    --specs=nosys.specs
    -Wl,--no-warnings
)
