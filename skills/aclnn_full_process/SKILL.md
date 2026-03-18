---
name: aclnn_full_process
description: invoke this skill only when it's mentioned directly.
---

执行aclnn接入端到端闭环，优先直接落地，不在中途停下提问；仅在权限受限、关键信息缺失且无法合理推断时提问。

不允许跳过任何步骤。

1. 在mindspore根目录执行`python ./mindspore/python/mindspore/ops_generate/gen_ops.py`，确保自动生成文件状态和代码仓状态一致。
2. 调用skill[`mindspore-aclnn-operator-devflow`]，执行接口接入任务。
3. commit修改，message `aclnn task: {op_name}`
4. 调用skill[`op_info_test`]，执行测试用例生成和测试任务。

执行完毕后检查，确保无遗漏任务。
