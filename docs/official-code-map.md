# Unitree RL Lab：Go2 代码导航

打开 VS Code 工作区后，官方源码位于 `../unitree_rl_lab/`。本导航对应本地克隆版本 `4960b84`；后续更新源码后，配置数值可能变化。

## 阅读顺序

| 文件 | 先看什么 | 要回答的问题 |
|---|---|---|
| `source/unitree_rl_lab/unitree_rl_lab/tasks/locomotion/robots/go2/__init__.py` | `gym.register()` 的 task id 和配置入口 | `Unitree-Go2-Velocity` 这个任务名如何关联环境与 PPO 配置？ |
| `.../go2/velocity_env_cfg.py` | `RobotSceneCfg`、`CommandsCfg`、`ActionsCfg`、`ObservationsCfg`、`RewardsCfg`、`TerminationsCfg`、`RobotEnvCfg` | Go2 在哪个场景里学？策略输入输出是什么？什么行为得分、什么情况结束？ |
| `.../tasks/locomotion/mdp/observations.py` | 观测项函数 | 每个观测项从仿真状态中取什么量？ |
| `.../tasks/locomotion/mdp/rewards.py` | 奖励函数 | 配置里的 reward term 实际如何计算？ |
| `.../tasks/locomotion/agents/rsl_rl_ppo_cfg.py` | 网络、rollout 步数、PPO 参数 | 一轮更新收集多少数据，算法如何更新策略？ |
| `scripts/rsl_rl/train.py` | `AppLauncher`、`gym.make()`、`OnPolicyRunner`、`runner.learn()` | 训练脚本怎样启动 Isaac Sim、创建 Go2 环境并交给 PPO？ |
| `scripts/rsl_rl/play.py` | checkpoint 加载和 inference policy | 训练出的策略怎样只做推理回放？ |

## 当前 Go2 任务的关键结构

- 注册任务 ID 为 `Unitree-Go2-Velocity`。
- `RobotEnvCfg` 组合场景、观测、动作、速度指令、奖励、终止、事件和课程配置。
- 仿真步长 `dt=0.005s`，`decimation=4`，策略每次动作推进 0.02 秒仿真时间，约 50Hz。
- actor 的 `policy` 观测与 critic 的 `critic` 观测分组配置；两者不一定相同。
- `JointPositionAction` 将动作映射为关节位置目标；实际控制接口由 Isaac Lab articulation/action 管理器执行。
- PPO 默认每个环境每轮采样 24 步；Go2 环境默认 `num_envs=4096`，初学启动时用命令行缩小并行环境数。

## 训练与回放入口

仓库 README 给出：

```bash
./unitree_rl_lab.sh -l
./unitree_rl_lab.sh -t --task Unitree-Go2-Velocity --num_envs 64 --max_iterations 10
./unitree_rl_lab.sh -p --task Unitree-Go2-Velocity
```

训练命令里的较小环境数和迭代数仅用于验证启动链路；真正训练前要依据 GPU 显存逐步增加规模。`-p` 回放时需要已存在的 checkpoint。

## Isaac Lab 的组织方式和旧代码区别

旧 `unitree_rl_gym` 把较多环境逻辑集中在 `LeggedRobot`。Lab 版本通常通过 ManagerBasedRLEnv 配置对象组合 scene、actions、observations、rewards、terminations、events 和 curriculum，再由 Isaac Lab 管理器运行。阅读时按配置项找到对应函数，比寻找一个“总的 step 函数”更有效。
