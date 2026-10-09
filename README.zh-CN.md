# app-restore-probe

[English](README.md)

**规划阶段（Planning），尚无可执行发布（no executable release yet）。** 本仓库目前提供范围说明和路线图。下文所有能力均为计划项；尚无 CLI、可安装包、经验证的版本兼容矩阵或集成测试结果。

app-restore-probe 拟验证某个具体自托管应用的备份能否同时恢复数据库中的记录、文件和一项实际业务操作。项目计划复用已有备份与恢复工具，再补充小范围的应用校验。

## 首个候选应用

[Paperless-ngx](https://github.com/paperless-ngx/paperless-ngx) 是首个拟选目标（proposed initial target）；正式发布前须选定并验证具体版本组合。本项目独立维护，与 Paperless-ngx 没有隶属关系，也未获得其背书。当前没有任何已支持的应用、数据库或 Docker Compose 版本。

首版计划完成以下流程：

1. 接收 Paperless-ngx 官方 document-exporter 产生的目录，以及描述一份已有 canary 文档的 manifest。
2. 校验输入和已选版本，由 Python 3.11+ 编排，在专用测试主机或测试 Docker daemon 上创建全新的 Docker Compose 环境。
3. 按所选版本的官方 `document_importer` 流程重建应用状态，再通过应用 API 读取已有 canary 文档、下载其原文件，核对文件哈希以及标签等预期元数据。
4. 生成 JSON 与 Markdown 报告，记录检查结果、证据引用、耗时、失败原因和清理状态。

必须先完成版本选择和可复现的合成导出 fixture，之后才实现恢复流程与演示报告。应用导出属于逻辑恢复输入，不能据此判断原始数据库备份是否可恢复，也不能证明所有生产配置已还原。

## 隔离要求

计划实现必须为每次运行生成唯一 run ID，并为全部容器、网络和卷使用唯一名称；应用端口只能监听 loopback；输入导出目录须保持只读；禁止修改生产服务、路径或凭据。清理只能使用工具生成的 Compose 文件，并逐项核实资源确实属于本次运行。绝不能使用用户传入的生产 Compose 文件进行清理。

首版拟要求专用测试环境和明确的资源限制。容器仍共享主机资源，仅有名称与网络隔离不能避免生产主机受到资源压力影响。本规划仓库不提供生产操作命令。

## 结果的含义

一次 canary 检查通过，只能说明该输入与版本组合下，报告列出的检查通过。它不能证明全部业务正确、全部数据完整、恢复时间目标达成，也不是灾备认证。失败和未完成的检查必须明确展示。

## 已有工具与拟选切口

[restore-drill](https://github.com/RamazanKara/restore-drill) 已覆盖数据库隔离恢复、检查和报告；[backupdrill](https://github.com/backupdrill/cli) 已覆盖 Supabase 恢复演练。本项目拟缩小到具体应用校验，将恢复后的记录、文件与一次真实读取操作串起来。在自建通用恢复基础设施之前，应先评估复用或集成已有工具。

## 项目文档

- [MVP 范围与安全边界](docs/MVP.md)
- [路线图与发布门槛](ROADMAP.md)
- [参与贡献](CONTRIBUTING.md)
- [MIT 许可证](LICENSE)
