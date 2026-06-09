# 机械臂抓取模块

## 学生作业提交

- **学生**：minshu22
- **日期**：2026-04-20

## 项目简介

基于 MuJoCo 3.4.0 物理仿真引擎的 **3自由度机械臂精准取放系统**，支持自动复位、抓取计时、速度调节和力度控制等功能。

机械臂由三个关节组成：
- **关节1（基座旋转）**：绕 Z 轴旋转，范围 [-3.14, 3.14] 弧度
- **关节2（大臂俯仰）**：绕 Y 轴俯仰，范围 [-1.5, 1.5] 弧度
- **关节3（小臂伸缩）**：沿 X 轴滑动，范围 [0, 0.4] 米

末端配备平行夹爪，用于抓取蓝色球体目标并放置到绿色目标区域。

## 修改内容

将机械臂的 MuJoCo XML 模型从 `main.py` 中分离到独立的 `robot_arm.xml` 文件，提高代码可维护性。

### 修改前

XML 模型以字符串形式内嵌在 `main.py` 的 `robot_xml` 变量中：

```python
def robot_arm_auto_reset_demo():
    robot_xml = """
    <mujoco model="3-DOF Robot Arm with Auto Reset">
      ...
    </mujoco>
    """
    model = mujoco.MjModel.from_xml_string(robot_xml)
```

### 修改后

XML 模型独立存储在 `robot_arm.xml` 文件中，通过文件路径加载：

```python
model = mujoco.MjModel.from_xml_path("robot_arm.xml")
```

## 环境依赖

- Python 3.8+
- MuJoCo 3.4.0+
- NumPy

安装依赖：

```bash
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

## 使用方法

### 基本运行

```bash
python main.py
```

### 命令行参数

```bash
python main.py [抓取次数] [速度模式] [最低成功率] [夹爪力度]
```

| 参数 | 说明 | 可选值 | 默认值 |
|------|------|--------|--------|
| 抓取次数 | 机械臂抓取的总轮数 | 任意正整数 | 5 |
| 速度模式 | 关节运动速度 | slow / medium / fast | medium |
| 最低成功率 | 成功率达到此阈值才返回成功 | 0~100 浮点数 | 0（不检查） |
| 夹爪力度 | 夹爪闭合力度 | light / medium / heavy | medium |

### 使用示例

```bash
# 默认配置：抓取5次，中速，中力度
python main.py

# 快速抓取10次，要求80%成功率，重力度
python main.py 10 fast 80 heavy

# 慢速抓取3次，轻力度
python main.py 3 slow 0 light
```

## 代码架构

### 主要文件

| 文件 | 说明 |
|------|------|
| `main.py` | 主程序入口，包含完整仿真流程和控制逻辑 |
| `robot_arm.xml` | MuJoCo 机械臂模型定义文件 |
| `robot_arm_control.py` | 机械臂控制接口（预留扩展） |

### 配置参数

#### 速度配置 `SPEED_CONFIG`

| 模式 | 关节运动时长 | 夹爪动作时长 | 暂停间隔 |
|------|-------------|-------------|----------|
| slow | 3.0s | 1.5s | 0.02s |
| medium | 2.0s | 1.0s | 0.001s |
| fast | 1.0s | 0.5s | 0.0005s |

#### 夹爪力度配置 `GRIP_CONFIG`

| 力度 | 闭合速度 |
|------|---------|
| light | 0.15 |
| medium | 0.25 |
| heavy | 0.35 |

### 核心函数

| 函数 | 说明 |
|------|------|
| `robot_arm_auto_reset_demo()` | 主入口，初始化模型和仿真流程 |
| `joint_move(joint_name, target_val, duration, viewer, step_desc)` | 单关节精准移动，带进度条显示 |
| `gripper_close(viewer, desc, force)` | 闭合夹爪抓取目标，支持力度调节 |
| `gripper_open(viewer, desc)` | 张开夹爪放置目标 |
| `robot_auto_reset(viewer)` | 机械臂自动复位到初始位置 |
| `target_auto_reset(viewer)` | 目标物体随机重置到新位置 |
| `grab_and_place(viewer, retry_max, speed, grip_force)` | 完整取放流程，含重试机制和计时 |

### 抓取流程

1. **对准目标**：旋转基座 → 俯仰大臂 → 伸缩小臂，对准蓝色球体
2. **抓取目标**：闭合夹爪抓取球体
3. **转移目标**：抬升大臂 → 旋转基座 → 降低大臂，移动到绿色区域
4. **放置目标**：张开夹爪释放球体
5. **自动复位**：机械臂和目标各自复位，准备下一次抓取

## 运行效果

程序运行后会：

- 打开 MuJoCo 可视化窗口，展示3自由度机械臂仿真
- 控制台实时显示进度条、力矩信息、抓取状态
- 每次抓取完成后输出计时统计
- 所有抓取完成后输出总成功率、总耗时和平均耗时
- 自动将日志保存到 `grasp_log.txt` 文件

## 文件结构

```
src/mechanical_arm_grasping/
├── README.md              # 模块文档（本文件）
├── main.py                # 主程序
├── robot_arm.xml          # MuJoCo 机械臂模型
├── robot_arm_control.py   # 控制接口
└── grasp_log.txt          # 运行日志（自动生成）
```
