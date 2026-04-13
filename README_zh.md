# TienKung-Lab：基于强化学习的运控系统

**[English](./README.md)｜简体中文**

## 概览

该框架是一个基于强化学习（RL）的运动控制系统，适用于全尺寸人形机器人天工行者。它将 AMP 风格[[本项目代码文件]](https://github.com/jayjayhust/Ubtech_TienKung-Lab/blob/ubetech-comp-2026/rsl_rl/rsl_rl/algorithms/amp_ppo.py)[[AMP原始项目]](https://github.com/linden713/humanoid_amp)的奖励与周期性步态奖励相结合，促进了自然、稳定和高效的行走和跑步行为。

该代码库建立在 IsaacLab 之上，支持 Sim2Sim 迁移到 MuJoCo，并具有模块化架构，便于无缝定制和扩展。此外，它还结合了基于光线投射的传感器以增强感知能力，能够精确地与环境交互并避障。该框架已在真机得到成功验证。



## 项目架构

	TienKung-Lab/
	├── legged_lab/              # 核心运动控制框架
	│   ├── scripts/             # 可执行脚本
	│   ├── envs/                # 环境定义
	│   ├── mdp/                 # 奖励函数
	│   ├── sensors/             # 传感器模块
	│   ├── terrains/            # 地形生成
	│   ├── assets/              # 机器人模型
	│   └── utils/               # 工具函数
	│
	├── rsl_rl/                  # 强化学习库
	│   ├── algorithms/          # PPO/AMP-PPO 算法
	│   ├── modules/             # 神经网络模块
	│   ├── runners/             # 训练运行器
	│   ├── storage/             # 数据存储
	│   └── utils/               # 工具函数
	│
	├── Exported_policy/         # 预训练策略
	├── docs/                    # 文档
	└── setup.py                 # 安装配置



## 安装

TienKung-Lab 基于 IsaacSim 4.5.0 和 IsaacLab 2.1.0 构建。

- 请遵循 [安装指南](https://isaac-sim.github.io/IsaacLab/main/source/setup/installation/index.html) 安装 Isaac Lab。我们建议使用 conda 安装，因为它简化了从终端调用 Python 脚本的过程。

- 在 Isaac Lab 安装目录之外单独克隆此仓库（即不要放在 `IsaacLab` 目录下）

- 使用已安装 Isaac Lab 的 python 解释器安装本库

```bash
cd TienKung-Lab
pip install -e .
```

- 安装 rsl-rl 库

```bash
cd TienKung-Lab/rsl_rl
pip install -e .
```

- 运行以下命令验证扩展是否安装正确：

```bash
python legged_lab/scripts/train.py --task=walk  --logger=tensorboard --headless --num_envs=64
```



## 用法

### 运动重定向（Motion Retargeting）

本节使用 [GMR](https://github.com/YanjieZe/GMR) 进行运动重定向，Walker TienKung 目前仅支持 SMPLX 类型（AMASS, OMOMO）的运动重定向。

**1. 准备数据集并使用 [GMR](https://github.com/YanjieZe/GMR) 进行运动重定向。**

```bash
python scripts/smplx_to_robot.py --smplx_file <path_to_smplx_data> --robot tienkung  --save_path <path_to_save_robot_data.pkl>
```

**2. 数据处理与保存。**

数据集由功能和格式各异的两部分组成，需要分两步进行转换。

- **`motion_visualization/`**  
  用于通过 `play_amp_animation.py` 回放动作，以检查动作的正确性和质量。  
  数据字段： [root_pos, root_rot, dof_pos, root_lin_vel, root_ang_vel, dof_vel]

- **`motion_amp_expert/`**  
  在训练期间用作 AMP 的专家参考数据。  
  数据字段： [dof_pos, dof_vel, end-effector pos]

- **步骤 1：数据处理与可视化数据保存。**

```bash
python legged_lab/scripts/gmr_data_conversion.py --input_pkl <path_to_save_robot_data.pkl> --output_txt legged_lab/envs/tienkung/datasets/motion_visualization/motion.txt
```

**注意**：在开始步骤 2 之前，请将配置中的 `amp_motion_files_display` 路径设置为步骤 1 生成的文件。

- **步骤 2：动作可视化与专家数据保存。**

```bash
python legged_lab/scripts/play_amp_animation.py --task=walk --num_envs=1 --save_path legged_lab/envs/tienkung/datasets/motion_amp_expert/motion.txt --fps 30.0
```

**注意**：步骤 2 完成后，请将配置中的 `amp_motion_files` 路径设置为步骤 2 生成的文件。

### 可视化动作

通过使用 tienkung/datasets/motion_visualization 中的数据更新仿真来可视化动作。

```bash
python legged_lab/scripts/play_amp_animation.py --task=walk --num_envs=1
python legged_lab/scripts/play_amp_animation.py --task=run --num_envs=1
```

### 可视化带传感器的动作

通过使用 tienkung/datasets/motion_visualization 中的数据更新仿真来可视化带传感器的动作。

```bash
python legged_lab/scripts/play_amp_animation.py --task=walk_with_sensor --num_envs=1
python legged_lab/scripts/play_amp_animation.py --task=run_with_sensor --num_envs=1
```

### 训练

使用 tienkung/datasets/motion_amp_expert 中的 AMP 专家数据训练策略。

```bash
python legged_lab/scripts/train.py --task=walk --headless --logger=tensorboard --num_envs=4096
python legged_lab/scripts/train.py --task=run --headless --logger=tensorboard --num_envs=4096
```

### 运行（Play）

运行已训练的策略。

```bash
python legged_lab/scripts/play.py --task=walk --num_envs=1
python legged_lab/scripts/play.py --task=run --num_envs=1
```

### Sim2Sim (MuJoCo)

在 MuJoCo 中评估已训练的策略，以进行交叉仿真验证。

Exported_policy/ 包含项目提供的预训练策略。使用 play 脚本时，训练好的策略会自动导出并保存到类似 logs/run/[timestamp]/exported/policy.pt 的路径中。

```bash
python legged_lab/scripts/sim2sim.py --task walk --policy Exported_policy/walk.pt --duration 100
```

### Sim2Real（真机部署）

TienKung-Lab 的结果已在真实机器人上成功验证。

关于部署详情，请参考 [此处](https://github.com/UBTECH-Robot/Deploy_Tienkung) 的仓库。

**安全提示：** 在真机上测试存在风险。RL 策略可能会导致意外或剧烈的动作，因此请确保已购买意外保险并且紧急停止按钮工作正常。




## 代码格式化

我们提供了一个 pre-commit 模板来自动格式化代码。
安装 pre-commit：

```bash
pip install pre-commit
```

然后可以通过以下命令运行 pre-commit：

```bash
pre-commit run --all-files
```



## 故障排除

### Pylance 缺失扩展索引

在某些 VsCode 版本中，部分扩展的索引可能会丢失。这种情况下，请在 `.vscode/settings.json` 的 `"python.analysis.extraPaths"` 键下添加扩展的路径。

```json
{
    "python.analysis.extraPaths": [
        "${workspaceFolder}/legged_lab",
        "<path-to-IsaacLab>/source/isaaclab_tasks",
        "<path-to-IsaacLab>/source/isaaclab_mimic",
        "<path-to-IsaacLab>/source/extensions",
        "<path-to-IsaacLab>/source/isaaclab_assets",
        "<path-to-IsaacLab>/source/isaaclab_rl",
        "<path-to-IsaacLab>/source/isaaclab",
    ]
}
```


## 致谢
**特别感谢北京人形机器人创新中心提供的宝贵支持与指导。**

**项目链接：** [TienKung-Lab](https://github.com/Open-X-Humanoid/TienKung-Lab)

