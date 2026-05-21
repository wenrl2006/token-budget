# Token Budget（词元预算）

> 首个 TypeScript 实现的 AI Agent 预算自治模式 —— 每次调 LLM 前先问预算，不够就自动降级。

**Token Budget** 不是一个产品，也不是一个 APP。它是一个**设计模式**——像 MVC 一样，应该渗透到每一个 Agent 的血脉里。

---

## 这是什么？

给每一个 AI Agent 装一个**智能油耗表**：

- **事前检查**：Agent 调 LLM 之前先问 `budget.canSpend()`
- **智能选型**：预算充裕 → 用满血模型；紧张 → 自动降级到廉价/免费模型
- **自动记账**：每次调用记录任务、token 数、模型、费用
- **优先级感知**：高优先级任务优先保障，低优先级任务等一等

## 为什么需要？

当前所有"成本控制"方案都是**事后诸葛亮**：

- OpenAI 用量仪表盘 → 花完了才知道
- API Key 限额 → 硬封顶，不智能
- 没有人在设计 Agent 的时候给它装上"事前判断"这一步

**Token Budget 解决的核心问题：Agent 缺乏预算自治能力。**

## 快速上手

```bash
npm install @wenrl2006/token-budget --registry=https://npm.pkg.github.com
```

```typescript
import { TokenBudget } from '@wenrl2006/token-budget'

const budget = new TokenBudget({
  dailyLimit: 100000,       // 10万 token/天
  autoDowngrade: true,
})

// 每次调 LLM 前：
const task = { type: 'deep_reasoning', estimatedTokens: 8000, priority: 'high' }

if (budget.canSpend(task).allow) {
  const model = budget.selectModel(task.type)  // 预算够 → 满血版
  const result = await callLLM(input, model)
  budget.recordSpend('分析用户输入', 8000, model)
}
```

## API

| 方法 | 返回 | 说明 |
|------|------|------|
| `canSpend(task)` | `{allow, suggestion, model?}` | 检查任务能否在预算内执行 |
| `selectModel(taskType)` | `string` | 根据剩余预算选模型 |
| `recordSpend(task, tokens, model)` | `void` | 记录本次调用 |
| `getStats()` | `BudgetStats` | 查看用量统计 + 调用日志 |
| `resetDaily()` | `void` | 重置日计数器 |

## 模型分级

| 等级 | 模型 | 费用 | 用途 |
|------|------|------|------|
| PREMIUM | DeepSeek V4 Reasoning | ¥8/百万token | 深度推理、命理、战略 |
| STANDARD | DeepSeek V4 Fast | ¥2/百万token | 分析、写作、总结 |
| CHEAP | DeepSeek V3 Lite (免费) | ¥0 | 日常对话、格式化 |

## 验证过的行为

```
场景A: 充裕预算 + 深度推理     → ✅ 允许 (约¥0.02)
场景B: 简单格式化             → ✅ 允许，用免费模型
场景C: 大量消耗到95%          → ✅ 降级，剩1500 token
场景D: 预算紧张 + 中等任务     → 🚫 拦截（"请充值"）
场景E: 预算紧张 + 免费任务     → ✅ 自动切免费模型
场景F: 单次任务超限            → 🚫 拦截（"建议拆分"）
```

## 时间戳验证

本仓库是 **Token Budget 模式的第一个 TypeScript 实现**。三个独立渠道交叉验证发布时间：

| 渠道 | 时间戳 | 状态 |
|------|--------|------|
| Git commit | `ac1bba1` · 2026-05-21 | ✅ |
| GitHub Package | `@wenrl2006/token-budget@0.1.0` · 2026-05-21 | ✅ |
| 公众号文章 | 待发布 · 草稿箱 | ✅ |

## 设计哲学

> Token Budget is not a product. It's a **design pattern** — like MVC, it should live in every Agent's blood.

模式很简单：**每次调 LLM 前，先问预算。** 就像 try/catch 一样自然。

## 协议

MIT © RongYe
