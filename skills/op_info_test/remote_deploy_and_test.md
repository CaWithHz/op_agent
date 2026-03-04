# 远端部署与测试实现（单任务穿刺）

## 当前范围

当前只支持单客户端、单任务，不引入 queue。目标是先跑通闭环：

1. 拉取分支并部署
2. 执行新增 `op_info` 用例
3. 回传结构化结果
4. 基于结果迭代修正并重跑

## 已实现文件

1. `.agents/skills/op_info_test/scripts/remote_runner_server.py`
2. `.agents/skills/op_info_test/scripts/remote_runner_client.py`
3. 使用说明：`.agents/skills/op_info_test/remote_deploy_and_test_usage.md`

## 架构

```text
Client -> API Server -> SingleTaskRunner -> Artifact Store
                    -> /tmp/op_info_state.json
```

实现要点：

1. 单任务互斥：`/tmp/op_info_runner.lock`（`flock`）
2. 状态持久化：`/tmp/op_info_state.json`
3. 工件目录：`/tmp/op_info_artifacts/<job_id>/`
4. 保留扩展字段：`client_id`、`priority`、`resource_need`

## 服务端启动

```bash
python .agents/skills/op_info_test/scripts/remote_runner_server.py \
  --host 0.0.0.0 \
  --port 18080 \
  --state-file /tmp/op_info_state.json \
  --lock-file /tmp/op_info_runner.lock \
  --artifact-root /tmp/op_info_artifacts \
  --workspace-root /tmp/op_info_workspace
```

## API 说明

### `POST /jobs`

请求字段（最小集）：

1. `repo`
2. `branch`
3. `test_cmd`
4. `timeout_sec`

可选字段：

1. `commit`
2. `client_id`
3. `priority`
4. `resource_need`

若已有任务在跑，返回 `409`：

```json
{
  "code": 409,
  "message": "runner busy",
  "running_job_id": "job_xxx",
  "api_version": "v1"
}
```

### `GET /jobs/{job_id}`

返回任务状态与摘要。

### `GET /jobs/current`

查询当前运行任务。

### `POST /jobs/{job_id}/cancel`

取消当前任务。

## 执行流程（服务端）

### 1. 部署代码

服务端在每个任务目录执行：

```bash
git clone <repo> <workspace>/src
cd <workspace>/src
git fetch origin --prune
git checkout -B <branch> origin/<branch>
# 若未指定 commit:
git reset --hard origin/<branch>
# 若指定 commit:
git checkout <commit>
```

### 2. 执行测试

测试命令执行规则：

1. `cwd` 固定为 MindSpore 仓库根目录（`<workspace>/src`）
2. 自动将仓库根目录前置到 `PYTHONPATH`

等价于：

```bash
cd <workspace>/src
export PYTHONPATH="<workspace>/src:$PYTHONPATH"
<test_cmd>
```

建议提交任务时使用：

```bash
pytest tests/st/ops/op_info_tests/*.py \
  -q --maxfail=1 --tb=short
```

说明：

1. `pytest.log` 由 runner 自动采集
2. 若 `test_cmd` 是 `pytest` 且未显式设置 `--junitxml`，runner 会自动追加 `--junitxml=<artifact_dir>/junit.xml`
3. 若你已手动设置 `--junitxml`，runner 不会覆盖

### 3. 生成工件

每个任务输出：

1. `pytest.log`
2. `junit.xml`（若命令中生成）
3. `env.txt`
4. `deploy_meta.json`
5. `summary.json`

`summary.json` 关键字段：

1. `job_id`
2. `status`（`success`/`failed`/`timeout`/`canceled`）
3. `failed_cases`
4. `error_type`（`infra`/`testcase`）
5. `top_traceback`

## 客户端使用

### 提交任务

```bash
python .agents/skills/op_info_test/scripts/remote_runner_client.py \
  --server http://127.0.0.1:18080 \
  submit \
  --repo /home/panzh/mindspore \
  --branch master \
  --test-cmd "pytest tests/st/ops/op_info_tests/*.py -q --maxfail=1 --tb=short" \
  --timeout-sec 3600
```

### 查询状态

```bash
python .agents/skills/op_info_test/scripts/remote_runner_client.py \
  --server http://127.0.0.1:18080 \
  status --job-id <job_id>
```

### 阻塞等待

```bash
python .agents/skills/op_info_test/scripts/remote_runner_client.py \
  --server http://127.0.0.1:18080 \
  wait --job-id <job_id> --poll-interval-sec 10 --wait-timeout-sec 7200
```

### 取消任务

```bash
python .agents/skills/op_info_test/scripts/remote_runner_client.py \
  --server http://127.0.0.1:18080 \
  cancel --job-id <job_id>
```

## error_type 约定

标记为 `infra` 的典型场景：

1. `git clone/fetch/checkout` 失败
2. 运行环境缺失（依赖、设备、权限、命令不存在）
3. runner 超时、被取消、进程异常退出

标记为 `testcase` 的典型场景：

1. pytest 执行完成且存在断言失败
2. junit/日志可定位到具体失败用例

## 与 op_info_test 主流程衔接

建议固定循环：

1. 本地修改并提交（`op_info_test: add/refine <op>`）
2. 推送分支
3. 远端提交任务并等待
4. 读取 `summary.json`
5. 若 `error_type=testcase` 则修正后进入下一轮
6. 若 `error_type=infra` 连续出现则停止自动修正并转人工排障

## 后续扩展（保持兼容）

1. 保持 `POST /jobs` 与 `GET /jobs/{job_id}` 不变
2. 后续可在内部替换为多 worker/多客户端调度
3. 预留字段直接启用，无需改接口
