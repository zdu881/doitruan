绘图待办 (TODO_drawings.md)

目标：为 `caogao.markdown` 补充可读、可渲染的说明图（PlantUML 源文件），便于开发、评审与实现对齐。

任务清单（优先级与说明）

1. 系统总体架构图（必做）
   - 文件：`docs/images/architecture.puml`
   - 目的：展示 Frontend / Backend / CastRay / Ray Head & Workers / Object Store / DB / 外部 GIS 等组件与协议（REST/WS/gRPC）
   - 输出：PlantUML 源 + 导出 SVG/PNG

2. 仿真生命周期时序图（必做）
   - 文件：`docs/images/simulation_sequence.puml`
   - 目的：说明从前端提交仿真到任务在 Ray 上执行并回传结果的完整时序（含 job_id、ObjectRef 流转、WebSocket 推送）

3. API 数据模型图（必做）
   - 文件：`docs/images/data_model.puml`
   - 目的：图示 Scenario / Drone / BaseStation / NetworkParams / Runtime 等请求/响应对象与主要字段

4. 部署拓扑图（高）
   - 文件：`docs/images/deployment_topology.puml`（可选）
   - 目的：展示单机开发、单机 Ray 和多节点生产部署的拓扑

5. 说明文档：渲染与导出方法
   - 文件：`docs/images/README.md`
   - 目的：说明如何使用 PlantUML 渲染这些 `.puml` 为 SVG/PNG（本地命令与 CI 自动化示例）

渲染建议
- 本地渲染：使用 PlantUML jar 或 Docker 镜像（示例命令将在 `docs/images/README.md` 中给出）。
- CI 渲染：可在 GitHub Actions 中加入 PlantUML 渲染步骤自动生成 images。

逐步计划
- 本次提交：创建 TODO 文件 + 生成前三个 PlantUML 源文件 + README（渲染说明）。
- 下一步（可选）：在本地或 CI 中渲染 SVG/PNG 并提交。
