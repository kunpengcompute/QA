# Pytest 自动化用例开发规范（v4.0）

> 适用范围：蓝区框架（`BaseTestCase`）下基于 pytest 的自动化测试用例开发
> 依据：《蓝区框架自动化框架切换》9.3.1 用例级别标记、10.3 脚本适配（原文以 CIDA 用例名表述，本规范统一表述为「用例编号」）
> 结构：命名规范 → 用例编写规范 → 用例标签与级别规范 → 日志使用规范 → 标签执行方式
> 说明：本文件为 v4.0 版本，v1.0/v2.0 见同目录历史文件 `pytest自动化用例开发规范.md`

---

## 目录

1. [命名规范](#1-命名规范)
2. [用例编写规范](#2-用例编写规范)
3. [用例标签规范以及用例级别规范](#3-用例标签规范以及用例级别规范)
4. [日志使用规范](#4-日志使用规范)
5. [使用标签执行用例的方式方法](#5-使用标签执行用例的方式方法)

---

## 1. 命名规范

### 1.1 核心约束：一个文件 = 一个用例 = 一个 def

蓝区框架下**每个 `.py` 文件代表一个用例，文件中只包含一个 `def` 测试方法**，该方法名称必须与用例编号严格对应。

```python
# 文件：tests/kunpeng_storage/.../reedsolomon_performance_001_test.py

class TestReedsolomonPerformance001(BaseTestCase):

    def test_reedsolomon_performance_001(self):
        ...
```

### 1.2 命名对照表

| 对象 | 规则 | 示例 |
|------|------|------|
| **文件名** | `<用例编号>_test.py`，**必须带 `_test` 后缀** | `reedsolomon_performance_001_test.py` |
| **类名** | `Test` + 用例编号驼峰化，**必须 `Test` 前缀** | `TestReedsolomonPerformance001` |
| **def 方法名** | `test_<用例编号>`，**与用例编号逐字对应** | `test_reedsolomon_performance_001` |
| **基类** | 继承 `BaseTestCase` | `class TestXxx(BaseTestCase):` |

> 文件名、类名、方法名三者**必须由同一个用例编号派生**，保证一一对应、可反向检索。

### 1.3 def 名称规则

✅ **正确**：

```python
def test_reedsolomon_performance_001(self):     # 与用例编号一致
def test_hyperscan_lily_function_0040(self):    # 与用例编号一致
```

❌ **禁止**：

```python
def test_step1(self):                 # 不是用例编号
def test_reedsolomon_001(self):       # 与用例编号不一致（缺 performance）
def testReedsolomonPerformance001(self):  # 缺少 test_ 前缀与下划线
def test_a(self):                     # 无业务含义
```

规则要点：

1. 必须以 `test_` 开头（pytest 收集要求）。
2. `test_` 之后的部分**必须与用例编号完全相同**，不得改写、缩写、调序。
3. 名称**全小写 + 下划线**，不要用驼峰。
4. 一个 def 内**不允许再拆多个 def**，多个测试步骤合并到同一个 def 中顺序执行。

### 1.4 一个文件一个用例的目录要求

```text
tests/kunpeng_storage/kps_features/AlgorithmLibrary/EC/go_reedsolomon/performance/
├── reedsolomon_performance_001_test.py     # 用例 1
├── reedsolomon_performance_002_test.py     # 用例 2
└── reedsolomon_performance_003_test.py     # 用例 3
```

同一业务场景的多个用例**平铺为多个文件**，不要放进同一个文件里的多个 def。

## 2. 用例编写规范

### 2.1 文件骨架（标准模板）

每个文件套用以下骨架，**只允许一个 def**：

```python
import pytest

from lib.Base.BaseTestCase import BaseTestCase

class TestReedsolomonPerformance001(BaseTestCase):
    """Reed-Solomon 性能用例：reedsolomon_performance_001"""

    @pytest.mark.case_info(level="P0", type="Performance")
    def test_reedsolomon_performance_001(self):
        """用一句话说明本用例的验证目标与预期结果。"""
        # ---------- 步骤 1：准备 ----------
        self.info("步骤1：准备测试数据")
        block_size = 16 * 1024 * 1024

        # ---------- 步骤 2：执行 ----------
        self.info("步骤2：执行编码")
        result = self.rs_encode(block_size)

        # ---------- 步骤 3：断言 ----------
        self.info("步骤3：校验结果")
        assert result.throughput > 0, f"吞吐量异常: {result.throughput}"
```

### 2.2 核心编写约束

| 约束 | 说明 |
|------|------|
| **一个文件一个用例** | 每个 `.py` 只含一个 def，见第 1 章 |
| **def 名 = 用例编号** | `def test_<用例编号>(self)` |
| **必须打 `case_info` 标签** | 每条用例必带 level 与 type，见第 3 章 |
| **多步骤合并** | 原 pistar 多个测试步合并到同一 def 内顺序执行，用注释/日志分段 |
| **继承 `BaseTestCase`** | `from lib.Base.BaseTestCase import BaseTestCase` |
| **必须带 `self` 参数** | 方法签名 `def test_xxx(self)` |
| **单条断言意图** | 多个相关断言可共存，禁止验证多个无关功能 |
| **禁止 `print`** | 使用 `self.info(...)` 或 `logging.info(...)` |
| **禁止硬编码** | URL、工号、账号、路径通过配置/fixture 注入 |
| **无副作用** | 通过 teardown 清理自身产生的数据 |

### 2.3 多步骤合并写法

pistar 的多测试步写法（**已废弃**）：

```python
# ❌ 不再支持：一个文件多个 teststep
@teststep()
def step1(self): ...
@teststep()
def step2(self): ...
```

改为单 def 顺序步骤（**推荐**）：

```python
def test_reedsolomon_performance_001(self):
    # ---------- 步骤1：初始化环境 ----------
    self.info("步骤1：初始化环境")
    self.init_env()

    # ---------- 步骤2：执行被测操作 ----------
    self.info("步骤2：执行编码")
    result = self.rs_encode(16 * 1024 * 1024)

    # ---------- 步骤3：校验结果 ----------
    self.info("步骤3：校验吞吐")
    assert result.throughput > 0
```

> **说明**：pytest 不支持用例内部测试步骤并发。若某用例原本依赖 `@teststep(start=xxx.start)` 定义步骤顺序或并发，合并后必须改为顺序执行，或用 `ThreadPoolExecutor` 显式实现并发。

### 2.4 teardown / failure 语义

| 重写情况 | 正常结束 | 异常结束 |
|---------|---------|---------|
| 只重写 `teardown` | 执行 `teardown` | 执行 `teardown` |
| 只重写 `failure` | 不执行 | 执行 `failure` |
| 两者均重写 | 执行 `teardown` | 执行 `failure` |

> 该能力已由公共基类适配，测试人员照旧书写即可。

### 2.5 condition.py 会话级前后置

原用 pistar `condition` 特性的用例，按优先级处理：

1. **只用** condition 中的文件夹级前置步/后置步；
2. condition 中初始化的变量需后续共享/处理时，单独改造；
3. 以上都不满足，**不用 condition**。

## 3. 用例标签规范以及用例级别规范

对应《蓝区框架自动化框架切换》**9.3.1 用例级别标记**。

### 3.1 标签定义

```python
@pytest.mark.case_info(level="P0", type="Functional")
```

| 参数 | 取值（**枚举，禁止自造**） | 含义 |
|------|--------------------------|------|
| `level` | `P0` / `P1` / `P2` / `P3` | 用例级别（`P0`~`P3` 依次对应 Level 0 ~ Level 3） |
| `type` | `Functional` / `Performance` / `Security` / `Compatibility` / `Reliability` | 用例类型 |

> 级别越小，重要程度越高；级别越大，重要程度越低。

### 3.2 用例级别（level）规范

| 级别 | Level | 名称 | 定义 | 版本对应 |
|------|-------|------|------|---------|
| `P0` | Level 0 | 门槛 | 标识**转测试的门槛用例**，验证版本是否可以转测试，用于开发的 HLT、版本集成转测试 | 转测试门槛 |
| `P1` | Level 1 | 基本 | 标识**基本功能用例**，每个 B 版本必须验证，保障系统、特性的基本功能没有问题 | B 版本 |
| `P2` | Level 2 | 重要 | **非回归的系统测试版本中基本上都需要验证**，以保证系统所有的重要功能都能正常实现 | C 版本（必须验证） |
| `P3` | Level 3 | 一般 | 使用频率较 Level 2 低，**每个发布 C 版本根据测试策略进行针对性验证** | C 版本（针对性验证） |

#### 各级别说明

- **Level 0（门槛，`P0`）**：标识转测试的门槛用例，验证版本是否可以转测试，用于开发的 HLT、版本集成转测试。用例不通过即代表版本不具备转测试条件。
- **Level 1（基本，`P1`）**：标识基本功能用例，对应 B 版本，每个 B 版本必须验证，保障系统、特性的基本功能没有问题。
- **Level 2（重要，`P2`）**：2 级测试用例在非回归的系统测试版本中基本上都需要进行验证，以保证系统所有的重要功能都能够正常实现，对应 C 版本，每个发布 C 版本必须验证，保障系统、特性的主业务场景、重要功能没有问题。
- **Level 3（一般，`P3`）**：3 级测试用例使用频率较 2 级测试用例低，对应 C 版本，每个发布 C 版本根据测试策略进行针对性验证。

> **扩展级别（预留，`case_info` 暂不支持）**：
>
> - **Level 4（生僻）**：该类用例对应较生僻的预置条件和数据设置，此类用例一般为一次性用例，新增版本测试，后续版本继承用例筛选时不会再进行测试。
> - **Level T**：标识用于测试自动化工具的用例。
>
> 当前 `case_info` 的 `level` 取值为 `P0`~`P3`（对应 Level 0~Level 3）。如需使用 Level 4 / Level T，需先在 `pytest.ini` 与 `conftest.py` 的 `ALLOWED_LEVELS` 中扩展取值，经团队评审后再启用。

#### 定级原则

1. **按版本阶段定级**：能否决定"版本可转测试" → `P0`；B 版本必须验证的基本功能 → `P1`；C 版本必须验证的重要功能 → `P2`；C 版本按策略针对性验证 → `P3`。
2. **就高不就低**：介于两级之间时，取更高级别（即更小的 Level 编号）。
3. **就低不就高（执行成本）**：仅有生僻预置条件、后续版本不再复测的一次性用例，不应占用 `P0`~`P2`，应归入 `P3` 或申请扩展 Level 4。
4. **`P0` 必须是门槛集**：`P0` 用例集合用于版本转测试判定，应精简、稳定、可快速执行。
5. **`P3` 不得占比过高**：单个测试套中 `P3` 占比建议 ≤ 20%。

#### 定级决策流程

```text
该用例是否决定"版本能否转测试"（开发的 HLT / 版本集成转测试门槛）?
├── 是 → P0（Level 0 门槛）
└── 否 → 是否属于基本功能，每个 B 版本都必须验证?
        ├── 是 → P1（Level 1 基本）
        └── 否 → 是否重要功能，每个发布 C 版本都必须验证?
                ├── 是 → P2（Level 2 重要）
                └── 否 → P3（Level 3 一般，按 C 版本测试策略针对性验证）
                        └── 若是生僻预置条件的一次性用例 → 申请扩展 Level 4
```

### 3.3 用例类型（type）规范

| 类型 | 定义 | 验证内容 | 示例 |
|------|------|---------|------|
| `Functional` | 功能正确性 | 返回值、返回码、数据结果、业务流程 | 编码结果比对、接口返回 200 |
| `Performance` | 性能指标 | 吞吐、时延、CPU/内存占用、并发能力 | 16MB 块编码吞吐 > 阈值 |
| `Security` | 安全性 | 权限越权、注入防护、敏感信息泄露 | 越界入参被拒绝、密钥不落盘 |
| `Compatibility` | 兼容性 | 多 OS/多版本/多硬件组合、升级回退 | 跨内核版本、升级后回退 |
| `Reliability` | 可靠性 | 长时间稳定运行、异常恢复、故障注入、断连重连、数据一致性、资源泄漏 | 72 小时长稳无异常退出、断网后自动重连恢复业务、进程崩溃后数据不丢失 |

#### 类型判定原则

- 一条用例**只选一个最主要的类型**；若同时涉及性能与功能，**优先定为 `Performance`**（因为性能断言通常更严格、耗时更长）。
- 同一用例编号不允许多个类型标签；确需多维度验证，拆为多条用例（不同用例编号）。

#### 可靠性（Reliability）与其他类型的边界

| 对比 | 区别要点 |
|------|---------|
| 与 `Functional` | `Functional` 验证"一次正确"；`Reliability` 验证"持续/异常后仍正确"（长时间、重复、故障恢复） |
| 与 `Performance` | `Performance` 关注指标是否达标（吞吐、时延）；`Reliability` 关注是否稳定不出错（不崩溃、不泄漏、不丢数据） |
| 与 `Security` | `Security` 关注恶意攻击与越权；`Reliability` 关注非恶意的故障与异常场景 |
| 与 `Compatibility` | `Compatibility` 关注跨环境差异；`Reliability` 关注单一环境下的长期稳定性与容错 |

> 判定参考：用例若包含**长稳运行、故障注入/异常恢复、断连重连、资源泄漏检测、数据一致性校验**之一，优先定为 `Reliability`。

### 3.4 标签书写规范

```python
class TestReedsolomonPerformance001(BaseTestCase):

    @pytest.mark.case_info(level="P0", type="Performance")
    def test_reedsolomon_performance_001(self):
        ...
```

规则：

1. **`level` 与 `type` 两个参数都必须写**，不接受省略。
2. **严格区分大小写**：`P0`~`P3` 大写（对应 Level 0~Level 3），`type` 首字母大写；不要写成 `p0`、`level0`、`L0`。
3. 一条用例**只能有一个** `case_info` 标签。
4. 不允许空字符串或 `None`。
5. 标签加在 **def 方法上**（不推荐加类上，因一个文件只有一个 def，两者等价，但方法级更直观）。

### 3.5 合规与禁止

✅ **推荐**：

```python
@pytest.mark.case_info(level="P0", type="Functional")
```

❌ **禁止**：

```python
@pytest.mark.case_info(level="p0", type="functional")            # 大小写不规范
@pytest.mark.case_info(level="L0")                                # 写成 Level/L 形式，非枚举取值
@pytest.mark.case_info(level="Level 0")                           # 含空格，非枚举取值
@pytest.mark.case_info(level="P4")                                # 枚举外取值（Level 4 需先扩展）
@pytest.mark.case_info(type="功能")                                # 中文取值
@pytest.mark.case_info(level="P0", type="Functional", owner="张三")  # 未定义参数
def test_x(self): ...                                             # 不打卡
```

### 3.6 与执行策略标记的关系

`case_info` 表示**用例属性**，可与 `smoke`/`regression`/`slow` 等**执行策略标记**叠加：

```python
@pytest.mark.case_info(level="P0", type="Functional")
@pytest.mark.smoke
def test_reedsolomon_performance_001(self):
    ...
```

- `case_info` = 是什么、多重要
- `smoke/regression/slow` = 这次跑不跑

---

## 4. 日志使用规范

### 4.1 日志写法对照

| 位置 | 正确写法 | 禁止写法 |
|------|---------|---------|
| **测试步内**（def 方法中） | `self.info("...")` | `print(...)` |
| **公共方法/工具函数** | `logging.info("...")` | `user_logger.info(...)`（pistar 旧写法） |

**依据**：《蓝区框架自动化框架切换》—— “公共方法中 `user_logger.info` 需替换成 `logging.info`，并去除 pistar 相关导入；测试步中日志记录还是延续 pistar 的习惯，使用 `self.info`”。

### 4.2 def 内日志写法

```python
def test_reedsolomon_performance_001(self):
    # 步骤开始
    self.info("步骤1：准备 16MB 数据块")
    data = self.build_data(16 * 1024 * 1024)

    self.info("步骤2：执行 RS 编码")
    result = self.rs_encode(data)

    # 关键结果输出
    self.info(f"编码吞吐: {result.throughput} MB/s")

    assert result.throughput > 0
```

规则：

1. **每个步骤开始打一条 `self.info`**，说明当前在做什么。
2. **关键输入/输出打日志**（数据规模、耗时、吞吐等），便于失败定位。
3. **断言失败信息写清楚**：`assert x > 0, f"吞吐量异常: {x}"`。
4. 日志**避免拼接大对象**，用懒格式化：`self.info("result: %s", result)`。

### 4.3 公共方法日志写法

```python
import logging

logger = logging.getLogger(__name__)

class DiskTool:
    def format_disk(self, dev):
        logger.info("开始格式化磁盘: %s", dev)      # ✅
        # logger 中不要用 self.info（非测试类）
```

> 从 pistar 迁移时，**全部 `user_logger.info` 替换为 `logging.info`**，并删除 `from ... import user_logger` 之类的旧导入。

### 4.4 日志级别使用

| 级别 | 使用场景 | 示例 |
|------|---------|------|
| `DEBUG` | 详细调试信息 | SQL、完整请求体、中间变量 |
| `INFO` | 关键业务步骤 | “开始编码”“编码完成，吞吐 xx” |
| `WARNING` | 可疑但非错误 | 重试、降级、指标接近阈值 |
| `ERROR` | 错误但用例未失败 | 非关键清理失败 |

现网默认 `log_cli_level = INFO`，`DEBUG` 信息仅在日志文件与失败时排查使用。

### 4.5 日志配置（pytest.ini）

```ini
[pytest]
log_cli = true
log_cli_level = INFO
log_cli_format = %(asctime)s [%(levelname)s] %(message)s
log_cli_date_format = %Y-%m-%d %H:%M:%S
```



## 5. 使用标签执行用例的方式方法

### 5.1 方式一（推荐）：专用参数 `--level` / `--case-type`

| 场景 | 命令 |
|------|------|
| 只跑 P0（Level 0 门槛，版本转测试判定） | `python -m pytest --level P0` |
| 跑 P0 + P1（门槛 + 基本，B 版本必验） | `python -m pytest --level P0 --level P1` |
| 跑 P0~P2（含 Level 2 重要，C 版本必验） | `python -m pytest --level P0 --level P1 --level P2` |
| 只跑性能用例 | `python -m pytest --case-type Performance` |
| 只跑安全用例 | `python -m pytest --case-type Security` |
| 只跑可靠性用例 | `python -m pytest --case-type Reliability` |
| 跑功能 + 兼容 | `python -m pytest --case-type Functional --case-type Compatibility` |
| 跑可靠性 + 性能 | `python -m pytest --case-type Reliability --case-type Performance` |
| **P0 的性能用例** | `python -m pytest --level P0 --case-type Performance` |
| P0/P1 的安全用例 | `python -m pytest --level P0 --level P1 --case-type Security` |
| P1 及以上的可靠性用例 | `python -m pytest --level P0 --level P1 --case-type Reliability` |
| 小写也可（容错） | `python -m pytest --level p0` |
| 全部已打标用例 | `python -m pytest -m case_info` |

**实测效果**（8 条用例样例，其中 P0 三条、Performance 一条、P0+Performance 一条）：

```text
$ pytest --level P0
3 passed, 5 deselected

$ pytest --level P0 --level P1
4 passed, 4 deselected

$ pytest --case-type Performance
1 passed, 7 deselected

$ pytest --level P0 --case-type Performance
1 passed, 7 deselected

$ pytest --level p0          # 小写容错
3 passed, 5 deselected
```

### 5.2 方式二：`-m` 表达式（派生标记）

自动派生 `level_P0`、`type_Functional` 等标记，可用布尔表达式：

| 场景 | 命令 |
|------|------|
| 只跑 P0 | `python -m pytest -m "level_P0"` |
| P0 或 P1 | `python -m pytest -m "level_P0 or level_P1"` |
| P0 的功能用例 | `python -m pytest -m "level_P0 and type_Functional"` |
| 只要安全用例 | `python -m pytest -m "type_Security"` |
| 只要可靠性用例 | `python -m pytest -m "type_Reliability"` |
| 排除性能用例 | `python -m pytest -m "case_info and not type_Performance"` |
| 排除可靠性用例 | `python -m pytest -m "case_info and not type_Reliability"` |
| P0 且是冒烟 | `python -m pytest -m "level_P0 and smoke"` |
| 性能且慢 | `python -m pytest -m "type_Performance and slow"` |
| 可靠性或安全 | `python -m pytest -m "type_Reliability or type_Security"` |

> ⚠️ `-m "case_info and not Performance"` **不会生效**，必须用派生标记 `type_Performance`。

### 5.3 方式三：标签合规性校验（CI 必跑）

```bash
python -m pytest --collect-only --strict-case-info -q
```

存在违规时输出：

```text
ERROR: case_info 标签校验失败，请修正后重试:
  tests/.../reedsolomon_performance_001_test.py::TestReedsolomonPerformance001::test_reedsolomon_performance_001  -> 缺少 case_info 标签
  tests/.../xxx_test.py::TestXxx::test_xxx  -> level 非法: 'P4'
```

### 5.4 单个 / 文件夹执行

```bash
# 执行单个用例（推荐的日常调试方式）
python -m pytest -vvv --tb=short tests/kunpeng_storage/.../reedsolomon_performance_001_test.py

# 执行文件夹
python -m pytest -vvv --tb=short tests/kunpeng_storage/.../performance/

# 文件夹内按标签进一步筛选
python -m pytest -vvv --tb=short tests/kunpeng_storage/.../performance/ --level P0
```

### 5.5 与看板 / 流水线组合

标签筛选可与 9.3.3 对接参数**自由组合**：

```bash
# 只跑 P0，生成看板结果并上传
python -m pytest -vvv --tb=short tests/ \
    --level P0 \
    --junitxml=test_result.xml \
    --upload-results

# 只收集 P0 + P1 用例信息（生成 metadata.xml）
python3 -m pytest tests/ --level P0 --level P1 --collect-only

# 黄区 pyagent 按类型跑安全用例
python -m pytest tests/ --case-type Security --junitxml=test_result.xml
```

### 5.6 CI 分层执行

```bash
# 0. 标签合规校验（必跑，阻断漏标）
python -m pytest --collect-only --strict-case-info -q

# 1. 版本转测试门槛 —— 仅 P0（Level 0，开发的 HLT / 版本集成转测试）
python -m pytest --level P0 -x --maxfail=3

# 2. B 版本必验 —— P0 + P1（Level 0 门槛 + Level 1 基本）
python -m pytest --level P0 --level P1 -n auto

# 3. C 版本必验 —— P0~P2（Level 0~2；Level 3 一般按测试策略针对性验证）
python -m pytest --level P0 --level P1 --level P2 --reruns 2 --reruns-delay 5

# 4. C 版本针对性验证 —— P3（Level 3 一般，按测试策略单独排期）
python -m pytest --level P3

# 5. 专项 —— 兼容性/安全/可靠性可单独排期
python -m pytest --case-type Compatibility
python -m pytest --case-type Security --level P0 --level P1
python -m pytest --case-type Reliability
python -m pytest --case-type Reliability --level P0 --level P1
```

### 5.7 执行命令速查卡

| 目的 | 命令 |
|------|------|
| 标签合规校验 | `pytest --collect-only --strict-case-info -q` |
| 跑 P0（Level 0 门槛） | `pytest --level P0` |
| 跑 P0+P1（Level 0+1） | `pytest --level P0 --level P1` |
| 跑 P0~P2（Level 0~2） | `pytest --level P0 --level P1 --level P2` |
| 跑 P3（Level 3 一般） | `pytest --level P3` |
| 跑性能 | `pytest --case-type Performance` |
| 跑可靠性 | `pytest --case-type Reliability` |
| 跑 P0 功能 | `pytest --level P0 --case-type Functional` |
| 跑 P1/P2 安全 | `pytest --level P1 --level P2 --case-type Security` |
| 跑 P0/P1 可靠性 | `pytest --level P0 --level P1 --case-type Reliability` |
| `-m` 等价写法 | `pytest -m "level_P0 and type_Functional"` |
| 排除性能 | `pytest -m "case_info and not type_Performance"` |
| 只要可靠性 | `pytest -m "type_Reliability"` |
| 全部已打标用例 | `pytest -m case_info` |
| 仅收集不执行 | `pytest --level P0 --collect-only` |
| 带看板输出 | `pytest --level P0 --junitxml=test_result.xml --upload-results` |

### 5.8 常见错误

| 错误写法 | 现象 | 正确写法 |
|---------|------|---------|
| `pytest -m "Performance"` | 0 条被选中 | `pytest --case-type Performance` |
| `pytest -m "case_info and not Performance"` | 排除失效 | `-m "case_info and not type_Performance"` |
| `level="p0"` | 可运行但违规 | `level="P0"`（对应 Level 0） |
| `level="L0"` / `level="Level 0"` | 筛选不到、报非法 | `level="P0"` |
| `level="P4"` | 报 level 非法 | 先扩展枚举再使用（Level 4） |
| `type="功能"` | 筛选不到 | `type="Functional"` |
| `type="reliability"` / `type="可靠性"` | 报 type 非法、筛选不到 | `type="Reliability"` |
| 未注册 `case_info` 标记 | `--strict-markers` 下报错 | 在 `pytest.ini` 注册 |
| `pytest.ini` 带 BOM | `unexpected line: '\ufeff[pytest]'` | 无 BOM UTF-8 保存 |

---

## 6. 版本历史

| 版本 | 日期 | 修订内容 |
|------|------|---------|
| v1.0 | 2026-11-07 | 初版发布（通用 pytest 规范） |
| v2.0 | 2026-11-07 | 新增 case_info 用例标签规范与统一执行方式 |
| v3.0 | 2026-11-07 | 按 5 章节重构；新增「一个 py 文件一个用例、def 名与 CIDA 用例名对应」硬约束；补充 level/type 定级判定标准 |
| v3.1 | 2026-11-09 | 更新用例级别（level）定义：`P0`~`P3` 依次对应 Level 0 门槛 / Level 1 基本 / Level 2 重要 / Level 3 一般，重写各级别说明、定级原则与决策流程；预留 Level 4 生僻、Level T 自动化工具级别；同步更新 CI 分层与执行速查 |
| v4.0 | 2026-11-09 | 用例类型（type）新增 `Reliability`（可靠性），补充其定义、验证内容与示例；新增「可靠性与其他类型的边界」判定说明；同步更新执行命令、`-m` 表达式、CI 专项与速查卡 |
| v4.1 | 2026-11-09 | 统一命名口径：所有「与 CIDA 用例名对齐」的规定统一改为「与用例编号对齐」，覆盖第 1 章命名规范（1.1 核心约束、1.2 对照表与派生说明、1.3 def 名称规则）、第 2 章核心编写约束、第 3 章类型判定原则；文件名、类名、def 名均改由用例编号派生 |

> 历史版本文件保留于同目录：`pytest自动化用例开发规范.md`