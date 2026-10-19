# Unitree RL Learning Lab

这是学习宇树 Go2 强化学习的笔记和实验记录项目。当前实操主线切换为宇树官方 [`unitree_rl_lab`](https://github.com/unitreerobotics/unitree_rl_lab)，使用 Isaac Lab。官方源码在本目录旁边单独克隆，保留它自己的 Git 历史。

## 从这里开始

1. 阅读 [新的学习路线](docs/learning-roadmap.md)。
2. 按 [Lab 代码导航](docs/official-code-map.md) 在 VS Code 里查看 Go2 任务。
3. 按 [环境与服务器准备](docs/environment-setup.md) 确认 GPU、磁盘和软件版本。
4. 每次实验复制 [实验记录模板](experiments/templates/experiment.md)。

## 本地目录

- `../unitree_rl_lab/`：宇树官方 Isaac Lab 源码，已加入 VS Code 工作区。
- `docs/`：当前 Isaac Lab 学习路线、代码导航和环境说明。
- `docs/archive/unitree_rl_gym/`：之前 Isaac Gym 学习笔记存档；Gym 源码不再放在本机工作区。
- `experiments/`：实验计划和结果。

没有真机也可以完成仿真训练、策略回放和 Sim2Sim 学习。真机部署是后续选修阶段。

## 官方仓库

- [Unitree RL Lab](https://github.com/unitreerobotics/unitree_rl_lab)
- [Unitree RL Gym（旧学习笔记所用）](https://github.com/unitreerobotics/unitree_rl_gym)
- [NVIDIA Isaac Lab 安装说明](https://isaac-sim.github.io/IsaacLab/)
