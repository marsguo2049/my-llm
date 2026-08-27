# My LLM

[English](README.md) | **简体中文**

关于学习和使用本地及在线大语言模型的笔记、实践教程与实验记录。

这个仓库是一份公开的 LLM 学习笔记。我希望通过实践、教学和分享来加深理解，同时把别人可以照着完成的教程与可能持续变化的个人观察分开整理。

## 这里记录什么

仓库分为两个主要部分：

- **`guides/`**：可复现的实践教程，读者可以按照步骤操作。
- **`notes/`**：个人思考、模型比较、踩坑经验和可能随时间变化的判断。

内容可以涉及本地部署、模型选择、使用工具、在线模型、隐私、成本，以及完整的 LLM 实践工作流。

## 仓库结构

```text
my-llm/
├── README.md
├── README.zh-CN.md
│
├── guides/                       教程与可分享的实践方法
│   ├── README.md                 教程索引
│   ├── local-deployment/         本地运行大模型
│   ├── model-selection/          根据硬件和任务选择模型
│   ├── tools-and-clients/        LM Studio 等使用工具
│   ├── online-llms/              在线模型与服务
│   └── practical-workflows/      面向具体任务的完整工作流
│
├── notes/                        个人笔记与观察
│   └── README.md                 笔记索引与写作约定
│
├── .gitignore
└── LICENSE                       MIT 许可证
```

## 已发布教程

- [在 Windows 本地运行文本生成大模型](guides/local-deployment/README.zh-CN.md)——面向零基础用户，涵盖硬件判断、GGUF 模型选择、LM Studio、加载参数、本地 API 验证、隐私和故障排查。

## 当前实践项目

- [Local LLM Word Translator](https://github.com/marsguo2049/local-llm-word-translator)——通过 LM Studio 调用本地大模型、以隐私保护为优先并支持断点续传的 Word 翻译工作流。

## 新的项目方向

- [LLM Workflow OR](notes/llm-workflow-or.md)——把文本、提示词、图像、视频与评估等 AI 能力抽象成 Python 可调用的模块，并进一步研究如何针对具体任务选择和优化模型、参数、计算预算与工作流结构。

## 写作原则

1. 从第一性原理出发，并解释读者可能不熟悉的术语。
2. 遵循二八法则：优先介绍能够产生实际结果的最小工作流。
3. 将经过核实的事实与个人体验、判断明确区分。
4. 对模型推荐、软件行为等可能变化的信息标注核查日期。
5. 优先引用官方文档和第一手资料。
6. 不公开私人文档、身份凭证、运行日志、本机专用路径或个人模型配置。

## 当前状态

仓库目前处于起步阶段。第一份 Windows + LM Studio + GGUF 零基础部署教程已经发布，并会随着软件生态变化持续更新。

## 许可证

[MIT](LICENSE)
