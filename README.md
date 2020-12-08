# cubeide-sd-card

This project is designed as an example of a STM32CubeIDE-generated system with FatFs middleware controlling
an SPI-connected MMC/SD memory card.

The project was initially created in CubeIDE, and then code written by ChaN was ported to fit the CubeIDE generator.

This is a remake of the original project provided [here](https://github.com/kiwih/cubemx-mmc-sd-card) for CubeIDE instead of CubeMX.

I also wrote a [blog post](https://01001000.xyz/2020-08-09-Tutorial-STM32CubeIDE-SD-card/) which accompanies this repository and explains how to use it.

## Adoption Guide

1. Copy the driver files `FATFS/Target/user_diskio_spi.c` and `FATFS/Target/user_diskio_spi.h` into your CubeIDE project which has been configured to use FatFS.
2. In `FATFS/Target/user_spi.c` add `#include "user_diskio_spi.h"`
3. In `FATFS/Target/user_spi.c` call `USER_SPI_initialize(...)` in `USER_initialize(...)`, `USER_SPI_status(...)` in `USER_status(...)`, `USER_SPI_read(...)` in `USER_read(...)`, `USER_SPI_write(...)` in `USER_write(...)`, and `USER_SPI_ioctl(...)` in `USER_ioctl(...)`.
  - We embed in this manner to avoid conflicts with the CubeIDE code generator. Make sure these function calls are inside the `USER` comment blocks.
4. In `main.h` ensure that you have `#define`s for `SD_SPI_HANDLE` (e.g. hspi2), `SD_CS_GPIO_Port`, and `SD_CS_Pin`.
5. Double check what would be suitable high/low speeds for your SPI driver and the prescalars you will need to use for the SPI port. Configure these at in `user_diskio_spi.c:`
```c
//(Note that the _256 is used as a mask to clear the prescalar bits as it provides binary 111 in the correct position)

#define FCLK_SLOW() { MODIFY_REG(SD_SPI_HANDLE.Instance->CR1, SPI_BAUDRATEPRESCALER_256, SPI_BAUDRATEPRESCALER_128); }	/* Set SCLK = slow, approx 280 KBits/s*/
#define FCLK_FAST() { MODIFY_REG(SD_SPI_HANDLE.Instance->CR1, SPI_BAUDRATEPRESCALER_256, SPI_BAUDRATEPRESCALER_8); }	/* Set SCLK = fast, approx 4.5 MBits/s */

```


You can now call the FatFS functions from your `main()`. For more detailed explanation of this, with examples, check out [the blog post](https://01001000.xyz/2020-08-09-Tutorial-STM32CubeIDE-SD-card/).
