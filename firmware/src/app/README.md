# app：业务编排层（App）

每一道电赛题的核心流程、状态机和算法编排放在这里。

命名规则：

- 文件：`app_<功能>.c/.h`
- 公开 API：`App_<Module>_...`

App 可以调用 driver、bsp 和 common；不要在这里堆标准库寄存器级初始化细节。
