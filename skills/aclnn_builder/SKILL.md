---
name: aclnn_builder
description: Guides end-to-end ACLNN custom operator development and adaptation in MindSpore (PyBoost/Pynative + KBK/Graph paths), including YAML definitions, code generation, GeneralInfer, kernel registration, bprop wiring, tests (UT), and docs. Use when the user mentions ACLNN, Ascend, operator adaptation/operator development, PyBoost, KBK, op_def YAML, gen_ops.py, bprop, or Ascend operator adaptation tasks.
---

# ACLNN Operator Development End-to-End Flow (MindSpore Adaptation)

## Goal

Land an ACLNN operator on the Ascend platform in MindSpore **end to end**: forward and backward paths, both PyBoost (Pynative) and KBK (Graph), dynamic shape/rank support, UT, documentation, export, and the required quality checks and validation.

## How To Use This Skill

- When the user says things like "integrate/adapt an ACLNN operator into MindSpore", "add an xxx interface to MindSpore that matches `torch_npu`", "use the skill to add an NPU operator", "add `xxx_op.yaml`", "how should PyBoost/KBK be written", "how is bprop registered", or "how should UT be added", proceed directly through this skill's workflow.

> **Important rule**: before executing each step, you must read the corresponding workflow file
> (`workflows/XX-xxx.md`) to get the detailed steps, constraints, and success criteria.
> Shared documents such as `reference.md` and `aclnn_doc` are stored under `../_shared/`.

## Execution Flow

### Workflow Execution Checklist

When using this skill to develop an ACLNN operator, **create a TODOLIST** and execute the following workflows in order.
**Steps marked `🔒 must not be skipped` are mandatory in every scenario.**
**Places marked `⛔ HARD GATE` must be completed before you continue, otherwise stop and wait for user confirmation.**

- [ ] **[Pre](workflows/00-pre-checks.md)** `🔒 must not be skipped`: pre-checks (Pre-A inventory check + Pre-B solution design + Pre-C call-chain inventory)
  - Input: operator name, PTA reference interface
  - **Required outputs**: PTA source review report, initialized Feature document
    > **⛔ HARD GATE 1**: before entering Step 1, you must confirm that these files have been generated into the workspace
  - Composite scenarios must also produce: the call-chain inventory based on `templates/aclnn-callchain-inventory.md`
  - After each later step, backfill the corresponding section of the Feature document
- [ ] **[Step 1](workflows/01-yaml-definition.md)**: YAML definition -> backfill Feature [5. YAML Definition](templates/feature-document.md#feature-yaml-definition)
  - Input: PTA source review report, Feature document
  - Output: `op_def` + `api_def` + doc YAML files
- [ ] **[Step 2](workflows/02-code-generation.md)**: code generation
  - Input: YAML files
  - Output: `gen_ops.py` runs successfully
- [ ] **[Step 3](workflows/03-general-infer.md)**: GeneralInfer -> backfill Feature [9. Dynamic Shape/Rank Support](templates/feature-document.md#feature-dynamic-shape) / [10. Validation And Error Handling](templates/feature-document.md#feature-validation-and-errors)
  - Input: YAML, PTA output-shape logic
  - Output: Infer implementation files
- [ ] **[Step 4](workflows/04-pyboost.md)**: PyBoost (Pynative) -> backfill Feature [7. Execution Modes And Adaptation](templates/feature-document.md#feature-execution-modes)
  - **Path 1**: skip handwritten implementation and only validate the auto-generated outputs
  - **Path 2**: handwrite Customize implementation files (argument conversion + ACLNN call)
  - Input: YAML, ACLNN invocation details
  - Output: Customize implementation files (Path 2) / validation evidence (Path 1)
- [ ] **[Step 5](workflows/05-kbk.md)**: KBK (Graph) -> backfill Feature [7. Execution Modes And Adaptation](templates/feature-document.md#feature-execution-modes)
  - **Path 1**: skip handwritten implementation and only validate the auto-registration
  - **Path 2**: handwrite kernel files (`GetWorkSpaceInfo` + `Launch` + registration)
  - Input: YAML, ACLNN invocation details
  - Output: kernel implementation files
- [ ] **[Step 6](workflows/06-bprop.md)**: BPROP registration -> backfill Feature [11. Backward (BPROP)](templates/feature-document.md#feature-bprop)
  - Input: `derivatives.yaml` analysis, backward kernel
  - Output: bprop implementation
- [ ] **[Step 7](workflows/07-export.md)**: export and placeholder behavior
  - Input: operator implementation
  - Output: exports under the `ops` package, interface files, and non-Ascend placeholder behavior; if interface overloads are involved, see [`reference.md` 25 Interface Overload Adaptation](reference.md#api-overload-adaptation)
- [ ] **[Step 8](workflows/08-testing.md)**: testing -> backfill Feature [12. Test Plan](templates/feature-document.md#feature-test-plan)
  - Input: all implementations, PTA reference
  - Output: C++ UT (must be newly created)
    See Step 2 in `workflows/08-testing.md`.
- [ ] **[Step 9](workflows/09-docs.md)**: documentation
  - Input: interface implementation
  - Output: English `function_doc` (created in Step 1 and refined here) + **Chinese RST (required for public APIs)**
  - **Important**: English doc YAML does not mean the documentation step is complete. Chinese RST is a separate deliverable and is the most common omission.
  - **Important**: public `mint`/`ops`/`nn`/`Tensor` interfaces must not skip this step. Only internal operators may skip it; see the conditional skip table.
- [ ] **Feature document finalization** `🔒 must not be skipped`: complete [13. Code And File Change Summary](templates/feature-document.md#feature-code-change-summary), [14. Acceptance Report](templates/feature-document.md#feature-acceptance-report), and update [3. Task List](templates/feature-document.md#feature-task-list)
  - Even if Step 9/10 is skipped or deferred, the Feature document must still be completed and delivered to the user once coding is done
- [ ] **[Step 10]**: finalize the Feature document

## Validation Loop (Evidence Required At Every Step) `🔒 must not be skipped`

After every completed step, you **must** present an execution report to the user using the template below. It may not be omitted, merged away, or deferred.
**This is a mandatory user-facing deliverable, not an internal note.**

```text
━━━ Step X Execution Report ━━━

Execution basis (which skill requirement I followed):
- workflow file: workflows/XX-xxx.md
- corresponding skill requirement: (quote the relevant item from SKILL.md / the workflow)
- success criteria for this step: (copied from the workflow success criteria)

What I did (deliverables):
- ...

Key evidence (code snippets / file paths / search results):
- ...
- Which existing operator implementation I compared against: ...

Validation result:
- ...

Item-by-item success criteria check:
- [ ] Criterion 1: ✅/❌
- [ ] Criterion 2: ✅/❌
- ...

Open issues / risks / next step:
- ...
```

## Key Constraints (Must Be Followed)

**Trust the repository's real code, not the workflow docs blindly.**
This skill's flow, templates, and naming conventions may become outdated as MindSpore evolves.
When the documentation disagrees with the repository state, **the repository state wins**.

## Additional Materials (Read As Needed)

- **Knowledge reference and code skeletons**: `../_shared/reference.md`
- **Trigger examples**: `examples.md`
- **PTA probing script template**: `scripts/probe_pta_sparse_flash_attention.py`
