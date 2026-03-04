# remote_deploy_and_test 使用说明

## 1. 目标

本文档用于快速跑通单客户端、单任务的远端部署与测试流程。

## 2. 前置条件

1. 远端机器可执行 `python`、`git`、`pytest`
2. 可访问目标仓库（`repo`）
3. 当前实现脚本已存在：`.agents/skills/op_info_test/scripts/remote_runner_server.py`
4. 当前实现脚本已存在：`.agents/skills/op_info_test/scripts/remote_runner_client.py`

## 3. 启动服务端

在远端机器执行：

```bash
cd /home/panzh/mindspore
python .agents/skills/op_info_test/scripts/remote_runner_server.py \
  --host 0.0.0.0 \
  --port 18080 \
  --state-file /tmp/op_info_state.json \
  --lock-file /tmp/op_info_runner.lock \
  --artifact-root /tmp/op_info_artifacts \
  --workspace-root /tmp/op_info_workspace
```

说明：

1. 同时仅允许一个任务运行
2. 若有任务在跑，新任务会返回 `409 runner busy`

## 4. 提交任务

在客户端机器执行：

```bash
cd /home/panzh/mindspore
python .agents/skills/op_info_test/scripts/remote_runner_client.py \
  --server http://<server_ip>:18080 \
  submit \
  --repo /home/panzh/mindspore \
  --branch master \
  --test-cmd "pytest tests/st/ops/op_info_tests/*.py -q --maxfail=1 --tb=short" \
  --timeout-sec 3600
```

返回示例：

```json
{
  "api_version": "v1",
  "job_id": "job_20260303_200000_abcd1234",
  "status": "running"
}
```

## 5. 查询与等待

查询状态：

```bash
python .agents/skills/op_info_test/scripts/remote_runner_client.py \
  --server http://<server_ip>:18080 \
  status --job-id <job_id>
```

阻塞等待结束：

```bash
python .agents/skills/op_info_test/scripts/remote_runner_client.py \
  --server http://<server_ip>:18080 \
  wait --job-id <job_id> --poll-interval-sec 10 --wait-timeout-sec 7200
```

## 6. 查看结果

任务工件目录：

```bash
/tmp/op_info_artifacts/<job_id>/
```

关键文件：

1. `pytest.log`
2. `junit.xml`
3. `env.txt`
4. `deploy_meta.json`
5. `summary.json`

查看摘要：

```bash
cat /tmp/op_info_artifacts/<job_id>/summary.json
```

`summary.json` 重点字段：

1. `status`：`success` / `failed` / `timeout` / `canceled`
2. `error_type`：`infra` / `testcase`
3. `failed_cases`：失败用例列表
4. `top_traceback`：关键报错片段

## 7. 取消任务

```bash
python .agents/skills/op_info_test/scripts/remote_runner_client.py \
  --server http://<server_ip>:18080 \
  cancel --job-id <job_id>
```

## 8. 执行行为说明

1. 服务端会把代码检出到 `<workspace_root>/<job_id>/src`
2. 测试在仓库根目录执行（`cwd=<...>/src`）
3. 运行测试前自动设置 `PYTHONPATH=<...>/src:$PYTHONPATH`
4. 若 `test_cmd` 是 `pytest` 且未设置 `--junitxml`，服务端会自动追加

## 9. 常见问题

1. `409 runner busy`
   当前已有任务运行中，等待结束或取消后重试
2. `error_type=infra`
   通常是环境、依赖、权限、设备、git 拉取等问题
3. `error_type=testcase`
   说明测试逻辑或期望不满足，按失败用例修正后重跑

## 10. 端到端闭环示例（op_info_test）

1. 本地完成用例修改并提交

```bash
git add tests/st/ops/share/_op_info/op_database.py
git commit -s -m "op_info_test: add <op_name>"
```

2. 推送分支

```bash
git push origin <your_branch>
```

3. 提交远端任务并记录 `job_id`

```bash
python .agents/skills/op_info_test/scripts/remote_runner_client.py \
  --server http://<server_ip>:18080 \
  submit \
  --repo /home/panzh/mindspore \
  --branch <your_branch> \
  --test-cmd "pytest tests/st/ops/op_info_tests/*.py -q --maxfail=1 --tb=short" \
  --timeout-sec 3600
```

4. 等待任务结束并读取摘要

```bash
python .agents/skills/op_info_test/scripts/remote_runner_client.py \
  --server http://<server_ip>:18080 \
  wait --job-id <job_id> --poll-interval-sec 10 --wait-timeout-sec 7200

cat /tmp/op_info_artifacts/<job_id>/summary.json
```

5. 根据结果进入下一轮

- `status=success`：闭环结束
- `error_type=testcase`：按 `failed_cases/top_traceback` 修正用例，重新提交并重复步骤 1-4
- `error_type=infra`：先修复环境/部署问题，再重提任务
