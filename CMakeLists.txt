cmake_minimum_required(VERSION 3.3)
project(bldc)
enable_language(C ASM)

set( CMAKE_VERBOSE_MAKEFILE on )
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

set(CMAKE_CACHEFILE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/)


function(getconfig varname config_file)
    get_filename_component(FILE_PATH ${config_file} ABSOLUTE)
    if(NOT EXISTS ${FILE_PATH})
        message(WARNING "File ${config_file} doesn't exist")
        return()
    endif()
    file(READ ${config_file} CONFIG_TEXT)
    if ("${CONFIG_TEXT}" STREQUAL "")
        message(WARNING "File ${config_file} empty")
        return()
    endif()
    string(REGEX MATCH "${varname}[ ]+(TRUE|FALSE)" RESULT_TEXT ${CONFIG_TEXT})
    if ("${RESULT_TEXT}" STREQUAL "")
        message(WARNING "Didn't found boolean variable ${varname} in the file ${config_file}")
        return()
    endif()
    string(REGEX MATCH "(TRUE|FALSE)" RESULT ${RESULT_TEXT})
    set(${varname} ${RESULT} PARENT_SCOPE)
endfunction()

function(set_compile_flags objects base_flag)
    #    message(WARNING ${${base_flag}})
    foreach(FILE ${${objects}})
        get_filename_component(FILE_WE ${FILE} NAME_WE)
        get_filename_component(FILE_EXT ${FILE} EXT)
        set(FILE_FLAG ${base_flag}${FILE_WE}.lst)
        #        message(WARNING "flag: ${FILE_FLAG}")
        set_source_files_properties(${FILE} PROPERTIES COMPILE_FLAGS ${FILE_FLAG})
    endforeach()
endfunction()



set(build_args ""                 CACHE STRING "COMPILE")





#message(WARNING "CACH-DTHUMB_NO_INTERWORKING"E FILEDIR ${CMAKE_CACHEFILE_DIR}" )





## Compiler options here.
#ifeq (${USE_OPT},)
#USE_OPT = -O2 -ggdb -fomit-frame-pointer -falign-functions=16 -std=gnu99
#USE_OPT += -DBOARD_OTG_NOVBUSSENS ${build_args}
#USE_OPT += -fsingle-precision-constant -Wdouble-promotion
#endif
set(USE_OPT
        -O2 -ggdb -fomit-frame-pointer -falign-functions=16 -std=gnu99
        -DBOARD_OTG_NOVBUSSENS
        -fsingle-precision-constant -Wdouble-promotion CACHE STRING "COMPILE")

set(USE_OPT ${USE_OPT} ${build_args})
## C specific options here (added to USE_OPT).
#ifeq (${USE_COPT},)
#USE_COPT =
#endif
set(USE_COPT ""                 CACHE STRING "COMPILE")


## C++ specific options here (added to USE_OPT).
#ifeq (${USE_CPPOPT},)
#USE_CPPOPT = -fno-rtti
#endif
set(USE_CPPOPT -fno-rtti      CACHE STRING "COMPILE")



## Enable this if you want the linker to remove unused code and data
#ifeq (${USE_LINK_GC},)
#USE_LINK_GC = yes
#endif
set(USE_LINK_GC                 TRUE      CACHE BOOL    "COMPILE")


## Linker extra options here.
#ifeq (${USE_LDOPT},)
#USE_LDOPT =
#endif
set(USE_LDOPT                   ""      CACHE STRING "COMPILE")


## Enable this if you want link time optimizations (LTO)
#ifeq (${USE_LTO},)
#USE_LTO = no
#endif
set(USE_LTO                     FALSE   CACHE BOOL      "COMPILE")

## If enabled, this option allows to compile the application in THUMB mode.
#ifeq (${USE_THUMB},)
#USE_THUMB = yes
#endif
set(USE_THUMB                   TRUE    CACHE BOOL      "COMPILE")

## Enable this if you want to see the full log while compiling.
#ifeq (${USE_VERBOSE_COMPILE},)
#USE_VERBOSE_COMPILE = yes
#endif
set(USE_VERBOSE_COMPILE         TRUE    CACHE BOOL      "COMPILE")

## If enabled, this option makes the build process faster by not compiling
## modules not used in the current configuration.
#ifeq (${USE_SMART_BUILD},)
#USE_SMART_BUILD = yes
#endif
set(USE_SMART_BUILD             TRUE    CACHE BOOL      "COMPILE")


## Stack size to be allocated to the Cortex-M process stack. This stack is
## the stack used by the main() thread.
#ifeq (${USE_PROCESS_STACKSIZE},)
#USE_PROCESS_STACKSIZE = 0x400
#endif
set(USE_PROCESS_STACKSIZE       0x400      CACHE STRING "COMPILE")

## Stack size to the allocated to the Cortex-M main/exceptions stack. This
## stack is used for processing interrupts and exceptions.
#ifeq (${USE_EXCEPTIONS_STACKSIZE},)
#USE_EXCEPTIONS_STACKSIZE = 0x400
#endif
set(USE_EXCEPTIONS_STACKSIZE       0x400      CACHE STRING "COMPILE")

## Enables the use of FPU on Cortex-M4 (no, softfp, hard).
#ifeq (${USE_FPU},)
#USE_FPU = hard
#endif
set(USE_FPU       hard      CACHE STRING "COMPILE")

## Enable this if you really want to use the STM FWLib.
#ifeq (${USE_FWLIB},)
#USE_FWLIB = yes
#endif
set(USE_FWLIB       TRUE      CACHE BOOL "COMPILE")


# Define project name here
#PROJECT = BLDC_4_ChibiOS
set(PROJECT       BLDC_4_ChibiOS     CACHE STRING "COMPILE")

## Imported source files and paths
#CHIBIOS = ChibiOS_3.0.2
set(CHIBIOS ./ChibiOS_3.0.2)

## Define linker script file here
#LDSCRIPT= ld_eeprom_emu.ld
get_filename_component(LDSCIRPT_ABS ld_eeprom_emu.ld ABSOLUTE)
message(WARNING "LDSCIRPT_ABS ${LDSCIRPT_ABS}")
set(LDSCRIPT ${LDSCIRPT_ABS})


################### STARTUP #########################

#include ${CHIBIOS}/os/common/ports/ARMCMx/compilers/GCC/mk/startup_stm32f4xx.mk
set(STARTUPSRC
        ${CHIBIOS}/os/common/ports/ARMCMx/compilers/GCC/crt1.c
        ${CHIBIOS}/os/common/ports/ARMCMx/compilers/GCC/vectors.c)
set(STARTUPASM
        ${CHIBIOS}/os/common/ports/ARMCMx/compilers/GCC/crt0_v7m.s)
set(STARTUPINC
        ${CHIBIOS}/os/common/ports/ARMCMx/devices/STM32F4xx
        ${CHIBIOS}/os/ext/CMSIS/include
        ${CHIBIOS}/os/ext/CMSIS/ST)
set(STARTUPLD
        ${CHIBIOS}/os/common/ports/ARMCMx/compilers/GCC/ld)



################### HAL #########################
#include ${CHIBIOS}/os/hal/hal.mk
if(USE_SMART_BUILD)

    set(HALSRC
            ${CHIBIOS}/os/hal/src/hal.c
            ${CHIBIOS}/os/hal/src/st.c
            ${CHIBIOS}/os/hal/src/hal_queues.c
            ${CHIBIOS}/os/hal/src/hal_mmcsd.c)

    getconfig(HAL_USE_ADC halconf.h)
    if(HAL_USE_ADC)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/adc.c)
    endif(HAL_USE_ADC)

    getconfig(HAL_USE_CAN halconf.h)
    if(HAL_USE_CAN)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/can.c)
    endif(HAL_USE_CAN)

    getconfig(HAL_USE_DAC halconf.h)
    if(HAL_USE_DAC)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/dac.c)
    endif(HAL_USE_DAC)

    getconfig(HAL_USE_EXT halconf.h)
    if(HAL_USE_EXT)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/ext.c)
    endif(HAL_USE_EXT)

    getconfig(HAL_USE_GPT halconf.h)
    if(HAL_USE_GPT)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/gpt.c)
    endif(HAL_USE_GPT)

    getconfig(HAL_USE_I2C halconf.h)
    if(HAL_USE_I2C)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/i2c.c)
    endif(HAL_USE_I2C)

    getconfig(HAL_USE_I2S halconf.h)
    if(HAL_USE_I2S)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/i2s.c)
    endif(HAL_USE_I2S)

    getconfig(HAL_USE_ICU halconf.h)
    if(HAL_USE_ICU)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/icu.c)
    endif(HAL_USE_ICU)

    getconfig(HAL_USE_MAC halconf.h)
    if(HAL_USE_MAC)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/mac.c)
    endif(HAL_USE_MAC)

    getconfig(HAL_USE_MMC_SPI halconf.h)
    if(HAL_USE_MMC_SPI)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/mmc_spi.c)
    endif(HAL_USE_MMC_SPI)

    getconfig(HAL_USE_PAL halconf.h)
    if(HAL_USE_PAL)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/pal.c)
    endif(HAL_USE_PAL)

    getconfig(HAL_USE_PWM halconf.h)
    if(HAL_USE_PWM)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/pwm.c)
    endif(HAL_USE_PWM)

    getconfig(HAL_USE_RTC halconf.h)
    if(HAL_USE_RTC)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/rtc.c)
    endif(HAL_USE_RTC)

    getconfig(HAL_USE_SDC halconf.h)
    if(HAL_USE_SDC)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/sdc.c)
    endif(HAL_USE_SDC)

    getconfig(HAL_USE_SERIAL halconf.h)
    if(HAL_USE_SERIAL)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/serial.c)
    endif(HAL_USE_SERIAL)

    getconfig(HAL_USE_SERIAL_USB halconf.h)
    if(HAL_USE_SERIAL_USB)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/serial_usb.c)
    endif(HAL_USE_SERIAL_USB)

    getconfig(HAL_USE_SPI halconf.h)
    if(HAL_USE_SPI)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/spi.c)
    endif(HAL_USE_SPI)

    getconfig(HAL_USE_UART halconf.h)
    if(HAL_USE_UART)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/uart.c)
    endif(HAL_USE_UART)

    getconfig(HAL_USE_USB halconf.h)
    if(HAL_USE_USB)
        set(HALSRC	${HALSRC}
             ${CHIBIOS}/os/hal/src/usb.c)
    endif(HAL_USE_USB)

else(USE_SMART_BUILD)

    set(HALSRC
            ${CHIBIOS}/os/hal/src/hal.c
            ${CHIBIOS}/os/hal/src/hal_queues.c
            ${CHIBIOS}/os/hal/src/hal_mmcsd.c
            ${CHIBIOS}/os/hal/src/adc.c
            ${CHIBIOS}/os/hal/src/can.c
            ${CHIBIOS}/os/hal/src/dac.c
            ${CHIBIOS}/os/hal/src/ext.c
            ${CHIBIOS}/os/hal/src/gpt.c
            ${CHIBIOS}/os/hal/src/i2c.c
            ${CHIBIOS}/os/hal/src/i2s.c
            ${CHIBIOS}/os/hal/src/icu.c
            ${CHIBIOS}/os/hal/src/mac.c
            ${CHIBIOS}/os/hal/src/mmc_spi.c
            ${CHIBIOS}/os/hal/src/pal.c
            ${CHIBIOS}/os/hal/src/pwm.c
            ${CHIBIOS}/os/hal/src/rtc.c
            ${CHIBIOS}/os/hal/src/sdc.c
            ${CHIBIOS}/os/hal/src/serial.c
            ${CHIBIOS}/os/hal/src/serial_usb.c
            ${CHIBIOS}/os/hal/src/spi.c
            ${CHIBIOS}/os/hal/src/st.c
            ${CHIBIOS}/os/hal/src/uart.c
            ${CHIBIOS}/os/hal/src/usb.c)

endif(USE_SMART_BUILD)
set(HALINC
        ${CHIBIOS}/os/hal/include)



################### PLATFORM #########################
#include ${CHIBIOS}/os/hal/ports/STM32/STM32F4xx/platform.mk
if(USE_SMART_BUILD)

    set(PLATFORMSRC
            ${CHIBIOS}/os/hal/ports/common/ARMCMx/nvic.c
            ${CHIBIOS}/os/hal/ports/STM32/STM32F4xx/stm32_dma.c
            ${CHIBIOS}/os/hal/ports/STM32/STM32F4xx/hal_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/TIMv1/st_lld.c)

    getconfig(HAL_USE_ADC halconf.h)
    if(HAL_USE_ADC)
        set(PLATFORMSRC	${PLATFORMSRC}
             ${CHIBIOS}/os/hal/ports/STM32/STM32F4xx/adc_lld.c)
    endif(HAL_USE_ADC)

    getconfig(HAL_USE_EXT halconf.h)
    if(HAL_USE_EXT)
        set(PLATFORMSRC ${PLATFORMSRC}
            ${CHIBIOS}/os/hal/ports/STM32/STM32F4xx/ext_lld_isr.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/ext_lld.c)
    endif(HAL_USE_EXT)

    getconfig(HAL_USE_CAN halconf.h)
    if(HAL_USE_CAN)
        set(PLATFORMSRC	${PLATFORMSRC}
             ${CHIBIOS}/os/hal/ports/STM32/LLD/can_lld.c)
    endif(HAL_USE_CAN)

    getconfig(HAL_USE_MAC halconf.h)
    if(HAL_USE_MAC)
        set(PLATFORMSRC	${PLATFORMSRC}
             ${CHIBIOS}/os/hal/ports/STM32/LLD/mac_lld.c)
    endif(HAL_USE_MAC)

    getconfig(HAL_USE_SDC halconf.h)
    if(HAL_USE_SDC)
        set(PLATFORMSRC	${PLATFORMSRC}
             ${CHIBIOS}/os/hal/ports/STM32/LLD/sdc_lld.c)
    endif(HAL_USE_SDC)

    getconfig(HAL_USE_DAC halconf.h)
    if(HAL_USE_DAC)
        set(PLATFORMSRC	${PLATFORMSRC}
             ${CHIBIOS}/os/hal/ports/STM32/LLD/DACv1/dac_lld.c)
    endif(HAL_USE_DAC)

    getconfig(HAL_USE_PAL halconf.h)
    if(HAL_USE_PAL)
        set(PLATFORMSRC	${PLATFORMSRC}
             ${CHIBIOS}/os/hal/ports/STM32/LLD/GPIOv2/pal_lld.c)
    endif(HAL_USE_PAL)

    getconfig(HAL_USE_I2C halconf.h)
    if(HAL_USE_I2C)
        set(PLATFORMSRC	${PLATFORMSRC}
             ${CHIBIOS}/os/hal/ports/STM32/LLD/I2Cv1/i2c_lld.c)
    endif(HAL_USE_I2C)

    getconfig(HAL_USE_USB halconf.h)
    if(HAL_USE_USB)
        set(PLATFORMSRC	${PLATFORMSRC}
             ${CHIBIOS}/os/hal/ports/STM32/LLD/OTGv1/usb_lld.c)
    endif(HAL_USE_USB)

    getconfig(HAL_USE_RTC halconf.h)
    if(HAL_USE_RTC)
        set(PLATFORMSRC	${PLATFORMSRC}
             ${CHIBIOS}/os/hal/ports/STM32/LLD/RTCv2/rtc_lld.c)
    endif(HAL_USE_RTC)

    getconfig(HAL_USE_I2S halconf.h)
    if(HAL_USE_I2S)
        set(PLATFORMSRC	${PLATFORMSRC}
             ${CHIBIOS}/os/hal/ports/STM32/LLD/SPIv1/i2s_lld.c)
    endif(HAL_USE_I2S)

    getconfig(HAL_USE_SPI halconf.h)
    if(HAL_USE_SPI)
        set(PLATFORMSRC	${PLATFORMSRC}
             ${CHIBIOS}/os/hal/ports/STM32/LLD/SPIv1/spi_lld.c)
    endif(HAL_USE_SPI)

    getconfig(HAL_USE_GPT halconf.h)
    if(HAL_USE_GPT)
        set(PLATFORMSRC	${PLATFORMSRC}
             ${CHIBIOS}/os/hal/ports/STM32/LLD/TIMv1/gpt_lld.c)
    endif(HAL_USE_GPT)

    getconfig(HAL_USE_ICU halconf.h)
    if(HAL_USE_ICU)
        set(PLATFORMSRC	${PLATFORMSRC}
             ${CHIBIOS}/os/hal/ports/STM32/LLD/TIMv1/icu_lld.c)
    endif(HAL_USE_ICU)

    getconfig(HAL_USE_PWM halconf.h)
    if(HAL_USE_PWM)
        set(PLATFORMSRC	${PLATFORMSRC}
             ${CHIBIOS}/os/hal/ports/STM32/LLD/TIMv1/pwm_lld.c)
    endif(HAL_USE_PWM)

    getconfig(HAL_USE_SERIAL halconf.h)
    if(HAL_USE_SERIAL)
        set(PLATFORMSRC	${PLATFORMSRC}
             ${CHIBIOS}/os/hal/ports/STM32/LLD/USARTv1/serial_lld.c)
    endif(HAL_USE_SERIAL)

    getconfig(HAL_USE_UART halconf.h)
    if(HAL_USE_UART)
        set(PLATFORMSRC	${PLATFORMSRC}
             ${CHIBIOS}/os/hal/ports/STM32/LLD/USARTv1/uart_lld.c)
    endif(HAL_USE_UART)


else(USE_SMART_BUILD)
    set(PLATFORMSRC
            ${CHIBIOS}/os/hal/ports/common/ARMCMx/nvic.c
            ${CHIBIOS}/os/hal/ports/STM32/STM32F4xx/stm32_dma.c
            ${CHIBIOS}/os/hal/ports/STM32/STM32F4xx/hal_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/STM32F4xx/adc_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/STM32F4xx/ext_lld_isr.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/can_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/ext_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/mac_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/sdc_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/DACv1/dac_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/GPIOv2/pal_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/I2Cv1/i2c_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/OTGv1/usb_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/RTCv2/rtc_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/SPIv1/i2s_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/SPIv1/spi_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/TIMv1/gpt_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/TIMv1/icu_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/TIMv1/pwm_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/TIMv1/st_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/USARTv1/serial_lld.c
            ${CHIBIOS}/os/hal/ports/STM32/LLD/USARTv1/uart_lld.c)
endif(USE_SMART_BUILD)
set(PLATFORMINC
        ${CHIBIOS}/os/hal/ports/common/ARMCMx
        ${CHIBIOS}/os/hal/ports/STM32/STM32F4xx
        ${CHIBIOS}/os/hal/ports/STM32/LLD
        ${CHIBIOS}/os/hal/ports/STM32/LLD/DACv1
        ${CHIBIOS}/os/hal/ports/STM32/LLD/GPIOv2
        ${CHIBIOS}/os/hal/ports/STM32/LLD/I2Cv1
        ${CHIBIOS}/os/hal/ports/STM32/LLD/OTGv1
        ${CHIBIOS}/os/hal/ports/STM32/LLD/RTCv2
        ${CHIBIOS}/os/hal/ports/STM32/LLD/SPIv1
        ${CHIBIOS}/os/hal/ports/STM32/LLD/TIMv1
        ${CHIBIOS}/os/hal/ports/STM32/LLD/USARTv1
        ${CHIBIOS}/os/hal/ports/STM32/LLD/FSMCv1)

################### BOARD #########################
#include ${CHIBIOS}/os/hal/boards/ST_STM32F4_DISCOVERY/board.mk
set(BOARDSRC
        ${CHIBIOS}/os/hal/boards/ST_STM32F4_DISCOVERY/board.c)

set(BOARDINC
        ${CHIBIOS}/os/hal/boards/ST_STM32F4_DISCOVERY)

################### OSAL #########################
#include ${CHIBIOS}/os/hal/osal/rt/osal.mk
set(OSALSRC
        ${CHIBIOS}/os/hal/osal/rt/osal.c)

set(OSALINC
        ${CHIBIOS}/os/hal/osal/rt)

################### RT #########################
#include ${CHIBIOS}/os/rt/rt.mk
if(USE_SMART_BUILD)
    message(WARNING "Using Smart build")
    set(KERNSRC
            ${CHIBIOS}/os/rt/src/chsys.c
            ${CHIBIOS}/os/rt/src/chdebug.c
            ${CHIBIOS}/os/rt/src/chvt.c
            ${CHIBIOS}/os/rt/src/chschd.c
            ${CHIBIOS}/os/rt/src/chthreads.c)

    getconfig(CH_CFG_USE_TM chconf.h)
    if(CH_CFG_USE_TM)
	    set(KERNSRC	${KERNSRC}
            ${CHIBIOS}/os/rt/src/chtm.c)
    endif(CH_CFG_USE_TM)

    getconfig(CH_DBG_STATISTICS chconf.h)
    if(CH_DBG_STATISTICS)
        set(KERNSRC	${KERNSRC}
             ${CHIBIOS}/os/rt/src/chstats.c)
    endif(CH_DBG_STATISTICS)

    getconfig(CH_CFG_USE_DYNAMIC chconf.h)
    if(CH_CFG_USE_DYNAMIC)
        set(KERNSRC	${KERNSRC}
             ${CHIBIOS}/os/rt/src/chdynamic.c)
    endif(CH_CFG_USE_DYNAMIC)

    getconfig(CH_CFG_USE_REGISTRY chconf.h)
    if(CH_CFG_USE_REGISTRY)
        set(KERNSRC	${KERNSRC}
             ${CHIBIOS}/os/rt/src/chregistry.c)
    endif(CH_CFG_USE_REGISTRY)

    getconfig(CH_CFG_USE_SEMAPHORES chconf.h)
    if(CH_CFG_USE_SEMAPHORES)
        set(KERNSRC	${KERNSRC}
             ${CHIBIOS}/os/rt/src/chsem.c)
    endif(CH_CFG_USE_SEMAPHORES)

    getconfig(CH_CFG_USE_MUTEXES chconf.h)
    if(CH_CFG_USE_MUTEXES)
        set(KERNSRC	${KERNSRC}
             ${CHIBIOS}/os/rt/src/chmtx.c)
    endif(CH_CFG_USE_MUTEXES)

    getconfig(CH_CFG_USE_CONDVARS chconf.h)
    if(CH_CFG_USE_CONDVARS)
        set(KERNSRC	${KERNSRC}
             ${CHIBIOS}/os/rt/src/chcond.c)
    endif(CH_CFG_USE_CONDVARS)

    getconfig(CH_CFG_USE_EVENTS chconf.h)
    if(CH_CFG_USE_EVENTS)
        set(KERNSRC	${KERNSRC}
             ${CHIBIOS}/os/rt/src/chevents.c)
    endif(CH_CFG_USE_EVENTS)

    getconfig(CH_CFG_USE_MESSAGES chconf.h)
    if(CH_CFG_USE_MESSAGES)
        set(KERNSRC	${KERNSRC}
             ${CHIBIOS}/os/rt/src/chmsg.c)
    endif(CH_CFG_USE_MESSAGES)

    getconfig(CH_CFG_USE_MAILBOXES chconf.h)
    if(CH_CFG_USE_MAILBOXES)
        set(KERNSRC	${KERNSRC}
             ${CHIBIOS}/os/rt/src/chmboxes.c)
    endif(CH_CFG_USE_MAILBOXES)

    getconfig(CH_CFG_USE_QUEUES chconf.h)
    if(CH_CFG_USE_QUEUES)
        set(KERNSRC	${KERNSRC}
             ${CHIBIOS}/os/rt/src/chqueues.c)
    endif(CH_CFG_USE_QUEUES)

    getconfig(CH_CFG_USE_MEMCORE chconf.h)
    if(CH_CFG_USE_MEMCORE)
        set(KERNSRC	${KERNSRC}
             ${CHIBIOS}/os/rt/src/chmemcore.c)
    endif(CH_CFG_USE_MEMCORE)

    getconfig(CH_CFG_USE_HEAP chconf.h)
    if(CH_CFG_USE_HEAP)
        set(KERNSRC	${KERNSRC}
             ${CHIBIOS}/os/rt/src/chheap.c)
    endif(CH_CFG_USE_HEAP)

    getconfig(CH_CFG_USE_MEMPOOLS chconf.h)
    if(CH_CFG_USE_MEMPOOLS)
        set(KERNSRC	${KERNSRC}
             ${CHIBIOS}/os/rt/src/chmempools.c)
    endif(CH_CFG_USE_MEMPOOLS)


else(USE_SMART_BUILD)

    set(KERNSRC
            ${CHIBIOS}/os/rt/src/chsys.c
            ${CHIBIOS}/os/rt/src/chdebug.c
            ${CHIBIOS}/os/rt/src/chvt.c
            ${CHIBIOS}/os/rt/src/chschd.c
            ${CHIBIOS}/os/rt/src/chthreads.c
            ${CHIBIOS}/os/rt/src/chtm.c
            ${CHIBIOS}/os/rt/src/chstats.c
            ${CHIBIOS}/os/rt/src/chdynamic.c
            ${CHIBIOS}/os/rt/src/chregistry.c
            ${CHIBIOS}/os/rt/src/chsem.c
            ${CHIBIOS}/os/rt/src/chmtx.c
            ${CHIBIOS}/os/rt/src/chcond.c
            ${CHIBIOS}/os/rt/src/chevents.c
            ${CHIBIOS}/os/rt/src/chmsg.c
            ${CHIBIOS}/os/rt/src/chmboxes.c
            ${CHIBIOS}/os/rt/src/chqueues.c
            ${CHIBIOS}/os/rt/src/chmemcore.c
            ${CHIBIOS}/os/rt/src/chheap.c
            ${CHIBIOS}/os/rt/src/chmempools.c)
endif(USE_SMART_BUILD)

set(KERNINC
        ${CHIBIOS}/os/rt/include)

################### PORT #########################
#include ${CHIBIOS}/os/rt/ports/ARMCMx/compilers/GCC/mk/port_v7m.mk
set(PORTSRC
        ${CHIBIOS}/os/rt/ports/ARMCMx/chcore.c
        ${CHIBIOS}/os/rt/ports/ARMCMx/chcore_v7m.c)

set(PORTASM
        ${CHIBIOS}/os/rt/ports/ARMCMx/compilers/GCC/chcoreasm_v7m.s)

set(PORTINC
        ${CHIBIOS}/os/rt/ports/ARMCMx
        ${CHIBIOS}/os/rt/ports/ARMCMx/compilers/GCC)


################### HWCONF #########################
#include hwconf/hwconf.mk
set(HWSRC
        hwconf/hw_40.c
        hwconf/hw_45.c
        hwconf/hw_r2.c
        hwconf/hw_46.c
        hwconf/hw_48.c
        hwconf/hw_49.c
        hwconf/hw_410.c
        hwconf/hw_victor_r1a.c)

set(HWINC
        hwconf)

################### APPLICATIONS #########################
#include applications/applications.mk
set(APPSRC
        applications/app.c
        applications/app_ppm.c
        applications/app_adc.c
        applications/app_sten.c
        applications/app_uartcomm.c
        applications/app_nunchuk.c)

set(APPINC
        applications)

################### NRF #########################
#include nrf/nrf.mk
set(NRFSRC
        nrf/spi_sw.c
        nrf/rf.c
        nrf/rfhelp.c
        nrf/nrf_driver.c)

set(NRFINC
        nrf)


# C sources that can be compiled in ARM or THUMB mode depending on the global
# setting.
set(CSRC
        ${STARTUPSRC}
        ${KERNSRC}
        ${PORTSRC}
        ${OSALSRC}
        ${HALSRC}
        ${PLATFORMSRC}
        ${BOARDSRC}
        ${CHIBIOS}/os/hal/lib/streams/chprintf.c
        ${CHIBIOS}/os/various/syscalls.c
        main.c
        comm_usb_serial.c
        irq_handlers.c
        buffer.c
        comm_usb.c
        crc.c
        digital_filter.c
        ledpwm.c
        mcpwm.c
        servo_dec.c
        utils.c
        servo.c
        servo_simple.c
        packet.c
        terminal.c
        conf_general.c
        eeprom.c
        commands.c
        timeout.c
        comm_can.c
        ws2811.c
        led_external.c
        encoder.c
        flash_helper.c
        mc_interface.c
        mcpwm_foc.c
        ${HWSRC}
        ${APPSRC}
        ${NRFSRC})



# C++ sources that can be compiled in ARM or THUMB mode depending on the global
# setting.
set(CPPSRC "")

# C sources to be compiled in ARM mode regardless of the global setting.
# NOTE: Mixing ARM and THUMB mode enables the -mthumb-interwork compiler
#       option that results in lower performance and larger code size.
set(ACSRC "")

# C++ sources to be compiled in ARM mode regardless of the global setting.
# NOTE: Mixing ARM and THUMB mode enables the -mthumb-interwork compiler
#       option that results in lower performance and larger code size.
set(ACPPSRC "")

# C sources to be compiled in THUMB mode regardless of the global setting.
# NOTE: Mixing ARM and THUMB mode enables the -mthumb-interwork compiler
#       option that results in lower performance and larger code size.
set(TCSRC "")

# C sources to be compiled in THUMB mode regardless of the global setting.
# NOTE: Mixing ARM and THUMB mode enables the -mthumb-interwork compiler
#       option that results in lower performance and larger code size.
set(TCPPSRC "")

# List ASM source files here
set(ASMSRC  ${STARTUPASM} ${PORTASM} ${OSALASM})

#Include dirs
set(INCDIR
        ${STARTUPINC} ${KERNINC} ${PORTINC} ${OSALINC}
        ${HALINC} ${PLATFORMINC} ${BOARDINC} ${TESTINC}
        ${CHIBIOS}/os/various
        ${CHIBIOS}/os/hal/lib/streams
        mcconf
        appconf
        ${HWINC}
        ${APPINC}
        ${NRFINC})


## Compiler settings
##
#
#MCU  = cortex-m4
#
##TRGT = arm-elf-
#TRGT = arm-none-eabi-
#CC   = ${TRGT}gcc
#CPPC = ${TRGT}g++
## Enable loading with g++ only if you need C++ runtime support.
## NOTE: You can use C++ even without C++ support if you are careful. C++
##       runtime support makes code size explode.
#LD   = ${TRGT}gcc
##LD   = ${TRGT}g++
#CP   = ${TRGT}objcopy
#AS   = ${TRGT}gcc -x assembler-with-cpp
#AR   = ${TRGT}ar
#OD   = ${TRGT}objdump
#SZ   = ${TRGT}size
#HEX  = ${CP} -O ihex
#BIN  = ${CP} -O binary

set(MCU cortex-m4)

set(TRGT arm-none-eabi-)

#set(CMAKE_SYSTEM_NAME Linux)
# which compilers to use for C and C++
set(CMAKE_C_COMPILER ${TRGT}gcc)
set(CMAKE_ASM_COMPILER ${TRGT}gcc)
set(CMAKE_ASM_FLAGS "-x assembler-with-cpp")
set(CMAKE_AR ${TRGT}ar)
set(CMAKE_RANLIB ${TRGT}ranlib)
set(CMAKE_CXX_COMPILER ${TRGT}g++)

message(WARNING "${CMAKE_CXX_COMPILER}")
# here is the target environment located
set(CMAKE_FIND_ROOT_PATH  /usr/arm-none-eabi)

# adjust the default behaviour of the FIND_XXX() commands:
# search headers and libraries in the target environment, search
# programs in the host environment
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)

set(CP_COMMAND ${TGRT}objcopy)
set(BIN_COMMAND ${CP_COMMAND} -O binary)
set(HEX_COMMAND ${CP_COMMAND} -O ihex)




# ARM-specific options here
set (AOPT "")

# THUMB-specific options here
set(TOPT -mthumb -DTHUMB )

# Define C warning options here
set(CWARN -Wall -Wextra -Wundef -Wstrict-prototypes)

# Define C++ warning options here
set(CPPWARN -Wall -Wextra -Wundef)

#
# Compiler settings
##############################################################################

##############################################################################
# Start of user section
#

# List all user C define here, like -D_DEBUG=1
set(UDEFS "")

# Define ASM defines here
set(UADEFS "")

# List all user directories here
set(UINCDIR "")

# List the user directory to look for the libraries here
set(ULIBDIR "")

## List all user libraries here
#ULIBS = -lm
set(ULIBS m)

#
#ifeq (${USE_FWLIB},yes)
#include ${CHIBIOS}/ext/stdperiph_stm32f4/stm32lib.mk
#CSRC += ${STM32SRC}
#INCDIR += ${STM32INC}
#USE_OPT += -DUSE_STDPERIPH_DRIVER
#endif
if(USE_FWLIB)
    #include ${CHIBIOS}/ext/stdperiph_stm32f4/stm32lib.mk
    # STM32F4 STDPERIPH files.
    set(STM32SRC
            ${CHIBIOS}/ext/stdperiph_stm32f4/src/misc.c
            ${CHIBIOS}/ext/stdperiph_stm32f4/src/stm32f4xx_adc.c
            ${CHIBIOS}/ext/stdperiph_stm32f4/src/stm32f4xx_dma.c
            ${CHIBIOS}/ext/stdperiph_stm32f4/src/stm32f4xx_exti.c
            ${CHIBIOS}/ext/stdperiph_stm32f4/src/stm32f4xx_flash.c
            ${CHIBIOS}/ext/stdperiph_stm32f4/src/stm32f4xx_rcc.c
            ${CHIBIOS}/ext/stdperiph_stm32f4/src/stm32f4xx_syscfg.c
            ${CHIBIOS}/ext/stdperiph_stm32f4/src/stm32f4xx_tim.c
            ${CHIBIOS}/ext/stdperiph_stm32f4/src/stm32f4xx_wwdg.c)

    set(STM32INC ${CHIBIOS}/ext/stdperiph_stm32f4/inc)

    set(CSRC ${CSRC} ${STM32SRC})
    set(INCDIR ${INCDIR} ${STM32INC})
    set(USE_OPT ${USE_OPT} -DUSE_STDPERIPH_DRIVER)
endif()


############## make upload ####################
## Program
#upload: build/${PROJECT}.bin
#   #qstlink2 --cli --erase --write build/${PROJECT}.bin
#   openocd -f interface/stlink-v2.cfg -c "set WORKAREASIZE 0x2000" -f target/stm32f4x_stlink.cfg -c "program build/${PROJECT}.elf verify reset"
#   #openocd -f board/stm32f4discovery.cfg -c "reset_config trst_only combined" -c "program build/${PROJECT}.elf verify reset exit" # For openocd 0.9

############## BIN creation ###################
#build/${PROJECT}.bin: build/${PROJECT}.elf
#    ${BIN} build/${PROJECT}.elf build/${PROJECT}.bin


############## ELF creation ###################
#%.elf: ${OBJS} ${LDSCRIPT}
#ifeq (${USE_VERBOSE_COMPILE},yes)
#@echo
#${LD} ${OBJS} ${LDFLAGS} ${LIBS} -o $@
#else
#@echo Linking $@
#@${LD} ${OBJS} ${LDFLAGS} ${LIBS} -o $@
#endif

############### OBJS #########################

#creating directories
#${OBJS}: | ${BUILDDIR} ${OBJDIR} ${LSTDIR}


#OBJS	  = ${ASMXOBJS} ${ASMOBJS} ${ACOBJS} ${TCOBJS} ${ACPPOBJS} ${TCPPOBJS}

#$(ASMXOBJS) : $(OBJDIR)/%.o : %.S Makefile
#ifeq ($(USE_VERBOSE_COMPILE),yes)
#@echo
#$(CC) -c $(ASXFLAGS) $(TOPT) -I. $(IINCDIR) $< -o $@
#else
#@echo Compiling $(<F)
#@$(CC) -c $(ASXFLAGS) $(TOPT) -I. $(IINCDIR) $< -o $@
#endif

#$(ASMOBJS) : $(OBJDIR)/%.o : %.s Makefile
#ifeq ($(USE_VERBOSE_COMPILE),yes)
#@echo
#$(AS) -c $(ASFLAGS) -I. $(IINCDIR) $< -o $@
#else
#@echo Compiling $(<F)
#@$(AS) -c $(ASFLAGS) -I. $(IINCDIR) $< -o $@
#endif

#$(ACOBJS) : $(OBJDIR)/%.o : %.c Makefile
#ifeq ($(USE_VERBOSE_COMPILE),yes)
#@echo
#$(CC) -c $(CFLAGS) $(AOPT) -I. $(IINCDIR) $< -o $@
#else
#@echo Compiling $(<F)
#@$(CC) -c $(CFLAGS) $(AOPT) -I. $(IINCDIR) $< -o $@
#endif

#$(TCOBJS) : $(OBJDIR)/%.o : %.c Makefile
#ifeq ($(USE_VERBOSE_COMPILE),yes)
#@echo
#$(CC) -c $(CFLAGS) $(TOPT) -I. $(IINCDIR) $< -o $@
#else
#@echo Compiling $(<F)
#@$(CC) -c $(CFLAGS) $(TOPT) -I. $(IINCDIR) $< -o $@
#endif

#$(ACPPOBJS) : $(OBJDIR)/%.o : %.cpp Makefile
#ifeq ($(USE_VERBOSE_COMPILE),yes)
#@echo
#$(CPPC) -c $(CPPFLAGS) $(AOPT) -I. $(IINCDIR) $< -o $@
#else
#@echo Compiling $(<F)
#@$(CPPC) -c $(CPPFLAGS) $(AOPT) -I. $(IINCDIR) $< -o $@
#endif

#$(TCPPOBJS) : $(OBJDIR)/%.o : %.cpp Makefile
#ifeq ($(USE_VERBOSE_COMPILE),yes)
#@echo
#$(CPPC) -c $(CPPFLAGS) $(TOPT) -I. $(IINCDIR) $< -o $@
#else
#@echo Compiling $(<F)
#@$(CPPC) -c $(CPPFLAGS) $(TOPT) -I. $(IINCDIR) $< -o $@
#endif


#TODO: separate each target





#RULESPATH = ${CHIBIOS}/os/common/ports/ARMCMx/compilers/GCC
set(RULESPATH $(CHIBIOS)/os/common/ports/ARMCMx/compilers/GCC)

#include ${RULESPATH}/rules.mk

# Compiler options
set(OPT ${USE_OPT})
set(COPT ${USE_COPT})
set(CPPOPT ${USE_CPPOPT})

# Garbage collection
if(USE_LINK_GC)
    set(OPT ${OPT} -ffunction-sections -fdata-sections -fno-common)
    set(LDOPT ,--gc-sections)
else()
    set(LDOPT "")
endif()


# Linker extra options
if (NOT "${USE_LDOPT}" STREQUAL "")
    set(LDOPT ${LDOPT},${USE_LDOPT})
endif()


# Link time optimizations
if (USE_LTO)
    set(OPT ${OPT} -flto)
endif()

# FPU-related options
if ("${USE_FPU}" STREQUAL "")
    set(USE_FPU "no")
endif()

if (NOT "${USE_FPU}" STREQUAL "no")
    set(OPT ${OPT} -mfloat-abi=${USE_FPU} -mfpu=fpv4-sp-d16 -fsingle-precision-constant)
    #TODO: defines
    set(DDEFS ${DDEFS} -DCORTEX_USE_FPU=TRUE)
    set(DADEFS ${DADEFS} -DCORTEX_USE_FPU=TRUE)
else()
    set(DDEFS ${DDEFS} -DCORTEX_USE_FPU=FALSE)
    set(DADEFS ${DADEFS} -DCORTEX_USE_FPU=FALSE)
endif()


# Process stack size
if("${USE_PROCESS_STACKSIZE}" STREQUAL "")
    set(LDOPT ${LDOPT},--defsym=__process_stack_size__=0x400)
else()
    set(LDOPT ${LDOPT},--defsym=__process_stack_size__=${USE_PROCESS_STACKSIZE})
endif()

# Exceptions stack size

if("${USE_EXCEPTIONS_STACKSIZE}" STREQUAL "")
    set(LDOPT ${LDOPT},--defsym=__main_stack_size__=0x400)
else()
    set(LDOPT ${LDOPT},--defsym=__main_stack_size__=${USE_EXCEPTIONS_STACKSIZE})
endif()

## Output directory and files
#ifeq (${BUILDDIR},)
#BUILDDIR = build
#endif
set(BUILDDIR ${CMAKE_CURRENT_SOURCE_DIR}/build)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${BUILDDIR})


if (USE_THUMB)
    set(TCSRC ${TCSRC} ${CSRC})
    set(TCPPSRC ${TCPPSRC} ${CPPSRC})
else(USE_THUMB)
    set(ACSRC ${ACSRC} ${CSRC})
    set(ACPPSRC ${ACPPSRC} ${CPPSRC})
endif(USE_THUMB)


set(ASRC ${ACSRC} ${ACPPSRC})
set(TSRC ${TCSRC} ${TCPPSRC})

# Various directories
set(OBJDIR ${BUILDDIR}/obj)
set(LSTDIR ${BUILDDIR}/lst)



#ACOBJS    = $(addprefix ${OBJDIR}/, $(notdir $(ACSRC:.c=.o)))
set(ACOBJS ${ACSRC})
#ACPPOBJS  = $(addprefix ${OBJDIR}/, $(notdir $(ACPPSRC:.cpp=.o)))
set(ACPPOBJS ${ACPPSRC})
#TCOBJS    = $(addprefix ${OBJDIR}/, $(notdir $(TCSRC:.c=.o)))
set(TCOBJS ${TCSRC})
#TCPPOBJS  = $(addprefix ${OBJDIR}/, $(notdir $(TCPPSRC:.cpp=.o)))
set(TCPPOBJS ${TCPPSRC})
#ASMOBJS   = $(addprefix ${OBJDIR}/, $(notdir $(ASMSRC:.s=.o)))
set(ASMOBJS ${ASMSRC})
#ASMXOBJS  = $(addprefix ${OBJDIR}/, $(notdir $(ASMXSRC:.S=.o)))
set(ASMXOBJS ${ASMXSRC})


#add_library(bldc_asmx ${ASMXOBJS})
#target_compile_options(bldc_asmx ${CMAKE_ASM_FLAGS})

#OBJS	  = ${ASMXOBJS} ${ASMOBJS} ${ACOBJS} ${TCOBJS} ${ACPPOBJS} ${TCPPOBJS}
set(OBJS ${ASMXOBJS} ${ASMOBJS} ${ACOBJS} ${TCOBJS} ${ACPPOBJS} ${TCPPOBJS})


# Paths
#IINCDIR   = $(patsubst %,-I%,${INCDIR} ${DINCDIR} ${UINCDIR})
set(IINCDIR ${INCDIR} ${DINCDIR} ${UINCDIR} ./)
#LLIBDIR   = $(patsubst %,-L%,${DLIBDIR} ${ULIBDIR})
set(LLIBDIR ${DLIBDIR} ${ULIBDIR})

## Macros
#DEFS      = ${DDEFS} ${UDEFS}
set(DEFS ${DDEFS} ${UDEFS})
#ADEFS 	  = ${DADEFS} ${UADEFS}
set(ADEFS ${DADEFS} ${UADEFS})

## Libs
#LIBS      = ${DLIBS} ${ULIBS}
set(LIBS ${DLIBS} ${ULIBS})

## Various settings
#MCFLAGS   = -mcpu=${MCU}
#ODFLAGS	  = -x --syms
#ASFLAGS   = ${MCFLAGS} -Wa,-amhls=${LSTDIR}/$(notdir $(<:.s=.lst)) ${ADEFS}
#ASXFLAGS  = ${MCFLAGS} -Wa,-amhls=${LSTDIR}/$(notdir $(<:.S=.lst)) ${ADEFS}
#CFLAGS    = ${MCFLAGS} ${OPT} ${COPT} ${CWARN} -Wa,-alms=${LSTDIR}/$(notdir $(<:.c=.lst)) ${DEFS}
#CPPFLAGS  = ${MCFLAGS} ${OPT} ${CPPOPT} ${CPPWARN} -Wa,-alms=${LSTDIR}/$(notdir $(<:.cpp=.lst)) ${DEFS}
#LDFLAGS   = ${MCFLAGS} ${OPT} -nostartfiles ${LLIBDIR} -Wl,-Map=${BUILDDIR}/${PROJECT}.map,--cref,--no-warn-mismatch,--library-path=${RULESPATH},--script=${LDSCRIPT}${LDOPT}

set(MCFLAGS -mcpu=${MCU})
set(ODFLAGS	-x --syms)
set(ASFLAGS ${MCFLAGS} ${ADEFS})
set(ASXFLAGS ${MCFLAGS} ${ADEFS})
set(CFLAGS ${MCFLAGS} ${OPT} ${COPT} ${CWARN} ${DEFS})
set(CPPFLAGS ${MCFLAGS} ${OPT} ${CPPOPT} ${CPPWARN} ${DEFS})
set(LDFLAGS ${MCFLAGS} ${OPT} -nostartfiles ${LLIBDIR} -Wl,-Map=${BUILDDIR}/${PROJECT}.map,--cref,--no-warn-mismatch,--library-path=${RULESPATH},--script=${LDSCRIPT}${LDOPT})

set_compile_flags(ASMOBJS "-Wa,-amhls=${LSTDIR}/")
set_compile_flags(ASMXOBJS "-Wa,-amhls=${LSTDIR}/")
set_compile_flags(TCOBJS "-Wa,-amhls=${LSTDIR}/")
set_compile_flags(TCPPOBJS "-Wa,-amhls=${LSTDIR}/")
#message(WARNING "tcobjs: ${TCOBJS}")


# Thumb interwork enabled only if needed because it kills performance.
if(NOT "${TSRC}" STREQUAL "")
    set(CFLAGS ${CFLAGS} -DTHUMB_PRESENT)
    set(CPPFLAGS ${CPPFLAGS} -DTHUMB_PRESENT)
    set(ASFLAGS  ${ASFLAGS} -DTHUMB_PRESENT)
    if(NOT "${ASRC}" STREQUAL "")
    # Mixed ARM and THUMB mode.
        set(CFLAGS   ${CFLAGS} -mthumb-interwork)
        set(CPPFLAGS ${CPPFLAGS} -mthumb-interwork)
        set(ASFLAGS  ${ASFLAGS} -mthumb-interwork)
        set(LDFLAGS  ${LDFLAGS} -mthumb-interwork)
    else()
        # Pure THUMB mode, THUMB C code cannot be called by ARM asm code directly.
        set(CFLAGS   ${CFLAGS} -mno-thumb-interwork -DTHUMB_NO_INTERWORKING)
        set(CPPFLAGS ${CPPFLAGS} -mno-thumb-interwork -DTHUMB_NO_INTERWORKING)
        set(ASFLAGS  ${ASFLAGS} -mno-thumb-interwork -DTHUMB_NO_INTERWORKING -mthumb)
        set(LDFLAGS  ${LDFLAGS} -mno-thumb-interwork -mthumb)
    endif()
else()
    # Pure ARM mode
    set(CFLAGS   ${CFLAGS} -mno-thumb-interwork)
    set(CPPFLAGS ${CPPFLAGS} -mno-thumb-interwork)
    set(ASFLAGS  ${ASFLAGS} -mno-thumb-interwork)
    set(LDFLAGS  ${LDFLAGS} -mno-thumb-interwork)
endif()

add_custom_target(build_dir ALL COMMAND ${CMAKE_COMMAND} -E make_directory ${BUILDDIR})
add_custom_target(obj_dir ALL COMMAND ${CMAKE_COMMAND} -E make_directory ${OBJDIR})
add_custom_target(lst_dir ALL COMMAND ${CMAKE_COMMAND} -E make_directory ${LSTDIR})



if (NOT "${ACPPOBJS}" STREQUAL "")
    add_library(acpp_objs OBJECT ${ACPPOBJS})
#    add_dependencies(acpp_objs build_dir obj_dir lst_dir)
    target_include_directories(acpp_objs PUBLIC ${IINCDIR})
    target_compile_options(acpp_objs PUBLIC ${CPPFLAGS} ${AOPT})
    target_compile_definitions(acpp_objs PUBLIC "")
    set(BIN_TARGETS ${BIN_TARGETS} $<TARGET_OBJECTS:acpp_objs>)
endif()

if (NOT "${TCPPOBJS}" STREQUAL "")
    add_library(tcpp_objs OBJECT ${TCPPOBJS})
#    add_dependencies(tcpp_objs build_dir obj_dir lst_dir)
    target_include_directories(tcpp_objs PUBLIC ${IINCDIR})
    target_compile_options(tcpp_objs PUBLIC ${CPPFLAGS} ${TOPT})
    target_compile_definitions(tcpp_objs PUBLIC "")
    set(BIN_TARGETS ${BIN_TARGETS} $<TARGET_OBJECTS:tcpp_objs>)
endif()

if (NOT "${ACOBJS}" STREQUAL "")
    add_library(ac_objs OBJECT ${ACOBJS})
#    add_dependencies(ac_objs build_dir obj_dir lst_dir)
    target_include_directories(ac_objs PUBLIC ${IINCDIR})
    target_compile_options(ac_objs PUBLIC ${CFLAGS} ${AOPT})
    target_compile_definitions(ac_objs PUBLIC "")
    set(BIN_TARGETS ${BIN_TARGETS} $<TARGET_OBJECTS:ac_objs>)

endif()

if (NOT "${TCOBJS}" STREQUAL "")
#    message(WARNING "tc_objs ${TCOBJS}")
    add_library(tc_objs OBJECT ${TCOBJS})
#    add_dependencies(tc_objs build_dir obj_dir lst_dir)
    target_include_directories(tc_objs PUBLIC ${IINCDIR})
    target_compile_options(tc_objs PUBLIC ${CFLAGS} ${TOPT})
    target_compile_definitions(tc_objs PUBLIC "")
    set(BIN_TARGETS ${BIN_TARGETS} $<TARGET_OBJECTS:tc_objs>)

endif()

if (NOT "${ASMOBJS}" STREQUAL "")
    add_library(asm_objs OBJECT ${ASMOBJS})
#    add_dependencies(asm_objs build_dir obj_dir lst_dir)
    target_include_directories(asm_objs PUBLIC ${IINCDIR})
    target_compile_options(asm_objs PUBLIC ${ASFLAGS})
    target_compile_definitions(asm_objs PUBLIC "")
    set(BIN_TARGETS ${BIN_TARGETS} $<TARGET_OBJECTS:asm_objs>)

endif()

if (NOT "${ASMXOBJS}" STREQUAL "")
    add_library(asmx_objs OBJECT ${ASMXOBJS})
#    add_dependencies(asmx_objs build_dir obj_dir lst_dir)
    target_include_directories(asmx_objs PUBLIC ${IINCDIR})
    target_compile_options(asmx_objs PUBLIC ${ASXFLAGS} ${TOPT})
    target_compile_definitions(asmx_objs PUBLIC "")
    set(BIN_TARGETS ${BIN_TARGETS} $<TARGET_OBJECTS:asmx_objs>)

endif()

#set(OBJS ${ASMXOBJS} ${ASMOBJS} ${ACOBJS} ${TCOBJS} ${ACPPOBJS} ${TCPPOBJS})

if (NOT "${ACOBJS}" STREQUAL "")
    message(WARNING "ACOBJS ${ACOBJS}")
    add_library(ac_objs OBJECT ${ACOBJS})
#    add_dependencies(ac_objs build_dir obj_dir lst_dir)
    target_include_directories(ac_objs PUBLIC ${IINCDIR})
    target_compile_options(ac_objs PUBLIC ${CFLAGS} ${AOPT})
    target_compile_definitions(ac_objs PUBLIC "")
    set(BIN_TARGETS ${BIN_TARGETS} $<TARGET_OBJECTS:ac_objs>)

endif()
function(JOIN VALUES GLUE OUTPUT)
    string (REPLACE ";" "${GLUE}" _TMP_STR "${VALUES}")
    set (${OUTPUT} "${_TMP_STR}" PARENT_SCOPE)
endfunction()

JOIN("${LDFLAGS}" " " LDFLAGS_SPACE)

set(CMAKE_SHARED_LIBRARY_LINK_C_FLAGS "")
add_executable(bldc ${BIN_TARGETS})
#target_compile_options(bldc PUBLIC )
#target_include_directories(bldc PUBLIC ${INCDIR} ${UINCDIR} ./)
target_link_libraries(bldc PUBLIC ${LIBS})
#target_compile_definitions(bldc PUBLIC ${UDEFS} ${UADEFS})
#target_compile_options(bldc PUBLIC ${AOPT} ${TOPT} ${CWARN} ${CPPWARN} )
set_target_properties(bldc PROPERTIES LINKER_LANGUAGE "C")
set_target_properties(bldc PROPERTIES LINK_FLAGS ${LDFLAGS_SPACE})
#link_directories(${ULIBDIR})
