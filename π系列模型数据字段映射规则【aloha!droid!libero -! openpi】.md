# 1. 项目介绍

1.1 ALOHA (A Low-cost Open-source Hardware for Autonomous manipulation)

**Mobile ALOHA** 是一个低成本、开源的双手操作机器人平台，主要用于自主操作任务的学习和演示。

1.2 DROID (Distributed Robot Interaction Dataset)

**DROID** 是一个大规模机器人操作数据集，基于 Franka Emika 机械臂平台收集。

1.3 LIBERO (Benchmarking Knowledge Transfer in Lifelong Robot Learning)

**LIBERO** 是一个专为机器人终身学习决策（LLDM）研究而设计的基准测试平台。用于研究终身学习和知识迁移



在openpi中，他们更多的是被作为一种数据定义方式，需要按照指定的方式转换为openpi使用的数据格式。不过这些不需要用户自己转换，openpi提供了默认的转换策略，只需要定义使用的策略名称即可。进行推理具体的字段映射规则参看&#x20;

[ π系列模型数据字段映射规则.html](https://qcnvwhpk3inb.feishu.cn/wiki/CQjKwUAOLiZVp0kM6G0c6ft3nGe)



# 2. 构建自己的数据【推理或者训练】

### 2.1 推理

客户端将自己的数据组织成对应的环境/策略【上面三者之一】要求，服务端使用对应的环境/策略。

### 2.2 训练

将数据转换为lerobot的格式，注意字段的组织方式需要和对应策略/环境匹配。以LIBERO数据为&#x4F8B;**，**&#x9700;要 **openpi/src/openpi/policies** 下的policy文件、数据集下&#x7684;**&#x20;info.json&#x20;**&#x6587;件，以及 openpi/src/openpi/training/config.py 中的 TrainConfig 的 **RepackTransform** 对应起来。下面是Libero的默认的**RepackTransform**

```json
_transforms.RepackTransform(
                    {
                        "observation/image": "image",
                        "observation/wrist_image": "wrist_image",
                        "observation/state": "state",
                        "actions": "actions",
                        "prompt": "prompt",
                    }
                )
```

```json
image -> observation/image -> image["base_0_rgb"]
image -> observation/wrist_image -> image["left_wrist_0_rgb"]
```

