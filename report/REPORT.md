# Báo Cáo Lab 7: Embedding & Vector Store

**Họ tên:** Đồng Mạnh Hùng - 2A202600465  
**Ngày:** 2026-04-10

---

## 1. Warm-up (5 điểm)

### Cosine Similarity (Ex 1.1)

**High cosine similarity nghĩa là gì?**  
> High cosine similarity nghĩa là hai đoạn text có vector gần cùng hướng, tức là chúng có nội dung hoặc ý nghĩa khá giống nhau. Trong retrieval, điều này thường cho thấy chunk đó liên quan hơn tới câu hỏi.

**Ví dụ HIGH similarity:**
- Sentence A: Students may submit a grade appeal within five working days.
- Sentence B: Learners can file a grade appeal within 5 business days.
- Tại sao tương đồng: Cả hai câu đều nói về cùng chủ đề phúc khảo điểm và cùng mốc thời gian.

**Ví dụ LOW similarity:**
- Sentence A: Graduate students must meet the language requirement before graduation.
- Sentence B: The cafeteria serves lunch from 11 AM to 2 PM.
- Tại sao khác: Hai câu thuộc hai chủ đề hoàn toàn khác nhau, một câu về học vụ và một câu về dịch vụ sinh hoạt.

**Tại sao cosine similarity được ưu tiên hơn Euclidean distance cho text embeddings?**  
> Cosine similarity tập trung vào hướng của vector nên phù hợp hơn để đo mức độ giống nhau về ngữ nghĩa. Với text embeddings, độ lớn vector thường ít quan trọng hơn hướng biểu diễn.

### Chunking Math (Ex 1.2)

**Document 10,000 ký tự, chunk_size=500, overlap=50. Bao nhiêu chunks?**  
> *Trình bày phép tính:*  
> `num_chunks = ceil((10000 - 50) / (500 - 50)) = ceil(9950 / 450) = ceil(22.11...)`  
> *Đáp án:* `23 chunks`

**Nếu overlap tăng lên 100, chunk count thay đổi thế nào? Tại sao muốn overlap nhiều hơn?**  
> Khi overlap tăng lên 100 thì số chunk là `ceil((10000 - 100) / (500 - 100)) = ceil(9900 / 400) = 25`. Overlap lớn hơn làm số chunk tăng, nhưng giúp giữ ngữ cảnh tốt hơn tại ranh giới giữa hai chunk.

---

## 2. Document Selection — Nhóm (10 điểm)

### Domain & Lý Do Chọn

**Domain:** Vinlex - chatbot cho sinh viên tra cứu chính sách VinUni

**Tại sao nhóm chọn domain này?**  
> Đây là domain thực tế và sát nhu cầu sử dụng vì sinh viên thường xuyên cần tra cứu quy chế học vụ, phúc khảo điểm, điều kiện tốt nghiệp, nghỉ học tạm thời và các guideline liên quan đến campus safety. Bộ dữ liệu policy cũng có cấu trúc rõ ràng theo section, điều, khoản nên rất phù hợp để thử nghiệm chunking, retrieval và metadata filtering.

### Data Inventory

| # | Tên tài liệu | Nguồn | Số ký tự | Metadata đã gán |
|---|--------------|-------|----------|-----------------|
| 1 | Sexual Misconduct and Response Guideline | VinUni policy PDF -> Markdown | ~65,000 | `category`, `language`, `doc_type`, `audience`, `effective_date` |
| 2 | Quy chế đào tạo đại học hệ chính quy theo hệ thống tín chỉ | VinUni policy PDF -> Markdown | ~110,000 | `category`, `language`, `doc_type`, `audience`, `effective_date` |
| 3 | Quy chế đào tạo trình độ thạc sĩ | VinUni policy PDF -> Markdown | ~69,000 | `category`, `language`, `doc_type`, `audience`, `effective_date` |
| 4 | Student Grade Appeal Procedure | VinUni policy PDF -> Markdown | ~11,000 | `category`, `language`, `doc_type`, `audience`, `effective_date` |
| 5 | Quy định bổ sung cho học viên / tài liệu hỗ trợ cao học | VinUni policy / Markdown | ~8,000 - 10,000 | `category`, `language`, `doc_type`, `audience`, `effective_date` |

### Metadata Schema

| Trường metadata | Kiểu | Ví dụ giá trị | Tại sao hữu ích cho retrieval? |
|----------------|------|---------------|-------------------------------|
| `category` | string | `undergraduate_academic_policy` | Giúp lọc theo nhóm policy như đại học, cao học, appeals, safety |
| `audience` | string | `undergraduate_students` | Hữu ích khi câu hỏi dành riêng cho đại học hay cao học |
| `language` | string | `vi`, `en` | Dữ liệu có cả tiếng Việt và tiếng Anh |
| `doc_type` | string | `regulation`, `procedure`, `guideline` | Phân biệt văn bản quy chế, quy trình và guideline |
| `effective_date` | string | `2024-10-30` | Có ích nếu sau này cần ưu tiên văn bản mới hơn |

---

## 3. Chunking Strategy — Cá nhân chọn, nhóm so sánh (15 điểm)

### Baseline Analysis

Chạy `ChunkingStrategyComparator().compare()` trên 3 tài liệu:

| Tài liệu | Strategy | Chunk Count | Avg Length | Preserves Context? |
|-----------|----------|-------------|------------|-------------------|
| `data1.md` | FixedSizeChunker (`fixed_size`) | 131 | 498.40 | Trung bình, dễ cắt ngang ý |
| `data1.md` | SentenceChunker (`by_sentences`) | 79 | 697.33 | Giữ câu tốt nhưng chunk dài |
| `data1.md` | RecursiveChunker (`recursive`) | 100 | 419.96 | Cân bằng giữa coherence và kích thước |
| `data2.md` | FixedSizeChunker (`fixed_size`) | 188 | 498.28 | Ổn về kích thước nhưng dễ cắt giữa điều khoản |
| `data2.md` | SentenceChunker (`by_sentences`) | 222 | 366.70 | Giữ câu tốt nhưng khá vụn |
| `data2.md` | RecursiveChunker (`recursive`) | 189 | 401.08 | Hợp với policy dài, nhiều section |
| `data4.md` | FixedSizeChunker (`fixed_size`) | 25 | 484.40 | Dễ cắt giữa bước thủ tục |
| `data4.md` | SentenceChunker (`by_sentences`) | 12 | 857.42 | Chunk quá dài với procedure |
| `data4.md` | RecursiveChunker (`recursive`) | 19 | 398.26 | Giữ block step-by-step tốt hơn |

### Strategy Của Tôi

**Loại:** RecursiveChunker

**Mô tả cách hoạt động:**  
> RecursiveChunker thử tách văn bản theo thứ tự separator từ lớn đến nhỏ như paragraph, newline, sentence, word rồi cuối cùng mới cắt cứng theo ký tự. Cách này giúp giữ các đơn vị có nghĩa như section hoặc block policy trước khi chia nhỏ hơn.

**Tại sao tôi chọn strategy này cho domain nhóm?**  
> Với domain Vinlex, tài liệu là các policy documents có cấu trúc theo section, điều và khoản. RecursiveChunker phù hợp vì giữ được block ý nghĩa tốt hơn FixedSizeChunker, đồng thời ít tạo chunk quá dài hơn SentenceChunker ở một số file PDF convert.

**Code snippet (nếu custom):**
```python
# Tôi dùng built-in RecursiveChunker, không dùng custom strategy.
```

### So Sánh: Strategy của tôi vs Baseline

| Tài liệu | Strategy | Chunk Count | Avg Length | Retrieval Quality? |
|-----------|----------|-------------|------------|--------------------|
| `data2.md` | best baseline: `SentenceChunker` | 222 | 366.70 | Giữ câu tốt nhưng hơi vụn |
| `data2.md` | **của tôi: `RecursiveChunker`** | 189 | 401.08 | Cân bằng hơn giữa coherence và kích thước |
| `data4.md` | best baseline: `FixedSizeChunker` | 25 | 484.40 | Ổn nhưng dễ cắt ngang step |
| `data4.md` | **của tôi: `RecursiveChunker`** | 19 | 398.26 | Tốt hơn cho procedure |

### So Sánh Với Thành Viên Khác

| Thành viên | Strategy | Retrieval Score (/10) | Điểm mạnh | Điểm yếu |
|-----------|----------|----------------------|-----------|----------|
| Tôi | RecursiveChunker | 7 | Giữ block policy hợp lý, dễ kết hợp metadata | Cần tune thêm tham số |
| Nguyễn Công Nhật Tân | FixedSizeChunker | 6 | Đơn giản, dễ kiểm soát | Dễ cắt ngang ý |
| Phan Anh Ly Ly | SentenceChunker | 7 | Giữ mạch câu tốt | Một số chunk quá dài |
| Trần Nhật Minh | RecursiveChunker (tuned) | 7 | Phù hợp với policy dài | Cần thêm benchmark |
| Phan Nguyễn Việt Nhân | FixedSizeChunker / custom | 6 | Dễ so sánh baseline | Coherence chưa tốt bằng recursive |

**Strategy nào tốt nhất cho domain này? Tại sao?**  
> RecursiveChunker là phù hợp nhất cho domain này vì tài liệu có cấu trúc rõ ràng theo section và paragraph. Strategy này giữ ngữ cảnh tốt hơn FixedSizeChunker và ít tạo ra các chunk quá dài hơn SentenceChunker trong một số trường hợp.

---

## 4. My Approach — Cá nhân (10 điểm)

### Chunking Functions

**`SentenceChunker.chunk`** — approach:  
> Tôi dùng regex để tách câu theo `.`, `!`, `?` và khoảng trắng hoặc newline phía sau. Sau đó tôi nhóm các câu theo `max_sentences_per_chunk` để tạo chunk có nghĩa và ổn định.

**`RecursiveChunker.chunk` / `_split`** — approach:  
> Tôi triển khai đệ quy: nếu đoạn hiện tại đủ ngắn thì trả về luôn; nếu quá dài thì thử split theo separator hiện tại. Nếu mảnh sinh ra vẫn quá dài thì gọi lại với separator nhỏ hơn, đến khi cần thì cắt cứng theo độ dài ký tự.

### EmbeddingStore

**`add_documents` + `search`** — approach:  
> `add_documents` nhận list `Document`, embed `doc.content`, rồi lưu record gồm `content`, `metadata` và `embedding` vào store. `search` embed query, tính dot product giữa query vector và các record trong store, sort giảm dần và trả về top-k.

**`search_with_filter` + `delete_document`** — approach:  
> `search_with_filter` lọc metadata trước rồi mới similarity search trên tập đã lọc. `delete_document` xóa tất cả record có `metadata["doc_id"]` trùng `doc_id` và trả về `True` nếu có record bị xóa.

### KnowledgeBaseAgent

**`answer`** — approach:  
> Hàm `answer` gọi `store.search()` để lấy top-k context, ghép các context đó thành prompt rồi truyền sang `llm_fn`. Trong `main.py` hiện tại, `llm_fn` đang là `demo_llm`, nên output dùng để minh họa pipeline RAG chứ chưa phải câu trả lời của LLM thật.

### Test Results

```text
..........................................                               [100%]
42 passed in 0.10s
```

**Số tests pass:** 42 / 42

---

## 5. Similarity Predictions — Cá nhân (5 điểm)

| Pair | Sentence A | Sentence B | Dự đoán | Actual Score | Đúng? |
|------|-----------|-----------|---------|--------------|-------|
| 1 | Students can appeal final grades within five working days. | Learners may submit a grade appeal within 5 business days. | high | 0.1179 | Có |
| 2 | VinUni students must meet graduation requirements. | The university has policies on degree completion. | high | -0.1719 | Không |
| 3 | Graduate students must meet language requirements. | The cafeteria serves lunch from 11 AM. | low | 0.1233 | Không |
| 4 | A complainant may report to the police. | VinUni can support a complainant in making a police report. | high | -0.1174 | Không |
| 5 | Undergraduate academic regulations apply to all VinUni students. | Sexual misconduct reports are handled confidentially. | low | -0.0778 | Có |

**Kết quả nào bất ngờ nhất? Điều này nói gì về cách embeddings biểu diễn nghĩa?**  
> Cặp 4 là kết quả bất ngờ nhất vì hai câu khá gần nghĩa nhưng điểm lại âm. Điều này cho thấy mock embeddings trong bài lab không phản ánh semantic similarity tốt như embedding model thật, nên phần này chủ yếu có ý nghĩa minh họa cách dùng `compute_similarity()`.

---

## 6. Results — Cá nhân (10 điểm)

### Benchmark Queries & Gold Answers (nhóm thống nhất)

| # | Query | Gold Answer |
|---|-------|-------------|
| 1 | Sinh viên có thể phúc khảo điểm trong bao lâu sau khi điểm được công bố trên Canvas? | Trong vòng 05 ngày làm việc sau khi điểm được công bố trên Canvas. |
| 2 | Điều kiện tối thiểu về CGPA để sinh viên đại học được công nhận tốt nghiệp là gì? | CGPA tối thiểu 2.00/4.00. |
| 3 | Học viên thạc sĩ cần đáp ứng những điều kiện nào để được công nhận tốt nghiệp? | Hoàn thành học phần và luận văn/đề án đạt yêu cầu, đạt chuẩn ngoại ngữ đầu ra, hoàn thành trách nhiệm theo quy định, không bị truy cứu trách nhiệm hình sự và không trong thời gian bị kỷ luật đình chỉ học tập. |
| 4 | Nếu sinh viên cho rằng điểm cuối cùng bị nhập sai hoặc chấm thiên vị, bước đầu tiên trong quy trình grade appeal là gì? | Liên hệ trực tiếp với giảng viên ngay sau khi điểm được công bố để trao đổi trước khi escalate. |
| 5 | Theo quy định của VinUni, complainant trong vụ sexual misconduct có quyền báo công an không, và Trường có thể hỗ trợ gì? | Có quyền báo công an và VinUni có thể hỗ trợ việc thực hiện police report khi phù hợp. |

### Kết Quả Của Tôi

| # | Query | Top-1 Retrieved Chunk (tóm tắt) | Score | Relevant? | Agent Answer (tóm tắt) |
|---|-------|--------------------------------|-------|-----------|------------------------|
| 1 | Phúc khảo điểm sau Canvas bao lâu? | Top-1 chưa đúng hoàn toàn nếu dùng mock embeddings | 0.1908 | Không | Trả chuỗi demo từ context |
| 2 | CGPA tối thiểu để tốt nghiệp đại học? | `data2` nằm trong top kết quả nhưng có lúc không đứng đầu nếu không filter | 0.1183 | Một phần | Trả chuỗi demo từ context |
| 3 | Điều kiện tốt nghiệp thạc sĩ? | Top-1 là `data3`, đúng policy cao học | 0.2460 | Có | Trả chuỗi demo từ context |
| 4 | Bước đầu tiên của grade appeal là gì? | Top-1 là `data4`, đúng procedure | 0.1760 | Có | Trả chuỗi demo từ context |
| 5 | Complainant có quyền báo công an không? | Top-1 chưa ổn định khi dùng mock embeddings | 0.2159 | Không | Trả chuỗi demo từ context |

**Bao nhiêu queries trả về chunk relevant trong top-3?** 4 / 5

### Metadata filtering có giúp ích không?

> Có. Với các query mơ hồ như tốt nghiệp đại học và tốt nghiệp thạc sĩ, metadata filtering giúp rõ rệt. Ví dụ khi filter `{"category": "graduate_academic_policy"}` thì query về thạc sĩ trả đúng `data3`; khi filter `{"category": "undergraduate_academic_policy"}` thì query về CGPA đại học trả đúng `data2`.

---

## 7. What I Learned (5 điểm — Demo)

**Điều hay nhất tôi học được từ thành viên khác trong nhóm:**  
> Tôi thấy rằng cùng một bộ tài liệu nhưng mỗi strategy chunking lại có điểm mạnh riêng. FixedSizeChunker dễ làm baseline, SentenceChunker giữ câu tốt, còn RecursiveChunker hợp hơn với policy docs dài và nhiều section.

**Điều hay nhất tôi học được từ nhóm khác (qua demo):**  
> Tôi nhận ra metadata schema có ảnh hưởng rất lớn đến retrieval. Khi metadata được thiết kế tốt theo `category`, `topic`, hoặc `audience`, các query mơ hồ vẫn có thể retrieve đúng tài liệu nhanh hơn.

**Nếu làm lại, tôi sẽ thay đổi gì trong data strategy?**  
> Tôi sẽ chuẩn hóa dữ liệu kỹ hơn sau khi convert PDF sang Markdown, đặc biệt là heading và layout. Tôi cũng sẽ chunk toàn bộ tài liệu trước khi index và dùng embedding model thật để benchmark phản ánh đúng semantic retrieval hơn.

---

## Tự Đánh Giá

| Tiêu chí | Loại | Điểm tự đánh giá |
|----------|------|-------------------|
| Warm-up | Cá nhân | 5 / 5 |
| Document selection | Nhóm | 9 / 10 |
| Chunking strategy | Nhóm | 14 / 15 |
| My approach | Cá nhân | 10 / 10 |
| Similarity predictions | Cá nhân | 4 / 5 |
| Results | Cá nhân | 8 / 10 |
| Core implementation (tests) | Cá nhân | 30 / 30 |
| Demo | Nhóm | 0 / 5 |
| **Tổng** | | **80 / 100** |
