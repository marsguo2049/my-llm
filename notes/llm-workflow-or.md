# LLM Workflow OR

> From calling models to optimizing AI workflows.

## Idea

The core goal is not to build one more image or video generator. It is to make heterogeneous AI capabilities reusable from Python as modular building blocks, then compose them into workflows and eventually optimize which models, configurations, and compute budgets should be used for each task.

A model is therefore treated as a **resource that provides a capability**, rather than as the application itself.

Examples of capabilities:

- understand a user request;
- write or transform text;
- create a storyboard;
- generate prompts;
- generate or edit images;
- generate video from text, images, or keyframes;
- evaluate text, images, and video;
- revise a failed result;
- choose the next action in a workflow.

Python acts as the orchestration layer that connects these capabilities.

## First demo: story to video

The first end-to-end demo should deliberately stay small and inspectable:

```text
User idea
  -> story / shot planning
  -> structured storyboard
  -> changing prompts for each shot
  -> keyframe generation
  -> short video generation between keyframes
  -> evaluation / retry
  -> stitching
  -> final video
```

A minimal target is:

```text
one sentence
  -> 4 keyframes
  -> 3 short clips
  -> 1 final video
```

The important result is not cinematic quality by itself. The important result is proving that text, image, video, and evaluation models can be called from one Python workflow through stable interfaces.

ComfyUI can remain underneath as a model execution backend when useful, while Python owns the higher-level workflow, state, branching, retries, and model selection.

## Capability abstraction

The first software layer should expose simple capability-oriented interfaces such as:

```python
generate_text(...)
generate_prompt(...)
generate_image(...)
generate_video(...)
evaluate_image(...)
evaluate_video(...)
```

Each backend can implement one or more capabilities. A backend might be a local model, a ComfyUI API workflow, a hosted API, or another inference engine.

The orchestration layer should not need to know the internal node graph of every backend.

## Capability registry

Each usable model/configuration should eventually be represented by structured metadata, for example:

```text
provider / backend
model
capabilities
supported inputs
supported outputs
parameter schema
latency
monetary cost
GPU / VRAM requirement
historical quality
reliability
known task strengths
known task weaknesses
```

This turns the available AI models into a heterogeneous resource pool.

## From routing to workflow optimization

The longer-term research question is broader than choosing a single model for one query.

For workflow step i, an action can be represented as:

```text
a_i = (model, configuration, prompt strategy, compute budget)
```

The outcome can be evaluated on several objectives:

- task quality;
- prompt / instruction adherence;
- identity or style consistency;
- temporal consistency for video;
- latency;
- API or compute cost;
- memory / GPU demand;
- failure and retry probability.

A simple multi-objective view is:

```text
maximize quality
minimize cost
minimize latency
minimize compute
```

The quality of one workflow step may depend on decisions made earlier, so optimizing each component independently is not necessarily optimal. Model and parameter combinations can have interaction effects across the workflow.

## Learning from feedback

After each execution, an evaluator can produce structured feedback and scores. These observations become experience data:

```text
Task context
  -> chosen workflow / model / parameters
  -> generated result
  -> evaluation
  -> performance record
```

Over time the system can learn which actions work well for which task contexts instead of repeatedly trying every model and parameter combination.

This creates a natural bridge to topics such as:

- model routing;
- contextual bandits;
- stochastic optimization;
- multi-objective optimization;
- sequential decision-making;
- adaptive resource allocation;
- workflow / DAG optimization.

## Connection to MAS

A multi-agent system can describe **who does what**:

```text
Planner Agent
Prompt Agent
Image Agent
Video Agent
Critic Agent
```

Operations Research can decide **which resources those roles should use and when**:

```text
Which model?
Which configuration?
How much compute?
Retry or continue?
Use a long-video model or a short-shot pipeline?
Which workflow topology is appropriate for this task?
```

A useful interpretation is:

> MAS provides the organizational structure; OR provides the resource-allocation and workflow-decision layer.

## Development roadmap

### Phase 0 — Capability interfaces

- Define task, result, artifact, model, configuration, and evaluation schemas.
- Implement a small capability registry.
- Keep providers replaceable.

### Phase 1 — Story-to-video MVP

- User idea -> storyboard JSON.
- Generate a small set of keyframes.
- Generate short clips between keyframes.
- Stitch clips into one output.
- Log every model call, parameter set, runtime, and output.

### Phase 2 — Evaluation loop

- Add automated and human evaluation.
- Support retry / revise / accept decisions.
- Store structured performance history.

### Phase 3 — Multiple interchangeable backends

- Register alternative models for the same capability.
- Compare quality, latency, cost, and resource use.
- Build reproducible benchmark tasks.

### Phase 4 — Router / optimizer

- Select model + configuration dynamically from task context.
- Start with transparent baselines before complex learning methods.
- Compare rule-based routing, exhaustive small-instance search, bandit-style learning, and OR formulations where appropriate.

### Phase 5 — Workflow optimization

Move from selecting a model for one step to selecting the complete workflow:

```text
Task
  -> workflow topology
  -> model assignment
  -> configuration
  -> compute budget
  -> execution
  -> evaluation
  -> learning
```

The system may decide, for example, between a single long-video model and a storyboard -> keyframes -> short-clips workflow based on task requirements and budget.

## Repository direction

Recommended standalone repository:

- **Display name:** LLM Workflow OR
- **Repository slug:** `llm-workflow-or`
- **Positioning:** Modular multi-model workflow orchestration and optimization for generative AI.

Although the short name says LLM, the scope intentionally includes multimodal foundation models for text, image, audio, and video. The research object is the workflow and resource-allocation problem rather than one specific model family.

## Visibility

Recommended: **public**.

Public content should include architecture, capability interfaces, demos, benchmark definitions, and reproducible research code. Keep credentials, private model paths, unpublished datasets, sensitive inputs, and any confidential experiments outside the repository.

## Status

**Concept -> MVP.**

The immediate next milestone is the smallest working Python pipeline that proves model capabilities can be treated as interchangeable building blocks.
