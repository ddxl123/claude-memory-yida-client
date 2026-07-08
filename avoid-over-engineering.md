---
name: avoid-over-engineering
description: 实现方案优先最轻手段，避免引入额外类型/包装机制，可扩展性不过度设计
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 737f9a40-c38a-4569-927c-aaa282a4a260
---

解决代码扩展性问题时，优先选最轻手段（如运行期 try-catch 兜底转异常），而非引入签名表 / 枚举 / 包装类等显式类型机制。

**Why:** 在 memory_algorithm_edit_page 的 checkJsHandle 改造中，为解决「硬编码 num 不通用」我引入了 ParamType 枚举 + BuiltinHandlerEntry 包装类 + validateParams 签名表，用户反馈「太复杂」。回退为 call 里一个 try-catch（catch TypeError / RangeError 转 paramValidation）后用户认可。

**How to apply:** 先用最简方案满足需求（如 lambda 内 as 强转 + call 兜底 catch），确有多个调用方需要显式类型约束时再考虑签名表。可扩展性是「以后可能需要」，不要为假设的未来需求提前搭重结构。注释要详，结构要简。
