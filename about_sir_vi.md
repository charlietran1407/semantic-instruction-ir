# Giới thiệu về SIR (Semantic Instruction IR)

> **SIR mã hóa ngữ nghĩa triển khai, cho phép tái tạo reasoning.**

---

# SIR là gì?

SIR (Semantic Instruction IR) là một định dạng nén ngữ nghĩa được thiết kế để bảo tồn và truyền tải kiến thức triển khai kỹ thuật một cách hiệu quả.

Khác với các dạng tóm tắt truyền thống, SIR tập trung bảo toàn:

- quyết định triển khai (implementation decisions)
- reasoning nhân quả (causal reasoning)
- thay đổi hệ thống (system changes)
- ràng buộc kỹ thuật (technical constraints)
- tham số quan trọng (concrete parameters)
- công việc được trì hoãn (deferred engineering work)

Mục tiêu của SIR không phải tạo ra tài liệu ngắn hơn.

Mục tiêu của SIR là giữ lại đủ thông tin ngữ nghĩa để kỹ sư hoặc LLM trong tương lai có thể tái tạo reasoning phía sau các quyết định triển khai.

---

# Vì sao SIR tồn tại?

Các dự án phần mềm hiện đại tích lũy rất nhiều kiến thức kỹ thuật:

- quyết định kiến trúc
- lịch sử tối ưu hóa
- kết quả debugging
- lựa chọn refactoring
- chiến lược migration
- trade-off về hiệu năng

Tuy nhiên, kiến thức này thường nằm trong các dạng khó tái sử dụng:

- tài liệu dài
- thảo luận Pull Request
- commit message
- meeting notes
- issue tracker

Những dạng này lưu được lịch sử, nhưng thường yêu cầu người đọc phải phân tích lại trước khi hiểu được reasoning triển khai.

SIR tạo ra một lớp semantic trung gian:

```

Technical Document
↓
Implementation Semantics
↓
SIR
↓
Reasoning Reconstruction
↓
Future Engineering Work

```

---

# Triết lý cốt lõi

## SIR không phải là bản tóm tắt

Một bản tóm tắt trả lời:

> "Tài liệu này nói về điều gì?"

SIR trả lời:

> "Những quyết định triển khai nào đã được đưa ra, vì sao cần chúng, và reasoning nào cần được tiếp tục trong tương lai?"

---

## Encode decisions, not explanations

SIR loại bỏ phần diễn giải không cần thiết nhưng giữ lại:

- điều gì đã thay đổi
- vì sao thay đổi
- constraint nào tồn tại
- công việc nào còn lại

Ví dụ:

Nguồn:

```

Sau khi phân tích vấn đề hiệu năng startup,
chúng tôi quyết định chuyển database initialization
sang background execution vì UI bị block trong quá trình loading.

```

SIR:

```

Ctx:
database initialization
→ UI blocking
→ background execution

Act:

* upd:databaseInitialization [logic:background execution]

```

Phần diễn giải bị loại bỏ, nhưng reasoning triển khai vẫn được giữ lại.

---

# Mục tiêu thiết kế

SIR tối ưu theo ba ưu tiên:

---

## 1. Bảo toàn Key Information (100%)

SIR bảo tồn:

- hành động triển khai
- state transition
- quan hệ nhân quả
- constraint
- parameter và giá trị cụ thể
- deferred task

Mất thông tin ở các nhóm này được xem là lỗi.

---

## 2. Độ chính xác ngữ nghĩa (≥96%)

SIR giữ lại:

- tên entity gốc
- thuật ngữ kỹ thuật
- intent triển khai
- quan hệ giữa các component

Mục tiêu không phải giống câu chữ.

Mục tiêu là tương đương về semantic.

---

## 3. Compression (≥40%)

SIR loại bỏ:

- prose dư thừa
- mô tả passive
- giải thích lặp lại

nhưng vẫn giữ implementation meaning.

---

# Cấu trúc SIR

Một tài liệu SIR bao gồm các semantic block:

```

@Meta

@Chg

Ctx

Act

defer

Cons

```

Ví dụ:

```

@Chg:LazyStrategyIndexers

Ctx:
eager component loading
→ high memory usage
→ lazy loading

Act:

* rm:@Component
* cr:indexerRegistry
* defer:loadIndexer

```

---

# Các thành phần

## @Chg

Đại diện cho một concern triển khai.

Mỗi block nên tập trung vào một nhóm thay đổi logic độc lập.

Ví dụ:

```

@Chg:GraphCallResolution

```

Một dự án có thể có nhiều thay đổi độc lập:

```

@Chg:UIThreadFix

@Chg:DatabaseOptimization

@Chg:GraphVisualization

```

---

## Ctx

Mô tả quan hệ nhân quả phía sau thay đổi.

Format:

```

cause → effect → outcome

```

Ví dụ:

```

Ctx:
absolute file paths
→ project relocation breaks indexing
→ store relative paths

```

Ctx giải thích tại sao một Act tồn tại.

---

## Act

Đại diện cho quyết định triển khai.

Ví dụ:

```

Act:

* rm:fallbackNameMatching
* upd:CallResolver
* cr:FQNMethodVertex

```

Act mô tả thay đổi, không viết lại toàn bộ câu chuyện.

---

## defer

Đại diện cho công việc được trì hoãn có chủ đích.

Ví dụ:

```

defer:

* migrateLegacyData [cond:after_release]

```

---

## Cons

Chỉ dùng khi source nêu rõ constraint hoặc requirement.

Ví dụ:

```

Cons:

* wrong [pitfall:name-only matching]

- must [why:preserve compatibility]

```

---

# Loại tài liệu phù hợp

SIR phù hợp với tài liệu chứa implementation semantics.

---

## Software Change Records

Ví dụ:

- feature implementation notes
- Pull Request technical description
- commit summary
- changelog có chứa quyết định kỹ thuật

---

## Architecture và Design Documents

Ví dụ:

- Architecture Decision Record (ADR)
- technical proposal
- system design document
- architecture evolution record

---

## Debugging và Root Cause Analysis

Ví dụ:

- production incident
- performance investigation
- bug analysis

SIR bảo tồn:

```

Problem
↓
Root Cause
↓
Decision
↓
Implementation Change

```

---

## Refactoring và Optimization Records

Ví dụ:

- memory optimization
- database tuning
- performance improvement
- dependency migration

---

## Engineering Knowledge Base

Ví dụ:

- subsystem knowledge
- onboarding document
- project memory
- maintenance notes

---

# Không phù hợp với

SIR không tối ưu cho tài liệu không chứa quyết định triển khai.

Ví dụ:

## Giao tiếp thông thường

- email
- announcement
- lịch họp

## Marketing

- product description
- promotional document

## Tài liệu học tập thuần túy

- tutorial
- conceptual explanation

## Báo cáo dạng tường thuật

Các tài liệu chỉ mô tả sự kiện nhưng không có technical decision.

---

# Ứng dụng thực tế

## Knowledge Base / Memory OS

SIR có thể lưu lịch sử phát triển của dự án:

- thay đổi kiến trúc
- tối ưu hiệu năng
- bug fix quan trọng
- quyết định thiết kế

LLM trong tương lai có thể phục hồi context mà không cần nạp toàn bộ lịch sử.

---

## Architecture Changelog

Changelog truyền thống:

```

Changed module A.
Fixed issue B.

```

SIR:

```

Ctx:
problem
→ impact
→ decision

Act:

* upd:moduleA

```

SIR lưu reasoning phía sau thay đổi.

---

## Code Handoff và Onboarding

SIR hỗ trợ truyền kiến thức giữa:

- kỹ sư
- team
- AI agent

Nó lưu intent phía sau code, không chỉ cấu trúc code.

---

## LLM-to-LLM Knowledge Transfer

SIR tạo một lớp trung gian để AI trao đổi kiến thức triển khai.

Một LLM có thể chuyển technical context thành SIR.

Một LLM khác có thể phục hồi reasoning từ SIR.

---

# FAQ

## 1. SIR có phải chỉ là một dạng tóm tắt tài liệu không?

Không.

Summary tối ưu việc giảm độ dài văn bản.

SIR tối ưu việc bảo toàn implementation semantics.

Mục tiêu không phải:

```

document ngắn hơn

```

Mà là:

```

engineering reasoning có thể phục hồi

````

---

## 2. Vì sao không dùng Markdown thông thường?

Markdown rất tốt cho tài liệu con người đọc.

Nhưng tài liệu kỹ thuật lớn thường chứa:

- giải thích lặp lại
- lịch sử thảo luận
- narrative context

SIR tạo semantic layer chỉ giữ phần cần cho reasoning trong tương lai.

SIR không thay thế documentation.

Nó bổ sung cho documentation.

---

## 3. Vì sao LLM cần SIR?

LLM có thể đọc tài liệu bình thường.

Thách thức nằm ở tính liên tục:

- repository lớn
- lịch sử nhiều năm
- context window giới hạn
- nhiều AI agent

SIR giúp duy trì knowledge continuity bằng cách lưu lại quyết định quan trọng.

---

## 4. SIR có làm mất thông tin không?

Có, một cách có chủ đích.

Mọi compression đều có trade-off.

SIR ưu tiên:

1. Implementation decisions
2. Causal relationships
3. Constraints
4. Important parameters
5. Deferred work

Các phần narrative không ảnh hưởng reasoning sẽ được loại bỏ.

---

## 5. SIR có thay thế ADR không?

Không.

ADR và SIR phục vụ các mục tiêu khác nhau.

ADR tập trung vào:
- ghi nhận quyết định kiến trúc
- lưu trữ rationale, trade-offs và lifecycle của quyết định

SIR tập trung vào:
- nén semantic information
- bảo toàn causal relationships, decisions, constraints và actions
- tối ưu cho retrieval và reasoning của LLM

SIR không thay thế ADR.
Một ADR, cũng như README, design document, changelog hoặc technical report, có thể được chuyển đổi sang SIR để lưu trữ và xử lý hiệu quả hơn bởi LLM.

---

## 6. Vì sao không dùng JSON/YAML?

JSON/YAML tốt cho structured data.

Nhưng implementation knowledge cần causal relationship.

Ví dụ:

JSON:

```json
{
  "action": "remove",
  "target": "fallbackMatching"
}
````

SIR:

```
Ctx:
name-only matching
→ false positive edges

Act:
- rm:fallbackMatching
```

SIR cân bằng giữa:

* structure
* readability
* reasoning

---

## 7. SIR có phải là standard chính thức không?

Không.

Hiện tại SIR là một semantic representation format được đề xuất cho việc bảo tồn implementation semantics.

Nó chưa phải:

* ISO standard
* IEEE standard
* industry standard

Mục tiêu hiện tại là chứng minh tính hữu ích trong workflow kỹ thuật sử dụng LLM.

---

## 8. Đánh giá chất lượng SIR như thế nào?

Một SIR tốt phải giúp LLM khác:

* hiểu vấn đề ban đầu
* hiểu vì sao thay đổi xảy ra
* xác định component liên quan
* tiếp tục phát triển mà không cần source document

Tiêu chí cuối cùng:

> Khả năng phục hồi implementation reasoning.

---

# Lợi ích chính

## Semantic Compression

Giảm token sử dụng nhưng vẫn giữ ý nghĩa triển khai.

## Context Recovery

Giúp LLM phục hồi quyết định kỹ thuật trong các session tương lai.

## Knowledge Continuity

Duy trì kiến thức giữa:

* kỹ sư
* team
* AI agent
* các giai đoạn phát triển

## Human và Machine Readability

SIR vừa có thể đọc bởi kỹ sư, vừa phù hợp cho xử lý bởi LLM.

---

# Nguyên tắc cuối cùng

SIR không nén text.

SIR nén implementation semantics.

Mục tiêu của SIR là bảo tồn những quyết định giúp reasoning kỹ thuật có thể tiếp tục trong tương lai.
