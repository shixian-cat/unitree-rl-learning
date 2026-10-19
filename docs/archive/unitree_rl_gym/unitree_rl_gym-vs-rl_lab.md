# Unitree 两套强化学习代码的区别

宇树官方有两套相关但独立的训练实现：`unitree_rl_gym` 和 `unitree_rl_lab`。它们不是同一个项目的两个可互换版本。

| 对比项 | `unitree_rl_gym` | `unitree_rl_lab` |
|---|---|---|
| 仿真框架 | NVIDIA Isaac Gym Preview 4 | NVIDIA Isaac Lab，配套 Isaac Sim |
| 当前状态 | 代码仍可读和使用；底层 Isaac Gym 已被 NVIDIA 标为 legacy | 宇树较新的 Isaac Lab 实现 |
| 常见代码结构 | `legged_gym` 风格；`LeggedRobot` 中集中实现环境交互，配置通过类继承 | Isaac Lab 的场景、动作、观测、奖励、终止管理器和配置体系，更模块化 |
| 机器人 | 官方仓库 README 列出 Go2、H1、H1_2、G1 | 官方 README 列出 Go2、H1、G1-29dof；具体任务看当前分支 |
| 环境要求 | 官方 Unitree 安装页给出 Python 3.8、PyTorch 2.3.1/CUDA 12.1 和 Isaac Gym 安装步骤；Isaac Gym 官方包较旧，显卡架构和系统版本可能带来兼容问题 | Unitree README 当前标注 Isaac Sim 5.1 / Isaac Lab 2.3；需要兼容的 RTX GPU 和更充足的系统资源 |
| 适合的用途 | 学习经典腿式机器人 PPO 流程、理解观测/动作/奖励/终止及 `env.step()` | 在较新的仿真平台上训练 Go2 等任务，并使用其更丰富的场景与传感器仿真生态 |
| 策略与 checkpoint | 和 Lab 的具体配置绑定 | 与 Gym 的观测、动作、网络和环境配置不保证兼容；不能直接假设 checkpoint 可互换 |

## 学习建议

1. 先继续读当前本地的 `unitree_rl_gym`：它的训练入口和环境交互链较短，适合理解 PPO 如何与仿真环境循环交互。
2. 需要在 RTX PRO 6000 这类新显卡上实操时，优先按宇树 `unitree_rl_lab` 的安装说明搭配 Isaac Lab；它的 README 中包含 Go2。
3. 迁移到 Lab 时，把它当作另一套实现重新读任务配置和训练入口。强化学习概念能复用，机器人模型、环境配置、依赖和模型文件则要分别管理。

## 官方资料

- [Unitree RL Gym](https://github.com/unitreerobotics/unitree_rl_gym)
- [Unitree RL Gym 安装说明](https://github.com/unitreerobotics/unitree_rl_gym/blob/main/doc/setup_en.md)
- [Unitree RL Lab](https://github.com/unitreerobotics/unitree_rl_lab)
- [NVIDIA Isaac Gym Preview 下载与旧版要求](https://developer.nvidia.com/isaac-gym/download)
- [NVIDIA Isaac Sim 5.1 要求](https://docs.isaacsim.omniverse.nvidia.com/5.1.0/installation/requirements.html)
