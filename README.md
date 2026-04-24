# About this Repo
This is an template of using cmake to build the stm32 related project.

For this template, STM32F103VET6 have been used.

Serial Wire Enabled
PC13 Output

---

# Tested Environment
* GCC 14.3.1 arm-none-eabi

---

# File Structure
```bash
.
├── cmake
│   └── stm32-toolchain.cmake
├── CMakeLists.txt
├── Drivers
│   ├── CMSIS
│   └── STM32F1xx_HAL_Driver
├── linker
│   └── STM32F103VETx_FLASH.ld
├── openocd.cfg
├── README.md
├── src
│   ├── main.c
│   ├── stm32f1xx_hal_conf.h
│   ├── stm32f1xx_hal_msp.c
│   ├── stm32f1xx_it.c
│   └── system_stm32f1xx.c
└── startup
    └── startup_stm32f103xe.s
```

---

# Tutorial for recreating this repo
1. Create below file structure

2. Get the HAL Driver Library
    > git clone --recurse-submodules https://github.com/STMicroelectronics/STM32CubeF1.git

3. And copy below files from STM32CubeF1
* `STM32CubeF1/Drivers/STM32F1xx_HAL_Driver/` -> `Drivers/STM32F1xx_HAL_Driver/`
* `STM32CubeF1/Drivers/CMSIS/Device/ST/STM32F1xx/` -> `Drivers/CMSIS/Device/ST/STM32F1xx/`
* `STM32CubeF1/Drivers/CMSIS/Include/` -> `Drivers/CMSIS/Include/`

<br/>

4. Create `cmake/stm32-toolchain.cmake`
    <details>
    <summary>Click to toggle contents</summary>

    ```
    set(CMAKE_SYSTEM_NAME Generic)
    set(CMAKE_SYSTEM_PROCESSOR arm)

    # Specify the cross compiler
    set(CMAKE_C_COMPILER arm-none-eabi-gcc)
    set(CMAKE_CXX_COMPILER arm-none-eabi-g++)
    set(CMAKE_ASM_COMPILER arm-none-eabi-gcc)
    set(CMAKE_AR arm-none-eabi-ar)
    set(CMAKE_OBJCOPY arm-none-eabi-objcopy)
    set(CMAKE_OBJDUMP arm-none-eabi-objdump)
    set(CMAKE_SIZE arm-none-eabi-size)

    # Processor specific flags
    set(CPU_FLAGS "-mcpu=cortex-m3 -mthumb -mfloat-abi=soft")

    set(CMAKE_C_FLAGS_INIT "${CPU_FLAGS}")
    set(CMAKE_CXX_FLAGS_INIT "${CPU_FLAGS}")
    set(CMAKE_ASM_FLAGS_INIT "${CPU_FLAGS}")

    # Don't try to run the linker during compiler test
    set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)
    ```
    </details>
    <br/>

5. Create `src/stm32f1xx_hal_conf.h`
    <details>
    <summary>Click to toggle contents</summary>
    
    ```
    #ifndef __STM32F1xx_HAL_CONF_H
    #define __STM32F1xx_HAL_CONF_H

    #ifdef __cplusplus
    extern "C" {
    #endif

    /* Module Selection */
    #define HAL_MODULE_ENABLED
    #define HAL_CORTEX_MODULE_ENABLED
    #define HAL_DMA_MODULE_ENABLED
    #define HAL_FLASH_MODULE_ENABLED
    #define HAL_GPIO_MODULE_ENABLED
    #define HAL_PWR_MODULE_ENABLED
    #define HAL_RCC_MODULE_ENABLED
    #define HAL_UART_MODULE_ENABLED
    // Add other modules as needed

    /* Oscillator Values */
    #define HSE_VALUE    8000000U
    #define HSE_STARTUP_TIMEOUT    100U
    #define HSI_VALUE    8000000U
    #define LSI_VALUE    40000U
    #define LSE_VALUE    32768U
    #define LSE_STARTUP_TIMEOUT    5000U

    /* System Configuration */
    #define VDD_VALUE    3300U
    #define TICK_INT_PRIORITY    0x0FU
    #define USE_RTOS    0U
    #define PREFETCH_ENABLE    1U

    /* Assert macro */
    #ifdef  USE_FULL_ASSERT
    #define assert_param(expr) ((expr) ? (void)0U : assert_failed((uint8_t *)__FILE__, __LINE__))
    void assert_failed(uint8_t* file, uint32_t line);
    #else
    #define assert_param(expr) ((void)0U)
    #endif

    /* HAL includes */
    #include "stm32f1xx_hal_rcc.h"
    #include "stm32f1xx_hal_gpio.h"
    #include "stm32f1xx_hal_dma.h"
    #include "stm32f1xx_hal_cortex.h"
    #include "stm32f1xx_hal_flash.h"
    #include "stm32f1xx_hal_pwr.h"
    #include "stm32f1xx_hal_uart.h"
    // Include other HAL modules you enabled

    #ifdef __cplusplus
    }
    #endif

    #endif /* __STM32F1xx_HAL_CONF_H */
    ```
    </details>
    <br/>

6. Create `src/main.c`
    <details>
        <summary>Click to toggle contents</summary>
        
    ```
    #include "stm32f1xx_hal.h"

    void SystemClock_Config(void);
    static void GPIO_Init(void);

    int main(void)
    {
        HAL_Init();
        SystemClock_Config();
        GPIO_Init();

        while (1)
        {
            HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_0);
            HAL_Delay(500);
        }
    }

    void SystemClock_Config(void)
    {
        RCC_OscInitTypeDef RCC_OscInitStruct = {0};
        RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

        RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSE;
        RCC_OscInitStruct.HSEState = RCC_HSE_ON;
        RCC_OscInitStruct.HSEPredivValue = RCC_HSE_PREDIV_DIV1;
        RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
        RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSE;
        RCC_OscInitStruct.PLL.PLLMUL = RCC_PLL_MUL9; // 8MHz * 9 = 72MHz
        HAL_RCC_OscConfig(&RCC_OscInitStruct);

        RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK | RCC_CLOCKTYPE_SYSCLK
                                    | RCC_CLOCKTYPE_PCLK1 | RCC_CLOCKTYPE_PCLK2;
        RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
        RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
        RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV2;
        RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;
        HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_2);
    }

    static void GPIO_Init(void)
    {
        __HAL_RCC_GPIOB_CLK_ENABLE();

        // GPIOB, Output Pin Configuration
        GPIO_InitTypeDef GPIO_InitStruct = {0};
        GPIO_InitStruct.Pin = GPIO_PIN_0;
        GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
        GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
        HAL_GPIO_Init(GPIOB, &GPIO_InitStruct);
    }

    void SysTick_Handler(void)
    {
        HAL_IncTick();
    }
    ```
    </details>
    <br/>

7. Create `src/stm32f1xx_it.c`
    <details>
    <summary>Click to toggle contents</summary>

    ```
    #include "stm32f1xx_hal.h"

    void NMI_Handler(void) {}
    void HardFault_Handler(void) { while (1) {} }
    void MemManage_Handler(void) { while (1) {} }
    void BusFault_Handler(void) { while (1) {} }
    void UsageFault_Handler(void) { while (1) {} }
    void SVC_Handler(void) {}
    void DebugMon_Handler(void) {}
    void PendSV_Handler(void) {}
    ```
    </details>
    <br/>

8. Create `src/stm32f1xx_hal_msp.c`
    <details>
    <summary>Click to toggle contents</summary>

    ```
    #include "stm32f1xx_hal.h"

    void HAL_MspInit(void)
    {
        __HAL_RCC_AFIO_CLK_ENABLE();
        __HAL_RCC_PWR_CLK_ENABLE();
    }
    ```
    </details>
    </br>

9. Get Startup and Linker Files
    
    Copy from `STM32CubeF1`:

    * Startup file: `STM32CubeF1/Drivers/CMSIS/Device/ST/STM32F1xx/Source/Templates/gcc/startup_stm32f103xe.s` to `startup/`
    * Linker script: You can find examples in CubeIDE projects or create one. The linker script defines memory regions (512KB Flash, 64KB RAM for VET6).

    Create `linker/STM32F103VETx_FLASH.ld`
    <details>
    <summary>Click to toggle contents</summary>

    ```
    ENTRY(Reset_Handler)

    _estack = 0x20010000;  /* 64KB RAM */

    _Min_Heap_Size = 0x200;
    _Min_Stack_Size = 0x400;

    MEMORY
    {
    RAM (xrw)    : ORIGIN = 0x20000000, LENGTH = 64K
    FLASH (rx)   : ORIGIN = 0x8000000, LENGTH = 512K
    }

    SECTIONS
    {
    .isr_vector :
    {
        . = ALIGN(4);
        KEEP(*(.isr_vector))
        . = ALIGN(4);
    } >FLASH

    .text :
    {
        . = ALIGN(4);
        *(.text)
        *(.text*)
        *(.glue_7)
        *(.glue_7t)
        *(.eh_frame)
        KEEP (*(.init))
        KEEP (*(.fini))
        . = ALIGN(4);
        _etext = .;
    } >FLASH

    .rodata :
    {
        . = ALIGN(4);
        *(.rodata)
        *(.rodata*)
        . = ALIGN(4);
    } >FLASH

    .ARM.extab :
    {
        *(.ARM.extab* .gnu.linkonce.armextab.*)
    } >FLASH

    .ARM :
    {
        __exidx_start = .;
        *(.ARM.exidx*)
        __exidx_end = .;
    } >FLASH

    .preinit_array :
    {
        PROVIDE_HIDDEN (__preinit_array_start = .);
        KEEP (*(.preinit_array*))
        PROVIDE_HIDDEN (__preinit_array_end = .);
    } >FLASH

    .init_array :
    {
        PROVIDE_HIDDEN (__init_array_start = .);
        KEEP (*(SORT(.init_array.*)))
        KEEP (*(.init_array*))
        PROVIDE_HIDDEN (__init_array_end = .);
    } >FLASH

    .fini_array :
    {
        PROVIDE_HIDDEN (__fini_array_start = .);
        KEEP (*(SORT(.fini_array.*)))
        KEEP (*(.fini_array*))
        PROVIDE_HIDDEN (__fini_array_end = .);
    } >FLASH

    _sidata = LOADADDR(.data);

    .data :
    {
        . = ALIGN(4);
        _sdata = .;
        *(.data)
        *(.data*)
        . = ALIGN(4);
        _edata = .;
    } >RAM AT> FLASH

    .bss :
    {
        _sbss = .;
        __bss_start__ = _sbss;
        *(.bss)
        *(.bss*)
        *(COMMON)
        . = ALIGN(4);
        _ebss = .;
        __bss_end__ = _ebss;
    } >RAM

    ._user_heap_stack :
    {
        . = ALIGN(8);
        PROVIDE ( end = . );
        PROVIDE ( _end = . );
        . = . + _Min_Heap_Size;
        . = . + _Min_Stack_Size;
        . = ALIGN(8);
    } >RAM

    /DISCARD/ :
    {
        libc.a ( * )
        libm.a ( * )
        libgcc.a ( * )
    }
    }
    ```
    </details>
    </br>

10. Copy `STM32CubeF1/Drivers/CMSIS/Device/ST/STM32F1xx/Source/Templates/system_stm32f1xx.c` -> `src/`
</br>

11. Build the project
    ```
    # Install ARM toolchain first
    # On Ubuntu: sudo apt-get install gcc-arm-none-eabi

    # Create build directory
    mkdir build
    cd build

    # Configure with CMake
    cmake ..

    # Build
    make

    # You'll get:
    # - stm32_project.elf (executable)
    # - stm32_project.hex (for flashing)
    # - stm32_project.bin (for flashing)
    ```

---

# License

MIT License

Feel free to use, modify, and distribute.

---

## 📫 Let's Connect

I'm always interested in collaborating on embedded systems projects, robotics competitions, or autonomous vehicle development!

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077b5?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/hang020713)

[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=for-the-badge&logo=github)](https://github.com/hang020713)

[![Email](https://img.shields.io/badge/Email-Contact-red?style=for-the-badge&logo=gmail)](mailto:hang020713@gmail.com)

---

<p align="right">(<a href="#readme-top">Back to Top</a>)</p>