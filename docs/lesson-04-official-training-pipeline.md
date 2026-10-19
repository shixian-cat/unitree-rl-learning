# 第 4 课：官方代码如何把 Go2 训练跑起来

## 本课先记住一张图

```text
train.py
  ├─ task_registry.make_env()       选 Go2 配置并创建 Isaac Gym 环境
  ├─ task_registry.make_alg_runner() 创建 RSL-RL 的 PPO runner
  └─ ppo_runner.learn()             反复采样、更新策略、保存 checkpoint

每轮采样：策略给动作 → env.step(actions) → 仿真推进 → 返回观测/奖励/done
```

## 1. `train.py` 是入口和装配工

打开 `unitree_rl_gym/legged_gym/scripts/train.py`。文件很短，核心只有三步：

```python
args = get_args()
env, env_cfg = task_registry.make_env(name=args.task, args=args)
ppo_runner, train_cfg = task_registry.make_alg_runner(env=env, name=args.task, args=args)
ppo_runner.learn(num_learning_iterations=train_cfg.runner.max_iterations,
                 init_at_random_ep_len=True)
```

- `get_args()` 读取命令行参数，例如 `--task=go2`、设备和并行环境数。
- `make_env()` 按任务名找到 Go2 环境类和配置，创建仿真环境。
- `make_alg_runner()` 把环境和 PPO 配置交给 `rsl_rl`，得到训练器。
- `learn()` 开始训练。PPO 算法主体来自独立的 `rsl_rl` 包，不在这个短文件里实现。

文件开头的 `from legged_gym.envs import *` 也有作用：导入环境模块时会执行任务注册代码。`legged_gym/envs/__init__.py` 将名字 `go2` 关联到 `LeggedRobot`、`GO2RoughCfg` 和 `GO2RoughCfgPPO`。所以 `--task=go2` 能查到对应的机器人、环境配置和训练配置。

## 2. `task_registry` 把名字变成实际对象

看 `legged_gym/utils/task_registry.py`：

- `register()` 保存任务名、环境类、环境配置和 PPO 配置。
- `make_env()` 找到注册项，应用命令行覆盖，设置随机种子，解析仿真参数，然后实例化环境。
- `make_alg_runner()` 将 PPO 配置转成普通字典，创建 `OnPolicyRunner`；需要续训时才加载 checkpoint。

你可以把 registry 理解为一个任务目录：`go2` 是索引，配置和环境类是查出来的内容。

## 3. `env.step(actions)` 是策略与仿真之间的接口

`legged_robot.py` 中的 `step()` 接收形状为 `(num_envs, 12)` 的动作。它会先裁剪动作，再按 `control.decimation` 重复执行物理步进：

1. `_compute_torques()` 将策略动作转换成关节力矩；P 控制时，动作先缩放为关节目标角的偏移，再由 PD 控制器计算力矩。
2. Isaac Gym 用这些力矩推进物理仿真。
3. `post_physics_step()` 刷新机身和接触状态，检查摔倒/超时，计算奖励，重置结束的环境，再生成下一份观测。
4. `step()` 返回 `(obs, privileged_obs, rewards, dones, infos)` 给训练器。

Go2 的 `dt=0.005` 秒、`decimation=4`，因此每次策略动作之间推进 0.02 秒物理时间，也就是策略控制频率约 50 Hz。一次 `env.step()` 代表一个策略时间步，不等于只调用一次物理引擎。

## 4. PPO 在接口外做什么

`OnPolicyRunner.learn()` 内部循环可以概括为：

1. 用当前策略根据观测采样动作；
2. 把动作交给 `env.step()`，收集奖励、下一观测和结束标记；
3. 将一段 rollout 存入缓冲区；
4. 用 PPO 更新 actor（产生动作的策略）和 critic（估计未来回报的价值网络）；
5. 继续采样，定期写日志和 checkpoint。

Go2 通用配置的 `num_steps_per_env=24` 表示每个环境每次更新先收集 24 个策略时间步。若有 `N` 个并行环境，一轮约收集 `24 × N` 条转移数据，再做 PPO 更新。`max_iterations` 是更新轮数，不是秒数，也不是 episode 数。

## 5. `play.py` 与训练有什么不同

`play.py` 创建环境、加载 checkpoint，然后调用推理策略生成动作并不断 `env.step()`。它不调用 `learn()`，因此不更新策略。回放时关闭部分噪声和随机化，便于观察已有策略行为；代码还会将策略导出为 TorchScript，供后续部署代码使用。

## 你现在只需会回答这三点

1. `train.py` 为什么这么短？——主要训练循环在 `rsl_rl` 的 runner 里，环境逻辑在 `legged_robot.py`。
2. `env.step()` 是谁调用的？——训练时由 PPO runner 调用；回放时由 `play.py` 的推理循环调用。
3. 改任务名会影响什么？——registry 会换成另一套已注册的机器人环境和配置。

## 阅读位置

- `legged_gym/scripts/train.py`
- `legged_gym/scripts/play.py`
- `legged_gym/envs/__init__.py`
- `legged_gym/utils/task_registry.py`
- `legged_gym/envs/base/legged_robot.py` 的 `step()`、`post_physics_step()` 和 `_compute_torques()`
- `legged_gym/envs/base/legged_robot_config.py` 的 `runner.num_steps_per_env`、`runner.max_iterations`

本课只读代码，不需要在当前服务器安装训练环境。该服务器的 Isaac Gym 兼容性还没有验证。
