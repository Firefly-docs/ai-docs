# 终端命令详解

`llamapi-cli` 是 LlamaPi 的命令行组件，命令入口为 `llamapi`，用于检查硬件、查询和下载模型、运行对话、管理模型实例以及配置自动加载。

如果希望按任务了解完整使用流程，请先阅读[模型下载与管理](../getting-started/model-download-and-management.md)、[模型运行与部署](../getting-started/model-load-and-run.md)和[持久化部署](../getting-started/model-persistence.md)。本章集中说明每个命令的语法、参数、前置条件、执行行为和输出。

## 命令格式

```text
llamapi [OPTIONS] <COMMAND>
```

当前命令包括：

| 命令 | 用途 |
|:---:|:---:|
| `run` | 运行对话模型并进入交互模式或执行单次提问 |
| `list` | 列出本地或远程模型 |
| `pull` | 下载模型 |
| `rm` | 删除本地模型 |
| `load` | 加载模型或调整实例数 |
| `unload` | 卸载模型组 |
| `ps` | 查看运行状态和自动加载配置 |
| `platform` | 查看硬件、平台和服务状态 |
| `enable` | 配置 `llamapi-server` 启动时自动加载模型 |
| `disable` | 取消自动加载配置 |

## 全局选项

| 选项 | 说明 |
|:---:|:---:|
| `--config <PATH>` | 指定 `llamapi-cli` 配置文件；默认 `~/.config/llamapi-cli/config.toml` |
| `-h, --help` | 显示帮助信息 |
| `-V, --version` | 显示版本号 |

`--config` 是顶层选项，必须放在子命令之前：

```bash
llamapi --config /path/to/config.toml load qwen3:4b
```

查看版本：

```bash
llamapi --version
```

当前版本输出类似：

```text
✦ Firefly AI · LlamaPi v0.2.0
```

`firefly-llamapi-cli` 与 `firefly-llamapi-server` 的版本号始终保持一致。

## `llamapi-cli` 配置文件

默认路径：

```text
~/.config/llamapi-cli/config.toml
```

配置文件缺失时使用默认值，不需要手动创建。完整示例：

```toml
[model_store]
path = "/var/lib/llamapi/models"

[download]
source = "auto"

[server]
config_path = "/etc/llamapi-server/config.toml"
```

| 字段 | 默认值 | 说明 |
|:---:|:---:|:---:|
| `model_store.path` | `/var/lib/llamapi/models` | 本地模型存储根目录 |
| `download.source` | `auto` | 下载源：`auto`、`modelscope` 或 `huggingface` |
| `server.config_path` | `/etc/llamapi-server/config.toml` | `llamapi-server` 配置文件路径 |

`llamapi-cli` 从 `llamapi-server` 配置中读取 `host` 和 `port`。如果 `host` 是 `0.0.0.0` 或其他绑定地址，`llamapi-cli` 会使用 `127.0.0.1` 进行本地连接。本文档中的 `llamapi-server` 端口统一为 `9265`。

## 模型参数约定

### 模型名称

支持：

```text
qwen3:4b
bge-m3
```

显示模型变体时使用：

```text
model(platform/chip)
```

例如：

```text
qwen3:4b(rkllm/rk3588)
```

### 平台参数

需要明确指定变体时使用：

```text
-p, --platform <platform>/<chip>
```

平台和芯片名称区分大小写，建议使用 `llamapi platform` 和 `llamapi list --online` 返回的值。

### 运行时 ID

默认 ID 为：

```text
name@platform-chip
```

自定义 ID：

- 可以包含中文。
- 不能包含空白字符。
- 不能包含 `/`。
- 显示宽度不能超过 20。
- 不能与已加载模型组或自动加载配置中的其他模型冲突。

## 环境与状态查询

### `platform`

显示当前硬件平台和 `llamapi-server` 状态。

```bash
llamapi platform
```

该命令没有子命令选项。

服务运行时示例：

```text
SoC:                    RK3588
Coprocessor:            null
Supported platforms:    rkllm, rknn2
Unsupported platforms:  rknn3
Service:                running  (http://127.0.0.1:9265/v1)
```

服务未运行时示例：

```text
SoC:                    unknown
Coprocessor:            unknown
Supported platforms:    unknown (service not running)
Unsupported platforms:  unknown (service not running)
Service:                not running  (http://127.0.0.1:9265/v1)

Run 'llamapi run <model>' to start the service.
```

字段说明：

- SoC、协处理器和平台来自 `GET /v1/platforms`。
- 同型号芯片数量大于 1 时显示为 `型号 ×数量`。
- 未检测到协处理器时显示 `null`。
- 服务运行但没有可用平台时显示 `none`。
- 服务未运行时无法查询硬件，相关字段显示 `unknown`。

### `list`

列出模型。默认显示本地已下载模型。

```text
llamapi list [OPTIONS]
```

| 选项 | 说明 |
|:---:|:---:|
| `-o, --online` | 显示远程仓库中兼容当前硬件的模型 |
| `-a, --all` | 显示远程仓库中的全部模型，不按当前硬件过滤 |

示例：

```bash
llamapi list
llamapi list --online
llamapi list --all
```

输出列：

| 列 | 说明 |
|:---:|:---:|
| `MODEL` | 模型显示名称 |
| `TYPE` | `chat`、`embedding` 或未知时的 `-` |
| `PLATFORM` | `<platform>/<chip>` |
| `SIZE` | 模型大小；无可信数据时为 `-` |
| `LOCAL` | `● installed` 或 `○ not installed` |

`list --online` 依赖 `llamapi-server` 的 `/v1/platforms`，不会自动启动服务。只有 `available: true` 且芯片匹配的模型变体会显示。

`list --all` 不进行硬件过滤，因此不依赖平台接口。

远程查询规则：

- `auto` 并发查询 Hugging Face 和 ModelScope，采用最先返回的有效结果。
- 任一来源成功后取消另一来源尚未完成的请求。
- 每个来源最多尝试 3 次，重试前等待 500 ms、1000 ms。
- 建连超时为 3 秒，单次请求超时为 6 秒，整个查询最多等待 20 秒。
- 连接、发送、读取、解析错误以及 HTTP `429`、`5xx` 会重试。
- 其他 `4xx` 和空仓库列表不会重试。

排序规则：

1. 先按 `MODEL` 自然排序。
2. 同系列中，无参数规模的模型排在前面。
3. 带规模的模型按数值升序，例如 `0.6b`、`4b`、`14b`。
4. 同名模型再按完整 `PLATFORM` 自然排序。

### `ps`

显示当前已加载模型和自动加载配置。

```bash
llamapi ps
```

该命令没有子命令选项。

输出示例：

```text
ID       MODEL       TYPE        PLATFORM          STATUS        INSTANCES    ENABLE
abc      qwen3:4b    chat        rkllm/rk3588      ● active      2            yes(1)
demo     qwen3:4b    chat        rkllm/rk3588      ○ inactive    -            yes(1)
```

| 列 | 说明 |
|:---:|:---:|
| `ID` | 运行时模型 ID |
| `MODEL` | 模型显示名 |
| `TYPE` | `chat` 或 `embedding` |
| `PLATFORM` | 平台和芯片 |
| `STATUS` | `● active` 或 `○ inactive` |
| `INSTANCES` | 当前运行实例数；未运行时为 `-` |
| `ENABLE` | `yes(N)` 表示配置了 N 个自动加载实例 |

只下载到本地、但没有加载且没有自动加载配置的模型不会显示。如果没有符合条件的模型，命令只输出表头。

## 本地模型管理

### `pull`

下载模型到本地模型目录。

```text
llamapi pull <MODEL> [OPTIONS]
```

| 参数或选项 | 说明 |
|:---:|:---:|
| `MODEL` | 模型名称，例如 `qwen3:4b` |
| `-p, --platform <PLATFORM>` | 指定平台和芯片，例如 `rkllm/rk3588` |
| `--source <SOURCE>` | `auto`、`modelscope` 或 `huggingface` |

示例：

```bash
llamapi pull qwen3:4b
llamapi pull qwen3:4b --platform rknn3/rk1828
llamapi pull qwen3:4b --source modelscope
```

执行流程：

1. 从 `/v1/platforms` 获取当前可用平台和芯片。
2. 根据模型名和平台选择远程仓库。
3. 多个兼容候选时，在交互终端提示选择；非交互环境要求 `--platform`。
4. 模型已存在时询问是否重新下载。
5. 下载模型文件并显示进度。
6. 验证 `model.toml` 完整性。
7. 显示模型总大小。

`pull` 不会自动启动 `llamapi-server`，也不会在下载后自动加载模型。

成功输出类似：

```text
✦ Firefly AI · LlamaPi
✔ Download complete: qwen3:4b(rkllm/rk3588)
  Platform  rkllm/rk3588
  Size      3.8 GB
```

### `rm`

删除本地模型文件。

```text
llamapi rm <MODEL> [OPTIONS]
```

| 参数或选项 | 说明 |
|:---:|:---:|
| `MODEL` | 模型名称 |
| `-p, --platform <PLATFORM>` | 指定要删除的平台变体 |

示例：

```bash
llamapi rm qwen3:4b
llamapi rm qwen3:4b --platform rkllm/rk3588
```

如果模型存在运行中的 ID，`llamapi-cli` 会提示是否先卸载所有匹配模型组。确认后删除模型目录并显示释放的磁盘空间。

多个本地变体且未指定平台时，交互终端提示选择；非交互环境要求显式指定平台。

## 模型运行

### `run`

按需启动 `llamapi-server`、下载和加载对话模型，然后进入交互模式或执行单次提问。

```text
llamapi run <MODEL_OR_ID> [PROMPT] [OPTIONS]
```

| 参数或选项 | 说明 |
|:---:|:---:|
| `MODEL_OR_ID` | 模型名称或已加载运行时 ID |
| `PROMPT` | 可选；提供后执行单次提问 |
| `-p, --platform <PLATFORM>` | 指定平台和芯片 |
| `--system <TEXT>` | 设置系统提示词 |
| `--temperature <FLOAT>` | 覆盖采样温度 |
| `--top-p <FLOAT>` | 覆盖 top-p |
| `--max-tokens <INT>` | 覆盖最大生成 token 数 |
| `--stats` | 回答后显示 token、首 token 延迟和生成速度 |

示例：

```bash
llamapi run qwen3:4b
llamapi run qwen3:4b "你好"
llamapi run qwen3:4b --system "你是一名 Python 助手"
llamapi run qwen3:4b --platform rknn3/rk1828
llamapi run abc
llamapi run qwen3:4b "写一首诗" --temperature 0.9 --max-tokens 512
llamapi run qwen3:4b "你好" --stats
```

关键行为：

1. `llamapi-server` 未运行时自动启动服务。
2. 参数匹配已加载 ID 时直接使用该 ID。
3. 显式指定平台时严格使用该平台，不执行自动性能选择。
4. 未指定平台时，本地兼容对话模型优先。
5. 没有本地变体时查询远程仓库并选择兼容变体。
6. 如果只有同名 Embedding 模型，拒绝进入对话并提示使用 `load`。
7. 模型未下载时自动下载。
8. 按模型名运行时使用默认运行时 ID。
9. 默认 ID 已加载且指向同一目录时直接复用，不调整实例数。
10. 默认 ID 指向其他目录时报告冲突。
11. 退出时只卸载本次命令新加载的模型。
12. `run` 自动启动的 `llamapi-server` 不会随命令退出而停止。

使用 `--stats` 后，回答结束时会显示以下统计信息：

```text
--- stats ---
prompt_tokens:     13
completion_tokens: 14
first_token:       380 ms
speed:             18.7 token/s
```

#### 并发使用机制

多个 `run` 进程共享同一个运行时模型组，但当前版本没有会话租约或引用计数。负责加载模型的 `run` 退出时，可能卸载其他进程正在复用的模型。

需要稳定并发服务时：

1. 使用 `llamapi load` 预先加载模型。
2. 让客户端直接调用 HTTP API。
3. 按硬件资源配置实例数和请求队列。

#### 交互模式命令

执行 `llamapi run <MODEL>` 后进入交互模式。

| 命令 | 说明 |
|:---:|:---:|
| `/help` | 显示 REPL 帮助 |
| `/clear` | 清空对话历史 |
| `/system <prompt>` | 设置系统提示词；无参数时清除 |
| `/set <key> <value>` | 设置生成参数 |
| `/show info` | 显示模型信息 |
| `/show params` | 显示当前生成参数 |
| `/show system` | 显示系统提示词 |
| `/exit` | 退出会话 |

`/set` 支持：

| 参数 | 取值 |
|:---:|:---:|
| `temperature` | 浮点数 |
| `top_p` | 浮点数 |
| `max_tokens` | 整数 |
| `enable_thinking` | `true` 或 `false` |

输入 `"""` 开始多行输入，再次输入 `"""` 结束。生成过程中按 `Ctrl+C` 中断当前回答；在输入提示符处按 `Ctrl+C` 或按 `Ctrl+D` 退出。

## 模型实例管理

### `load`

加载本地模型，或者通过已有运行时 ID 调整实例数。

```text
llamapi load <MODEL_OR_ID> [OPTIONS]
```

| 参数或选项 | 默认值 | 说明 |
|:---:|:---:|:---:|
| `MODEL_OR_ID` | — | 本地模型名或当前活动运行时 ID |
| `-p, --platform <PLATFORM>` | 自动选择 | 指定本地模型变体；ID 分支用于精确校验平台 |
| `--instance <N>` | `1` | 目标实例数，必须大于等于 1 |
| `--id <ID>` | 默认运行时 ID | 为新加载模型指定自定义 ID |

示例：

```bash
llamapi load qwen3:4b
llamapi load qwen3:4b --instance 2
llamapi load qwen3:4b --platform rkllm/rk3588 --id abc --instance 2
llamapi load abc --instance 3
```

行为：

- 位置参数精确匹配活动 ID 时，按 `--instance` 调整该模型组。
- 位置参数作为本地模型名时，加载指定数量的实例。
- 默认 ID 已加载且指向同一模型时，执行实例调整。
- 实例数相同则提示已加载，不重复操作。
- `llamapi-server` 只能创建部分实例时，仍可能返回成功，并显示实际实例数。
- 位置参数是已有 ID 时不能同时指定 `--id`。
- 多个本地变体且未指定平台时，交互选择；非交互环境要求 `--platform`。

前置条件：`llamapi-server` 必须正在运行。按模型名加载时，模型必须已经下载到本地。

**协处理器限制**：在协处理器芯片上使用 `--instance` 加载多个模型实例可能导致加载失败、芯片通信失败和服务异常。当前应保持 `--instance 1`。故障恢复步骤见[协处理器加载多个模型实例后通信失败](../llamapi/faq.md#协处理器加载多个模型实例后通信失败)。

### `unload`

卸载一个模型组及其全部实例。

```text
llamapi unload <MODEL_OR_ID> [OPTIONS]
```

| 参数或选项 | 说明 |
|:---:|:---:|
| `MODEL_OR_ID` | `llamapi ps` 中的模型名或 ID |
| `-p, --platform <PLATFORM>` | 精确匹配运行平台和芯片 |

示例：

```bash
llamapi unload qwen3:4b
llamapi unload abc
llamapi unload qwen3:4b --platform rkllm/rk3588
```

匹配规则：

1. 优先将参数作为 ID 精确匹配。
2. ID 未匹配时，再按模型名匹配。
3. 只有一个模型组匹配时直接卸载。
4. 多个模型组匹配时，交互终端提示选择。
5. 非交互环境必须使用明确的 ID。

ID、模型名和平台均区分大小写。`llamapi-server` 必须正在运行。

## 自动加载配置

### `enable`

将模型写入 `llamapi-server` 配置，使其在服务启动时自动加载。

```text
llamapi enable <MODEL> [OPTIONS]
```

| 参数或选项 | 默认值 | 说明 |
|:---:|:---:|:---:|
| `MODEL` | — | 本地模型名称 |
| `-p, --platform <PLATFORM>` | 自动选择 | 指定平台和芯片 |
| `--instance <N>` | `1` | 自动加载实例数 |
| `--id <ID>` | 默认运行时 ID | 自定义运行时 ID |
| `--now` | — | 写入配置后立即加载 |

示例：

```bash
llamapi enable qwen3:4b
llamapi enable qwen3:4b --instance 2 --now
llamapi enable qwen3:4b --platform rkllm/rk3588 --id abc --instance 2
```

同一 ID 已存在且指向同一模型目录时，命令更新实例数和默认参数；同一 ID 指向其他模型目录时报告冲突。

多个本地变体且未指定平台时，交互终端提示选择；非交互环境要求显式指定平台。

### `disable`

从 `llamapi-server` 配置中移除模型自动加载条目。

```text
llamapi disable <MODEL_OR_ID> [OPTIONS]
```

| 参数或选项 | 说明 |
|:---:|:---:|
| `MODEL_OR_ID` | 模型名或自动加载配置中的 ID |
| `-p, --platform <PLATFORM>` | 指定平台和芯片 |
| `--now` | 移除配置后立即卸载模型 |

示例：

```bash
llamapi disable qwen3:4b
llamapi disable qwen3:4b --now
llamapi disable abc
```

参数精确匹配配置 ID 时，移除该 ID；否则按模型名和平台生成默认 ID 后移除。自定义 ID 必须显式指定，`disable qwen3:4b` 不会间接删除自定义 ID。
