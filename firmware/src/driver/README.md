# driver：STM32 片内外设驱动层（Driver）

放置 ADC、TIM、DMA、USART、I2C、SPI 等片内外设的项目驱动。

命名规则：

- 文件：`drv_<外设>.c/.h`
- 公开 API：`Drv_<Module>_...`

模板故意不预置固定 ADC/TIM/UART 配置，因为不同赛题的通道、触发源、DMA 和工作模式差异很大。
