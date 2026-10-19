# Isaac Lab 环境与服务器准备

本项目已切换到宇树官方 `unitree_rl_lab`。该仓库 README 标注 Isaac Sim 5.1 / Isaac Lab 2.3，并通过 `unitree_rl_lab.sh` 在 Isaac Lab 环境中安装自己的任务包。优先按仓库 README 和所选 Isaac Lab 版本配套安装，不要把 Isaac Gym 的 Python/CUDA 依赖混进来。

## 服务器规格

Isaac Sim 5.1 官方工作站要求给出的最低参考是 Ubuntu 22.04/24.04、32GB RAM、50GB SSD、RTX 4080 级别 GPU、16GB VRAM、Linux NVIDIA driver 580.65.06；理想 GPU 列为 RTX PRO 6000 Blackwell。Isaac Lab 训练和缓存还会额外占用资源，建议预留至少 100GB 持久化磁盘。

租实例时优先检查：

- GPU 是否是完整的 RTX PRO 6000 / RTX 4090 / RTX 5090 等受支持 RTX 卡，显存至少 16GB；vGPU 先问清底层型号及 Isaac Sim 是否支持。
- Ubuntu 22.04，驱动满足所选 Isaac Sim 版本要求。
- 32GB 以上系统内存，至少 100GB 持久化磁盘空间。
- 远程无桌面训练可用 headless；首次环境和资产缓存仍会占用时间与磁盘。

## 先前服务器检查结果

之前连接的实例为 Ubuntu 22.04.3、RTX 3080 Ti 12GB、驱动 595.71.05、根盘 30GB。系统版本和驱动满足较新的 Isaac Sim 条件，但显存低于 16GB 的官方最低参考，根盘也小于官方最低 50GB。它不作为本次 Lab 实操服务器的首选；也不要在旧 Gym 环境和 Isaac Lab 环境间混装依赖。

## 软件安装路径

1. 先根据 [宇树 RL Lab README](https://github.com/unitreerobotics/unitree_rl_lab#installation) 安装其要求的 Isaac Lab 环境。
2. 克隆/使用独立的 `unitree_rl_lab` 仓库，不要把它放进 Isaac Lab 安装目录。
3. 准备 Unitree 模型资产：README 提供 USD 数据集路径；Isaac Sim 5.0+ 也可按说明使用 Unitree URDF。
4. 在已安装 Isaac Lab 的 Conda 环境中，进入 Lab 仓库执行 `./unitree_rl_lab.sh -i`。
5. 运行 `./unitree_rl_lab.sh -l` 检查 task 是否注册，再用小规模 Go2 task 做启动验证。

完整训练开始前，先确认模型资产、GPU、日志目录和持久化磁盘都可用。不要直接用默认 4096 个并行环境起步。

## 本机与服务器分工

- 本机 VS Code：阅读源码、修改配置、维护学习笔记和实验记录。
- Linux GPU 服务器：安装 Isaac Lab/Isaac Sim、运行仿真训练、保存 checkpoint。
- 没有真实机器人也能完成环境交互训练、策略回放和 MuJoCo Sim2Sim 学习。

## 官方参考

- [Unitree RL Lab 安装说明](https://github.com/unitreerobotics/unitree_rl_lab#installation)
- [Isaac Sim 5.1 系统要求](https://docs.isaacsim.omniverse.nvidia.com/5.1.0/installation/requirements.html)
- [Isaac Lab 安装说明](https://isaac-sim.github.io/IsaacLab/)
