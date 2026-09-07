# 期末考评1：教具无人机外挂负载PID调参

### 教学示例代码

|下载链接|代码说明|
|---|---|
|<a href="../file/score_code.zip">📥 score_code</a>|自主评分程序|

# 课程任务

本课程要求学生在教具无人机上安装规定重量的外挂配重，用以模拟相机、激光雷达、计算单元等传感器或机载设备。在载荷发生变化后，学生需要通过调整 PX4 飞控 PID 参数以及上位机 px4ctrl 控制参数，使无人机在规定测试条件下保持稳定飞行，并获得良好的轨迹跟踪、控制响应和振动性能。

课程考核分为两个独立部分：**Bag 轨迹性能考核**和 **PX4 ULog 飞控性能考核**。两部分均采用 100 分制独立评分，互不依赖——Bag 与 ULog 不要求来自同一次飞行，也不会自动合并为一个总分。

# 统一测试要求

为保证不同学生、不同参数方案之间的可比性，所有测试均在以下统一条件下进行：

1. 所有学生使用相同型号的教具无人机、桨叶、电池、定位系统和软件版本。

2. 外挂配重的质量、安装位置和固定方式由课程统一规定，测试时记录实际载荷质量。

3. Bag 轨迹考核统一执行规定参数的 8 字轨迹，保证轨迹持续时间、最大速度和几何形状一致。

4. ULog 飞控性能考核使用课程规定的标准飞行动作，保证不同学生的测试动态强度具有可比性。

5. 学生主要通过调整 PX4 PID、px4ctrl 控制参数以及课程允许的滤波参数改善飞行性能。

# Bag 轨迹性能考核

Bag 部分用于评价无人机在标准 8 字轨迹上的轨迹跟踪质量和动态响应。实际状态读取自 `/MVD/odom`，期望位置和速度读取自 `/setpoints_cmd`。评分程序根据消息时间戳完成时间对齐，并自动截取有效运动区间。

## 数据与轨迹有效性

在进入具体指标评分之前，程序首先检查数据完整性和轨迹有效性，两项均为二值评分（满足即得满分，不满足得 0 分）。

|项目|分值|判定方式|说明|
|---|---|---|---|
|数据完整性|4|二值评分|/MVD/odom 与 /setpoints\_cmd 数据可正常读取并具有有效时间重叠。|
|轨迹完整性|8|二值评分|有效轨迹持续时间不少于 18 s，且期望轨迹路径长度不少于 5 m。|

## 位置、路径与速度评分

通过有效性检查后，程序从位置精度、路径跟踪、速度匹配和响应延迟四个维度对轨迹性能进行连续型评分。各项指标及阈值如下：

|指标|分值|满分阈值|0 分阈值|含义|
|---|---|---|---|---|
|Position RMSE|30|≤ 0\.05 m|≥ 0\.35 m|三维实际位置与同一时刻期望位置的均方根误差。|
|Position P95|10|≤ 0\.08 m|≥ 0\.40 m|95% 采样点的位置误差不超过该值，用于反映大部分时间的跟踪稳定性。|
|Position Max|6|≤ 0\.15 m|≥ 0\.80 m|整段轨迹中的最大瞬时位置误差。|
|Cross\-track RMSE|13|≤ 0\.05 m|≥ 0\.30 m|实际位置到期望 8 字几何曲线最近点距离的均方根误差。|
|Velocity RMSE|19|≤ 0\.10 m/s|≥ 0\.50 m/s|三维实际速度与期望速度的均方根误差。|
|Tracking Delay|10|≤ 0\.05 s|≥ 0\.30 s|通过时间平移寻找实际轨迹与期望轨迹最佳匹配位置，得到整体响应延迟。|

## Bag 总分

Bag 考核满分为 100 分，各项权重之和为 100 分。连续型指标采用"越小越好"的线性评分方式：当指标优于满分阈值时获得该项满分，当指标达到或超过 0 分阈值时该项得 0 分，中间区域线性扣分。

<div style="background-color:#edf2fc; border:1px solid #94b8f0; padding:20px 24px; margin:16px 0; border-radius:12px; color:#222; line-height:2.4;">
📐 <strong>连续型指标评分公式：</strong>对于某一连续型指标 x，设该项满分为 W，满分阈值为 E，0 分阈值为 F，则评分为：
  <div style="margin-top:10px;padding-left:8px;">
    x ≤ E 时得 W 分；E &lt; x &lt; F 时得 W × (F − x) / (F − E) 分；x ≥ F 时得 0 分。
  </div>
</div>


# PX4 ULog 飞控性能考核

ULog 部分只评价飞控控制响应速度和高频振动质量，**不评价**姿态误差、角速度误差、P95、Failsafe、Motor Saturation 或 Gyro Clipping。ULog 评分与 Bag 评分独立，满分同样为 100 分。

## 姿态响应延迟

姿态响应延迟通过 `vehicle_attitude_setpoint` 与 `vehicle_attitude` 进行比较。程序分别估计 Roll 与 Pitch 的最佳时间偏移，Yaw 仅作为诊断信息，不参与课程评分。用于评分的姿态延迟取 Roll、Pitch 两轴绝对延迟中的较大值。

|指标|分值|满分阈值|0 分阈值|说明|
|---|---|---|---|---|
|姿态响应延迟|20|≤ 0\.05 s|≥ 0\.30 s|Roll/Pitch 姿态响应延迟的较大值，反映姿态控制链路的整体响应速度。|
|角速度响应延迟|20|≤ 0\.03 s|≥ 0\.20 s|vehicle\_rates\_setpoint 与 vehicle\_angular\_velocity 的最佳时间偏移，反映最内层角速度环响应速度。|

## 振动性能

振动评分主要检查高频机械振动和高频电机控制输出。Gyro 高频振动从 `sensor_gyro_fifo` 数据计算频谱并统计指定频带内 RMS；Actuator 高频振动从 `actuator_motors.control` 计算高频频带能量。

|指标|分值|分析频带|满分阈值|0 分阈值|含义|
|---|---|---|---|---|---|
|Gyro 高频振动|40|80\~900 Hz|≤ 0\.08 rad/s|≥ 0\.50 rad/s|反映机架、桨叶、电机、外挂负载及滤波对 IMU 高频振动的影响。|
|Actuator 高频振动|20|20\~150 Hz|≤ 0\.015|≥ 0\.10|反映飞控输出中高频抖动程度，可用于观察控制器是否过激或噪声是否被放大。|

## ULog 总分

ULog 考核满分为 100 分，其中姿态响应延迟 20 分、角速度响应延迟 20 分、Gyro 高频振动 40 分、Actuator 高频振动 20 分。四项均采用"越小越好"的线性评分方式，评分规则与 Bag 部分连续型指标一致。

# 成绩等级

Bag 与 ULog 分别按照以下等级规则独立给出成绩。除非课程另行规定，不对两项成绩进行自动加权或合并。

|分数范围|等级|
|---|---|
|90\~100|A|
|80\~\<90|B|
|70\~\<80|C|
|60\~\<70|D|
|\<60|F|

# 效果演示

<p >
  <video width="950" controls>
    <source src="https://diffrobots.oss-cn-hangzhou.aliyuncs.com/nju-wiki/%E5%A4%96%E6%8C%82%E8%B4%9F%E8%BD%BDPID%E8%B0%83%E5%8F%82/8%E5%AD%97%E9%A3%9E%E8%A1%8C.mp4" type="video/mp4">
  </video>
</p>

# 提交材料

学生需在课程截止日期前提交以下全部材料：

1. 标准 8 字轨迹飞行 ROS Bag。

2. 用于飞控性能分析的 PX4 ULog。

3. 最终使用的 PX4 PID 与 px4ctrl 参数。

4. 外挂载荷质量及安装说明。

5. 自动评分程序生成的 Bag 评分结果和 ULog 评分结果。

6. 飞行演示视频。

# 考核目的

通过统一外挂载荷和统一测试条件，使学生能够根据飞行日志定量分析控制性能，并通过调整 PX4 与 px4ctrl 参数改善轨迹跟踪、控制响应和振动表现。评分标准采用自动化、连续化方式，减少人工主观判断，使不同学生、不同参数方案之间具有可重复和可比较的评价依据。


## 补充：飞行数据评分工具使用说明

## 1. 功能说明

工具包含两个相互独立的评分程序：

| 程序 | 输入数据 | 评分内容 | 满分 |
| --- | --- | --- | ---: |
| `bag_score.py` | ROS1 `.bag` | 轨迹完整性、位置和速度跟踪、响应延迟 | 100 分 |
| `ulog_score.py` | PX4 `.ulg` | 姿态和角速度延迟、陀螺仪和电机振动 | 100 分 |

Bag 与 ULog 分别评分，不要求来自同一次飞行，也不会自动合并成绩。

## 2. 文件摆放

建议使用以下目录结构：

```text
Score_Code/
├── bag_score.py
├── bag_scoring.yaml
├── ulog_score.py
├── ulog_scoring.yaml
├── requirements.txt
├── data/
│   ├── flight.bag
│   └── flight.ulg
└── results/
```

将需要处理的 `.bag` 和 `.ulg` 文件放入 `data/`。文件也可以放在其他位置，只需在命令中填写正确路径。

`data`、`results`文件夹需要自己手动创建。

## 3. 安装环境

推荐使用 Ubuntu 20.04、Python 3 和 ROS Noetic。

进入程序目录：

```bash
cd ~/Score_Code
```

安装 Python 依赖：

```bash
python3 -m pip install -r requirements.txt
```

Bag 分析还需要 ROS Noetic。`rosbag` 由 ROS 提供，不要使用 pip 安装。

## 4. 处理单个 Bag 文件

### 4.1 检查数据

```bash
source /opt/ros/noetic/setup.bash
source ~/catkin_ws/devel/setup.bash
rosbag info data/flight.bag
```

默认需要：

- `/MVD/odom`：实际位置和速度，消息需兼容 `nav_msgs/Odometry`。
- `/setpoints_cmd`：期望位置和速度，消息中需包含 `position` 和 `velocity` 字段。

如果没有使用自定义消息工作空间，可省略第二条 `source` 命令。

### 4.2 运行评分

```bash
python3 bag_score.py \
  --bag data/flight.bag \
  --config bag_scoring.yaml \
  --output results/bag_result
```

如果实际主题名称不同：

```bash
python3 bag_score.py \
  --bag data/flight.bag \
  --config bag_scoring.yaml \
  --odom-topic /实际里程计主题 \
  --setpoint-topic /实际期望轨迹主题 \
  --output results/bag_result
```

## 5. 处理单个 ULog 文件

运行评分：

```bash
python3 ulog_score.py \
  --ulg data/flight.ulg \
  --config ulog_scoring.yaml \
  --output results/ulog_result
```

ULog 建议包含以下 PX4 数据：

- `vehicle_attitude`
- `vehicle_attitude_setpoint`
- `vehicle_angular_velocity`
- `vehicle_rates_setpoint`
- `sensor_gyro_fifo`
- `actuator_motors`

数据缺失时，对应指标可能按 0 分处理，并在结果中给出警告。

## 6. 常用可选参数

记录载荷质量，例如 200 g：

```bash
--payload 200
```

`--payload` 只写入结果，不参与评分。

不生成 PNG 图片：

```bash
--no-plots
```

完整示例：

```bash
python3 ulog_score.py \
  --ulg data/flight.ulg \
  --config ulog_scoring.yaml \
  --payload 200 \
  --no-plots \
  --output results/ulog_result
```

## 7. 查看结果

运行完成后：

```text
results/
├── bag_result/
│   ├── score_summary.json
│   ├── score_summary.csv
│   └── *.png
└── ulog_result/
    ├── score_summary.json
    ├── score_summary.csv
    └── *.png
```

- `score_summary.json`：完整评分结果和分析指标。
- `score_summary.csv`：各评分项的数值、得分和满分。
- `*.png`：轨迹、误差、延迟和振动图。

建议先查看终端是否有报错或警告，再检查 CSV 中的 `missing` 列：

- `missing=0`：指标计算正常。
- `missing=1`：指标缺失，该项按 0 分处理。

## 8. 常见问题

### 无法导入 `rosbag`

```bash
source /opt/ros/noetic/setup.bash
source ~/catkin_ws/devel/setup.bash
```

### 提示 `Missing bag topic`

使用下面的命令查看真实主题名：

```bash
rosbag info data/flight.bag
```

然后通过 `--odom-topic` 和 `--setpoint-topic` 指定。

### ULog 某项为 0 分

查看终端警告、`score_summary.json` 中的 `warnings`，以及 CSV 中的 `missing` 列。通常是 ULog 缺少对应数据、采样率不足或飞行中的姿态变化太小。

### 显示详细报错

```bash
UAV_SCORE_DEBUG=1 python3 bag_score.py \
  --bag data/flight.bag \
  --config bag_scoring.yaml \
  --output results/bag_result
```

ULog 脚本也可以使用相同方式调试。

## 9. 注意事项

- 修改 YAML 中的阈值会直接改变评分结果。
- YAML 中所有评分项的 `weight` 总和必须等于 `meta.max_score`。
- 正式评分时应固定配置文件版本，并保存原始 Bag 和 ULog 文件。
- 输出目录中已有的同名结果文件会被覆盖，建议每次使用独立的结果目录。
