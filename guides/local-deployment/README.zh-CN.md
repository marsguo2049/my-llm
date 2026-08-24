# 在 Windows 本地运行文本生成大模型

[English](README.md) | **简体中文** | [教程索引](../README.md)

> 范围：Windows、NVIDIA 显卡、LM Studio、GGUF 文本生成模型  
> 最后核查：2026-08-24

这份指南只追求一个明确结果：让零基础用户下载一个可信模型，在 LM Studio 中完成本地对话，并验证本地 API 可以正常调用。

## 1. 先理解要安装的四样东西

| 组成 | 它是什么 | 例子 |
| --- | --- | --- |
| 模型权重 | 模型学习到的参数，通常是最大的文件 | 一个 `.gguf` 文件 |
| 量化 | 用一定质量损失换取更小体积和更低内存占用 | `Q4_K_M` |
| 运行时 | 读取模型并执行推理计算的引擎 | CUDA llama.cpp runtime |
| 应用程序 | 管理模型、对话、参数和 API 的界面 | LM Studio |

LM Studio 不是模型，GGUF 文件也不是应用程序；运行时负责把两者连接起来。

## 2. 先检查电脑

LM Studio 官方对 Windows 的建议是：x64 CPU 支持 AVX2，至少 16 GB RAM，并推荐至少 4 GB 独立显存。内存越多，越有机会运行更大的模型或上下文。

在 PowerShell 中检查 NVIDIA 显卡：

```powershell
nvidia-smi
```

同时打开 **任务管理器 → 性能**，记下：

- 系统内存（RAM）；
- 独立 GPU 内存（显存/VRAM）；
- 磁盘剩余空间。

模型文件大小只是起点。加载时还需要运行开销和 KV Cache。因此“磁盘放得下”不代表“显存或内存装得下”。

零基础用户可以参考以下起点：

| 硬件 | 第一次尝试的模型 | 初始上下文 |
| --- | --- | --- |
| 16 GB RAM、4–6 GB 显存 | 3B–4B、4-bit | 4096 |
| 16–32 GB RAM、8 GB 显存 | 7B–8B、`Q4_K_M` | 4096–8192 |
| 32 GB RAM、12 GB 显存 | 8B–14B、`Q4_K_M` | 4096–8192 |
| 32–64 GB RAM、16 GB 显存 | 14B 或节省计算的 MoE 模型 | 8192 |
| 64 GB 以上 RAM、24 GB 显存 | 20B–32B 级别的 4-bit 模型 | 从 8192 开始 |

这只是估算，不是保证。模型架构、上下文、KV Cache 格式和 GPU Offload 都会改变内存占用。

## 3. 选择正确类型的模型

如果用途是聊天、写作、摘要或翻译，应选择 **Instruct**、**Chat** 或 **IT**（经过指令微调），不要下载 **Base** 模型。

第一次下载时，优先选择模型发布方的官方 GGUF。如果官方没有提供，可以选择 `ggml-org`、`lmstudio-community` 等可信转换来源，但仍要检查它链接的基础模型和许可证。

### 入门模型

| 模型 | 适合的入门用途 | 建议文件 | Hugging Face |
| --- | --- | --- | --- |
| Qwen3 8B | 主流消费级硬件上的通用对话与多语言任务 | `Q4_K_M` | [Qwen/Qwen3-8B-GGUF](https://huggingface.co/Qwen/Qwen3-8B-GGUF) |
| Qwen3 14B | 内存较充足时获得更好的通用质量 | `Q4_K_M` | [Qwen/Qwen3-14B-GGUF](https://huggingface.co/Qwen/Qwen3-14B-GGUF) |
| Gemma 3 4B IT | 适合硬件有限的指令模型 | `Q4_K_M` | [ggml-org/gemma-3-4b-it-GGUF](https://huggingface.co/ggml-org/gemma-3-4b-it-GGUF) |
| gpt-oss 20B | 在更强硬件上体验推理和工具任务 | 仓库提供的 `MXFP4` GGUF | [ggml-org/gpt-oss-20b-GGUF](https://huggingface.co/ggml-org/gpt-oss-20b-GGUF) |

模型推荐很容易过期。下载大模型前应阅读模型卡和许可证、确认文件大小，并用自己的真实任务试用。参数更多不代表在内存受限的电脑上体验一定更好。

### 用人话理解量化

| 格式 | 实际含义 |
| --- | --- |
| `BF16` / `F16` | 体积很大、精度高，一般不是入门首选 |
| `Q8_0` | 体积较大，质量损失较小 |
| `Q6_K` / `Q5_K_M` | 比 4-bit 更接近原始质量，但更占内存 |
| `Q4_K_M` | 体积、质量和兼容性的常用平衡点，推荐从这里开始 |
| `Q3_*` 或更低 | 更省内存，但质量损失更容易察觉 |

不确定时就从 `Q4_K_M` 开始。LM Studio 官方指南同样建议：电脑能力允许时，选择 4-bit 或更高的量化。

## 4. 安全安装 LM Studio

从官方页面下载桌面应用：

- [LM Studio 官方下载](https://lmstudio.ai/download?os=win32)
- [官方系统要求](https://lmstudio.ai/docs/app/system-requirements)

零基础用户直接使用普通 Windows 安装程序即可。

你可能在网上看到下面这条命令：

```powershell
irm https://lmstudio.ai/install.ps1 | iex
```

它会下载并立即执行远程 PowerShell 脚本。LM Studio 当前文档将它用于无界面的 `llmster` 后台服务，并不是普通桌面应用的常规安装方式。只有在明确需要 headless 部署并理解远程脚本执行风险时才使用。

## 5. 下载模型

最简单的方法是使用 LM Studio 的 **Discover** 页面：

1. 打开 **Discover**。
2. 搜索准确的 `发布者/模型名`，或粘贴完整 Hugging Face 地址。
3. 核对发布者和模型卡。
4. 第一次选择 `Q4_K_M`。
5. 检查下载体积和磁盘剩余空间。
6. 暂时只下载一个量化版本。

LM Studio 会自动处理分片 GGUF。如果在 Hugging Face 手动下载的文件名包含 `00001-of-0000N`，必须下载全部分片。

### 整理模型存放位置

可以在 LM Studio 的 **My Models** 中更改模型目录。这是把大模型统一放在专用磁盘或文件夹中最简单的方法。

如果已经在其他地方下载了 GGUF，也可以使用：

```powershell
lms import "D:\Models\example-model.Q4_K_M.gguf" --dry-run
lms import "D:\Models\example-model.Q4_K_M.gguf" --copy
```

默认导入操作会移动原文件。需要保留原文件时使用 `--copy`。`--hard-link` 不会制造第二份实体数据，但源文件和目标位置通常必须位于同一文件系统；两个路径随后指向同一份底层数据。

官方说明：[下载模型](https://lmstudio.ai/docs/app/basics/download-model)与[导入模型](https://lmstudio.ai/docs/cli/local-models/import)。

## 6. 加载模型

第一次采用保守参数：

```text
Context Length（上下文）:           4096（任务需要时再用 8192）
GPU Offload（GPU 卸载）:            Auto，或能够稳定加载的最高值
Max Concurrent Predictions（并发）: 1
Flash Attention:                    支持时开启
Speculative Decoding（推测解码）:   关闭
Thinking / Reasoning（思考）:       普通聊天或直接翻译时关闭
```

第一次运行时，Batch Size 和 RoPE 参数保持默认。

各参数的实际作用：

| 设置 | 改变什么 | 入门建议 |
| --- | --- | --- |
| Context Length | 模型一次能够参考的最大文本历史 | 越大越占 KV Cache；从 4096 或 8192 开始 |
| GPU Offload | 多少权重与计算放到显卡 | 先用 Auto；加载失败再降低 |
| Flash Attention | 更高效的注意力计算 | 支持时开启 |
| Offload KV Cache to GPU | 把对话缓存放入显存 | 更快但占显存；显存紧张时关闭 |
| KV Cache Quantization | 压缩对话缓存 | 先保持默认；内存压力大时再尝试 Q8 |
| Evaluation Batch Size | 一次处理多少输入 token | 先保持默认；分配内存失败时才降低 |
| Concurrent Predictions | 同时生成多少条回复 | 个人电脑设为 1 |
| Keep Model in Memory | 空闲时仍保留模型 | 内存紧张时关闭 |
| mmap | 从磁盘映射模型，而不是一次复制全部权重 | 通常开启；只有运行时警告或特殊 CPU 卸载才调整 |

点击 **Load Model**。如果内存估算已经超过可用资源，不要第一时间选择 “Load Anyway”，应先降低上下文或 Offload，或者换更小的模型。

## 7. 完成第一次对话

新建聊天并发送：

```text
请只回复：本地模型运行成功
```

正确回复说明模型、运行时、聊天模板和基础推理链路已经打通。GPU Offload 生效时，任务管理器或 `nvidia-smi` 中的显存占用应明显上升。

生成速度通常使用 tokens/s 表示。输入提示词处理速度和回复生成速度是两个不同指标。

## 8. 启动并验证本地 API

打开 LM Studio 的 **Developer / Local Server**：

1. 加载模型。
2. 在 `1234` 端口启动服务器。
3. 只在本机使用时，关闭 **Serve on Local Network**。
4. 除非应用明确需要，否则关闭 CORS 和 MCP 访问。
5. 服务器只监听 `127.0.0.1` 时可以暂不启用身份验证；对局域网开放前应启用认证。
6. 确认模型显示 `READY`。

在 PowerShell 中列出模型：

```powershell
$models = Invoke-RestMethod -Uri "http://127.0.0.1:1234/v1/models"
$models | ConvertTo-Json -Depth 5
```

通过 LM Studio 原生 API 发送 UTF-8 测试请求：

```powershell
$modelId = $models.data[0].id

$requestBody = @{
    model             = $modelId
    input             = "请只回复：本地 API 运行成功"
    temperature       = 0.1
    max_output_tokens = 64
    reasoning         = "off"
    store             = $false
} | ConvertTo-Json -Depth 5

$utf8Body = [System.Text.Encoding]::UTF8.GetBytes($requestBody)

$response = Invoke-RestMethod `
    -Uri "http://127.0.0.1:1234/api/v1/chat" `
    -Method Post `
    -ContentType "application/json; charset=utf-8" `
    -Body $utf8Body

$response.output |
    Where-Object { $_.type -eq "message" } |
    Select-Object -ExpandProperty content
```

预期结果：

```text
本地 API 运行成功
```

LM Studio 还提供 `/v1/chat/completions`、`/v1/responses` 等 OpenAI-compatible 接口。

## 9. 按正确顺序排查问题

### 内存不足或无法加载

1. Eject 重复加载的模型实例。
2. 关闭占用大量内存的应用。
3. 把 Context Length 降到 4096。
4. 并发数设为 1。
5. 视情况把 KV Cache 放到 RAM，或使用 Q8 KV Cache。
6. 显存不足时降低 GPU Offload。
7. 换更小的模型或量化。

### 能加载，但主要使用 CPU

- NVIDIA 显卡选择 CUDA llama.cpp runtime。
- 确认 GPU Offload 不是 0。
- 生成时使用任务管理器或 `nvidia-smi` 观察独立显存。
- 模型架构比运行时新时，更新当前选中的 CUDA runtime；不需要同时安装全部 CPU、Vulkan 和 CUDA runtime。

### 回复为空或只有思考内容

- 简单测试时关闭 Thinking/Reasoning。
- 如果思考用完了输出额度，提高最大输出 token。
- 确认下载的是指令模型，并使用正确聊天模板。

### 本地 API 返回 HTTP 500

1. 查看 LM Studio Developer Logs 中真正的运行时错误。
2. 用一句话请求重试，区分 API 健康问题和长上下文问题。
3. Eject 后只重新加载一个模型实例。
4. 重启本地服务器。
5. 日志显示内存分配失败时，降低上下文或 Batch Size。
6. 日志显示架构或解析器不受支持时，只更新兼容的当前运行时。

### PowerShell 中文显示乱码

像上面的 API 示例一样，把 JSON 转为 UTF-8 bytes 后发送。终端显示异常不一定代表模型收到乱码，可以在 Developer Logs 中核对请求正文。

## 10. 隐私检查清单

- 本地模型下载完成后，对话和本地 API 推理可以完全离线运行。
- Discover 搜索、模型下载、运行时下载和更新检查需要联网。
- “界面在本地”不代表“推理一定在本地”；如果选择云端模型或服务商，数据仍会离开电脑。
- 没有明确需要时，让服务器只绑定 `127.0.0.1`。
- 向其他设备开放前启用认证。
- 不要把 API 端口直接暴露到公网。
- 插件、MCP、脚本和第三方客户端可能进行自己的网络请求，需要分别检查。
- 不要把提示词、私人文档、日志、本机模型路径或 API Token 提交到 Git。

## 11. 卸载或删除模型

释放内存和显存时，在 LM Studio 中点击 **Eject**，或者执行：

```powershell
lms unload --all
```

删除聊天不会卸载模型。卸载只释放内存，不会删除 GGUF。只有在希望释放磁盘空间时，才到 **My Models** 删除模型文件。

## 12. 怎样才算部署完成

下面四项全部成立，就已经成功：

- 模型能在 LM Studio Chat 中回答；
- 任务管理器或 `nvidia-smi` 显示预期的 GPU 使用；
- `GET /v1/models` 返回模型标识；
- 原生 API 测试返回“本地 API 运行成功”。

完成后，本地服务器就可以支持 [Local LLM Word Translator](https://github.com/marsguo2049/local-llm-word-translator) 等实际应用。

## 官方参考资料

- [LM Studio 下载](https://lmstudio.ai/download?os=win32)
- [系统要求](https://lmstudio.ai/docs/app/system-requirements)
- [离线运行说明](https://lmstudio.ai/docs/app/offline)
- [下载模型](https://lmstudio.ai/docs/app/basics/download-model)
- [导入模型](https://lmstudio.ai/docs/cli/local-models/import)
- [使用 `lms` 加载与卸载](https://lmstudio.ai/docs/cli/local-models/load)
- [本地 API 服务器](https://lmstudio.ai/docs/developer/core/server)
- [LM Studio REST API](https://lmstudio.ai/docs/developer/rest)
- [原生 Chat 接口](https://lmstudio.ai/docs/developer/rest/chat)
- [OpenAI-compatible 接口](https://lmstudio.ai/docs/developer/openai-compat)
