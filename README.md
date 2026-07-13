# Semantic Instruction IR (SIR)

## A Compact Semantic Representation for LLM Instruction Memory

---

# 1. Overview

**Semantic Instruction IR (SIR)** là một định dạng biểu diễn trung gian (Intermediate Representation) dùng để chuyển đổi chỉ dẫn bằng ngôn ngữ tự nhiên thành cấu trúc ngữ nghĩa nhỏ gọn dành cho LLM.

SIR không nhằm thay thế ngôn ngữ tự nhiên hoặc tạo ra một ngôn ngữ lập trình mới.

Mục tiêu của SIR là:

* loại bỏ phần diễn đạt dư thừa.
* giữ lại ý định và thông tin ảnh hưởng đến hành động.
* lưu trữ instruction dưới dạng có cấu trúc.
* cung cấp lại context cho LLM khi cần.

---

# 2. Problem

Instruction dành cho LLM hiện thường được lưu dưới dạng văn bản:

```text
Before modifying code, analyze architecture.
After changes, run build verification.
Do not report completion without successful build.
```

Nhược điểm:

* nhiều từ mô tả.
* lặp lại ý nghĩa.
* tốn context window.
* khó truy xuất phần quan trọng.

SIR chuyển đổi thành:

```text
@Inst

Int:CodeModification

Act:
- rd:Architecture
- upd:Code
- run:Build

Cons:
+BuildSuccess
-ReportWithoutValidation
```

LLM vẫn giữ được ý định nhưng giảm lượng thông tin dư thừa.

---

# 3. Core Concept

SIR hoạt động như một lớp trung gian:

```text
Human Instruction
        |
        v
Semantic Extraction
        |
        v
Semantic Instruction IR
        |
        v
SIR Memory
(md / DB / Knowledge Repository)
        |
        v
LLM Context Retrieval
        |
        v
Agent Execution
```

---

# 4. Design Principle

## 4.1 Minimal Specification

Chỉ lưu những thành phần cần thiết để tái tạo ý nghĩa.

Bao gồm:

```text
Intent
Entity
Action
Parameter
Constraint
Causal Relation
```

Không lưu:

```text
Filler words
Repeated explanations
Formatting
Obvious knowledge
```

---

## 4.2 Maximal Semantic Leverage

SIR tận dụng khả năng suy luận có sẵn của LLM.

Không mã hóa lại kiến thức phổ biến.

Ví dụ:

Không cần:

```text
Constructor Injection is a dependency injection pattern.
```

Chỉ cần:

```text
ref:Service
change:ConstructorInjection
```

---

# 5. SIR Structure

SIR gồm các block chính:

```text
@Inst
@Meta
@Chg
```

---

# 6. @Inst - Instruction Representation

`@Inst` lưu instruction có tính hành vi.

Dùng cho:

* agent rules.
* workflow.
* coding conventions.
* operational instructions.

Cấu trúc:

```text
@Inst

Int:[Intent]

Trig:[Trigger]

Role:[Role]

Cap:[Capability]

Act:
- action:target

Cons:
+required
-forbidden
```

Ví dụ:

```text
@Inst

Int:JavaDevelopment

Role:DeveloperAgent

Act:
- rd:ArchitectureContext
- upd:SourceCode
- run:Build

Cons:
+BuildSuccess
-ReportWithoutValidation
```

---

# 7. @Chg - Change Representation

`@Chg` lưu các thay đổi kỹ thuật, quyết định hoặc lịch sử xử lý.

Dùng cho:

* changelog.
* architecture decision.
* bug resolution.
* optimization history.

Cấu trúc:

```text
@Meta

date:YYYY-MM-DD


@Chg:Name

Ctx:
cause
->
effect
->
problem


Act:
- verb:target
```

Ví dụ:

```text
@Meta

date:2026-07-09


@Chg:GraphOptimization

Ctx:
LinearCanvasScaling
->
GravityIncrease
->
NodeJitter


Act:
- upd:prefSize[Math.sqrt(nodeCount)]
- tune:gravity[0.1]
- defer:applyDefaultStyle
```

---

# 8. Semantic Operation

Các action verb phổ biến:

| Verb  | Meaning  |
| ----- | -------- |
| rd    | read     |
| cr    | create   |
| mk    | make     |
| upd   | update   |
| add   | add      |
| rm    | remove   |
| ref   | refactor |
| set   | set      |
| tune  | tune     |
| inj   | inject   |
| defer | defer    |

---

# 9. Semantic Relation

SIR sử dụng:

```text
->
```

để biểu diễn:

* nguyên nhân → kết quả.
* điều kiện → hành động.
* phụ thuộc → ảnh hưởng.

Ví dụ:

```text
MissingContext
->
WrongImplementation
->
BuildFailure
```

Quy tắc:

* Không tự thêm nguyên nhân.
* Không suy diễn ngoài dữ liệu được cung cấp.

---

# 10. Constraint Representation

Constraint mô tả giới hạn hành vi.

Bắt buộc:

```text
+
```

Ví dụ:

```text
+RunTest
```

Cấm:

```text
-
```

Ví dụ:

```text
-ReportWithoutValidation
```

---

# 11. Storage Model

SIR có thể lưu trong:

```text
.md
.json
.database
.vector memory
.knowledge repository
```

Ví dụ:

```text
sir-memory/

├── instructions/
│     └── coding.inst

├── changes/
│     └── graph-layout.chg

└── decisions/
      └── architecture.dec
```

---

# 12. Retrieval Model

Khi cần sử dụng:

```text
Task
 |
 v
Search SIR Memory
 |
 v
Retrieve Relevant SIR
 |
 v
Inject Into LLM Context
 |
 v
Generate Action
```

LLM không cần đọc lại toàn bộ lịch sử.

Chỉ cần semantic representation liên quan.

---

# 13. Difference From Prompt

Prompt truyền thống:

```text
Remember these rules:
Always check architecture before coding...
```

SIR:

```text
@Inst

Act:
- rd:Architecture
- upd:Code

Cons:
+Validation
```

Khác biệt:

| Prompt                 | SIR                      |
| ---------------------- | ------------------------ |
| Câu chữ                | Cấu trúc nghĩa           |
| Dành cho con người đọc | Dành cho LLM tái sử dụng |
| Nhiều mô tả            | Tập trung hành động      |
| Context dài            | Context nén              |

---

# 14. Technical Definition

**Semantic Instruction IR (SIR)** là một phương pháp biểu diễn trung gian nhằm nén instruction bằng cách chuyển đổi ngôn ngữ tự nhiên thành cấu trúc semantic gồm Intent, Entity, Action, Parameter, Constraint và Causal Relation, cho phép LLM lưu trữ, truy xuất và thực thi lại ý định với lượng context nhỏ hơn.

---

# 15. Fundamental Principle

```text
Minimal Specification
+
Maximal Semantic Leverage
=
Semantic Instruction IR
```

SIR không lưu toàn bộ những gì con người đã nói.

SIR lưu phần cần thiết để LLM hiểu lại điều con người muốn đạt được.
