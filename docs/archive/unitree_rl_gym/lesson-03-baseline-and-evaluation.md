# 第 3 课：基线回放与判断训练结果

## 本课目标

知道什么时候用 `play.py`、什么时候用 `train.py`，并学会从训练曲线和机器人行为一起判断一个基线是否值得继续。

## 1. Train 和 Play 的分工

- `train.py`：让当前策略在仿真中继续采样并更新参数。
- `play.py`：加载一个已有 checkpoint，在仿真中观察策略，不做训练更新。

Unitree README 的总体流程是 `Train → Play → Sim2Sim → Sim2Real`。本阶段先做到基线回放和短训练；还没有机器人并不影响这一步。

## 2. 第一次实验的顺序

1. **先尝试回放已有 checkpoint。**使用 Go2 task，明确记录加载的是哪个 run 和 checkpoint。仓库 README 说明 Play 默认读取实验目录最近一次 run；可用 `load_run` 和 `checkpoint` 指定模型。
2. **如果没有可用 checkpoint，就做短训练。**使用较小 `num_envs` 和较少 `max_iterations`，目标只是验证训练链、日志和 checkpoint 保存工作，不判断最终步态质量。
3. **训练完明确指定该次 run/checkpoint 做 Play。**不要只回放“最近一次”而不记模型来源。
4. **保存配置、种子、日志路径和模型文件。**这样才能复现和比较。

README 给出的入口是 `python legged_gym/scripts/play.py --task=go2` 和 `python legged_gym/scripts/train.py --task=go2`。训练输出在 `logs/<experiment_name>/<date_time>_<run_name>/` 下。正式执行前按官方 setup guide 安装并确认设备参数；没有 NVIDIA Linux 环境时不要在本机强行运行 Isaac Gym。

## 3. 判断基线时看三类证据

### A. 训练数字

- **episode reward / mean reward：**看整体趋势是否改善；它只是奖励函数的分数，不等于任务成功率。
- **episode length：**看机器人是否能持续运行，还是很快摔倒并 reset。
- **各奖励分项：**如果日志记录了 `rew_*`，看速度跟踪、力矩、动作变化、碰撞等分项，确认总分变化来自哪里。
- **训练是否稳定：**留意 NaN、突降、长期不变、频繁失败等迹象。

### B. 仿真行为

回放时看机器人是否能站稳、是否按前进/侧向/转向命令移动、是否只会一种方向、是否出现抖动或异常大动作。训练曲线上升但动作更差时，不应把它判作成功。

### C. 可复现性

记录 Git commit、task、配置覆盖、seed、num_envs、max_iterations、checkpoint 和结果。基线先固定条件；后续每次实验只改一个因素。

## 4. 本阶段的“完成标准”

不要求达到论文指标。完成以下事项就算基线流程跑通：

- 能说明回放使用的是哪个策略文件；
- 短训练可以启动并保存 checkpoint；
- 能用训练出的 checkpoint 回放；
- 同时记录一项训练指标和一项行为观察；
- 能说清这只是仿真结果，不是真机验证。

## 5. 当前不需要做什么

现在先不调奖励权重、不追求最佳步态、不跑长时间训练，也不需要收集真机视频。下一阶段再选一个问题做单因素实验。

## 练习

设想短训后 mean reward 上升，但机器人回放时经常摔倒。写下：

1. 为什么不能只看 mean reward？
2. 你会再检查哪两类证据？
3. 下次实验怎样保证和这次结果可比较？

## 官方说明

- [Unitree RL Gym README：Train/Play 流程和命令](https://github.com/unitreerobotics/unitree_rl_gym#-process-overview)
- [训练入口](https://github.com/unitreerobotics/unitree_rl_gym/blob/main/legged_gym/scripts/train.py)
- [回放入口](https://github.com/unitreerobotics/unitree_rl_gym/blob/main/legged_gym/scripts/play.py)
- [安装配置指南](https://github.com/unitreerobotics/unitree_rl_gym/blob/main/doc/setup_en.md)
