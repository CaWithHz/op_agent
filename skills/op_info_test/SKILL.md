---
name: op_info_test
description: add python st for op using the op_info test factory.
---

该 skill 覆盖 op_info 测试从生成到远端验证的端到端闭环。

## 执行顺序

1. 用例生成：`op_info_generation.md`
2. 生成提交：commit message 规范为 `op_info_test: add xxx`（`xxx` 为新增接口）
3. 仅保留新增接口测试（临时 patch）：`patch_out_old_tests.md`
4. 远端部署与测试：`remote_deploy_and_test.md`
5. 实际操作指令：`remote_deploy_and_test_usage.md`
6. 移除临时patch commit

## remote deploy && test 入口

1. 服务端脚本：`.agents/skills/op_info_test/scripts/remote_runner_server.py`
2. 客户端脚本：`.agents/skills/op_info_test/scripts/remote_runner_client.py`

执行测试时，runner 固定行为：

1. 在 MindSpore 仓库根目录执行 pytest（`cwd=<workspace>/<job_id>/src`）
2. 自动前置 `PYTHONPATH=<repo_root>:$PYTHONPATH`
3. 若 `test_cmd` 为 pytest 且未设置 `--junitxml`，自动追加 junit 输出路径

## 闭环完成标准

1. 远端任务 `status=success`
2. `summary.json` 中 `failed_cases` 为空
3. 若 `error_type=testcase`，继续修正并重跑，直到成功
4. 若 `error_type=infra`，停止自动改用例，转环境排障
