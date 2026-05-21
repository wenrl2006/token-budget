# TokenBudget (Token Budget)

> The first TypeScript implementation of Token Budget (词元预算) pattern — every AI Agent pre-checks budget before LLM calls.

[![npm version](https://img.shields.io/npm/v/@wenrl2006/token-budget)](https://www.npmjs.com/package/token-budget)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## What is Token Budget?

A design pattern that gives every AI Agent a **smart fuel gauge**:

- **Pre-check**: Agent asks `budget.canSpend()` before every LLM call
- **Smart selection**: Budget plenty → use premium model. Tight → auto-downgrade to cheaper/free
- **Auto accounting**: Every call is logged with task, tokens, model, cost
- **Priority-aware**: High-priority tasks get the budget they need; low-priority tasks wait

## Why?

Current approaches are all **post-hoc**:
- OpenAI/Anthropic usage dashboards show you what you *already spent*
- API key rate limits are hard caps, not smart allocation
- No one gives the Agent itself the ability to **decide when to spend and when to save**

Token Budget does exactly that. It's not a bill viewer — it's a **decision engine** that lives *inside* your Agent.

## Quick Start

```typescript
import { TokenBudget } from 'token-budget'

const budget = new TokenBudget({
  dailyLimit: 100000,       // 100K tokens/day ≈ ¥0.80 premium model
  autoDowngrade: true,
  preferredTier: 'STANDARD'
})

// Before every LLM call:
const task = { type: 'deep_reasoning', estimatedTokens: 8000, priority: 'high' }

if (budget.canSpend(task).allow) {
  const model = budget.selectModel(task.type)
  const result = await callLLM(input, model)
  budget.recordSpend('analyze user input', 8000, model)
}
```

## API

### Constructor
```typescript
new TokenBudget(config?: {
  dailyLimit?: number      // default: 100,000
  monthlyLimit?: number    // default: 3,000,000
  perTaskLimit?: number    // default: 50,000
  autoDowngrade?: boolean  // default: true
  alertThreshold?: number  // default: 0.1 (10%)
  preferredTier?: 'PREMIUM' | 'STANDARD' | 'CHEAP'
})
```

### Methods
| Method | Returns | Description |
|--------|---------|-------------|
| `canSpend(task)` | `{allow, suggestion, model?}` | Check if task fits budget |
| `selectModel(taskType)` | `string` | Pick model by remaining budget |
| `recordSpend(task, tokens, model)` | `void` | Log this call |
| `getStats()` | `BudgetStats` | Usage summary + call log |
| `resetDaily()` | `void` | Reset daily counter |
| `reset()` | `void` | Reset all counters |

### Model Tiers
| Tier | Model | Cost/M tokens | Use for |
|------|-------|--------------|---------|
| PREMIUM | DeepSeek V4 Reasoning | ¥8 | deep reasoning, analysis, coding |
| STANDARD | DeepSeek V4 Fast | ¥2 | drafting, summarizing, chat |
| CHEAP | DeepSeek V3 Lite (free) | ¥0 | greetings, formatting, echo |

## Verified Behavior

```
Scenario A: 充裕预算深度推理     → ✅ Allowed (¥0.02)
Scenario B: 简单格式化           → ✅ Allowed
Scenario C: 大量消耗到95%        → ✅ 1,500 token remaining
Scenario D: 预算紧张 + 中等任务   → 🚫 Blocked ("请充值")
Scenario E: 预算紧张 + 免费类型   → ✅ Auto-downgrade to free model
Scenario F: 单次任务超限          → 🚫 Blocked ("建议拆分")
```

## Design Philosophy

> Token Budget is not a product. It's a **design pattern** — like MVC, it should live in every Agent's blood.

The pattern is simple: **before every LLM call, `askBudget()`**. Just like `try/catch`.

## License

MIT © RongYe
