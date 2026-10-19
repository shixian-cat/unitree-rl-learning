# 第 1 课：把 RL 基本概念对应到 Go2

## 本课目标

看懂一个腿式运动 RL 任务中的观测、动作、奖励和 episode，并能从代码里指出它们的位置。

## 1. 强化学习的一步

每个仿真时刻，策略根据当前观测选择动作；环境执行动作后返回新观测、奖励，以及 episode 是否结束：

`观测 o_t → 策略 π → 动作 a_t → 仿真环境 → (o_{t+1}, r_t, done)`

策略的目标不是记住一段固定动作，而是学一个映射：不同状态和速度指令下都能给出合适动作。训练通过很多并行仿真环境收集交互经验，再用 PPO 更新策略。

## 2. MDP 五个关键概念

- **环境**：Go2 的刚体、关节、地面和物理仿真。
- **状态 s**：仿真器内部完整状态，例如位置、姿态、关节状态和接触信息。
- **观测 o**：实际交给策略的状态摘要。它通常不等于仿真器知道的全部状态。
- **动作 a**：策略每次输出的 12 个关节控制量。配置采用位置控制时，这些数会先缩放并加到默认关节角，形成目标角度，再由 PD 控制器转换为力矩。
- **奖励 r**：对本步行为的打分。奖励同时鼓励跟踪指令、稳定运动，也惩罚过大力矩、关节越界或碰撞等行为。

一个 episode 是从环境 reset 到结束的一段连续试验。训练重复很多 episode，使策略逐渐得到更高的累计回报。

## 3. 在 Unitree 代码中定位

- Go2 特定设置：`legged_gym/envs/go2/go2_config.py`
- 通用机器人环境与交互：`legged_gym/envs/base/legged_robot.py`
- 通用环境和 PPO 默认配置：`legged_gym/envs/base/legged_robot_config.py`
- 训练入口：`legged_gym/scripts/train.py`

Go2 配置继承通用配置，因此要一起看 Go2 和 base 两处。

## 4. 代码里的具体例子

- 通用配置设置 `num_observations = 48`、`num_actions = 12`、episode 时长 20 秒。
- `compute_observations()` 把机身线速度、角速度、重力方向、速度指令、关节位置差、关节速度和上一动作拼成观测。当前配置对应维数为 `3+3+3+3+12+12+12=48`。
- Go2 有 12 个关节默认角。动作缩放值是 0.25；`_compute_torques()` 将策略动作转成目标角差，再用 PD 控制计算仿真力矩。因此“策略动作”不等于“电机力矩”。
- 奖励权重由通用配置定义，Go2 配置会覆写其中部分值。目标包括线速度和角速度跟踪、足部腾空等；惩罚项包括力矩、关节限位、动作变化率、碰撞等。
- episode 在躯干接触、机身姿态超过阈值或达到时间上限时结束。结束与否由环境判断，不由策略说了算。
- 通用 PPO 配置使用 PPO；每个环境每轮采样 24 步，默认最多更新 1500 次。训练规模参数要根据 GPU 资源调整，入门时先缩小运行规模。

## 5. 本课最重要的区分

1. **观测 ≠ 完整状态**：策略只看到被挑选、缩放并可能加噪后的输入。
2. **动作 ≠ 力矩**：策略输出经过控制层解释后才作用于仿真器。
3. **奖励 ≠ 任务本身**：策略优化的是代码定义的奖励，因此奖励遗漏或权重失衡会让行为偏离人的直觉。
4. **一次跑通 ≠ 泛化**：同一配置同一种子训练成功，还需要回放、换种子或做 Sim2Sim 检查。

## 练习

打开官方仓库的 `go2_config.py` 和 `legged_robot.py`，用自己的话回答：

1. 48 维观测由哪七组量拼出来？
2. 为什么说 12 维动作不是直接的电机力矩？
3. 如果提高 `torques` 的负奖励权重，你预期策略会怎样变化？可能出现什么副作用？
4. 机器人躯干碰地时，环境会如何处理该 episode？

建议先不改代码。先在笔记中写出“代码事实”和“自己的预测”，把观察和解释分开。

## 官方源码

- [Go2 配置](https://github.com/unitreerobotics/unitree_rl_gym/blob/main/legged_gym/envs/go2/go2_config.py)
- [通用配置](https://github.com/unitreerobotics/unitree_rl_gym/blob/main/legged_gym/envs/base/legged_robot_config.py)
- [通用环境](https://github.com/unitreerobotics/unitree_rl_gym/blob/main/legged_gym/envs/base/legged_robot.py)
- [训练入口](https://github.com/unitreerobotics/unitree_rl_gym/blob/main/legged_gym/scripts/train.py)
