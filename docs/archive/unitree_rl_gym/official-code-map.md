# Unitree 官方代码导航：Go2 RL 入门该看哪里

这份导航只带你看第一次练习所需的配置和执行路径。打开官方文件后按下面的标题/搜索词查找即可；Go2 配置继承了通用配置，所以不能只看一个文件。

## 阅读顺序

1. **`go2_config.py`：先看 Go2 改了什么**
2. **`legged_robot_config.py`：再看继承来的默认值**
3. **`legged_robot.py`：追踪一步环境交互**
4. **`train.py`：最后看如何把环境交给 PPO**

## A. Go2 专属设置

文件：[go2_config.py](https://github.com/unitreerobotics/unitree_rl_gym/blob/main/legged_gym/envs/go2/go2_config.py)

| 代码位置 | 关键项 | 读它时问自己什么 |
|---|---|---|
| `GO2RoughCfg.init_state` | 初始机身位置、12 个关节默认角 | 仿真 reset 后机器人从什么姿态开始？动作 0 对应什么姿势？ |
| `GO2RoughCfg.control` | `control_type='P'`、刚度/阻尼、`action_scale=0.25`、`decimation=4` | 策略输出会如何变成目标关节角？策略多久更新一次？ |
| `GO2RoughCfg.asset` | URDF 路径、foot 名称、惩罚碰撞部位、躯干接触终止 | 环境加载哪个机器人模型？哪些碰撞算违规或终止？ |
| `GO2RoughCfg.rewards` | `base_height_target=0.25`、`soft_dof_pos_limit=0.9`、覆写的惩罚权重 | Go2 对继承来的奖励设置做了哪些改变？ |
| `GO2RoughCfgPPO` | `entropy_coef=0.01`、实验名 | 这是算法超参数/运行标识，不是机器人物理参数。 |

**初学最重要的结论：**这里没有写完所有观测、奖励或 PPO 设置。每个子类继承自通用 `LeggedRobotCfg` / `LeggedRobotCfgPPO`。

## B. 通用环境配置：维度、指令、奖励与 PPO

文件：[legged_robot_config.py](https://github.com/unitreerobotics/unitree_rl_gym/blob/main/legged_gym/envs/base/legged_robot_config.py)

| 配置段 | 初学先记的值 | 含义 |
|---|---|---|
| `env` | `num_observations=48`, `num_actions=12`, `episode_length_s=20` | 策略输入/输出维数和单次仿真试验的最长时长。`num_envs=4096` 是默认并行环境数，运行时可调整。 |
| `commands` | `lin_vel_x/y`、`heading` 的范围；每 10 秒重采样 | 任务给机器人随机速度/朝向指令，不是视频或固定轨迹标签。 |
| `terrain` | 地面种类、摩擦、地形课程 | 训练环境如何定义地面和地形变化。 |
| `domain_rand` | 摩擦随机化、定时推机器人 | 为仿真增加变化，练习鲁棒性。 |
| `rewards.scales` | `tracking_lin_vel`、`tracking_ang_vel`、`feet_air_time` 和力矩/动作/碰撞等惩罚 | 各项奖励函数乘上的权重；Go2 会覆写部分权重。 |
| `normalization` / `noise` | 观测缩放、裁剪和传感噪声 | 策略实际看到的是处理过的观测。 |
| `sim` | `dt=0.005` 秒 | 物理仿真的基础时间步。结合 Go2 `decimation=4`，策略动作每 0.02 秒更新一次，即约 50 Hz。 |
| `policy` | actor/critic 隐藏层 `[512,256,128]` | PPO 策略网络和价值网络的大致结构。 |
| `algorithm` | PPO、`gamma=0.99`、`lam=0.95` 等 | PPO 学习参数；入门第一次复现先不要动它们。 |
| `runner` | 每个环境每轮 24 步、默认最多 1500 次更新、每 50 次检查点保存 | 采样量、训练长度和保存节奏。实际初次练习应先缩小规模。 |

## C. 环境一步做了什么

文件：[legged_robot.py](https://github.com/unitreerobotics/unitree_rl_gym/blob/main/legged_gym/envs/base/legged_robot.py)

在 VS Code 搜索这些方法：

1. **`step(self, actions)`**：接收策略动作，按 `decimation` 执行多次物理步进，然后返回观测、奖励、结束标记和附加信息。
2. **`_compute_torques(self, actions)`**：动作乘 `action_scale`；P 控制时结合默认关节角、当前关节位置、刚度和阻尼，生成力矩。这里能直接看出 action ≠ torque。
3. **`compute_observations(self)`**：将七组量拼接：机身线速度 3、角速度 3、投影重力 3、运动指令 3、关节位置差 12、关节速度 12、上一动作 12，总计 48 维；之后可能加噪声。
4. **`compute_reward(self)`**：调用启用的各项奖励函数，乘配置权重后求和；注意 `only_positive_rewards` 会把总奖励下裁剪到 0，再加终止奖励。
5. **`check_termination(self)`**：碰撞力、机身 roll/pitch 超阈值或时间耗尽时置结束标记。
6. **`reset_idx(self, env_ids)`**：对结束的环境重置关节/机身状态、重新采样速度指令并清空 episode 计数。

把这一轮记成：

`step(actions) → 物理仿真 × 4 → check_termination → compute_reward → reset(如需) → compute_observations → 返回给 PPO`

## D. 训练入口只看 3 行

文件：[train.py](https://github.com/unitreerobotics/unitree_rl_gym/blob/main/legged_gym/scripts/train.py)

- `task_registry.make_env(...)`：按 `--task` 创建机器人环境。
- `task_registry.make_alg_runner(...)`：根据该任务配置创建 PPO runner。
- `ppo_runner.learn(...)`：开始采样和策略更新。

## VS Code 实操提示

在官方仓库打开后按 `⌘P` 搜索文件名；按 `⌘F` 搜索上表的类名或方法名。建议用 VS Code 双击拆分视图同时看 `go2_config.py` 和 `legged_robot_config.py`，继承关系就会很直观。

## 本次阅读任务

暂时只看 A、B、C 中列出的区域。在你的笔记里写三列：**源码里的配置值 / 它控制的内容 / 我预测改大或改小会怎样**。先不改代码。
