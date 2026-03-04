---
name: op_info_test
description: add python st for op using the op_info test factory.
---

该 skill 覆盖 op_info 测试从生成到远端验证的端到端闭环。除非涉及权限问题，否则在整个流程执行完成之前不要进行提问。

## 执行顺序

1. 测试用例生成：`workflow/op_info_generation.md`
2. 生成提交：commit message 规范为 `op_info_test: add xxx`（`xxx` 为新增接口）
3. 仅保留新增接口测试（临时 patch）：`workflow/patch_out_old_tests.md`
4. git push
5. 远端部署与测试：`workflow/remote_deploy_and_test.md`
6. 移除临时 patch commit，合并测试用例 commit; git push

## 闭环完成标准

1. 远端任务 `status=success`
2. `summary.json` 中 `failed_cases` 为空
3. 若 `error_type=testcase`，继续修正并重跑，直到成功
4. 若 `error_type=infra`，停止自动改用例，转环境排障
