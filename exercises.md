# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 14:15–17:00

**Domain:** OrbitTech Store Customer Support

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 14:15–14:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (14:30–14:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness (Độ trung thực / Không ảo giác) | Bot trả lời xã giao, chào hỏi hoặc bổ sung kiến thức thường thức hiển nhiên ngoài context. | Bot tự bịa đặt dữ liệu (hallucination) trong các nghiệp vụ nhạy cảm (y tế, tài chính, pháp lý, chính sách). | Tinh chỉnh prompt (bắt buộc dựa 100% vào context), giảm temperature, thêm guardrails kiểm tra trích dẫn. |
| Answer Relevance (Độ đúng trọng tâm câu hỏi) | Bot chủ động từ chối lịch sự do câu hỏi ngoài phạm vi, hoặc hỏi ngược lại để làm rõ ý người dùng. | Bot trả lời lan man, lạc đề hoàn toàn, nói chuyện vòng vo không giải quyết đúng ý định câu hỏi. | Siết lại prompt sinh câu trả lời (buộc trả lời trực diện), cải thiện bước phân tích/viết lại query (query rewriting). |
| Context Recall (Độ đầy đủ của dữ liệu tìm được) | Câu hỏi chỉ yêu cầu tóm tắt ý chính ngắn gọn, hoặc context dùng từ đồng nghĩa với ground truth. | Context tìm về thiếu các chi tiết sống còn, khiến LLM thiếu thông tin bắt buộc và phải trả lời cụt/đoán mò. | Tối ưu retrieval: tăng chunk size/overlap, chuyển sang hybrid search (BM25 + vector), mở rộng top-k. |
| Context Precision (Độ chính xác và thứ tự của dữ liệu tìm được) | Cần lấy nhiều chunk phụ để so sánh dữ liệu đa tài liệu, miễn là chunk đúng vẫn nằm trong kết quả. | Chunk quan trọng bị tụt xuống cuối danh sách hoặc lẫn quá nhiều chunk rác gây nhiễu cho LLM. | Thêm bước Re-ranking (ví dụ Cohere/BGE), tối ưu hóa embedding model hoặc áp dụng bộ lọc filter/compress context. |
| Completeness (Độ trọn vẹn của câu trả lời) | Người dùng chỉ yêu cầu câu trả lời nhanh (quick check), không cần liệt kê toàn bộ chi tiết phụ. | Câu trả lời bỏ sót bước quy trình bắt buộc, điều kiện tiên quyết hoặc các trường hợp ngoại lệ quan trọng. | Cải thiện prompt (yêu cầu cấu trúc checklist/bullet points đủ ý), tăng số lượng chunk liên quan gửi vào context. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> Lấy một tập cặp câu trả lời (A, B), rồi cho judge chấm mỗi cặp hai lần: điều kiện 1 - thứ tự (A, B); điều kiện 2 - đảo thứ tự (B, A), nội dung giữ nguyên. Nếu judge chọn câu đứng trước nhiều hơn hẳn mức 50%, hoặc đổi kết quả khi chỉ đảo vị trí, thì có position bias. Nên chạy trên nhiều cặp và so sánh tỷ lệ thắng của vị trí đầu giữa hai điều kiện.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> Ghi rõ trong rubric rằng độ dài không được cộng điểm. Chấm theo tiêu chí cụ thể (đúng, đủ ý, bám context) và trừ điểm phần lan man hoặc lặp ý. Có thể thêm yêu cầu câu trả lời ngắn gọn, đúng trọng tâm ở mức điểm cao nhất.

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> Judge cũng là một model nên có thể thiên lệch (quá dễ dãi, quá khắt khe, hoặc thích câu dài). Phải so điểm của judge với điểm người chấm trên một tập mẫu (ví dụ bằng hệ số tương quan hoặc Cohen's kappa) để biết judge có đáng tin không. Nếu lệch nhiều thì chỉnh rubric hoặc prompt cho đến khi gần với người chấm.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | 0.80 | Bịa chính sách, thời hạn bảo hành hay phí hoàn tiền gây hại trực tiếp cho khách, nên đặt ngưỡng cao nhất. |
| Answer Relevance | 0.70 | Lạc đề làm khách phải hỏi lại, gây khó chịu nhưng ít rủi ro hơn thông tin sai. |
| Completeness | 0.70 | Thiếu ý hoặc thiếu điều kiện làm khách hiểu sai, nhưng thường không sai hẳn như hallucination. |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> Offline evaluation: chạy trên golden dataset trước mỗi lần deploy, làm quality gate trong CI/CD để chặn thay đổi làm điểm giảm.

> Online evaluation: giám sát traffic thật sau khi deploy (điểm tự động trên mẫu hội thoại, phản hồi của khách, tỷ lệ chuyển sang nhân viên) để phát hiện lỗi mà dataset không bao phủ.

> Human review: dùng cho các trường hợp rủi ro cao hoặc mơ hồ (hoàn tiền, bảo mật tài khoản), các mẫu judge chấm thấp hoặc không chắc, và để calibrate judge định kỳ.

---

## Part 2 — Core Coding (14:45–15:40)

Hoàn thiện các TODO bắt buộc trong `template.py`.

### Task 1 — Data Models

- `QAPair`: question, expected answer, gold context, metadata và retrieved contexts.
- `EvalResult`: answer-side scores, optional retrieval scores, pass/failure fields.
- `overall_score()`: trung bình Faithfulness, Relevance và Completeness.

### Task 2 — RAGASEvaluator

Answer-side:

- `evaluate_faithfulness(answer, context)`
- `evaluate_relevance(answer, question)`
- `evaluate_completeness(answer, expected)`

Retrieval-side:

- `evaluate_context_recall(contexts, expected)`
- `evaluate_context_precision(contexts, expected)`

Full pipeline:

- `run_full_eval(..., contexts=None)` luôn tính ba answer metrics.
- Nếu có `contexts`, tính và lưu thêm Context Recall và Context Precision.
- Retrieval scores không làm thay đổi `overall_score()` và pass rule gốc.

### Task 3 — LLMJudge

- `score_response(question, answer, rubric)`
- `detect_bias(scores_batch)`

### Task 4 — BenchmarkRunner

- `run(qa_pairs, agent_fn, evaluator)`
- `generate_report(results)`
- `run_regression(new_results, baseline_results)`
- `identify_failures(results, threshold)`

`BenchmarkRunner.run()` phải truyền `pair.retrieved_contexts` vào
`run_full_eval()`. Report phải có average của hai retrieval metrics.

### Task 5 — FailureAnalyzer

- `categorize_failures(failures)`
- `find_root_cause(failure)`
- `generate_improvement_suggestions(failures)`
- `generate_improvement_log(failures, suggestions)`

Kiểm tra:

```bash
pytest tests/ -v
```

`rerank_by_overlap()` là TODO bonus của Exercise 3.5. Test tương ứng được skip
nếu bạn chưa làm bonus.

---

## Part 3 — Golden Dataset & Real Benchmark (15:40–16:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | ____ / 20 |
| Easy | ____ / 5 |
| Medium | ____ / 7 |
| Hard | ____ / 5 |
| Adversarial | ____ / 3 |
| Source documents được sử dụng | ____ / 10 |
| Validator status | PASS / FAIL |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| | | | |
| | | | |
| | | | |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:*

**Xác nhận:**

- [ ] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [ ] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [ ] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | | | | | | | | | |
| E02 | | | | | | | | | |
| E03 | | | | | | | | | |
| E04 | | | | | | | | | |
| E05 | | | | | | | | | |
| M01 | | | | | | | | | |
| M02 | | | | | | | | | |
| M03 | | | | | | | | | |
| M04 | | | | | | | | | |
| M05 | | | | | | | | | |
| M06 | | | | | | | | | |
| M07 | | | | | | | | | |
| H01 | | | | | | | | | |
| H02 | | | | | | | | | |
| H03 | | | | | | | | | |
| H04 | | | | | | | | | |
| H05 | | | | | | | | | |
| A01 | | | | | | | | | |
| A02 | | | | | | | | | |
| A03 | | | | | | | | | |

**Aggregate Report**

- Overall pass rate: ____%
- Avg Context Recall: ____
- Avg Context Precision: ____
- Avg Faithfulness: ____
- Avg Relevance: ____
- Avg Completeness: ____
- Failure type distribution: ____

**Ba cases có Overall Score thấp nhất**

1. ID: ____ | Score: ____ | Failure type: ____
2. ID: ____ | Score: ____ | Failure type: ____
3. ID: ____ | Score: ____ | Failure type: ____

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> *Câu trả lời:*

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [ ] Correctness
- [ ] Completeness
- [ ] Relevance
- [ ] Evidence/citation
- [ ] Actionability
- [ ] Safety/privacy
- [ ] Tone/clarity
- [ ] Dimension khác: __________

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | | |
| 4 | | |
| 3 | | |
| 2 | | |
| 1 | | |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| | | |
| | | |
| | | |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:*

### Exercise 3.4 — Framework Comparison (Bonus +5)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí | Framework 1: ____ | Framework 2: ____ |
|---|---|---|
| Setup complexity | | |
| Metrics available | | |
| CI/CD integration | | |
| Kết quả trên cùng dataset | | |
| Insight rút ra | | |

- Scores có nhất quán không?
- Framework nào strict hơn và vì sao?
- Hai framework có tìm ra cùng failure cases không?

> *Phân tích:*

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không
thay đổi Context Recall hay không.

1. Chọn ít nhất 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()` hoặc một reranker khác.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| **Avg** | | | | | |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:*

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:*

---

## Part 4 — Reflection (16:35–16:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 16:50–17:00.

- [ ] Tất cả required tests pass.
- [ ] `golden_dataset.json` validate thành công.
- [ ] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [ ] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [ ] Exercise 3.3 có rubric 1–5 và bias controls.
- [ ] `reflection.md` có ba failure analyses và regression strategy.
- [ ] Đã copy `template.py` thành `solution/solution.py`.
- [ ] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
