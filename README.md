# STM32F103C8T6 电赛标准库工程模板

用于后续电子设计竞赛项目的 STM32F103C8T6 基础脚手架。

当前 `main` 固定为：

- MCU：STM32F103C8T6
- STM32F10x Standard Peripheral Library V3.5.0
- arm-none-eabi-gcc
- CMake + Ninja
- VSCode + Cortex-Debug
- ST-Link
- 不使用 CubeMX
- 不使用 HAL

模板只保留可跨题复用的工程基础设施。ADC、TIM、DMA、量程、电机等具体业务配置不预置，避免上一道题的实现污染下一道题。

## 目录分层

```text
.
├── .github/
│   └── workflows/
│       └── firmware-build.yml
├── .vscode/
├── firmware/
│   ├── Start/
│   │   ├── startup.cmake
│   │   ├── cmsis-compat.cmake
│   │   └── STM32F103xx_FLASH.ld
│   ├── Libraries/
│   │   └── STM32F10x_StdPeriph_Lib/
│   │       ├── Libraries/
│   │       ├── VENDOR_INFO.md
│   │       └── VENDOR_MANIFEST.json
│   ├── cmake/
│   ├── src/
│   │   ├── user/
│   │   ├── app/
│   │   ├── driver/
│   │   ├── bsp/
│   │   └── common/
│   ├── CMakeLists.txt
│   └── CMakePresets.json
├── tests/
└── README.md
```

### 分层职责

| 目录 | 职责 |
| --- | --- |
| `Start` | 启动文件、CMSIS 兼容、链接脚本 |
| `user` | main、中断入口、标准库配置 |
| `app` | 赛题业务流程、状态机、算法编排 |
| `driver` | STM32 片内 ADC/TIM/DMA/UART 等驱动 |
| `bsp` | 板级 GPIO 与外部器件映射 |
| `common` | 与赛题无关的通用组件 |

当前 `common` 预置 `com_time.c/.h`，统一提供 1 ms SysTick 时间基准。

### 自定义源码命名规范

自定义目录和文件统一使用小写 `snake_case`，减少 macOS / Linux 大小写差异带来的构建问题：

- `app/app_<功能>.c/.h`，公开 API 使用 `App_<Module>_...`；
- `driver/drv_<外设>.c/.h`，公开 API 使用 `Drv_<Module>_...`；
- `bsp/bsp_<器件>.c/.h`，公开 API 使用 `Bsp_<Module>_...`；
- `common/com_<组件>.c/.h`，公开 API 使用 `Com_<Module>_...`；
- `user` 中的 `main.c`、`stm32f10x_it.c/.h`、`stm32f10x_conf.h` 等 STM32 约定文件保留官方命名。

文件名负责表达模块归属，函数名前缀负责表达“软件层 + 模块 + 动作”。不要混用 `App_Xxx.c`、`bsp_Xxx.c`、`Driver_xxx.c` 等文件命名风格。

### 标准库管理

`firmware/Libraries/STM32F10x_StdPeriph_Lib` 已作为普通 Git tracked files 固化到模板仓库，不再使用 Git Submodule。新项目通过 **Use this template** 创建或普通 `git clone` 后即可直接构建，不需要额外初始化子模块。

标准库来源与固定上游提交记录在 `VENDOR_INFO.md` / `VENDOR_MANIFEST.json`。业务开发不要修改该目录；CI 会校验 vendor 文件的 blob SHA。

## 为什么不把具体赛题的 ADC 和自动量程一起带进来

不同赛题可能需要完全不同的外设组合，例如：

- TIM 触发 ADC
- ADC + DMA Circular
- 多通道扫描
- 输入捕获
- PWM
- 编码器接口

具体 ADC 通道、GPIO 引脚、触发源和 DMA 工作方式属于项目实现，不是模板基础设施，因此模板只保留分层位置，不预设外设方案。

## 首次使用

标准库已经随模板仓库提供，无需执行任何 submodule 初始化命令。

确认工具链：

```bash
arm-none-eabi-gcc --version
cmake --version
ninja --version
```

## 构建

从仓库根目录执行：

```bash
cmake -S firmware -B firmware/build/Debug -G Ninja \
  -DCMAKE_BUILD_TYPE=Debug \
  -DCMAKE_TOOLCHAIN_FILE=firmware/cmake/gcc-arm-none-eabi.cmake

cmake --build firmware/build/Debug
```

构建输出：

```text
firmware/build/Debug/
├── stm32f103_std_template.elf
├── stm32f103_std_template.hex
├── stm32f103_std_template.bin
└── stm32f103_std_template.map
```

GitHub Actions 会同时验证 Debug / Release，并检查 CMSIS 兼容处理、源码分层和 BIN/HEX/MAP 产物。

## 新电赛题的推荐起步方式

1. 通过 GitHub 的 **Use this template** 创建新的题目仓库。
2. 修改 `CMAKE_PROJECT_NAME` 为题目仓库名。
3. 在 `app/` 创建业务入口，例如 `app_xxx.c/.h`，公开 API 使用 `App_Xxx_...` 前缀。
4. 按实际方案在 `driver/` 增加 ADC、TIM、DMA 等驱动，例如 `drv_adc.c/.h`。
5. 按实际硬件在 `bsp/` 增加板级控制，例如 `bsp_motor.c/.h`。
6. 在 `main.c` 中完成初始化，并在主循环调用 `App_Xxx_Task()`。
7. 本地 Debug 构建通过后再提交，由 GitHub Actions 验证 Debug / Release。

CMake 已使用 `CONFIGURE_DEPENDS` 自动发现 `user/app/driver/bsp/common` 下新增的 `.c` 文件，因此通常不需要每增加一个模块就手工修改源码列表。

## 架构原则

```text
main / IRQ
    │
    ▼
   App
  / | \
 ▼  ▼  ▼
Driver Bsp Common
   \   |   /
    标准外设库
        │
        ▼
      寄存器
        │
        ▼
      STM32
```

核心边界：

- App 决定“做什么”。
- Driver 决定“STM32 片内外设怎么工作”。
- Bsp 决定“当前板子具体接到哪里、怎么驱动外部器件”。
- Common 只放真正跨题通用的能力。
- 不修改固定版本的 vendor 标准库源码。

## GitHub Template Repository

建议在仓库 Settings → General 中勾选 **Template repository**。

这样以后不要复制旧项目历史，也不需要重新搭工程：

```text
embed-stm32c8t6-template
        ↓
Use this template
        ↓
embed-xxxx-xxxx
        ↓
app + driver + bsp
```
