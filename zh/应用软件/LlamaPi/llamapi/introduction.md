# 简介

LlamaPi（读作“拉马派”）是 Firefly AI 团队自研的大模型部署与管理工具，面向具备大模型推理能力的边缘设备，帮助用户在本地部署和管理开源大模型，并通过 OpenAI 兼容 API 为第三方应用提供推理能力。

LlamaPi 旨在降低边缘设备上的大模型部署门槛，缩短从模型获取、硬件适配到模型运行的流程，为离线、私有的 AI 应用提供轻量化推理底座。

如果你准备开始使用 LlamaPi，可以从[快速开始](../getting-started/quickstart.md)入手，完成安装并运行第一个模型。

## LlamaPi 是什么

在边缘算力设备上运行大模型通常涉及硬件识别、推理平台选择、模型格式匹配、运行库配置和服务部署等多个环节。LlamaPi 将这些环节整合为统一的命令行和服务接口，让用户可以用较短的操作流程完成模型下载、运行、管理和应用接入。

LlamaPi 不是单个模型或单一推理引擎，而是一套连接设备、模型、端侧推理平台和上层应用的部署与管理工具。

## 解决的问题

- **部署流程复杂**：将模型查找、下载、加载和运行整合为连续的操作流程。
- **硬件适配门槛高**：根据设备识别结果和可用推理平台筛选兼容模型。
- **模型管理分散**：通过统一命令管理本地模型、运行状态、实例数量和自动加载配置。
- **应用接入成本高**：通过 OpenAI 兼容 API 向第三方应用提供本地推理能力。

## 核心能力

- **本地部署开源大模型**：在设备上下载、加载并运行兼容的对话模型和 Embedding 模型。
- **统一模型管理**：查找模型、查看运行状态、调整实例、配置自动加载，以及卸载和删除模型。
- **适配多种推理平台**：根据当前设备识别可用推理平台及兼容模型。
- **提供 OpenAI 兼容 API**：当前支持 Chat Completions、Embeddings 和 Models 接口。
- **提供 LlamaPi 扩展接口**：支持模型实例管理和硬件平台查询。

## 典型使用场景

- 在无公网或数据不能离开本地环境时提供离线、私有 AI 推理。
- 在开发板或边缘设备上构建本地问答、文本生成和智能助手应用。
- 为支持自定义 OpenAI 服务地址的第三方应用提供本地模型服务。
- 为检索、搜索或知识库应用提供本地 Embedding 向量服务。

## 软件组成

LlamaPi 由两个主要组件组成：

| 组件 | 作用 |
|:---:|:---:|
| `llamapi-cli` | 用于发现、下载、运行和管理模型的命令行工具；命令入口为 `llamapi` |
| `llamapi-server` | 负责检测推理平台、加载模型、管理模型实例并提供 HTTP API 的服务组件 |

`llamapi-cli` 为交互操作和常见任务提供便捷入口，`llamapi-server` 负责持续运行的模型服务。两者配合完成从模型获取到应用接入的完整流程。

## 支持范围与当前限制

- LlamaPi 面向具备大模型推理能力的边缘设备。当前支持的硬件和推理平台见[模型下载与管理](../getting-started/model-download-and-management.md)。
- 当前 API 文档覆盖 OpenAI 兼容的 Chat Completions、Embeddings 和 Models 接口，以及 LlamaPi 自有扩展接口，并不代表兼容 OpenAI 的全部接口和字段。
- 当前版本尚未提供 Anthropic 格式兼容接口。
- 模型能否运行取决于设备硬件、可用推理平台和模型变体是否匹配。

## 文档导航

- 初次使用 LlamaPi：[快速开始](../getting-started/quickstart.md)
- 下载和管理本地模型：[模型下载与管理](../getting-started/model-download-and-management.md)
- 运行和部署模型：[模型运行与部署](../getting-started/model-load-and-run.md)
- 配置模型自动加载：[持久化部署](../getting-started/model-persistence.md)
- 将本地模型接入现有应用：[接入第三方应用](../getting-started/third-party-integration.md)
- 查询完整命令参数：[终端命令详解](../advanced-guides/cli-command-guide.md)
- 配置和维护服务：[服务配置与运维](../advanced-guides/server-operations.md)
- 开发 API 集成：[API 接口详解](../advanced-guides/api-reference.md)
