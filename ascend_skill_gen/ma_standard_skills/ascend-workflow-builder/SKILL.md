---
name: ascend-workflow-builder
description: Guide for building end-to-end AI pipelines using ModelArts Workflow on Ascend NPU. This skill should be used when users need to orchestrate data preparation, model training, model registration, and service deployment into automated low-code workflows, configure multi-branch execution, or integrate MRS big data capabilities.
license: Apache 2.0
---

# Ascend Workflow Builder Guide

## Overview

This skill provides guidance for building end-to-end AI pipelines using ModelArts Workflow on Ascend NPU hardware. It covers workflow creation, node orchestration, parameter configuration, multi-branch execution, and pipeline publishing.

---

## When to Use

- Building automated AI pipelines (data → training → deployment)
- Orchestrating Workflow nodes for Ascend NPU training jobs
- Configuring multi-branch conditional execution in workflows
- Publishing reusable workflows to ModelArts or AI Gallery
- Integrating MRS big data processing into AI pipelines

---

## Workflow Core Concepts

### What is a Workflow

A Workflow is a low-code AI pipeline that connects multiple processing stages:

```
Data Import → Data Labeling → Training → Model Registration → Service Deployment
```

Each stage is represented as a **node** in the workflow graph. Nodes execute sequentially or conditionally based on configuration.

### Node Types

| Node Type | Purpose | Ascend Relevance |
|---|---|---|
| Dataset node | Create or reference a dataset | Data source for training |
| Dataset labeling node | Trigger data annotation | Prepare labeled data |
| Dataset import node | Import data from OBS/DWS/MRS | Load training data |
| Dataset version node | Publish a dataset version | Version control for data |
| Training job node | Run model training | Execute on Ascend NPU |
| Model registration node | Register trained model | Package model artifacts |
| Service deployment node | Deploy as online service | Serve on Ascend NPU |

Reference: [什么是Workflow](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_workflow_0269.html)

---

## Building a Workflow

### Step 1: Define Workflow Parameters

Workflow parameters allow reuse across different runs:

```python
from modelarts.workflow import Workflow, Parameter

# Define reusable parameters
params = {
    "data_url": Parameter(name="data_url", type="str",
                          description="OBS path to training data"),
    "epochs": Parameter(name="epochs", type="int", default=10,
                        description="Number of training epochs"),
    "batch_size": Parameter(name="batch_size", type="int", default=32,
                            description="Training batch size"),
    "npu_count": Parameter(name="npu_count", type="int", default=8,
                           description="Number of NPU cards per node"),
}
```

Reference: [配置Workflow参数](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_workflow_0353.html)

### Step 2: Create Training Job Node (Ascend NPU)

The training job node is the core of an Ascend AI pipeline:

```python
from modelarts.workflow import TrainingJobNode

training_node = TrainingJobNode(
    name="ascend_training",
    algorithm_id="my-algorithm-id",
    inputs={
        "data_url": params["data_url"],
    },
    outputs={
        "train_url": "obs://bucket/output/model/",
    },
    hyperparameters={
        "epochs": params["epochs"],
        "batch_size": params["batch_size"],
    },
    spec={
        "resource_pool_id": "pool-xxx",
        "flavor": "modelarts.vm.cpu168.npu.8xAscend910B",
        "node_count": 1,
    }
)
```

Reference: [创建Workflow训练作业节点](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_workflow_0380.html)

### Step 3: Create Model Registration Node

Register the trained model for deployment:

```python
from modelarts.workflow import ModelRegistrationNode

model_node = ModelRegistrationNode(
    name="register_model",
    model_name="my-ascend-model",
    model_type="PyTorch",
    source_job=training_node,
    model_path=training_node.outputs["train_url"],
)
```

Reference: [创建Workflow模型注册节点](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_workflow_0385.html)

### Step 4: Create Service Deployment Node

Deploy the registered model as an online service:

```python
from modelarts.workflow import ServiceDeploymentNode

deploy_node = ServiceDeploymentNode(
    name="deploy_service",
    model=model_node,
    spec={
        "resource_pool_id": "pool-xxx",
        "flavor": "modelarts.vm.cpu8.npu.1xAscend910B",
        "instance_count": 1,
    }
)
```

Reference: [创建Workflow服务部署节点](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_workflow_0389.html)

### Step 5: Orchestrate and Publish

Connect nodes into a complete workflow and publish:

```python
from modelarts.workflow import Workflow

workflow = Workflow(
    name="ascend-e2e-pipeline",
    nodes=[training_node, model_node, deploy_node],
    params=params,
)

# Validate workflow
workflow.validate()

# Publish to ModelArts
workflow.publish()
```

Reference: [编排Workflow](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_workflow_0400.html)

---

## Multi-Branch Execution

### Conditional Branch Control

Use condition nodes to control which branch executes based on runtime metrics:

```python
from modelarts.workflow import ConditionNode

# Branch based on training accuracy
condition_node = ConditionNode(
    name="check_accuracy",
    condition="accuracy > 0.95",
    true_branch=deploy_node,       # Deploy if accuracy is high
    false_branch=retrain_node,     # Retrain if accuracy is low
)
```

### Use Cases for Multi-Branch

| Scenario | Condition | True Branch | False Branch |
|---|---|---|---|
| Quality gate | accuracy > threshold | Deploy model | Retrain with more data |
| A/B model selection | model_a_loss < model_b_loss | Deploy model A | Deploy model B |
| Data volume check | dataset_size > min_size | Full training | Data augmentation first |

Reference: [构建条件节点控制分支执行](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_workflow_0394.html)

---

## Workflow Management

### Run and Monitor

- **Run**: Trigger workflow execution from console or SDK
- **Monitor**: View node-level execution status and logs in real time
- **Retry**: Re-run failed nodes without restarting the entire workflow
- **Stop**: Terminate running workflow and release resources

Reference: [重试/停止/运行Workflow节点](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_workflow_0287.html)

### Publish Destinations

| Destination | Scope | Use Case |
|---|---|---|
| ModelArts | Within account | Internal team reuse |
| AI Gallery | Public marketplace | Community sharing |

Reference: [发布Workflow到ModelArts](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_workflow_0403.html)

---

## Reference Documentation

- [使用Workflow实现低代码AI开发](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/umn-standard-modelarts-0010.html)
- [开发Workflow的核心概念介绍](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_workflow_0292.html)
- [配置Workflow的输入输出目录](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_workflow_0357.html)
- [Workflow多分支运行介绍](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_workflow_0395.html)
- [在Workflow中使用大数据能力（MRS）](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_workflow_0410.html)
- [开发Workflow命令参考](https://support.huaweicloud.com/intl/zh-cn/usermanual-standard-modelarts/modelarts_workflow_0291.html)
