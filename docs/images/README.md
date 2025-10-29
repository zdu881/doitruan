PlantUML 渲染说明

这些 `.puml` 文件位于 `docs/images/`，包含可直接渲染的 PlantUML 源代码。

本地渲染（使用 PlantUML jar）示例：

1. 下载 PlantUML jar（如果尚未安装）：
   - https://plantuml.com/zh/download

2. 渲染为 SVG：

```bash
java -jar plantuml.jar -tsvg docs/images/architecture.puml
java -jar plantuml.jar -tsvg docs/images/simulation_sequence.puml
java -jar plantuml.jar -tsvg docs/images/data_model.puml
```

3. 渲染为 PNG：

```bash
java -jar plantuml.jar -tpng docs/images/architecture.puml
```

使用 Docker（无需直接安装 Java）：

```bash
docker run --rm -v $(pwd):/workspace plantuml/plantuml -tsvg /workspace/docs/images/architecture.puml
```

CI 自动化建议：在 GitHub Actions 中加入一个 job 执行上述命令并将生成的 SVG/PNG 上传为构建工件或直接提交回仓库（注意：需谨慎防止 CI 创建循环提交）。

注意：PlantUML 有多种渲染后端（Graphviz），如果包含复杂布局请确保系统安装 `dot`（Graphviz）。
