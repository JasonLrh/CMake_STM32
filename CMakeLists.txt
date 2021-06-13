cmake_minimum_required(VERSION 2.9)
set(HAVE_FLAG_SEARCH_PATHS_FIRST 0)
project(robosystem) # project name
set(CMAKE_C_LINK_FLAGS "")
enable_language(ASM)

# device and cc setings
set(CMAKE_BUILD_TYPE "Release")
set(MCU_FLAGS "-mcpu=cortex-m4 -mthumb -mfpu=fpv4-sp-d16 -mfloat-abi=hard")
# set(CMAKE_VERBOSE_MAKEFILE on)

set(CMAKE_C_FLAGS         "${MCU_FLAGS} -Wall -fdata-sections -ffunction-sections")
set(CMAKE_C_FLAGS_DEBUG   "-Og -g -gdwarf-2")
set(CMAKE_C_FLAGS_RELEASE "-O2")

# defines include and library settings
add_definitions(
    -DUSE_HAL_DRIVER
    -DSTM32F407xx
)
include_directories(
    "Inc"
    "Drivers/STM32F4xx_HAL_Driver/Inc"
    "Drivers/STM32F4xx_HAL_Driver/Inc/Legacy"
    "Middlewares/ST/STM32_USB_Device_Library/Core/Inc"
    "Middlewares/ST/STM32_USB_Device_Library/Class/CDC/Inc"
    "Drivers/CMSIS/Device/ST/STM32F4xx/Include"
    "Drivers/CMSIS/Include"
    "Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS"
    "Middlewares/Third_Party/FreeRTOS/Source/include"
    "Middlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_CM4F"
)

link_directories(
    # ${PROJECT_BINARY_DIR}
)
# delete *templete.c
exec_program("rm */*/*/*template.c */*/*/*/*/*template.c || echo \"\\033[32mDirectory is very clean\\033[0m\"" ${CMAKE_HOME_DIRECTORY})

# file input: asm,ldscript,c/c++
file(GLOB Startup_ASM "*.s")
file(GLOB Linker_Script "*.ld")
file(GLOB STM32_HAL_Src "Drivers/STM32*_HAL_Driver/Src/*.c")
file(GLOB STM32_HAL_USB_Src "Middlewares/ST/STM32_USB_Device_Library/Core/Src/*.c")
file(GLOB freertos_src "Middlewares/Third_Party/FreeRTOS/Source/*.c")
aux_source_directory("Src" SRC_LIST)

# add target
add_library(STM32_HAL_Library STATIC ${STM32_HAL_Src} ${Startup_ASM})
add_library(STM32_HAL_USB_Library STATIC ${STM32_HAL_USB_Src} "Middlewares/ST/STM32_USB_Device_Library/Class/CDC/Src/usbd_cdc.c")
add_library(freertos STATIC ${freertos_src} "Middlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_CM4F/port.c" 
                                    "Middlewares/Third_Party/FreeRTOS/Source/portable/MemMang/heap_4.c"
                                    "Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS/cmsis_os.c")

# library relationship
target_link_libraries(STM32_HAL_USB_Library STM32_HAL_Library)

# target rules
add_executable(${PROJECT_NAME}.elf ${SRC_LIST})
target_link_libraries(${PROJECT_NAME}.elf -lc -lm -lnosys STM32_HAL_Library STM32_HAL_USB_Library freertos)
################################################################################################################################################
#   you shouldn't   edit    the context below   unless  you know    what    you are doing
################################################################################################################################################

# compiler settings
set(CMAKE_C_COMPILER arm-none-eabi-gcc)
set(CMAKE_CXX_COMPILER arm-none-eabi-gcc)
set(CMAKE_OBJCOPY arm-none-eabi-objcopy) 
set(CMAKE_SIZE arm-none-eabi-size)
set(CMAKE_AR arm-none-eabi-ar)
set(CMAKE_RANLIB arm-none-eabi-ranlib)
set(CMAKE_ASM_COMPILER arm-none-eabi-gcc)
set(CMAKE_FLASH STM32_Programmer_CLI)

set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_PROCESSOR arm)
set(CMAKE_EXPORT_COMPILE_COMMANDS 1)
set(CMAKE_EXE_LINKER_FLAGS "-specs=nano.specs -T${Linker_Script} -Wl,-Map=${PROJECT_BINARY_DIR}/${PROJECT_NAME}.map,--cref -Wl,--gc-sections")
set(CMAKE_ASM_FLAGS "-x assembler-with-cpp ${CMAKE_C_FLAGS}")

# system settings
set(CMAKE_SHARED_LIBRARY_LINK_C_FLAGS "")
set(CMAKE_SHARED_LIBRARY_LINK_CXX_FLAGS "")
set(CMAKE_OSX_SYSROOT "")
set(CMAKE_OSX_DEPLOYMENT_TARGET "")

# end output
set(ELF_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.elf)
set(HEX_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.hex)
set(BIN_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.bin)

add_custom_target(flash 
    ${CMAKE_FLASH} -c port=SWD -d ${ELF_FILE} -s
    DEPENDS ${PROJECT_NAME}.elf)

# export hex and binary file
add_custom_command(TARGET "${PROJECT_NAME}.elf" POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -Obinary ${ELF_FILE} ${BIN_FILE}
    COMMAND ${CMAKE_OBJCOPY} -Oihex ${ELF_FILE} ${HEX_FILE}
    COMMENT "Building ${HEX_FILE} & ${BIN_FILE}"
    COMMAND ${CMAKE_SIZE} ${ELF_FILE}
)



# get_cmake_property(_variableNames VARIABLES)
# foreach (_variableName ${_variableNames})
#     message(STATUS "${_variableName}=${${_variableName}}")
# endforeach()
