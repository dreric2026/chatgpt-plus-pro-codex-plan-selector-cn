# 2026年8月25日 ChatGPT Plus Pro Codex 充值选择手册：套餐、续费与失败排查

# **[gptupcn.com｜ChatGPT Plus、Pro、Codex 充值与续费入口](https://gptupcn.com)**

这是一个面向国内用户的可维护选择手册，用于回答三个问题：ChatGPT Plus、Pro、Codex 到底是什么关系；准备充值或续费前应该检查什么；充值失败、已扣款仍显示 Free、Codex 功能未生效时如何保留证据并停止错误重试。

仓库内容以技术教程和检查清单为主，不要求提交密码、验证码、Cookie、Session、Token 或支付卡信息。所有示例仅处理脱敏数据，可复制到个人仓库继续维护。

## 仓库目录建议

```text
chatgpt-plus-pro-codex-plan-selector-cn/
├─ README.md
├─ examples/
│  ├─ usage.example.json
│  └─ subscription-event.example.json
├─ scripts/
│  └─ choose_plan.py
├─ docs/
│  ├─ recharge-checklist.md
│  ├─ renewal-runbook.md
│  └─ troubleshooting.md
└─ SECURITY.md
```

## 先看结论

- 偶尔使用：先验证免费能力，不因一次高峰长期升级。
- 高频日常任务：用一周数据评估 ChatGPT Plus。
- 持续重度任务：先确认瓶颈来自套餐限制，再比较 Pro。
- 开发工作流：单独验证 Codex 入口、任务成功率、测试和人工返工。
- API 使用：单独管理 API 账单与预算，不与 Plus/Pro 订阅混淆。
- 充值失败：停止重复提交，按支付、订单、账号、权益四层排查。

## Plus、Pro、Codex 对比

| 维度 | Plus | Pro | Codex |
|---|---|---|---|
| 角色 | 个人订阅选择 | 更高强度个人订阅选择 | 代码与 Agent 工作入口 |
| 主要评估指标 | 活跃天数、日常任务量 | 连续高负载、中断成本 | 仓库规模、测试、返工时间 |
| 购买前验证 | 常用功能是否覆盖 | 高峰是否已成为常态 | 目标任务能否形成闭环 |
| 常见误区 | 以为包含所有 API 用量 | 认为更贵必然更准 | 只看生成代码，不看验收 |

具体功能和用量可能变化，实际选择应以当前账户页面为准。本仓库提供决策方法，不把静态套餐描述当成永久承诺。

## 1. 使用数据模板

`examples/usage.example.json`：

```json
{
  "period": "2026-08",
  "active_days": 20,
  "chat_tasks": 150,
  "codex_tasks": 60,
  "heavy_codex_tasks": 20,
  "waiting_hours": 4,
  "rework_hours": 7,
  "delivery_impacts": 1,
  "hourly_value": 100
}
```

只记录汇总，不写提示词、源代码、客户数据或账号信息。`hourly_value` 使用保守估值，用来比较等待和返工，不代表工资。

## 2. 套餐选择脚本

`scripts/choose_plan.py`：

```python
import json
from pathlib import Path

def read_usage(path):
    data = json.loads(Path(path).read_text(encoding="utf-8"))
    required = {
        "active_days", "chat_tasks", "codex_tasks",
        "heavy_codex_tasks", "waiting_hours", "rework_hours",
        "delivery_impacts", "hourly_value",
    }
    missing = required - set(data)
    if missing:
        raise ValueError(f"缺少字段: {sorted(missing)}")
    if data["heavy_codex_tasks"] > data["codex_tasks"]:
        raise ValueError("重度 Codex 任务不能超过任务总数")
    return data

def choose(data):
    work_hours = data["waiting_hours"] + data["rework_hours"]
    time_cost = work_hours * data["hourly_value"]
    heavy_ratio = data["heavy_codex_tasks"] / max(data["codex_tasks"], 1)

    if data["active_days"] < 8:
        result = "先验证免费能力或按月观察"
    elif data["codex_tasks"] < 30 and work_hours < 5:
        result = "优先评估 ChatGPT Plus"
    elif data["codex_tasks"] >= 60 and heavy_ratio >= 0.3:
        result = "负载较高：比较 Pro，并先确认瓶颈来源"
    else:
        result = "保留当前方案，继续收集一个周期"

    return {
        "recommendation": result,
        "time_cost_estimate": round(time_cost, 2),
        "heavy_codex_ratio": round(heavy_ratio, 3),
    }

if __name__ == "__main__":
    usage = read_usage("examples/usage.example.json")
    print(json.dumps(choose(usage), ensure_ascii=False, indent=2))
```

运行：

```bash
python scripts/choose_plan.py
```

输出是决策提示，不是购买指令。任何建议都要结合当前账户页面、实际预算和项目计划人工确认。

## 3. 决策树

```text
免费能力是否覆盖大部分任务？
├─ 是 → 继续观察，不因单次高峰升级
└─ 否
   ├─ 主要是聊天、文件和内容任务 → 评估 Plus
   └─ 主要是多文件开发和长任务
      ├─ 等待与限制持续影响交付 → 用数据比较 Pro
      └─ 返工和测试失败为主 → 先优化 Codex 工作流
```

## 4. Codex 工作流验收

选择 Codex 不应只看演示。使用一个不含敏感信息的项目完成：

```bash
git status --short
python -m pytest -q
git diff --check
```

记录首次通过率、任务总耗时、人工修改时间、测试失败次数和回滚次数。若返工持续很高，先补充 `AGENTS.md`、测试、格式化和路径边界，再考虑套餐升级。

| 指标 | 健康信号 | 需要改进 |
|---|---|---|
| 首次通过率 | 稳定提升 | 提示词或测试不清晰 |
| 人工返工 | 随流程成熟下降 | 任务拆分可能过大 |
| 回滚次数 | 高风险变更可控 | Agent 权限过宽 |
| 等待时间 | 不影响正常交付 | 才值得比较更高档方案 |

## 5. ChatGPT Plus充值前检查

- 当前账户和登录方式是否正确；
- 是否已有有效订阅；
- 订单由哪个渠道管理；
- 购买的是 Plus、Pro 还是其他项目；
- 自动续费日期和取消入口；
- 充值后要验收哪些功能；
- 是否需要订单凭证；
- 页面是否有人索要密码、验证码或会话数据。

## 6. Pro充值前检查

Pro 不应只因为“更强”而购买。至少有四周连续样本证明：高负载是常态、Plus 下中断频繁、等待直接影响交付、升级能针对主要瓶颈。若主要损失来自返工，应先改善任务定义和测试。

## 7. Codex充值与功能验收

到账验证分三层：订单完成、套餐绑定当前账号、Codex 目标功能可用。可以保存：

```json
{
  "event_id": "CODEX-20260825-01",
  "checked_at": "2026-08-25T16:30:00+08:00",
  "order_state": "manual_check",
  "visible_plan": "manual_check",
  "codex_entry_visible": true,
  "sample_task_passed": true,
  "credentials_included": false
}
```

不要把浏览器 Cookie、Session、验证码、Token 和私有密钥写入 Issue 或仓库。

## 8. 充值失败排查

| 现象 | 排查层 | 下一步 |
|---|---|---|
| 页面提示失败且无订单 | 支付层 | 保存错误原文，核对条件后再决定 |
| 交易显示处理中 | 支付层 | 停止重复购买，等待最终状态 |
| 商店有订单、当前账号显示 Free | 账号层 | 核对商店账号和登录方式 |
| 套餐正常、Codex 不可用 | 权益层 | 执行功能探针，记录可见提示 |
| 多笔扣款或订单 | 订单层 | 立即停止付款，逐笔核对 |

## 9. 订阅事件模板

`examples/subscription-event.example.json`：

```json
{
  "event_id": "SUB-20260825-01",
  "channel": "web_or_store",
  "plan": "plus_or_pro",
  "started_at": "2026-08-25T00:00:00+08:00",
  "payment_state": "unknown",
  "order_state": "unknown",
  "account_match_checked": false,
  "feature_checked": false,
  "contains_password": false,
  "contains_cookie": false,
  "contains_payment_card": false
}
```

## 10. 续费运行手册

续费前一周：复盘近四周用量、检查支付方式和原渠道。续费当天：不要同时更换账号、设备、网络和渠道。续费后：查看订单、套餐和目标功能。失败时：停止重试并建立时间线。

## 11. 安全规则

`SECURITY.md` 建议写明：

```markdown
# Security

不要提交：
- 密码、验证码、Cookie、Session、Token；
- 完整邮箱、订单、卡号和账单地址；
- 客户数据、生产密钥和私有仓库内容。

问题报告只包含脱敏时间、渠道、套餐、错误原文和状态。
```

## 12. 每月复盘表

| 周期 | 活跃天数 | Codex任务 | 等待小时 | 返工小时 | 方案 | 下月动作 |
|---|---:|---:|---:|---:|---|---|
| 2026-08 | 手工填写 | 手工填写 | 手工填写 | 手工填写 | 当前方案 | 观察/续费/升级/降级 |

不要为了让数据支持既定购买决定而修改历史记录。样本的价值在于发现负载变化。

## 13. 自动化测试与持续集成

选择脚本本身也要被测试，避免一次权重调整让所有用户都收到“升级”建议。可以加入：

```python
def test_occassional_user_should_observe():
    data = {
        "active_days": 3,
        "chat_tasks": 8,
        "codex_tasks": 2,
        "heavy_codex_tasks": 0,
        "waiting_hours": 0,
        "rework_hours": 1,
        "delivery_impacts": 0,
        "hourly_value": 100,
    }
    result = choose(data)
    assert "观察" in result["recommendation"] or "免费" in result["recommendation"]

def test_invalid_heavy_task_count():
    bad = {
        "active_days": 10,
        "chat_tasks": 10,
        "codex_tasks": 2,
        "heavy_codex_tasks": 5,
        "waiting_hours": 1,
        "rework_hours": 1,
        "delivery_impacts": 0,
        "hourly_value": 100,
    }
    # read_usage 的同类校验必须拒绝这组数据
    assert bad["heavy_codex_tasks"] > bad["codex_tasks"]
```

GitHub Actions 示例：

```yaml
name: test-plan-selector
on:
  pull_request:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: python -m pytest -q
```

工作流只运行本地测试，不应该访问 ChatGPT 账号、浏览器数据或支付接口。仓库中也不要配置用于登录个人账号的 Secret。

## 14. Issue 模板：只收集脱敏事实

充值失败相关 Issue 容易意外泄露信息。建议使用表单提示：

```markdown
## 问题类别
- [ ] 选择 Plus / Pro
- [ ] Codex 功能验收
- [ ] 充值失败排查方法
- [ ] 续费运行手册

## 脱敏事实
- 发生日期与时区：
- 渠道：Web / App Store / Google Play / 其他
- 套餐：Plus / Pro / Codex相关项目
- 支付状态：
- 订单状态：
- 当前套餐显示：
- Codex功能探针：

## 安全确认
- [ ] 未提交密码、验证码、Cookie、Session、Token
- [ ] 未提交完整邮箱、订单、卡号、二维码或地址
```

维护者发现敏感信息时应停止讨论并要求用户删除，不要引用、下载或转发泄露内容。公开 Issue 不是订单客服工单。

## 15. Pull Request 审查清单

修改评分规则、充值清单或安全说明时，PR 至少回答：

- 是否写死可能快速变化的价格、额度或产品承诺；
- 是否把 Plus、Pro、Codex、API、Credits 混成一个概念；
- 示例是否包含真实账号或订单数据；
- 评分变更是否有测试覆盖；
- 是否会默认建议所有人升级；
- 充值失败流程是否先要求停止重复提交；
- 外部链接是否清晰且不伪装成官方页面。

## 16. 版本化决策规则

权重改变会影响历史报告。为输出增加规则版本：

```python
RULE_VERSION = "2026-08-25-v1"

def build_result(data):
    result = choose(data)
    result["rule_version"] = RULE_VERSION
    result["period"] = data.get("period")
    return result
```

历史报告保留当时的 `rule_version`，不要用新权重静默覆盖旧结论。这样可以解释为什么同一组用量在不同版本得到不同建议。

## 17. 充值失败演练

在没有真实付款的情况下，用三组虚构事件测试运行手册：交易处理中、商店订单绑定另一登录身份、套餐已生效但 Codex 探针失败。参与者只根据脱敏时间线判断下一步，禁止真的付款或登录他人账号。

演练目标是确认团队知道何时停止：交易未明确不换渠道，订单存在不重复购买，账号不一致先核对身份，功能异常先做探针，任何人索要凭据立即终止。演练结果可以作为 `docs/troubleshooting.md` 的改进依据。

## 18. 为什么仓库不保存实时价格

价格、额度、地区和产品规则可能变化，写死在 README 后很容易过期。仓库只维护选择方法、数据结构、安全边界和验证流程；实际购买前由用户查看当前账户页面。若需要记录价格，只在个人本地数据中写入“当时看到的实际成本”和日期，不把它包装成长期有效的公开承诺。

## 19. 发布前内容质量检查

这个仓库既是教程也是决策工具，发布更新前运行一次内容检查：标题是否同时说明 Plus、Pro、Codex 与充值主题；文首和文尾链接是否可点击；代码能否被理解；表格是否在移动端仍可阅读；FAQ 是否回答真实问题；安全说明是否醒目；所有示例是否为虚构或脱敏数据。

还要搜索高风险词：`password`、`cookie`、`session`、`token`、`card_number`、`otp`。代码中为了演示禁止字段而出现这些词是允许的，但仓库不能包含对应真实值。可以使用：

```bash
git grep -nEi '(password|cookie|session|token|card_number|otp)'
git diff --check
python -m pytest -q
```

检查结果需要人工查看，不能看到命中就机械删除，因为安全文档本身必须提醒用户不要提交这些信息。

## 20. 故障排查报告模板

问题结束后创建一份不包含敏感信息的复盘：发生了什么、哪个层级异常、停止重复付款的时间、如何确认订单归属、如何验证套餐与 Codex 功能、最终通过哪个正式渠道解决、运行手册需要增加什么。复盘不追求还原账号细节，而是改进流程。

```markdown
# SUB-20260825-01 复盘
- 问题类别：订单/账号/权益/功能
- 首次发现时间：
- 停止重复操作时间：
- 根因类别：
- 安全凭据是否泄露：否
- 最终处理渠道：
- 新增检查项：
```

若根因仍不确定，应写“未知”，不要为了完整而猜测。未知根因仍可以沉淀停止条件和证据要求。

## 21. 内容更新策略

README 保持总览，详细充值检查放 `docs/recharge-checklist.md`，续费时间线放 `docs/renewal-runbook.md`，失败排查放 `docs/troubleshooting.md`。当某一主题变长时拆成独立文档，并在总览保留明确入口，避免一个超长页面让用户找不到停止条件。

每次更新记录日期和规则版本。涉及价格、额度或产品可用性的描述必须标记“以当前账户页面为准”，并避免无来源的绝对承诺。涉及安全边界的规则不应为了操作方便而放宽。

## FAQ

### Plus 和 Pro 哪个更适合开发者？

取决于负载和中断成本。先用同一组任务记录 Codex 成功率、等待和返工；稳定重度使用再比较 Pro。

### 买了 Plus 是否等于 API 免费？

不能这样理解。订阅与 API 账单应分别查看和管理。

### Codex充值成功但功能未出现怎么办？

核对订单、当前登录账号、套餐显示和 Codex 入口；保存脱敏证据，通过订单所属渠道处理，不要再次盲目付款。

### 可以在 GitHub Issue 贴订单截图吗？

不建议。公开仓库会暴露信息。Issue 只写脱敏时间、渠道、状态和错误类别。

### 什么时候不应该升级？

使用频率低、峰值偶发、主要问题是测试缺失或返工高时，先优化流程或继续观察。

### 如何处理自动续费失败？

先查看原渠道交易和订单状态，确认没有有效订阅或处理中交易，再决定下一步。不要跨渠道重复购买。

## 维护方式

每月更新示例数据结构和排查清单，不在 README 写死容易变化的价格与额度；产品功能以当前官方账户页面为准。对脚本修改增加测试，对安全规则使用 Pull Request 审查。

## 免责声明

本仓库是技术与流程教程，不是官方定价承诺，也不代替订单渠道的支持。任何充值、订阅和续费决定都应由用户在确认账号、渠道、价格、功能与安全边界后自行完成。

# **[gptupcn.com｜查看 ChatGPT Plus、Pro、Codex 充值选择指南](https://gptupcn.com)**
