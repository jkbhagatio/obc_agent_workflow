# PRD

## Overview

This project builds **multimodal foundation models trained on neural data**, with a focus on
learning representations that unify neural recordings with aligned modalities (e.g., behavior,
video, stimuli, task variables, text/metadata) and enabling robust downstream evaluation. A core
deliverable is a reproducible training + inference stack, plus benchmark-facing tooling
(including CodaBench submissions).

## Goals

- Train scalable multimodal foundation models that learn useful, general representations from
  neural data.
- Support rigorous, reproducible evaluation across standardized benchmarks and internal tasks.
- Make experimentation fast: clear configs, modular architectures, deterministic pipelines where
  feasible.
- Keep the codebase extensible: new modalities, datasets, model variants, and evaluation tasks
  should be easy to add.

## References

- (Project-level references: dataset standards, benchmark specs, relevant papers, internal docs)

---

## Domain: codabench

### Overview

Code related to training models and generating/uploading predictions to CodaBench for
standardized evaluation. This domain must support reproducible training, deterministic inference,
correct submission output formatting, and easy iteration on model architectures.

### Goals

- Make it easy to iterate on model architectures without rewriting the submission pipeline.
- Ensure submission artifacts are self-contained (weights + config + entrypoint).
- Keep inference deterministic and resource-bounded by default.

### Requirement: Config-driven model selection via registry

The codabench pipeline MUST select the model architecture via configuration and instantiate
models through a registry (string name → constructor).

#### Scenario: Select PatchFormer by config

- **GIVEN** a config with `model.name = "patchformer"`
- **WHEN** the training or inference entrypoint initializes the model
- **THEN** PatchFormer is constructed with configured hyperparameters
- **AND** no code changes are required outside config to switch architectures

#### Scenario: Unknown model name

- **GIVEN** a config with an unknown `model.name`
- **WHEN** the entrypoint initializes the model
- **THEN** initialization fails with a clear error listing valid model names

### Requirement: PatchFormer architecture is available for sequence/time-series inputs

The codabench domain MUST include a PatchFormer model architecture suitable for
sequence/time-series style inputs, implemented with minimal dependencies.

#### Scenario: Forward pass shape contract

- **GIVEN** a batch of inputs shaped according to the codabench data contract
- **WHEN** PatchFormer runs a forward pass
- **THEN** the output tensor matches the expected prediction shape for the task

#### Scenario: Configurable patching knobs

- **GIVEN** a PatchFormer config specifying `patch_len` and `patch_stride`
- **WHEN** the model is instantiated
- **THEN** the model uses those values to form patch tokens
- **AND** changing them changes the effective token length (and compute)

### Requirement: Submission-safe checkpoint and config packaging

The codabench submission artifacts MUST include everything required for inference: model weights,
model config (including architecture choice), and a deterministic inference entrypoint.

#### Scenario: Inference round-trip

- **GIVEN** a trained checkpoint and its config
- **WHEN** inference loads them in a fresh environment
- **THEN** it produces predictions in the required output format

#### Scenario: Deterministic inference

- **GIVEN** a fixed seed and fixed checkpoint/config
- **WHEN** inference runs twice on identical input
- **THEN** predictions are identical (within floating tolerance if applicable)

### Requirement: Inference entrypoint preserves backwards compatibility

The codabench inference entrypoint MUST preserve existing behavior for the current default model
when configured to use it.

#### Scenario: Default architecture unchanged

- **GIVEN** a config selecting the legacy/default architecture
- **WHEN** inference runs
- **THEN** output formatting and semantics match the previous baseline behavior

### References

- (Domain references: benchmark rules, submission format docs, scoring protocol, internal
  baseline notes)

---

## Domain: scripts

### Overview

Project scripts and CLI tooling that support common workflows: dataset preparation, training
runs, evaluation, artifact packaging, and experiment bookkeeping.

### Goals

- Provide stable entrypoints for repeatable workflows.
- Keep scripts composable, testable, and predictable.
- Prefer configs over ad-hoc flags for complex workflows.

### Requirement: Scripts are runnable in the project environment

Scripts MUST be runnable via the project's standard environment tooling (e.g., `uv run ...`) and
fail with actionable error messages on missing dependencies or invalid inputs.

#### Scenario: Run script successfully

- **GIVEN** a valid invocation and required inputs
- **WHEN** the script is executed
- **THEN** it completes successfully and produces expected outputs

#### Scenario: Invalid arguments

- **GIVEN** invalid arguments
- **WHEN** the script is executed
- **THEN** it prints usage/help and exits non-zero

### Requirement: Script outputs are deterministic when applicable

When scripts generate artifacts intended for downstream automation (e.g., packaged checkpoints,
prediction files), they SHOULD be deterministic given fixed inputs/config.

#### Scenario: Stable output on repeated runs

- **GIVEN** identical inputs and config
- **WHEN** the script is executed twice
- **THEN** the produced artifacts match (byte-for-byte when feasible)

### References

- (Domain references: CLI conventions, internal runbooks, experiment tracking docs)
