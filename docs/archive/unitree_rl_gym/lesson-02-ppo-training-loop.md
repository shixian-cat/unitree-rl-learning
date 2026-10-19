# 第 2 课：PPO 怎样从仿真经验中更新策略

## 本课目标

读懂训练入口，并能解释一次 PPO iteration：采样 rollout、计算优势/回报、更新 actor/critic、保存日志和 checkpoint。

## 1. 从 train.py 看训练是怎么启动的

`legged_gym/scripts/train.py` 只有三步核心动作：

1. `task_registry.make_env(...)`：按 `--task` 选注册好的机器人任务，建立并行仿真环境。
2. `task_registry.make_alg_runner(...)`：读取这个任务对应的 PPO 配置，建立 `OnPolicyRunner`。
3. `ppo_runner.learn(...)`：开始训练，循环采样经验并更新策略。

`task_registry.py` 显示 runner 使用 `rsl_rl.runners.OnPolicyRunner`，并将配置转换为字典传入。该仓库安装说明固定了 `rsl_rl` v1.0.2；所以实际 PPO 实现主要在这个依赖仓库里，而不是 `train.py` 中。

## 2. Actor 和 Critic 分别做什么

- **Actor（策略）**读观测，输出动作分布；训练时会采样动作，部署时通常从策略得到动作。
- **Critic（价值函数）**估计当前观测/状态未来还能获得多少累计奖励。它帮助判断刚才的动作比预期好还是差。
- 机器人环境本身仍负责物理仿真和算奖励；PPO 不会替代 `step()` 或 PD 控制器。

PPO 使用新策略和采样时旧策略之间的概率比率更新 Actor，并用 clipping 限制单次更新幅度，降低策略突然变化导致性能崩坏的风险。

## 3. 一次训练 iteration 的直觉流程

```text
当前策略
  ↓ 在所有并行环境各走 24 个策略步
rollout：观测、动作、奖励、结束标记、价值估计
  ↓
计算每步的 advantage（比 Critic 预期好多少）和 return（学习目标）
  ↓
把 rollout 切成 mini-batches，重复多个 epoch 优化 Actor 和 Critic
  ↓
记录指标/按间隔保存 checkpoint
  ↓
用更新后的策略采集下一批 rollout
```

这是 on-policy 方法：当前这批数据主要用于更新产生它的当前策略，更新之后再用新策略采下一批。

## 4. 用配置算出训练规模

通用配置给出：

- `num_envs = 4096`：并行环境数（大规模默认值，不是初学者必须照搬的数）。
- `num_steps_per_env = 24`：每个环境每轮采样 24 个策略步。
- `num_mini_batches = 4`：每个 PPO epoch 将 rollout 划成 4 个 mini-batch。
- `num_learning_epochs = 5`：同一批 rollout 被用于 5 个优化 epoch。
- `max_iterations = 1500`：最多做 1500 轮“采样+更新”，不是 1500 个 episode。

按默认值计算，每轮 rollout 有 `4096 × 24 = 98,304` 个转移样本。每个 mini-batch 大约 `98,304 / 4 = 24,576` 个样本。每个 iteration 有 `5 × 4 = 20` 个 mini-batch 更新步骤。同一批数据在这 5 个 epoch 中会被重复使用。

如果每轮都完整运行 1500 次，名义采样量约为 147,456,000 个环境转移。实际时间与环境结束、reset、GPU、仿真速度等有关。初学时先减小 `num_envs` 和 `max_iterations` 验证流程，之后再增加。

一个 Go2 episode 上限是 20 秒；策略步长约 0.02 秒，所以一个完整 episode 上限约 1000 个策略步。一次 rollout 的 24 步远短于一个 episode，这没问题：runner 会跨 rollout 连续训练，episode 也可能在采样区间中结束并重置。

## 5. 关键 PPO 参数的学习顺序

- `learning_rate = 1e-3`：优化器更新参数的步幅；太大可能训练不稳定，太小可能进展慢。
- `gamma = 0.99`：折扣未来奖励的系数，影响策略更看重近期还是长远。
- `lam = 0.95`：GAE 参数，在优势估计的偏差和方差之间折中。
- `clip_param = 0.2`：PPO 更新策略概率比时的裁剪范围。
- `entropy_coef = 0.01`：探索奖励的权重，太低可能过早变得单一，太高则动作持续分散。
- `value_loss_coef = 1.0`：Critic 价值误差在总损失里的权重。
- `desired_kl = 0.01`、`schedule = 'adaptive'`：用于根据策略变化幅度调节学习率的配置。

本课先会读这些值即可，不建议一次改好几个。后续实验应先固定 seed、环境数和训练长度，再一次只调整一个参数。

## 6. 训练入口和配置怎么接上

- `train.py` 把 `args.task` 传给 `task_registry`。
- `task_registry.make_env()` 按注册表取得环境类和该任务的 `env_cfg`，并应用命令行覆盖。
- `task_registry.make_alg_runner()` 取得训练配置，将它传给 `OnPolicyRunner`。
- Go2 的 `GO2RoughCfgPPO` 继承通用 PPO 配置，只覆盖少数项目，例如 `experiment_name`。

因此配置值最终组合为：Go2 覆盖值 + 通用父类默认值 + 命令行指定的覆盖值。实验记录里应保存最终有效配置，而不只是记 Go2 文件里的几行。

## 7. 练习

1. 用默认值算出每轮 rollout 有多少转移样本、每个 mini-batch 多少样本。
2. 1500 iterations 为什么不是 1500 个 episode？
3. 如果训练显存不够，你会先调 `num_envs` 还是 `num_actions`？为什么？
4. 如果机器人很快就只学会站着不动，你认为应该先检查奖励、命令采样、episode 终止，还是直接加大学习率？请说明理由。

## 源码

- [训练入口 `train.py`](https://github.com/unitreerobotics/unitree_rl_gym/blob/main/legged_gym/scripts/train.py)
- [任务注册和 runner 创建 `task_registry.py`](https://github.com/unitreerobotics/unitree_rl_gym/blob/main/legged_gym/utils/task_registry.py)
- [PPO/runner 默认参数 `legged_robot_config.py`](https://github.com/unitreerobotics/unitree_rl_gym/blob/main/legged_gym/envs/base/legged_robot_config.py)
