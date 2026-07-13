# Semantic IR DSL: Semantic Memory Representation for LLM

## 1. Tổng quan

Semantic IR DSL là một kỹ thuật biểu diễn thông tin dạng nén dành cho LLM, nhằm chuyển đổi nội dung ngôn ngữ tự nhiên dài thành cấu trúc ngữ nghĩa nhỏ gọn để lưu trữ và tái sử dụng.

Mục tiêu không phải là thay thế ngôn ngữ tự nhiên, mà là lưu lại phần thông tin cần thiết để LLM có thể tái tạo ý định, hành động và bối cảnh khi cần.

Luồng hoạt động:

```
Natural Language
        |
        v
Semantic Compression
        |
        v
Semantic IR DSL
        |
        v
Semantic Memory Storage
(md / DB / Knowledge Repository)
        |
        v
Retrieve Context
        |
        v
LLM Reasoning / Execution
```

---

# 2. Vấn đề

Hiện nay, thông tin dành cho LLM thường được lưu dưới dạng:

* tài liệu dài
* ghi chú
* changelog
* prompt
* lịch sử hội thoại

Các dạng này có nhiều thông tin không cần thiết:

* câu chữ mô tả
* phần giải thích lặp lại
* ngữ cảnh hiển nhiên
* cách diễn đạt của con người

Ví dụ:

```
ArcadeDbService trước đây sử dụng field injection.
Điều này làm dependency bị ẩn và gây khó khăn khi viết unit test.
Đã chuyển sang constructor injection và cập nhật test.
```

Phần quan trọng đối với LLM chỉ là:

```
Component:
ArcadeDbService

Issue:
FieldInjection
->
HiddenDependency
->
TestComplexity

Action:
ConstructorInjection
Remove:ReflectionTestUtils
```

---

# 3. Mục tiêu kỹ thuật

Semantic IR DSL nhằm:

* Giảm kích thước context lưu trữ.
* Tăng mật độ thông tin.
* Giữ lại ý nghĩa thay vì câu chữ.
* Cho phép LLM sử dụng lại kiến thức trong quá khứ.
* Giảm việc gửi lại các mô tả dài.

Các thông tin cần bảo toàn:

```
Entity
Identifier
Intent
Action
Parameter
Constraint
Causal Relationship
```

Các thông tin có thể loại bỏ:

```
Filler words
Repeated explanation
Formatting
Obvious knowledge
```

---

# 4. Nguyên tắc thiết kế

## 4.1 Minimal Specification

Chỉ lưu thông tin ảnh hưởng đến việc hiểu hoặc hành động.

Không lưu:

```
upd means update
```

vì LLM đã hiểu.

Chỉ lưu:

```
upd:Parser
```

---

## 4.2 Maximal Semantic Leverage

Tận dụng kiến thức có sẵn của LLM.

Không mã hóa lại kiến thức phổ biến.

Ví dụ:

Không cần:

```
Constructor injection is a dependency injection technique
```

Chỉ cần:

```
ref:Service
change:constructor_injection
```

---

# 5. Cấu trúc Semantic IR DSL

Semantic IR DSL sử dụng các block chính:

```
@Inst
@Meta
@Chg
```

---

# 6. @Inst - Instruction Memory

`@Inst` lưu các quy tắc hoặc hành vi của agent.

Dùng cho:

* workspace rules
* coding rules
* agent behavior
* workflow

Cấu trúc:

```
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

```
@Inst

Int:JavaDevelopment

Act:
- rd:ArchitectureContext
- upd:SourceCode
- run:Build

Cons:
+BuildSuccess
-ReportWithoutValidation
```

---

# 7. @Chg - Change Memory

`@Chg` lưu các thay đổi đã xảy ra.

Dùng cho:

* changelog
* quyết định kỹ thuật
* lịch sử sửa lỗi
* kiến trúc hệ thống

Cấu trúc:

```
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

```
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

# 8. Semantic Operator

Các operator phổ biến:

| Operator | Ý nghĩa       |
| -------- | ------------- |
| rd       | đọc           |
| cr       | tạo           |
| mk       | tạo/thực hiện |
| upd      | cập nhật      |
| add      | thêm          |
| rm       | xóa           |
| ref      | tái cấu trúc  |
| set      | thiết lập     |
| tune     | điều chỉnh    |
| inj      | inject        |
| defer    | trì hoãn      |

---

# 9. Quan hệ ngữ nghĩa

Toán tử:

```
->
```

biểu diễn quan hệ:

* nguyên nhân → kết quả
* điều kiện → hành động
* phụ thuộc → ảnh hưởng

Ví dụ:

```
MissingContext
->
WrongModification
->
BuildFailure
```

LLM không được tự tạo thêm nguyên nhân không có trong dữ liệu.

---

# 10. Constraint

Constraint mô tả giới hạn hành vi.

Bắt buộc:

```
+
```

Ví dụ:

```
+RunTest
```

Không được:

```
-
```

Ví dụ:

```
-ReportWithoutValidation
```

---

# 11. Cách sử dụng

Semantic IR DSL được lưu như memory:

Ví dụ:

```
knowledge/
 ├── architecture/
 ├── decisions/
 ├── changes/
 └── instructions/
```

Khi cần xử lý task:

```
Task
 |
 v
Retrieve related Semantic IR
 |
 v
Inject into LLM context
 |
 v
Generate response/action
```

---

# 12. So sánh với lưu trữ truyền thống

## Văn bản truyền thống

```
10 trang tài liệu
```

Ưu:

* dễ đọc

Nhược:

* nhiều thông tin thừa
* tốn context

## Semantic IR DSL

```
50 dòng semantic graph
```

Ưu:

* nhỏ hơn
* tập trung vào quyết định
* LLM dễ tái sử dụng

---

# 13. Bản chất kỹ thuật

Semantic IR DSL không phải:

* ngôn ngữ lập trình.
* schema bắt buộc.
* hệ thống rule engine.

Nó là:

> Một định dạng biểu diễn bộ nhớ ngữ nghĩa tối giản, giúp LLM lưu trữ và tái sử dụng thông tin bằng cách giữ lại các thành phần có giá trị hành vi.

---

# 14. Định hướng mở rộng

Có thể mở rộng:

```
@Fact
```

cho kiến thức ổn định.

```
@Decision
```

cho quyết định kiến trúc.

```
@Task
```

cho workflow.

Nhưng nguyên tắc giữ nguyên:

```
Minimal Specification
+
Maximal Semantic Leverage
```

---

**Tóm tắt một câu:**

Semantic IR DSL là cách nén ngôn ngữ tự nhiên thành "ký ức có cấu trúc" cho LLM, giữ lại ý định, hành động, ràng buộc và quan hệ nguyên nhân để AI có thể tái sử dụng mà không cần lưu toàn bộ văn bản gốc.
