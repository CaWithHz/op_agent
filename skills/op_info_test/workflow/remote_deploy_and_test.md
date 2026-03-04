# remote_deploy_and_test 操作手册

## 1. 前置条件

本地已完成用例生成、修改并推送分支。

## 2. 标准操作流程

### 步骤 1：启动服务端（远端机器）

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

### 步骤 2：提交任务（客户端机器）

```bash
cd /home/panzh/mindspore
python .agents/skills/op_info_test/scripts/remote_runner_client.py \
  --server http://<server_ip>:18080 \
  submit \
  --repo /home/panzh/mindspore \
  --branch <your_branch> \
  --test-cmd "pytest tests/st/ops/op_info_tests/*.py -q --maxfail=1 --tb=short" \
  --timeout-sec 3600
```

记录返回的 `job_id`。

### 步骤 3：等待任务结束

```bash
python .agents/skills/op_info_test/scripts/remote_runner_client.py \
  --server http://<server_ip>:18080 \
  wait --job-id <job_id> --poll-interval-sec 10 --wait-timeout-sec 7200
```

如需手动查询状态：

```bash
python .agents/skills/op_info_test/scripts/remote_runner_client.py \
  --server http://<server_ip>:18080 \
  status --job-id <job_id>
```

### 步骤 4：读取测试摘要

```bash
cat /tmp/op_info_artifacts/<job_id>/summary.json
```

重点字段：

1. `status`
2. `error_type`
3. `failed_cases`
4. `top_traceback`

### 步骤 5：按结果进入下一轮

1. `status=success`：闭环结束。
2. `error_type=testcase`：按 `failed_cases/top_traceback` 修正用例，重新执行步骤 2 到步骤 5。
3. `error_type=infra`：停止自动改用例，先处理环境问题。

## 3. 可选操作

取消任务：

```bash
python .agents/skills/op_info_test/scripts/remote_runner_client.py \
  --server http://<server_ip>:18080 \
  cancel --job-id <job_id>
```
