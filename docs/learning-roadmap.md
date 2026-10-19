# 学习路线：Unitree Go2 + Isaac Lab

## 目标

在没有机器人、使用仿真的前提下，理解并跑通 Go2 速度跟踪强化学习：读懂任务配置，启动短训练，回放 checkpoint，记录实验，并理解策略如何导出用于后续 Sim2Sim。

## 阶段 1：读懂 Go2 任务

依次查看官方仓库中的：

1. `source/unitree_rl_lab/unitree_rl_lab/tasks/locomotion/robots/go2/__init__.py`：Gymnasium task ID 如何注册。
2. `.../go2/velocity_env_cfg.py`：场景、指令、动作、观测、奖励、终止、随机化和 episode 参数。
3. `.../tasks/locomotion/mdp/`：各项观测、奖励、指令和课程学习函数的具体实现。
4. `.../tasks/locomotion/agents/rsl_rl_ppo_cfg.py`：PPO policy/algorithm/runner 参数。
5. `scripts/rsl_rl/train.py` 与 `play.py`：训练环境如何创建、如何加载模型回放。

交付物：用自己的话解释观测→策略→动作→Isaac Lab 环境→奖励/新观测的数据流，并能指出配置中各关键项的位置。

## 阶段 2：检查环境并回放

先满足官方依赖和 GPU 要求，列出任务，检查 Go2 task 能被发现；如果仓库或资产提供可用预训练 checkpoint，先回放。否则使用较小 `num_envs` 和迭代数做启动验证。此阶段目标是确认仿真与训练链路可运行，不追求最终步态质量。

交付物：环境版本记录、运行命令、日志路径、回放观察。

## 阶段 3：短训 PPO 基线

记录 Git commit、task、seed、`num_envs`、`max_iterations`、训练配置和 checkpoint。观察训练日志，并用 `play.py` 检查实际动作；reward 上升不单独等同于机器人行为成功。

## 阶段 4：单因素对比

提出一个可检验问题，每次只改一个配置，例如速度跟踪奖励权重、地形课程或环境随机化。对比训练曲线和回放表现，区分观察结果与对原因的猜测。

## 阶段 5：Sim2Sim

使用仓库部署说明和 Unitree MuJoCo 检查训练策略。重点核对关节顺序、动作缩放、观测组成、控制频率和初始姿态。

## 阶段 6：扩展

流程稳定后，再学习模仿/Mimic 任务、地形泛化、恢复策略、相机观测或真机 sim-to-real。每次只增加一个新变量。

## 数据从哪里来

- 基础 PPO locomotion 的主要训练经验来自 Isaac Lab 仿真中策略与环境的交互，不需要预先收集真机数据。
- 真机数据用于之后的标定、真实评估和 sim-to-real 研究。
- 通用互联网视频不是这个 Go2 状态观测速度跟踪任务的直接训练输入。
- Mimic/模仿任务需要参考动作或轨迹数据，和基础 PPO 速度跟踪任务分开学习。
