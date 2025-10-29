
# doit

## 基于UE的6G无线通信系统数字孪生交互框架 — 软件设计说明书（草稿）

**版本**：草稿

---

## 目录

- [1 引言](#1-引言)
- [2 总体设计](#2-总体设计)
- [3 系统出错处理设计](#3-系统出错处理设计)
- [4 Ray 集群的调度与通信](#4-ray-集群的调度与通信)
- [5 基站报错预警（机器学习）](#5-基站报错预警机器学习)
- [6 应用](#6-应用)

---

## 1 引言

### 1.1 编写目的

本文档旨在详细阐述“基于UE的6G无线通信系统数字孪生交互框架”的软件设计方案。其主要目的是为项目开发团队提供一个清晰、统一的指导蓝图，确保各模块的设计和实现协调一致，并为开发、测试、维护与迭代提供参考。

### 1.2 背景说明

随着6G 技术的发展，其超高带宽、极低延迟与海量连接特性，为无人机（UAV）等 UE 在工业巡检、物流、应急通信等场景提供了更多可能。但真实环境中（如校园、城市）存在的信道干扰、基站覆盖与负载变化，会影响无人机通信质量与任务执行。

因此提出数字孪生系统，通过在虚拟空间精确复现无人机、基站与电磁环境，来模拟、预测与优化无人机行为与网络性能。本项目代号 `CastRay`，拟借助 `Ray` 框架构建可扩展的分布式数字孪生平台。

### 1.3 定义

| 术语 | 定义 |
| --- | --- |
| **数字孪生 (Digital Twin)** | 在虚拟空间中创建与物理实体对应的数字化模型，通过实时数据交互反映物理实体生命周期。 |
| **用户设备 (UE)** | User Equipment，指网络终端设备，本项目特指无人机（UAV）。 |
| **移动边缘计算 (MEC)** | 将计算与存储下沉到网络边缘，降低延迟并提升响应能力。 |
| **Ray** | 开源分布式计算框架，用于并行任务与 Actor 管理。 |
| **CastRay** | 本项目名称，基于 `Ray` 的数字孪生交互系统实现。 |
| **无人机代理 (Vehicle Agent)** | 仿真中代表真实无人机行为和通信的软件实体。 |
| **场景 (Scenario)** | 仿真环境配置集合（地图、基站布局、UE 初始状态、任务目标等）。 |

## 2 总体设计

### 2.1 需求规定

系统核心需求包括：环境建模、分布式仿真、UE 行为模拟、6G 通信模拟、数据可视化与交互，以及模块化与可扩展性。

### 2.2 运行环境

- 操作系统：Linux（建议 Ubuntu 20.04+）
- 后端：Python 3.10+、`Ray`、Flask / FastAPI
- 前端：HTML5、CSS3、JavaScript（ES6+）、Vue 或 React
- 相关库示例：`ray`, `numpy`, `pandas`, `matplotlib`, `websockets`

### 2.3 基本设计概念和处理流程

设计理念：仿真与现实分离，计算与交互分离。前端提交仿真任务至后端，后端将任务分解为 `Ray` 任务或 Actor 提交给集群并行执行，仿真数据通过 WebSocket 实时推送至前端进行可视化与交互。

主要流程：
- 场景配置（`droneOnCampus/src/frontend`）
- 任务提交（`droneOnCampus/src/backend/python/castray_backend.py`）
- 任务分发（`CastRay/ray_casting.py` 等）
- 分布式计算与状态更新（`CastRay/ray_cluster_discovery.py`, `vehicle_mec_agent.py`）
- 汇总、处理并通过 WebSocket 推送结果

<!-- 仿真生命周期时序图 -->
![仿真生命周期时序图](docs/images/simulation_sequence.svg)

*图：仿真生命周期时序图 — 描述从前端提交仿真到 Ray 执行并回传结果的端到端消息与时序。*

### 2.4 结构设计

主要模块：
- 表现层：`droneOnCampus/src/frontend`, `diaplayRayCluster`
- 应用层：`droneOnCampus/src/backend/python/castray_backend.py`
- 核心计算层：`CastRay/main.py`, `CastRay/models.py`, `CastRay/ray_casting.py`, `CastRay/file_transfer.py`
- 集群管理：`CastRay/start_ray_cluster.py`, `CastRay/ray_cluster_discovery.py`

<!-- 系统总体架构 -->
![系统总体架构](docs/images/architecture.svg)

*图：系统总体架构 — 展示前端、后端、CastRay 与 Ray 集群之间的交互与通信协议（REST / WebSocket / gRPC）。*

#### 功能映射（示例表）

| 功能需求 | 主要实现模块 |
| --- | --- |
| 环境建模 | `droneOnCampus/src/frontend`, `CastRay/models.py` |
| 分布式仿真 | `CastRay/ray_casting.py`, `ray` |
| UE 行为模拟 | `droneOnCampus/src/backend/python/vehicle_mec_agent.py` |
| 6G 通信模拟 | `CastRay/models.py`, `CastRay/ray_casting.py` |
| 数据可视化 | `droneOnCampus/src/frontend`, `diaplayRayCluster` |
| 任务管理 | `droneOnCampus/src/backend/python/castray_backend.py` |

### 2.5 接口设计（概要）

<!-- VehicleAgent 状态机 -->
![VehicleAgent 状态机](docs/images/vehicle_agent_state.svg)

*图：VehicleAgent 状态机 — 描述单个无人机代理在仿真中的生命周期与状态转换（Initialized → Flying → Communicating → Checkpointing → Error / Terminated）。*

- 前端-后端：RESTful（如 `POST /api/simulation/start`）与 WebSocket（如 `ws://server/api/simulation/stream`）。
- 应用-计算：通过 `ray.remote()` 提交任务或创建 Actor。

### 2.6 核心 API 规范（REST / WebSocket 示例）

下面给出若干核心接口的示例（请求/响应 JSON、状态码与错误示例），作为前后端与仿真执行模块对接的最低实现契约。

1) 启动仿真任务 — POST /api/simulation/start

Request (application/json):

```json
{
	"scenario_id": "campus-01",
	"job_name": "patrol-morning",
	"drones": [
		{"id": "u1", "init_pos": [10.0, 20.0, 5.0], "mission": "patrol"},
		{"id": "u2", "init_pos": [15.0, 25.0, 5.0], "mission": "inspection"}
	],
	"network_params": {
		"band": "sub-THz",
		"tx_power_dbm": 23,
		"bandwidth_mhz": 100
	},
	"runtime": {"duration_s": 1800, "update_interval_s": 1}
}
```

Response (202 Accepted):

```json
{
	"job_id": "job-20251029-0001",
	"status": "accepted",
	"message": "simulation job accepted"
}
```

错误示例 (400 Bad Request)：参数缺失或非法

```json
{ "error": "missing field: drones" }
```

2) 查询仿真状态 — GET /api/simulation/status/{job_id}

Response (200 OK):

```json
{
	"job_id": "job-20251029-0001",
	"status": "running",   
	"started_at": "2025-10-29T09:00:00Z",
	"progress": {"elapsed_s": 120, "duration_s": 1800},
	"metrics": {"active_agents": 2}
}
```

3) 停止仿真任务 — POST /api/simulation/stop

Request:

```json
{ "job_id": "job-20251029-0001", "graceful": true }
```

Response (200 OK):

```json
{ "job_id": "job-20251029-0001", "status": "stopping" }
```

4) WebSocket 实时数据流 — ws://{host}/api/simulation/stream?job_id={job_id}

连接后，后端以 JSON 消息推送仿真状态、Agent 状态与 KPI。建议使用统一的消息 envelope：

示例消息 - 状态更新（type = "state_update")：

```json
{
	"type": "state_update",
	"job_id": "job-20251029-0001",
	"timestamp": 1698612345,
	"vehicles": [
		{"id":"u1","pos":[10.5,20.3,5.0],"vel":[0.1,0.0,0.0],"snr_db": -65.2},
		{"id":"u2","pos":[15.1,25.4,5.0],"snr_db": -70.4}
	],
	"kpi": {"avg_throughput_mbps": 12.3, "max_latency_ms": 45}
}
```

示例消息 - 事件/告警（type = "event")：

```json
{
	"type": "event",
	"level": "warning",
	"job_id": "job-20251029-0001",
	"timestamp": 1698612400,
	"message": "Vehicle u1 SNR below threshold (-80 dB)"
}
```

设计建议：
- 所有 REST 接口使用标准 HTTP 状态码（200/202/400/404/500）；错误响应包含统一字段 `{code, error, details}`。
- WebSocket 建议采用心跳/心跳应答机制（例如每 30s 发送 ping），并支持客户端通过 message 类型 `control` 发送控制命令（如请求更高频率的更新或修改 agent 参数）。
- 将接口契约写成可机读文档（OpenAPI / Swagger）以便前后端自动对齐与生成客户端。

（注：以上示例为推荐最小契约；根据项目需要可扩展字段与认证机制，例如在 Header 中加入 `Authorization: Bearer <token>`。）

<!-- 数据模型图 -->
![核心 API 数据模型](docs/images/data_model.svg)

*图：核心 API 数据模型 — 显示 Scenario、Drone、BaseStation、NetworkParams 与 Runtime 等对象及主要字段（用于请求/响应校验）。*

## 3 系统出错处理设计

详见原文档；此处保留日志、前端提示、Ray 集群异常处理、参数校验、任务重试、检查点与优雅降级等机制的描述（已将相关文件名统一为反引号格式）。

## 4 Ray 集群的调度与通信

说明 `Ray` 在任务（Task）、Actor、对象存储（Object Store）与 gRPC 通信方面的使用模式，已在文中统一为标准术语与示例文件路径引用。

<!-- Ray 使用模式 -->
![Ray 使用模式](docs/images/ray_patterns.svg)

*图：Ray 使用模式 — 说明 Task 与 Actor 的分工、ObjectRef 的传递与失败重试示意。*

## 5 基站报错预警（机器学习）

保留并稍作精简：描述云-边协同、边缘轻量检测（如 LightGBM）、云端长序列模型（如 Transformer/FT-Transformer）与分级预警的思路。

<!-- ML 预警流水线 -->
![ML 预警流水线](docs/images/ml_pipeline.svg)

*图：ML 预警流水线 — 描述数据采集、边缘检测、云端聚合、模型训练/部署与告警下发的端到端流程。*

## 6 应用

保留原有应用场景：网络规划、资源优化、应急通信保障、算法验证平台。

<!-- 前端 mockup -->
![前端仪表板示意](docs/images/frontend_mockup.svg)

*图：前端仪表板 mockup — 示意 3D 场景视图、实时 KPI 曲线、事件日志与控制面板的典型布局。*
