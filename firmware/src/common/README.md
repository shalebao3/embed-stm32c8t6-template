# common：通用基础组件（Common）

放置与具体赛题无关、可被多个模块复用的纯通用能力。

命名规则：

- 文件：`com_<组件>.c/.h`
- 公开 API：`Com_<Module>_...`

当前预置：

- `com_time.c/.h`：1 ms SysTick 时间基准、毫秒读取和阻塞延时。

不要把具体 ADC 通道、GPIO 引脚、量程参数或题目算法放进 common。
