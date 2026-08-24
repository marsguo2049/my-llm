# Run a text-generation LLM locally on Windows

**English** | [简体中文](README.zh-CN.md) | [Guide index](../README.md)

> Scope: Windows, NVIDIA GPU, LM Studio, and GGUF text-generation models  
> Last verified: 2026-08-24

This guide helps a beginner reach one concrete result: download a trustworthy model, run it locally in LM Studio, complete a chat, and verify the local API.

## 1. The four pieces you are installing

| Piece | What it is | Example |
| --- | --- | --- |
| Model weights | The learned parameters; usually the largest files | A `.gguf` file |
| Quantization | A compressed representation that reduces memory use, with some quality tradeoff | `Q4_K_M` |
| Runtime | The engine that reads the model and performs inference | CUDA llama.cpp runtime |
| Application | The interface that manages models, chats, settings, and APIs | LM Studio |

LM Studio is not the model. A GGUF file is not an application. The runtime connects the two.

## 2. Check the computer first

LM Studio's official Windows guidance recommends an AVX2-capable x64 CPU, at least 16 GB RAM, and at least 4 GB dedicated VRAM. More memory allows a larger model or context.

Check the GPU in PowerShell:

```powershell
nvidia-smi
```

Also open **Task Manager → Performance** and note:

- system RAM;
- dedicated GPU memory (VRAM);
- free disk space.

Model size on disk is only the starting point. Loading also needs runtime overhead and a KV cache. A file fitting on disk does not mean it fits in VRAM or RAM.

Approximate beginner starting points:

| Hardware | First model class | Initial context |
| --- | --- | --- |
| 16 GB RAM, 4–6 GB VRAM | 3B–4B, 4-bit | 4096 |
| 16–32 GB RAM, 8 GB VRAM | 7B–8B, `Q4_K_M` | 4096–8192 |
| 32 GB RAM, 12 GB VRAM | 8B–14B, `Q4_K_M` | 4096–8192 |
| 32–64 GB RAM, 16 GB VRAM | 14B or a memory-efficient MoE model | 8192 |
| 64 GB+ RAM, 24 GB VRAM | 20B–32B-class 4-bit models | 8192, then test higher |

These are estimates, not guarantees. Architecture, context length, KV-cache format, and GPU offload all change memory use.

## 3. Choose the right kind of model

For chat, writing, summarization, or translation, choose **Instruct**, **Chat**, or **IT** (instruction-tuned), not **Base**.

For a first download, prefer a publisher's official GGUF repository. If none exists, use a recognized conversion publisher such as `ggml-org` or `lmstudio-community`, then verify the linked base model and license.

### Starter models

| Model | Good starting use | Suggested file | Hugging Face |
| --- | --- | --- | --- |
| Qwen3 8B | General chat and multilingual work on mainstream hardware | `Q4_K_M` | [Qwen/Qwen3-8B-GGUF](https://huggingface.co/Qwen/Qwen3-8B-GGUF) |
| Qwen3 14B | Better general quality when more memory is available | `Q4_K_M` | [Qwen/Qwen3-14B-GGUF](https://huggingface.co/Qwen/Qwen3-14B-GGUF) |
| Gemma 3 4B IT | Smaller instruction-tuned model for limited hardware | `Q4_K_M` | [ggml-org/gemma-3-4b-it-GGUF](https://huggingface.co/ggml-org/gemma-3-4b-it-GGUF) |
| gpt-oss 20B | Reasoning and tool-oriented experiments on stronger hardware | supplied `MXFP4` GGUF | [ggml-org/gpt-oss-20b-GGUF](https://huggingface.co/ggml-org/gpt-oss-20b-GGUF) |

Model recommendations age quickly. Read the model card and license, check the file size, and test your real task before downloading a larger model. A larger parameter count does not automatically mean a better experience on a memory-constrained computer.

### Quantization in plain language

| Format | Practical meaning |
| --- | --- |
| `BF16` / `F16` | Very large, high fidelity; rarely the beginner choice |
| `Q8_0` | Large, small quality loss |
| `Q6_K` / `Q5_K_M` | Higher fidelity but more memory than 4-bit |
| `Q4_K_M` | Recommended general balance of size, quality, and compatibility |
| `Q3_*` / lower | Smaller, but quality loss becomes easier to notice |

If uncertain, begin with `Q4_K_M`. LM Studio's official guide likewise recommends choosing 4-bit or higher when the machine can run it.

## 4. Install LM Studio safely

Download the desktop application from the official page:

- [LM Studio official download](https://lmstudio.ai/download?os=win32)
- [Official system requirements](https://lmstudio.ai/docs/app/system-requirements)

Use the normal Windows installer for this beginner workflow.

You may see this command online:

```powershell
irm https://lmstudio.ai/install.ps1 | iex
```

It downloads and immediately executes a remote PowerShell script. LM Studio currently documents it for the headless `llmster` daemon, not as the ordinary desktop-app installation path. Use it only when you intentionally want a headless setup and understand the script-execution implications.

## 5. Download a model

The easiest method is LM Studio's **Discover** page:

1. Open **Discover**.
2. Search the exact `publisher/model` name or paste the full Hugging Face URL.
3. Confirm the publisher and model card.
4. Select `Q4_K_M` for the first attempt.
5. Check the download size and free disk space.
6. Download only one quantization at first.

LM Studio's downloader handles split GGUF files. When downloading manually from Hugging Face, a model whose filename contains `00001-of-0000N` requires every shard.

### Keep models in an organized directory

LM Studio lets you change the models directory from **My Models**. This is the simplest way to keep large model files on a dedicated drive or folder.

For a GGUF already downloaded elsewhere, LM Studio also provides:

```powershell
lms import "D:\Models\example-model.Q4_K_M.gguf" --dry-run
lms import "D:\Models\example-model.Q4_K_M.gguf" --copy
```

The default import operation moves the source file. Use `--copy` to keep the original. `--hard-link` avoids a second physical copy but normally requires source and destination to be on the same filesystem; both paths then refer to the same underlying data.

Official references: [download models](https://lmstudio.ai/docs/app/basics/download-model) and [import models](https://lmstudio.ai/docs/cli/local-models/import).

## 6. Load the model

Start with conservative settings:

```text
Context Length:              4096 (use 8192 when the task needs it)
GPU Offload:                 Auto, or the highest value that loads reliably
Max Concurrent Predictions:  1
Flash Attention:             On, when supported
Speculative Decoding:        Off
Thinking / Reasoning:        Off for simple chat or direct translation
```

Leave batch sizes and RoPE values at their defaults for the first run.

Parameter meanings:

| Setting | What it changes | Beginner advice |
| --- | --- | --- |
| Context Length | Maximum working text history | Larger uses more KV-cache memory; start at 4096 or 8192 |
| GPU Offload | How much model computation/weight storage goes to the GPU | Auto first; reduce if loading fails |
| Flash Attention | More efficient attention implementation | Enable when supported |
| KV Cache on GPU | Keeps conversation cache in VRAM | Faster but consumes VRAM; disable if memory is tight |
| KV Cache quantization | Compresses the conversation cache | Keep default first; try Q8 only for memory pressure |
| Evaluation batch size | Number of prompt tokens processed together | Leave default; lower it only for memory errors |
| Concurrent predictions | Simultaneous generations | Use 1 on a personal machine |
| Keep model in memory | Avoids reloading between uses | Disable when RAM is limited |
| mmap | Maps model data from disk instead of eagerly copying all of it | Keep enabled unless a runtime warning or special CPU offload setup suggests otherwise |

Click **Load Model**. If the estimate is above available memory, do not select “Load Anyway” as the first response—reduce context/offload or choose a smaller model.

## 7. Complete the first chat

Open a new chat and send:

```text
Please reply with exactly: Local model is running.
```

A successful reply proves that the model, runtime, prompt template, and basic inference path work. In Task Manager or `nvidia-smi`, GPU memory usage should rise when GPU offload is active.

Generation speed is normally shown in tokens per second. Prompt processing speed and output generation speed are different measurements.

## 8. Start and verify the local API

Open **Developer / Local Server** in LM Studio:

1. Load the model.
2. Start the server on port `1234`.
3. Keep **Serve on Local Network** off for a local-only setup.
4. Keep CORS and MCP access off unless an application specifically needs them.
5. Authentication can remain off when the server listens only on `127.0.0.1`; enable it before exposing the server to a network.
6. Confirm that the model shows `READY`.

List models from PowerShell:

```powershell
$models = Invoke-RestMethod -Uri "http://127.0.0.1:1234/v1/models"
$models | ConvertTo-Json -Depth 5
```

Send a UTF-8 test request through LM Studio's native API:

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

Expected result:

```text
本地 API 运行成功
```

LM Studio also provides OpenAI-compatible endpoints such as `/v1/chat/completions` and `/v1/responses`.

## 9. Troubleshooting in the right order

### Not enough memory or model fails to load

1. Eject duplicate loaded instances.
2. Close memory-heavy applications.
3. Reduce Context Length to 4096.
4. Use one concurrent prediction.
5. Move KV cache to RAM or use Q8 cache if appropriate.
6. Reduce GPU Offload if VRAM is full.
7. Choose a smaller model or quantization.

### It loads but runs mostly on CPU

- Select the CUDA llama.cpp runtime for an NVIDIA GPU.
- Confirm GPU Offload is not zero.
- Watch dedicated GPU memory with Task Manager or `nvidia-smi` while generating.
- Update the selected CUDA runtime if the model architecture is newer than the runtime. You do not need to install every CPU, Vulkan, and CUDA runtime pack.

### Empty answer or only reasoning text

- Disable Thinking/Reasoning for a simple test.
- Increase the output token limit if reasoning consumed the entire allowance.
- Confirm that the model is instruction-tuned and has the correct chat template.

### HTTP 500 from the local API

1. Read LM Studio's Developer Logs for the actual runtime message.
2. Retry a one-line prompt to separate API health from long-context problems.
3. Eject and reload one model instance.
4. Restart the local server.
5. Reduce context or batch size if the log reports allocation failure.
6. Update only the compatible selected runtime when the log reports an unsupported model architecture or parser issue.

### Garbled Chinese in PowerShell

Send JSON as UTF-8 bytes, as shown in the API example. A terminal rendering issue does not always mean the model received corrupted text; confirm the request in Developer Logs.

## 10. Privacy checklist

- A downloaded local model can chat and serve local API requests without internet connectivity.
- Discover search, model downloads, runtime downloads, and update checks require internet access.
- “Local UI” does not guarantee local inference if a cloud model/provider is selected.
- Keep the server bound to `127.0.0.1` unless network access is intentional.
- Use authentication before allowing other devices to connect.
- Do not expose API ports directly to the public internet.
- Review plugins, MCP servers, scripts, and third-party clients separately; they may make their own network requests.
- Do not publish prompts, documents, logs, model paths, or API tokens in Git repositories.

## 11. Unload or remove a model

To release RAM and VRAM, click **Eject** in LM Studio or run:

```powershell
lms unload --all
```

Deleting a chat does not unload a model. Unloading releases memory but does not delete the GGUF file. Delete a model from **My Models** only when you intend to reclaim disk space.

## 12. What success looks like

You are finished when all four statements are true:

- the model answers in LM Studio Chat;
- Task Manager or `nvidia-smi` shows expected GPU use;
- `GET /v1/models` returns a model identifier;
- the native API test returns `本地 API 运行成功`.

From here, the local server can support applications such as [Local LLM Word Translator](https://github.com/marsguo2049/local-llm-word-translator).

## Official references

- [LM Studio download](https://lmstudio.ai/download?os=win32)
- [System requirements](https://lmstudio.ai/docs/app/system-requirements)
- [Offline operation](https://lmstudio.ai/docs/app/offline)
- [Download a model](https://lmstudio.ai/docs/app/basics/download-model)
- [Import a model](https://lmstudio.ai/docs/cli/local-models/import)
- [Load and unload with `lms`](https://lmstudio.ai/docs/cli/local-models/load)
- [Local API server](https://lmstudio.ai/docs/developer/core/server)
- [LM Studio REST API](https://lmstudio.ai/docs/developer/rest)
- [Native chat endpoint](https://lmstudio.ai/docs/developer/rest/chat)
- [OpenAI-compatible endpoints](https://lmstudio.ai/docs/developer/openai-compat)
