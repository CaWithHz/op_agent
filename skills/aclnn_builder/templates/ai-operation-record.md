# Full AI Operation Record

> This document is produced by the AI that used the `mindspore-aclnn-operator-devflow` skill at the end of a task or stage. It is intended for traceability, review, and later troubleshooting. Readers should be able to understand how the AI completed the task without reopening the conversation history.

**Suggested filename**: `ai_operation_record_{task_summary}_{date}.md` (for example, `ai_operation_record_xxx_op_pre_and_step1_20250227.md`)

---

## 1. Task And Execution Basis

| Item | Content |
| --- | --- |
| **Original user request** | (Summarize the user's request in one sentence) |
| **Execution basis** | Which rule in this skill or which workflow step was followed (for example: pre-development information gathering + Workflow Pre + Step 1) |
| **Goal and deliverables for this stage** | (What this session aimed to complete and what it delivered) |
| **Produced at** | (Date or session end time) |

---

## 2. Todo List And Completion Status

(Corresponds to the items in TodoWrite; if no tool was used, list them manually here.)

| No. | Todo Item | Status | Notes |
| --- | --- | --- | --- |
| 1 | (Example) Pre: pre-checks | ✅ Done / ⏳ In progress / ❌ Not done | |
| 2 | (Example) Feature document initialization | ✅ / ⏳ / ❌ | |
| 3 | ... | | |

---

## 3. Key Actions At Each Step

(Record step by step, either by time or by workflow order: what was done, which files were read or edited, and what conclusions or validation results were obtained.)

### 3.1 (Step name, for example Pre-A inventory check)

- **Action**: searched for xxx, read xxx files
- **Files/directories involved**: `path/to/file`
- **Conclusion/evidence**: (Key excerpts or conclusions)
- **Validation method and result**: (if any)

### 3.2 (Next step, for example Pre-B solution design)

- **Action**: ...
- **Files/directories involved**: ...
- **Conclusion/evidence**: ...
- **Validation method and result**: ...

### (Follow the same pattern for later steps)

---

## 4. Deliverables

| Deliverable Type | Path Or Description |
| --- | --- |
| New/modified files | `mindspore/ops/op_def/yaml/xxx_op.yaml`, etc. |
| Documents | Feature document path, PTA review report path, and so on |
| Scripts | (For example) validation scripts or probing scripts |

---

## 5. Remaining Issues And Next Step

- **Remaining issues**: (Items that require user confirmation, are unfinished, or still need device-side validation)
- **Recommended next step**: (What the user or the next session should do)

---

*(All sections above should be filled by the AI based on the real execution at task end. Sections may be added or removed as needed.)*
