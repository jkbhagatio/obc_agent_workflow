Target domain: <domain-name>

# Delta PRD: <change-title>

## Overview
- <1–3 sentences: what this delta changes in the target domain>

## ADDED Requirements

<!--
Use the canonical structure:
### Requirement: <stable requirement title>
<Requirement statement: MUST/SHOULD...>

#### Scenario: <name>
- GIVEN ...
- WHEN ...
- THEN ...
- AND ...
-->

### Requirement: <requirement-title>
<The system MUST/SHOULD ...>

#### Scenario: <scenario-name>
- **GIVEN** <preconditions>
- **WHEN** <action>
- **THEN** <expected result>
- **AND** <additional expected result (optional)>

## MODIFIED Requirements

<!--
For MODIFIED requirements, include the FULL updated requirement block (do not partial-patch).
Avoid renaming requirement titles unless absolutely necessary.
-->

### Requirement: <existing-requirement-title>
<Updated requirement statement>

#### Scenario: <scenario-name>
- **GIVEN** ...
- **WHEN** ...
- **THEN** ...

## REMOVED Requirements

<!--
List requirements to remove. If none, write "(None)".
-->

(None)

## Design
> Combine requirements ("what") and design ("how") in one document. Keep design scoped to the target domain.

### High-level design
- <component/flow 1>
- <component/flow 2>

### Key interfaces / data contracts (if applicable)
- Inputs:
  - <input contract>
- Outputs:
  - <output contract>

### Code snippets (only for non-obvious parts)
```python
# <snippet filename/path (optional)>
# <minimal snippet illustrating the non-obvious part>
```

### Non-obvious implementation details
- <detail 1>
- <detail 2>

### Memory/compute considerations
<compute cost notes>
<memory cost notes>
<defaults that bound cost>

## Beads breakdown (execution plan)

List executable issues you would create in Beads.

- <Issue title>
    - <1–3 sentence description>
- <Issue title>
    - <1–3 sentence description>

### Dependencies / blockers

- <Issue B> depends on <Issue A>
    - <blocker details>

### Handoff notes

- <work that should be done by a different agent, if any>
