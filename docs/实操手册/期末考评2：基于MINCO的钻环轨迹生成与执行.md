# 期末考评2：基于MINCO的钻环轨迹生成与执行

### 教学示例代码

|下载链接|代码说明|
|---|---|
|<a href="../file/fly_ws.zip">📥 fly_ws</a>|fly_ws 穿环示例代码包|

# 1\. 考核任务概述

本课程最终考核要求学生综合运用无人机轨迹规划、轨迹优化、飞行控制以及机器人系统集成等课程内容，完成基于 **MINCO（Minimum Control）轨迹生成方法**的无人机自主穿环任务。.

实验场地中设置 **4 个空间圆环**。学生需要根据圆环的位置、朝向以及无人机起始位置，自主设计 MINCO 轨迹的中间航点及相关轨迹参数，使生成的连续轨迹依次穿过全部圆环，并通过轨迹控制接口将生成的轨迹发送至 PX4Ctrl，控制无人机自主完成：

**起飞 → 加速 → 连续穿越四个圆环 → 飞向终点 → 降落**

<div style="background-color:#fffde0; border:1px solid #ffbc69; padding:18px 22px; margin:16px 0; border-radius:12px; color:#222; line-height:2.2;">
💡 <strong>整个任务原则上不得依赖飞行过程中的人工遥控干预。</strong>
</div>

考核重点不仅是无人机能否完成穿环，同时考察学生是否能够理解并合理利用 MINCO 轨迹的以下性质：

1. 中间航点对轨迹几何形状的影响；

2. 分段时间对速度、加速度和轨迹形态的影响；

3. 最大速度、最大加速度等动力学约束对轨迹的影响；

4. 轨迹平滑性与任务完成时间之间的权衡；

5. 轨迹规划模块与飞行控制系统之间的数据接口；

6. 完整自主任务流程的设计和实现。

# 2\. 实验任务要求

## 2\.1 基础任务

实验环境中设置编号为 Gate 1、Gate 2、Gate 3、Gate 4，共 4 个圆环。

学生需要规划一条从指定起飞位置出发的连续轨迹，并按照规定顺序依次穿过四个圆环。完成所有圆环穿越之后，无人机需要继续飞行至指定终点，并执行降落。

<div style="background-color:#ffeeee; border:1px solid #f28484; padding:18px 22px; margin:16px 0; border-radius:12px; color:#222; line-height:2.2;">
🚫 <strong>不得通过跳过圆环、改变规定穿环顺序等方式完成任务。</strong>
</div>


## 2\.2 MINCO 轨迹规划要求

学生应使用课程提供的 MINCO 轨迹生成框架完成轨迹规划，至少需要完成以下参数的设计或调整：

### （1）中间航点

根据圆环的空间位置及朝向设置合理的中间航点，使 MINCO 生成的连续轨迹能够正确穿过各圆环。中间航点不要求必须设置在圆环中心，但学生应考虑：

- 圆环法向方向；

- 进入圆环的飞行方向；

- 离开圆环后的飞行方向；

- 前后圆环之间的空间关系；

- 高速飞行情况下轨迹曲率变化。

<div style="background-color:#edf2fc; border:1px solid #94b8f0; padding:16px 20px; margin:16px 0; border-radius:12px; color:#222; line-height:2.2;">
💡 鼓励学生通过合理设置圆环前后的引导航点，而不是简单将所有圆环中心作为轨迹插值点。
</div>


### （2）轨迹分段时间

对于由 \(M\) 个轨迹段组成的轨迹，需要合理设置或优化各段飞行时间。轨迹时间过长会导致飞行速度较低、整体任务完成时间增加；轨迹时间过短则可能导致动力学约束超限或跟踪误差增大，并可能造成实际飞行中碰撞圆环。因此应根据实际轨迹综合调整时间分配。

## 2\.3 轨迹动力学要求

本实验规定无人机允许的最大规划速度，原则上规划轨迹不得超过该速度。推荐同时考虑最大加速度约束，具体数值可根据实验无人机性能和课程实验平台统一设置。

<div style="background-color:#fff8e8; border:1px solid #ffcc88; padding:18px 22px; margin:16px 0; border-radius:12px; color:#222; line-height:2.2;">
💡 实际考核过程中应保证飞行系统处于稳定、可控状态，不允许通过明显超出平台安全能力的轨迹获得更高成绩。
</div>


# 3\. 有效穿环判定

为避免不同实验人员对"穿过圆环"的判定标准不同，采用统一的穿环判定方法。

设圆环中心、圆环所在平面的单位法向量、无人机位置分别为对应参数。当轨迹从圆环平面的一侧运动到另一侧，并且轨迹与圆环平面的交点位于圆环有效内部区域时，判定为成功穿环。

实际实验中可采用位置记录、动作捕捉系统、定位系统或现场观察进行综合判定。

## 3\.1 成功穿环

同时满足以下条件，则该圆环记为 **有效穿环**：

1. 无人机按照规定顺序穿过该圆环；

2. 无人机主体完整通过圆环；

3. 无人机、桨叶或其他结构未与圆环发生明显碰撞；

4. 穿环过程中无人机未发生失控。

## 3\.2 无效穿环

出现以下任一情况，该圆环不得获得完整穿环得分：

- 未进入圆环内部；

- 从圆环外侧绕过；

- 飞行顺序错误；

- 无人机与圆环发生明显碰撞；

- 穿环过程中人工遥控修正位置；

- 无人机严重失控后偶然穿过圆环。

# 4\. 任务执行要求

最终应实现完整自主任务流程。轨迹生成模块需要将 MINCO 轨迹转换为 PX4Ctrl 所需要的轨迹指令。

轨迹指令至少应包含系统所要求的：

- 位置；

- 速度；

- 加速度。

如果实验接口支持，也可以进一步提供：

- jerk；

- yaw；

- yaw rate；

- trajectory time。

学生需要保证轨迹时间戳与控制系统时间正确对应。

# 5\. 评分标准

课程最终考核总分由以下四部分组成：

|**考核项目**|**分值**|
|---|---|
|圆环穿越完成度|60 分|
|轨迹性能与参数优化|15 分|
|全流程自主执行|20 分|
|系统实现与任务规范|5 分|
|**总计**|**100 分**|

# 6\. 圆环穿越完成度——60 分

四个圆环分别独立计分。每成功穿过一个圆环：15 分，四个圆环共：60 分。

|**单个圆环完成情况**|**得分**|
|---|---|
|正确、连续、无碰撞穿过圆环|15|
|成功穿过，但与圆环发生轻微接触|10|
|穿环过程中发生明显碰撞但无人机仍继续飞行|5|
|未成功穿过|0|

<div style="background-color:#fffde0; border:1px solid #ffbc69; padding:18px 22px; margin:16px 0; border-radius:12px; color:#222; line-height:2.2;">
💡 如果由于前一个圆环碰撞导致后续任务终止，则后续未完成圆环不得分。
</div>


---

# 7\. 轨迹性能与参数优化——15 分

本部分重点考察：**在保证安全穿环的前提下，提高轨迹飞行速度并保持合理的轨迹平滑性。**

<div style="background-color:#eef2fc; border:1px solid #88a8f0; padding:18px 22px; margin:16px 0; border-radius:12px; color:#222; line-height:2.2;">
📌 不以简单修改程序中的 max_vel 参数作为评分依据，而以实际执行轨迹的性能作为评价依据。
</div>


## 7\.1 飞行速度与任务时间——10 分

根据学生最终稳定完成任务所采用的轨迹速度等级评分。

|**稳定完成任务时的轨迹速度等级**|**得分**|
|---|---|
|最大速度稳定为5m/s左右|10|
|4\.0m/s到4\.5m/s|8|
|3\.0m/s到4\.0m/s|6|
|2\.0m/s到3\.0m/s|4|
|1\.0m/s到2\.0m/s|2|
|低于 1\.0 m/s|0|

<div style="background-color:#fffde0; border:1px solid #ffbc69; padding:18px 22px; margin:16px 0; border-radius:12px; color:#222; line-height:2.2;">
💡 速度得分必须建立在无人机成功完成相应轨迹段且没有因速度过高产生碰撞或明显失控的基础上。
</div>

例如：学生将最大速度设置为较高值，但由于轨迹参数不合理造成撞环，则不能仅凭参数设置获得 10 分。

## 7\.2 轨迹平滑性——5 分

根据无人机实际飞行状态以及轨迹速度、加速度变化进行评价。

### 5 分

轨迹整体连续平滑：

- 无明显速度突变；

- 无明显加速度突变；

- 无明显轨迹振荡；

- 穿环前后姿态变化自然；

- 无明显过冲。

### 3～4 分

能够完成任务，但存在：

- 部分位置曲率较大；

- 部分加减速过快；

- 穿环附近存在轻微摆动；

- 局部跟踪误差较大。

### 1～2 分

轨迹存在明显：

- 急停；

- 急转；

- 振荡；

- 大幅超调。

但仍基本能够执行。

### 0 分

轨迹存在严重不连续或者不可执行现象。

---

# 8\. 全流程自主执行——20 分

该部分考察系统集成能力。

## 8\.1 完整一键任务——20 分

程序启动后，只允许进行一次任务启动操作。之后无人机自动完成整个过程，期间：

- 不需要修改参数；

- 不需要重新启动节点；

- 不需要人工发送新的目标点；

- 不需要人工切换任务阶段；

- 不需要遥控修正无人机轨迹。

满足以上要求可得满分。

## 8\.2 分阶段自动运行——15 分

例如：

1. 人工启动起飞；

2. 自动完成四环穿越；

3. 人工触发降落。

或者需要人工执行少量任务阶段切换，根据人工干预程度确定具体分数。

## 8\.3 仅实现自动穿环——5～10 分

能够自动生成并执行 MINCO 轨迹完成穿环，但：

- 起飞依赖人工；

- 降落依赖人工；

- 中途需要多次人工启动轨迹。

根据完成程度获得相应分数。

## 8\.4 主要依赖人工控制——0～5 分

若轨迹执行过程中需要频繁遥控干预，则本部分最高 5 分。

---

# 9\. 系统实现与任务规范——5 分

主要考察程序完整性与实验规范。

## 9\.1 MINCO 轨迹正确使用——2 分

应能够说明并正确设置：

- 中间航点；

- 轨迹时间；

- 最大速度；

- 必要的轨迹约束。

<div style="background-color:#fef0f0; border:1px solid #f29494; padding:18px 22px; margin:16px 0; border-radius:12px; color:#222; line-height:2.2;">
🚫 不能仅直接使用教师预先提供的最终航点和参数。
</div>


## 9\.2 PX4Ctrl 轨迹接口正确——2 分

要求：

- 轨迹消息发送正常；

- 时间同步正确；

- 控制过程中不存在明显错误指令；

- 轨迹结束后能够正确退出或进入下一任务状态。

## 9\.3 实验程序规范——1 分

程序应至少能够明确显示：

- 当前任务状态；

- 当前穿环编号；

- 轨迹是否生成成功；

- 飞行任务是否完成。

---

# 10\. 附加规则

## 10\.1 碰撞规则

发生轻微圆环接触时，只扣除对应圆环分数。如果碰撞造成：

- 无人机坠落；

- 安全保护触发；

- 飞行任务被迫终止。

则本次飞行结束，并按照已经完成的任务计算成绩。

## 10\.2 人工干预

为保证安全，教师或学生始终可以执行紧急停止。由于安全原因执行紧急停止不会产生额外处罚，但本次任务中尚未完成的项目不得获得对应成绩。

## 10\.3 多次测试

正式考核建议允许学生进行多次正式飞行，最终成绩可采用 **最佳一次完整有效飞行成绩**。

若希望同时考察系统稳定性，也可以规定：

- 每名学生正式飞行 2 次；

- 取两次平均成绩。

以降低偶然成功对成绩的影响。

---

# 11\. 建议的考核流程

正式考核按照以下流程进行。

### 第一阶段：程序准备

学生完成：

- 圆环位置配置；

- MINCO 中间航点配置；

- 最大速度配置；

- 轨迹时间参数配置。

### 第二阶段：启动任务

学生执行规定的一键启动命令，开始正式计时。

### 第三阶段：自主飞行

无人机自主完成：

1. 起飞；

2. Gate 1；

3. Gate 2；

4. Gate 3；

5. Gate 4；

6. 飞向终点；

7. 降落。

教师记录：

- 穿环数量；

- 碰撞次数；

- 最大速度；

- 飞行状态；

- 人工干预次数；

- 总任务时间。

### 第四阶段：任务结束

无人机成功降落后停止计时，记录最终成绩。

---

# 12\. 推荐成绩记录表

|**项目**|**指标**|**满分**|**实际得分**|
|---|---|---|---|
|Gate 1|无碰撞穿越|15||
|Gate 2|无碰撞穿越|15||
|Gate 3|无碰撞穿越|15||
|Gate 4|无碰撞穿越|15||
|飞行速度|不超过 5 m/s|10||
|轨迹平滑性|速度、加速度及实际跟踪状态|5||
|一键自主任务|起飞—穿环—降落|20||
|MINCO 参数设置|航点、时间及约束|2||
|PX4Ctrl 接口|轨迹发送与执行|2||
|程序规范|状态输出及完整性|1||
|**总分**||**100**||

---

# 13\. 考核目标

通过本次考核，学生应能够理解：MINCO 并不仅是一个"经过若干航点的曲线生成器"，而是通过合理选择中间航点和轨迹参数，实现连续、高阶平滑并满足无人机动力学要求的轨迹生成方法。

学生需要在飞行速度、轨迹平滑性与任务安全性之间进行合理权衡。最终目标是在确保安全和稳定跟踪的前提下，通过合理设置 MINCO 中间航点和轨迹参数，使无人机以尽可能高的速度、尽可能平滑的轨迹完成四圆环连续自主穿越，并实现完整的起飞、穿环和降落自主任务流程。

---

# 14\. 效果演示

<div style="background-color:#edf1ff; border:1px solid #93b4f7; padding:18px 22px; margin:16px 0; border-radius:12px; color:#222; line-height:2.2;">
💡 本视频演示仅展示了无人机穿越两个圆环的效果，实际考核任务要求无人机依次穿越全部四个圆环。
</div>

<p >
  <video width="750" controls>
    <source src="https://diffrobots.oss-cn-hangzhou.aliyuncs.com/nju-wiki/%E5%9F%BA%E4%BA%8EMinco%E7%9A%84%E9%92%BB%E7%8E%AF%E8%BD%A8%E8%BF%B9%E7%94%9F%E6%88%90%E4%B8%8E%E6%89%A7%E8%A1%8C/%E7%A9%BF%E7%8E%AF.mp4" type="video/mp4">
  </video>
</p>

# `fly_ws`使用教程

## 1. 放置与目录

ZIP 解压后是 `fly_ws/{src, sh_file}`，直接放到家目录：

```bash
unzip fly_ws.zip -d ~/          # 得到 ~/fly_ws
cd ~/fly_ws
ls                              # 应看到 src  sh_file  README.md  README_MOCAP.md
```

## 2. 编译

### 2.1 依赖

```bash
sudo apt install -y \
  ros-noetic-mavros ros-noetic-mavros-extras \
  ros-noetic-vrpn \
  ros-noetic-cmake-modules \
  ros-noetic-gazebo-msgs \
  ros-noetic-rviz \
  libeigen3-dev \
  libarmadillo-dev
```

| 依赖 | 谁要它 | 说明 |
|---|---|---|
| `ros-noetic-mavros` | `px4ctrl` | 下位机通信 |
| `ros-noetic-vrpn` | `vrpn_client_ros` | 提供 `VRPN` 的 cmake 配置 |
| `ros-noetic-cmake-modules` | `uav_utils`、`px4ctrl` | `build_depend` 里点名要 |
| `ros-noetic-gazebo-msgs` | `groundtruth` | 只在 SITL 用 |
| `ros-noetic-rviz` | `decomp_ros_utils` | rviz 插件，Qt5 硬依赖 |
| `libeigen3-dev` | `trajectory`/`traj_server`/`px4ctrl`/`decomp_ros_utils` | — |
| `libarmadillo-dev` | **`pose_utils`** | CMake 里 `find_package(Armadillo REQUIRED)`，package.xml 却没声明，rosdep 查不出来 |

### 2.2 还原顶层 `src/CMakeLists.txt`（唯一必须动的工作空间文件）

**这不是包，是 catkin 工作空间顶层文件。** 压缩包里它被清成了 0 字节（打包时把软链打坏了）。0 字节不会报错，但会让 `catkin_make` 配出一个空工程——编完 `rospack` 一个包都找不到。

```bash
cd ~/fly_ws/src
wc -c CMakeLists.txt            # 若是 0，按下面处理
rm -f CMakeLists.txt
catkin_init_workspace           # 生成标准 toplevel.cmake
cd ~/fly_ws
```

`catkin_init_workspace` 需要先 `source /opt/ros/noetic/setup.bash`。

### 2.3 全量编译（13 个可编译包）

```bash
source /opt/ros/noetic/setup.bash
cd ~/fly_ws
catkin_make -DCATKIN_WHITELIST_PACKAGES="" -j4
```

- 不用白名单，`src` 下**所有能编译的包全部编**；白名单只在你之前用过、`CMakeCache.txt` 里残留旧值时才需要显式传空串。
- `decomp_ros_utils` 配置阶段会打印 `Using Qt5 based on the rviz_QT_VERSION: 5.12.8`，属正常。
- 结尾应是 `[100%] Built target ...`，没有 `Error` / `Failed`。

日常只改了一个包时增量编：

```bash
catkin_make --pkg px4ctrl
catkin_make --pkg trajectory
```

### 2.5 验收

```bash
source ~/fly_ws/devel/setup.bash

# ① 包数量：应为 13
rospack list | grep -E \
 "^(trajectory|traj_server|catkin_simple|cmake_utils|decomp_ros_msgs|decomp_ros_utils|pose_utils|quadrotor_msgs|uav_utils|groundtruth|vrpn_client_ros|vrpn_process|px4ctrl) " | wc -l

# ② 产物清单
find devel/lib -maxdepth 2 -type f | sort

```

| 包 | 期望产物 |
|---|---|
| `trajectory` | `devel/lib/trajectory/traj_gen`、`libtrajectory.so`、`libminco.so` |
| `traj_server` | `devel/lib/traj_server/traj_server_node` |
| `px4ctrl` | `devel/lib/px4ctrl/px4ctrl_node` |
| `quadrotor_msgs` | `libencode_msgs.so`、`libdecode_msgs.so`、`devel/include/quadrotor_msgs/*.h`（26 个消息） |
| `uav_utils` | 只有头文件（无 .so，正常） |
| `cmake_utils` | 只有 cmake 宏与 `FindEigen.cmake`（无产物，正常） |
| `catkin_simple` | 只有 cmake 宏（无产物，正常） |
| `pose_utils` | `devel/lib/pose_utils/libpose_utils.so` |
| `decomp_ros_msgs` | `devel/lib/python3/dist-packages/decomp_ros_msgs/` + `devel/include/decomp_ros_msgs/` |
| `decomp_ros_utils` | `devel/lib/decomp_ros_utils/libdecomp_rviz_plugins.so` |
| `groundtruth` | `devel/lib/groundtruth/gazebo_odom_node` |
| `vrpn_client_ros` | `libvrpn_client_ros.so`、`vrpn_client_node`、`vrpn_tracker_node` |
| `vrpn_process` | `devel/lib/vrpn_process/fusion_odom` |

`uav_utils` / `cmake_utils` / `catkin_simple` 本来就不产出二进制，别因为找不到 `.so` 以为没编过——看 `rospack find` 能否定位到它们即可。

## 3. 穿环：配置流程

穿环不是"改个参数就行"，**几何决定成败**。代码只做一件事：把 `rings` 里的环心按顺序串成一条 MINCO 轨迹，逐点飞过去。

### 3.1 先把环摆对（比调参重要得多）

代码是**只飞环心**模式（`gateApproachDist` 不生效），意味着：

```
起点 → c1 → c2 → c3 → c4
```

每一段的出入方向 = **相邻两个环心的连线方向**。所以：

| 规则 | 原因 |
|---|---|
| 环面法向 ≈ 来向（上一个环心 → 本环心的方向） | 否则飞机斜着切进环框，桨叶必打框 |
| 相邻环心间距 ≥ 1.5 m | 太近没有转弯余量，MINCO 会被 `maxAcc` 顶住 |
| 所有环心高度接近（±0.3 m） | 高度突变会变成"爬升+下降"，浪费航程且易失速 |
| 首环离起飞点 1.5~2 m，且在起飞点正前方 | 给第一段留加速距离，也避免起飞就贴框 |
| 环心按**飞行顺序**编号记录 | 顺序写错 = 轨迹乱穿 |

### 3.2 量环心

**方法：临时看话题**

**前置条件：在动捕启动、定位无误的情况下，通过话题进行查看。**

```bash
rostopic echo -n1 /MVD/odom/pose/pose/position
```

### 3.3 写进 `traj.yaml`

文件：`src/8fly/src/trajectory/parameters/traj.yaml`

```yaml
shape: 4                    # ← 必须改成 4，否则永远飞八字

# 穿环段 此处x、y、z仅作演示使用，不可照搬教学示例中的配置
ringsReturnHome: false      # false: 停在最后一个环；true: 原路返回起点
rings:
  - {x: 2.000, y: 0.000, z: 1.300, nx: 1.0, ny: 0.0, nz: 0.0}
  - {x: 4.000, y: 1.600, z: 1.400, nx: 0.0, ny: 1.0, nz: 0.0}
  - {x: 6.000, y: 0.000, z: 1.300, nx: 1.0, ny: 0.0, nz: 0.0}
```

三个必须知道的点：

1. **`z` 是绝对高度**（动捕世界系），不是相对起飞点。起飞高度设 1.5、环心 z=1.3，飞机会先降 0.2 m；
2. `nx/ny/nz` 在只飞环心模式下**不参与生成**，只做记录，填非零值即可（`loadRings` 会拒绝零向量）；
3. `rings` 里**不要**把起点写进去，起点是触发瞬间的悬停位置，由代码自动取。

### 3.4 离线校验

```bash
cd ~/fly_ws
python3 sh_file/verify_rings_geometry.py
```

期望输出：

```
ring[0] c=(2.0, 0.0, 1.3)
ring[1] c=(4.0, 1.6, 1.4)
ring[2] c=(6.0, 0.0, 1.3)
|wps_fwd|=3 |Q|=2 N=3 final=(6.0, 0.0, 1.3)
GEOMETRY_OK (center-only)
```

`N = 环数`，`|Q| = 环数 - 1`（Q 是不含首尾的中间点）。

## 4. 启动流程

### 4.1 冷启动顺序（不可乱）

```
① 动捕软件 → 刚体 MVD 可见 → 检查 VRPN 广播 3883
② 遥控器 → 模式档 AUTO_HOVER、齿轮档 CMD、四杆回中
③ 飞机上电 → 等 PX4 自检蜂鸣结束
④ T1 MAVROS  → T2 VRPN → T3 fusion → T4 px4ctrl → T5 traj
⑤ 检查 → 起飞 → 挪到起点 → 触发 → 飞行 → 降落
```

### 4.2 一键启动

```bash
#终端1
cd ~/fly_ws && source devel/setup.bash
./sh_file/run_mocap.sh
```

### 4.3 起飞 → 触发 → 结束

```bash
#终端2
# ① 起飞（约 6 s 到悬停：3s 电机怠速 + 1.5m/0.5m/s ≈ 3s）
cd ~/fly_ws && source devel/setup.bash
./sh_file/takeoff.sh
# 期望：MANUAL_CTRL(L1) --> AUTO_TAKEOFF --> AUTO_HOVER(L2)

# ② 用摇杆把飞机挪到"第一个环前 1.5~2 m、机头朝向第一个环"的位置
#    松杆，等 1~2 s，速度 < 0.3 m/s（触发门限）

#终端3
# ③ 触发穿环
cd ~/fly_ws && source devel/setup.bash
./sh_file/pub_trigger.sh
# 期望：[traj_gen] RINGS trigger from hover (x, y, z), yaw=..., 3 gates
#       [traj_gen] published traj: segments=3, duration≈Xs
#       随后 px4ctrl: AUTO_HOVER(L2) --> CMD_CTRL(L3)

#终端4
# ④ 飞完自动停：traj_server 停止发指令 → 0.5s 后自动回 AUTO_HOVER
cd ~/fly_ws && source devel/setup.bash
./sh_file/land.sh
# 期望：AUTO_HOVER(L2) --> AUTO_LAND
```

### 4.4 换一组环之后要做什么

| 改动 | 操作 | 要不要重编译 |
|---|---|---|
| 改 `rings` / `shape` / `maxVel` | **重启 终端1**（yaml 只在节点启动时读一次） | 不要 |
| 改 `ctrl_param_fpv.yaml` | 重启 终端1 | 不要 |
| 改 `takeoff_height` | 重启 终端1 | 不要 |
| 改 `.cpp` / `.hpp` | `catkin_make --pkg trajectory` 等 | 要 |
| 改 launch 文件 | 重启对应节点 | 不要 |

## 5. 参数说明

### 5.1 `traj.yaml`（轨迹层）

| 参数 | 默认 | 作用 | 穿环建议 | 是否真生效 |
|---|---|---|---|---|
| `shape` | 0 | 0 八字 / 1 圆 / 2 星 / 3 S / 4 穿环 | **必须 4** | ✅ |
| `rings` | 4 个占位 | 环心列表，顺序=飞行顺序 | 用实测值替换 | ✅ |
| `ringsReturnHome` | false | true：飞到最后一个环后原路返回起点 | 演示建议 true | ✅ |
| `maxVel` | 2.0 | 参考速度上限，决定轨迹总时长（越大越快） | 首飞 1.0 | ✅ |
| `maxAcc` | 5.0 | 参考加速度上限，决定过弯"硬度" | 首飞 2.0 | ✅ |
| `dt` | 0.05 | 可视化轨迹点间隔 | 不动 | ✅（仅 RViz） |
| `K` | 32 | MINCO 每段采样数，越大越平滑越慢 | 不动 | ✅ |
| `rhoT` | 10000 | 时间惩罚（越大越保守、越慢） | 不动 | ✅ |
| `rhoV` | 100000 | 速度惩罚 | 不动 | ✅ |
| `rhoA` | 100000 | 加速度惩罚 | 不动 | ✅ |
| `eightWidth/ eightLength` | 5.0 / 10.0 | 八字尺寸 | 穿环时不看 | 仅 shape=0 |
| `circleRadius` | 2.0 | 圆半径 | 不看 | 仅 shape=1 |
| `starRadius` | 3.0 | 星形半径 | 不看 | 仅 shape=2 |
| `sWidth/sLength` | 3.0 / 14.0 | S 形尺寸 | 不看 | 仅 shape=3 |
| `height` | 1.00 | — | — | ❌ **无效**，高度由 `takeoff_height` 决定 |
| `startX/Y` `endX/Y` | 0 | — | — | ❌ 无效（只读不用） |
| `gateApproachDist` | 0.5 | — | — | ❌ 无效（只飞环心） |
| `rings[].nx/ny/nz` | — | 环法向 | 填 `1,0,0` 等占位即可 | ❌ 不参与生成 |
| `pauseDebug` | false | MINCO 调试暂停 | 保持 false | ✅ |


