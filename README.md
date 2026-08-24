# My LLM

**English** | [简体中文](README.zh-CN.md)

Notes, practical guides, and experiments from learning and using local and online LLMs.

This repository is a public learning notebook for understanding large language models through practice, teaching, and sharing. It covers both local and online LLMs while keeping reproducible tutorials separate from personal observations.

## What belongs here

The repository has two main sections:

- **`guides/`** contains practical, reproducible tutorials that other readers can follow.
- **`notes/`** contains reflections, comparisons, lessons learned, and opinions that may evolve over time.

Topics may include local deployment, model selection, tools and clients, online model usage, privacy, cost, and practical LLM workflows.

## Repository structure

```text
my-llm/
├── README.md
├── README.zh-CN.md
│
├── guides/                       tutorials and shared practices
│   ├── README.md                 guide index
│   ├── local-deployment/         running LLMs locally
│   ├── model-selection/          choosing models for hardware and tasks
│   ├── tools-and-clients/        LM Studio and other user-facing tools
│   ├── online-llms/              using hosted models and services
│   └── practical-workflows/      complete task-oriented workflows
│
├── notes/                        personal notes and observations
│   └── README.md                 notes index and writing conventions
│
├── .gitignore
└── LICENSE                       MIT License
```

## Current practical project

- [Local LLM Word Translator](https://github.com/marsguo2049/local-llm-word-translator) — a privacy-first, resumable Word translation workflow using a local LLM through LM Studio.

## Writing principles

1. Start from first principles and explain unfamiliar terms.
2. Apply the 80/20 rule: prioritize the smallest workflow that produces a useful result.
3. Separate verified facts from personal experience and opinion.
4. Date information that may change, such as model recommendations or software behavior.
5. Prefer official documentation and primary sources.
6. Never publish private documents, credentials, logs, machine-specific paths, or personal model configurations.

## Status

This repository is at an early stage. The initial focus will be a beginner-friendly Windows guide for running a text-generation GGUF model locally with LM Studio.

## License

[MIT](LICENSE)
