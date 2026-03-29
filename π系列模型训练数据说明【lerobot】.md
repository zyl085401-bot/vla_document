# 1. lerobot数据集目录结构

```plain&#x20;text
<dataset_name>/
├── data/
│   └── chunk-000/
│       ├── episode_000000.parquet
│       ├── episode_000001.parquet
│       └── ...
├── videos/
│   └── chunk-000/
│       ├── observation.images.main/  (or your_camera_key_1)
│       │   ├── episode_000000.mp4
│       │   └── ...
│       ├── observation.images.secondary_0/ (or your_camera_key_2)
│       │   ├── episode_000000.mp4
│       │   └── ...
│       └── ...
├── meta/
│   ├── info.json
│   ├── episodes.jsonl
│   ├── tasks.jsonl
│   ├── episodes_stats.jsonl  (for v2.1) or stats.json (for v2.0)
│   └── README.md (often, for Hugging Face Hub)
└── README.md (top-level, for Hugging Face Hub)
```

# 2. 数据解释

### 2.1 Parquet 文件

* observation.state (list of numbers): 机器人状态，如关节位置、末端位姿

* action (list of numbers): 动作，如关节位置变化、末端位姿变化

* timestamp (number): 时间戳，当前数据距离任务开始的时间

* episode\_index (integer): 该片段数据的ID

* frame\_index (integer): 该片段数据内的数据帧的ID

* index (integer): 数据的唯一ID【相对于整个数据集】

* next.done (true/false, optional): 是否是该片段数据的最后一帧

* task\_index (integer, optional): 该片段数据隶属于哪一个任务ID

### 2.2 视频文件

可选文件，路径一般为`videos/chunk-000/camera_key/episode_xxxxxx.mp4`，其中`camera_key`对应相机名称。

# 3. **元数据解释**

3.1  数据组织结构文件 - info.json

info.json 包含数据集相关的通用信息

* codebase\_version (text): “v2.0” or “v2.1”。版本信息，尤其和 stats.json相关

* robot\_type (text): 机器人的类型

* fps (number): 数据采样帧率

* total\_episodes (integer): 数据集中所有片段数（不同任务的）

* total\_frames (integer): 数据集的所有帧数（不同任务不同片段数）

* total\_tasks (integer): 任务的类别墅

* total\_videos (integer): 视频文件的格式

* splits (dictionary): 按索引顺序划分数据集的约定。比如 {"train": "0:N"} 表示 0 \~ N-1 用于训练。

* **<span style="color: inherit; background-color: rgb(247,105,100)">features (dictionary)</span>**: 为数据集中的每个数据字段提供完整的元数据描述，包括：数据类型（整数、浮点数、字符串等）、形状/维度（数据是标量、向量、图像还是序列）、可选的语义信息（如通道的含义、坐标系的说明等）

特征数据格式示例

```json
"state": {"dtype": "float32", "shape": [7], "names": ["joint1", ...]}
```

```json
"image": {
    "dtype": "image",
    "shape": [256, 256, 3],
    "names": ["height", "width", "channel"]
},
```

```json
"actions": {
    "dtype": "float32",
    "shape": [7],
    "names": ["actions"]
},
```

### 3.2 片段级元数据 - episodes.jsonl

每行一个JSON对象，描述单个片段（episode）的元数据。

```json
{"episode_index": 0, "tasks": ["pick up the red block", "place it on the blue shelf"], "length": 150}
{"episode_index": 1, "tasks": ["open the drawer", "grasp the screwdriver"], "length": 200}
{"episode_index": 2, "tasks": ["pour water into cup"], "length": 120}
```

* episode\_index：片段ID

* tasks：片段隶属的任务命令

* length：片段的帧数

### 3.3 任务元数据 - tasks.jsonl

每一个JSON对象，描述一个任务（task）元数据

```json
{"task_index": 9, "task": "pick up the book and place it in the back compartment of the caddy"}
{"task_index": 10, "task": "put the bowl on the plate"}
```

* task\_index：任务ID

* task：任务命令

### 3.4 统计元数据 - stats.jsonl

包含每个特征数据的 {'max': ..., 'min': ..., 'mean': ..., 'std': ...}，若是v2.1 (`episodes_stats.jsonl` )，还定义了episode\_index。

# 4. 数据转换程序设计建议

1. 整体框架 ROS2 -> LeRobot

2. 使用rosbag记录数据

3. 使用python包 lerobot 辅助导出数据集
