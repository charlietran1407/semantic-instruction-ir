# Semantic Instruction IR (SIR)

SIR (Semantic Instruction IR) là định dạng nén ngữ nghĩa được thiết kế để lưu trữ và truyền tải **quyết định triển khai** một cách hiệu quả, giúp LLM có thể tái tạo lại reasoning một cách chính xác.

## Mục tiêu thiết kế

SIR ưu tiên ba nguyên tắc theo thứ tự:

1. **KEY INFORMATION (100%)** — Giữ lại toàn bộ hành động, thay đổi, causal chain, ràng buộc và công việc hoãn lại.
2. **SEMANTIC ACCURACY (≥96%)** — Giữ tên entity, thuật ngữ kỹ thuật và ý định gần với nguồn gốc nhất có thể.
3. **COMPRESSION (≥40%)** — Loại bỏ văn xuôi, giọng bị động, tối đa hóa mật độ thông tin.

---

## Instruction SIR (Phiên bản cuối cùng)

```markdown
You are an expert at transforming natural language into Semantic Instruction IR (SIR) for LLMs.

**SIR encodes implementation semantics, enabling reasoning reconstruction.**

Convert source text into SIR with priority order:

1. **KEY INFORMATION (100%)** — Preserve all actions, state transitions, causal chains, constraints, parameters, and deferred tasks.
2. **SEMANTIC ACCURACY (≥96%)** — Use entity names and verbs matching the source.
3. **COMPRESSION (≥40%)** — Maximize density, remove prose and passive voice.

## RULES
- Every `Act:` and `defer:` MUST have at least one corresponding `Ctx:` chain mentioning the target entity.
- Every `Ctx:` chain MUST justify at least one `Act:` or `defer:` item.
- "Replace X with Y" → `rm:X` + `cr/upd:Y`.
- "Remove X" → `rm:X` (never encode as `why:`).
- Split unrelated concerns into separate `@Chg` blocks.
- Never hallucinate or add metadata not present in the source.
- Preserve every concrete value exactly (numbers, constants, thresholds...).

## NOTATION
```markdown
@Meta date:[YYYY-MM-DD]
@Chg:[DescriptiveName] [scope:module]

Ctx: cause -> effect (-> outcome)

Act:
- verb:target [logic:..., param:..., cond:..., type:..., why:...]

defer:
- task [cond:...]

Cons: (only if explicitly stated in the source)
- wrong [pitfall:...]
+ must [why:...]
```

---

## SIR phù hợp với loại document nào?

SIR đặc biệt hiệu quả với các loại tài liệu sau:

- **Changelog & Release Notes**
- **Architecture Decision Records (ADR)**
- **Technical Guidelines / Implementation Guidelines**
- **Code Refactoring Summaries**
- **Meeting Notes về kỹ thuật**
- **Migration & Optimization Documents**
- **Knowledge Base entries** (đặc biệt cho Knowledge OS (Quado project))
- **Pull Request Descriptions** (technical part)

**Không khuyến khích** dùng SIR cho:
- Tài liệu hướng dẫn người dùng (User Guide)
- Tài liệu marketing hoặc mô tả sản phẩm
- Narrative writing / Story-telling
- Legal documents

---

## Triết lý cốt lõi

> **SIR encodes implementation semantics, enabling reasoning reconstruction.**

SIR không phải là tóm tắt văn bản, cũng không phải tài liệu mô tả. Nó là **biểu diễn ngữ nghĩa triển khai** dưới dạng có thể tái tạo reasoning, giúp LLM hoặc lập trình viên sau này dễ dàng tiếp nối và hiểu rõ quyết định đã đưa ra.
