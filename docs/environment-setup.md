# 环境与 GPU 服务器准备

## 推荐分工

- 本机（例如 Mac）：VS Code、Git、文档、阅读代码、整理实验记录。
- 远程 Linux 主机：Unitree 官方 Isaac Gym 训练、长时间并行仿真和需要 NVIDIA GPU 的任务。
- MuJoCo 回放：可在远程机或具备相应依赖的本机运行；图形回放取决于显示环境。

## 何时租服务器

先在本地完成代码阅读、选择 Go2、列出要查看的配置项。真正安装 Isaac Gym 和运行 `train.py` 时再租。开始时租小时计费的 NVIDIA Linux 实例，先跑 Isaac Gym 自带验证示例和 Unitree `play`，通过后再开始短训练。使用持久化磁盘并定期保存 checkpoint、日志和配置。

选机时核对：NVIDIA GPU/驱动、Linux x86_64、GPU 显存、磁盘、SSH、实例停止后磁盘是否保留。建议先从约 16 GB 显存档位尝试；实际并行环境容量与机器人任务、仿真设置和显存占用相关，先少量环境启动再逐步提高。

## 官方仓库当前安装方向

Unitree RL Gym 的 setup guide 给出 Ubuntu、Python 3.8、PyTorch 2.3.1 + CUDA 12.1、Isaac Gym Preview、`rsl_rl` v1.0.2。严格按该仓库当前 setup guide 复现，并记录安装日期与版本；不要把现代 Isaac Lab 的环境和旧 Isaac Gym 环境混装。

官方指南要求从 NVIDIA 开发者页面获取 Isaac Gym Preview。服务器上先验证 Isaac Gym 示例，再安装/验证 Unitree 仓库，最后运行训练。

## 建议软件

本机：VS Code、Git、SSH 客户端、Python（用于通用脚本/笔记，不要求本机装 Isaac Gym）。远程训练环境：Linux x86_64、Miniconda/Conda、仓库指定 PyTorch/CUDA 组合、Isaac Gym Preview、指定版本 `rsl_rl`、MuJoCo 和 Unitree 仓库依赖。

先把 Unitree 官方仓库作为独立 clone 管理，不要把它复制进本学习仓库后再各自修改。此仓库负责学习记录与实验元数据；官方仓库负责训练代码。实验记录保存两边对应的 Git commit。

## 关键官方链接

- Setup: <https://github.com/unitreerobotics/unitree_rl_gym/blob/main/doc/setup_en.md>
- README/workflow: <https://github.com/unitreerobotics/unitree_rl_gym>
- Isaac Gym Preview: <https://developer.nvidia.com/isaac-gym/download>
- Isaac Lab requirements (用于未来评估，不是本项目当前安装步骤): <https://isaac-sim.github.io/IsaacLab/main/source/setup/installation/>
