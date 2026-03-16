# 算子st用例生成

<a id="op-info-generation-goal"></a>
## 目标

新增算子Python ST，尽可能一次做到功能、精度、动态 shape 全覆盖。

**重要！！**：库上可能已有相似算子用例，但覆盖场景不全。参考类似算子用例实现时不应该以其覆盖场景作为目标覆盖场景。需要严格遵循该文档中各场景对生成用例的覆盖范围要求。

<a id="op-info-generation-inputs"></a>
## 输入

- **接口名**: 使用接口名收集相关信息，如算子接口定义yaml, 算子接口文档
- **torch对标接口**：测试标杆

<a id="op-info-generation-outputs"></a>
## 输出

> **⚠️ 以下是必须产出**

| 类型 | 文件位置 |
| --- | --- | --- | --- |
| **Python ST** | `tests/st/ops/share/_op_info/op_database.py`（OpInfo 注册） |

---

## 测试基本原则

测试接口行为应该完全和测试标杆接口对标
- 测试接口支持的输入和行为，该接口应该同样支持
- 测试接口不支持的输入和行为，该接口不需要支持

<a id="op-info-generation-steps"></a>
## 执行步骤

> 当前 ST 使用**op info测试框架**，核心操作是在 `op_database.py` 中注册 OpInfo，禁止另外手写独立测试文件。框架原理详见 [`../_shared/reference.md` 8.2 ST op-info测试框架](../../_shared/reference.md#testing-st-opinfo)。

**两种场景**：

| 适用场景 | 操作 |
| --- | --- |
| Unary/Binary/Reduction 等常规算子 | 在 `op_database.py` 添加 OpInfo → 加入对应 `xxx_op_db` → 自动纳入前端参数化用例 |
| 需要自定义测试逻辑的算子 | 继承 OpsFactory 写自定义测试套 + 新建前端测试文件 |

<a id="op-info-generation-common-ops"></a>
### Unary/Binary/Reduction 等常规算子 

Unary/Binary/Reduction 类算子在 `op_info.py` 中已提供丰富的通用输入生成函数（各种 shape 组合、
广播、非连续、特殊值、极端值等），注册 OpInfo 后自动覆盖。

1. **确定算子OpInfo类别**：Unary → `UnaryOpInfo` / Binary → `BinaryOpInfo` / Reduction → `ReductionOpInfo` / 其他 → `OpInfo`
2. **在 `op_database.py` 添加 OpInfo 实例**：配置 `name`、`op`、`ref`、`dtypes_support`（以及 `dtypes_grad`、`dtypes_dynamic` 等）
3. **将算子名加入对应 `xxx_op_db` 列表**（如 `binary_op_db`、`unary_op_db`）
4. **如需自定义输入场景**：编写 `op_basic_reference_inputs_func` / `op_extra_reference_inputs_func`，返回 `OpSampleInput` 列表
5. **判断是否需要加入 `xxx_op_kbk_db` 列表**（见下方约束）
6. **验证覆盖**：确认前端测试文件（如 `test_binary_ops.py`）的参数化用例已包含新算子

> **关于 KBK 列表（`xxx_op_kbk_db`）的添加约束**：
>
> KBK 场景耗时较长，不需要每个算子都加入。仅在以下情况下将算子加入对应的 `xxx_op_kbk_db`（如 `binary_op_kbk_db`、`unary_op_kbk_db`、`reduction_op_kbk_db` 等），使前端测试文件跑 KBK 前向/反向/动态 shape 用例：
>
> - 算子包含**较复杂的动态 shape 推导逻辑**（如输出 shape 依赖输入值、多分支推导），可通过确认算子infer代码确认。
> - 算子采用**组合实现**（PyBoost/aclnn kernelmod中串联多个算子调用）
> - 算子包含**前端接口重载**（mindspore/ops/api_def中存在接口yaml定义）
>
>
> **不需要添加的情况**：
> - 简单直通算子（单 ACLNN、无参数预处理）
> - KBK 列表中**已有同类型/同实现模式的算子**——例如 `unary_op_kbk_db` 已有 `mint.tanh`，则 `mint.cosh` 等同类三角函数无需重复添加

### 需要自定义测试逻辑的算子

**other 类算子**（加入 `other_op_db`）需要在 `op_database.py` 中**自行编写** 输入生成函数传给OpInfo的`op_basic_reference_inputs_func`和`op_extra_reference_inputs_func`输入，

#### 用例覆盖场景
- [ ] `[MUST]` **默认参数场景验证**：使用所有默认参数值调用前向+反向，确认基本路径可通。
- [ ] `[MUST]` **动态 shape 自验**：前端测试用例文件调用 `OpsFactory` 的 `test_op_dynamic` 方法。
- [ ] `[MUST]` **空张量输入**：验证空张量的前向/反向是否支持或正确报错。
- [ ] `[MUST]` **输入 dtype 全覆盖**：算子声明支持的所有 dtype 都有对应用例（含不支持类型的异常用例）。
- [ ] `[MUST]` **输入维度覆盖**：合法维度（输入维度覆盖 0D/8D和一个中间大小的维度（如果支持））和非法维度都有用例。
- [ ] `[MUST]` **输入取值范围验证**：边界值、极端值（极大/极小）、margin/reduction 等枚举参数全覆盖。
- [ ] `[MUST]` **输入间约束验证**：形状匹配/不匹配、dtype 一致/不一致、rank 一致/不一致。
- [ ] `[MUST]` **异常用例校验具体报错信息**：异常场景需断言 TypeError/ValueError/RuntimeError 的具体 message。
- [ ] `[MUST]` **多布局覆盖**：若算子支持多种 layout（如 BSND/TND/PA_BSND），覆盖所有布局组合的前后向。
- [ ] `[MUST]` **非连续张量**：通过 transpose/permute 构造非连续输入，验证正确性。
- [ ] `[MUST]` **特殊值健壮性**：inf/-inf/nan 场景验证（至少不 crash，形状/流程正确）。
- [ ] `[SHOULD]` **多 batch 变长序列**：若涉及 actual_seq_len 类参数，覆盖多 batch + 变长场景。
- [ ] `[MUST]` **bf16 场景**：bf16 支持情况确认（支持则测精度，不支持则有异常用例）；比较前升精到 float32。
- [ ] `[MUST]` **隐式类型转换**：确认是否支持输入间 dtype 不同时的自动提升；不支持则有异常用例。
- [ ] `[MUST]` **广播**：确认是否支持输入间 shape 广播；不支持则有异常用例。
- [ ] `[MUST]` **多 Tensor 输入 dtype 不一致场景**：确认是否支持多 Tensor 输入 dtype 不一致场景；不支持则有异常用例。非多Tensor输入的算子不需要。

| 必覆盖场景 | 编写方式 | 示例 |
| --- | --- | --- |
| **多种 shape**（含 0D scalar、1D、2D-3D 中间维、高维） | 多个 yield，不同 shape | `make_arg(())`, `make_arg((S,))`, `make_arg((S,M,S))` |
| **空 tensor**（某维为 0） | shape 中含 0 | `make_arg((0, S))`, `make_arg((S, 0, M))` |
| **非连续 tensor** | `discontiguous=True` 参数 | `make_tensor(shape, discontiguous=True)` |
| **边界参数值** | 覆盖参数的极端/边界 | `dim=0`, `dim=-1`, `dim=最后一维`; `p=1`, `p=2`, `p=inf` |
| **大 tensor** | 至少一个较大 shape | `make_arg((LARGE_DIM_SIZE, M))` |

编写参考：`op_info.py` 中 `basic_reference_inputs_binary_op_common_func` 和
`_generate_binary_op_broadcasting_and_discontiguous_tensor_inputs_func` 的写法模式。

如果算子支持 `op_extra_reference_inputs_func`（额外精度场景）或 `op_dynamic_inputs_func`
（动态 shape/rank），也应参照 `op_info.py` 中的同类写法编写。

#### 测试组合与稳定性
- [ ] `[MUST]` **测试组合覆盖**：接口形态（functional/nn/Tensor）× 后端 × 模式（Pynative/KBK）× shape 类型（静态/动态）。
- [ ] `[MUST]` **关闭退避验证**：`export MS_DISABLE_KERNEL_BACKOFF=1` 环境下用例全部通过（防止退回非 ACLNN 路径）。
<!-- - [ ] `[SHOULD]` **测试仓存量用例回归**：若存在测试仓 ST，确认全部 PASS。 -->

<!-- #### 功能合规性确认
- [ ] `[MUST]` **不影响存量接口**：新增算子/原语不会使运算符或存量 ops 接口调用到新增原语（除非设计如此）。
- [ ] `[SHOULD]` **AMP 混合精度**：确认是否已支持或不涉及（新增 Primitive 需关注 amp_white/black_list）。
- [ ] `[SHOULD]` **多 Tensor 输入 dtype 不一致**：若多输入算子，确认是否支持各输入 dtype 不同。
- [ ] `[SHOULD]` **输出 shape 是否依赖计算结果**：若是 compute-depend 输出，需要 SyncOutputShape 机制。 -->


<!-- <a id="op-info-generation-bitwise-validation"></a>
### 精度零偏差验证（[`../_shared/reference.md` 14.1 精度零偏差](../../_shared/reference.md#bitwise-accuracy-validation)，按需）

- 固定随机种子，保存输出为 `.npy`
- `md5sum` 对比 MS/PTA 输出哈希

<a id="op-info-generation-memory-validation"></a>
### 显存对齐验证（[`../_shared/reference.md` 14.2 显存占用对齐](../../_shared/reference.md#memory-alignment-validation)，按需）

- MS：`mindspore.runtime.max_memory_allocated()`
- PTA：`torch_npu.npu.max_memory_allocated()`
- 在相同阶段统计 -->

---

<a id="op-info-generation-gate"></a>
## 🔒 Step 8 完成前强制检查（不可跳过）

**在标记 Step 8 为完成之前，必须逐项确认以下清单：**

```text
测试产出检查清单：

Python ST（OpInfo 注册）：
  - 注册文件：tests/st/ops/share/_op_info/op_database.py
  - OpInfo 已注册？ ✅是（算子名：___）/ ❌否（原因：___）
  - 已加入对应 xxx_op_db 列表？ ✅是 / ❌否
  - 前端参数化用例已覆盖？ ✅是（测试文件：___）/ ❌否
  - 若需自定义输入：inputs_func 已编写？ ✅是 / ⏭不需要
  - 🚫 是否新建了独立测试脚本？ 必须为否（如误建需删除并迁移到 OpInfo）
```

> 如果 Python ST 的状态为 ❌，**必须说明原因并暂停等用户确认后再继续**。
> 不允许静默跳过。

<a id="op-info-generation-success-criteria"></a>
## 成功标准

- [ ] **Python ST OpInfo 已注册且纳入前端参数化用例**（自动覆盖多模式 + 前向精度 + 动态 shape）
- [ ] 覆盖场景：动态 shape / 静态 shape / 非连续 tensor / 空 tensor / 特殊值
