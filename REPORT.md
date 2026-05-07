# Lab 21 — Evaluation Report

**Học viên**: Huy Tran — 2A202600303
**Ngày nộp**: 2026-05-07
**Submission option**: A + B (lightweight ZIP + HF Hub)

## 1. Setup
- **Base model**: unsloth/Qwen2.5-3B-bnb-4bit
- **Dataset**: 5CD-AI/Vietnamese-alpaca-gpt4-gg-translated, 200 samples (180 train + 20 eval)
- **max_seq_length**: 1024 (p95 = 562, rounded up)
- **GPU**: Tesla T4, 15.6 GB VRAM
- **Training cost**: $0.07 (11.6 phút @ $0.35/hr)
- **HF Hub links**:
	- r=8: https://huggingface.co/HuyTran1301/qwen2.5-3b-vi-lab21-r8
	- r=16: https://huggingface.co/HuyTran1301/qwen2.5-3b-vi-lab21-r16
	- r=64: https://huggingface.co/HuyTran1301/qwen2.5-3b-vi-lab21-r64

## 2. Rank Experiment Results

| Rank | Trainable Params | Train Time (min) | Peak VRAM (GB) | Eval Loss | Perplexity |
|------|------------------|------------------|----------------|-----------|------------|
| 8    | 1,843,200        | 3.77             | 7.22           | 1.5577    | 4.7479     |
| 16   | 3,686,400        | 4.05             | 6.62           | 1.5161    | 4.5544     |
| 64   | 14,745,600       | 3.76             | 8.00           | 1.4768    | 4.3790     |
| Base | -                | -                | -              | N/A       | N/A        |

## 3. Loss Curve Analysis
- Loss curve: [fig/loss.png](fig/loss.png)
- Train loss giảm từ ~1.61 xuống ~1.39 sau 3 epochs (69 steps).
- Không có eval loss trong quá trình train (eval_strategy = no), nên chỉ nhận xét xu hướng giảm ổn định của train loss.
- Eval loss sau train ổn định (1.5161), chưa thấy dấu hiệu overfitting rõ ràng trên tập nhỏ.

## 4. Qualitative Comparison (5 examples)

### Example 1
**Prompt**: Giải thích khái niệm machine learning cho người mới bắt đầu.
**Base**: "Machine learning là một phân khúc của trí tuệ nhân tạo, nó tập trung vào việc thiết lập các mô hình..."
**Fine-tuned (r=16)**: "Machine learning là một bộ môn công nghệ máy tính dựa trên việc học tập và cải thiện các dự đoán..."
**Nhận xét**: Câu trả lời FT rõ ràng và giống văn phong giải thích hơn.

### Example 2
**Prompt**: Viết đoạn code Python tính số Fibonacci thứ n.
**Base**: "Để tính số Fibonacci thứ n, bạn có thể sử dụng hàm đệ quy hoặc vòng lặp..."
**Fine-tuned (r=16)**: "Để tính số Fibonacci thứ n, bạn có thể viết một đoạn code Python như sau: ..."
**Nhận xét**: FT có cấu trúc hướng dẫn + code rõ ràng, base dài dòng.

### Example 3
**Prompt**: Liệt kê 5 nguyên tắc thiết kế UI/UX.
**Base**: "Thân thiện với người dùng..."
**Fine-tuned (r=16)**: "Chuyển đổi, thích ứng, đơn giản..."
**Nhận xét**: FT concise hơn, nhưng chất lượng nội dung cần rõ hơn (nội dung có thể bị chung chung).

### Example 4
**Prompt**: Tóm tắt sự khác biệt giữa LoRA và QLoRA.
**Base**: "LoRA và QLoRA là hai phương pháp cải thiện hiệu năng..."
**Fine-tuned (r=16)**: "LoRA và QLoRA là hai phương pháp regularization..."
**Nhận xét**: FT có lỗi nhỏ (gọi sai ý nghĩa LoRA/QLoRA), cần bổ sung kiểm soát fact.

### Example 5
**Prompt**: Phân biệt prompt engineering, RAG, và fine-tuning.
**Base**: "Prompt engineering, RAG, và fine-tuning là ba cách khác nhau..."
**Fine-tuned (r=16)**: "Prompt engineering, RAG và fine-tuning là ba kỹ thuật khác nhau..."
**Nhận xét**: FT rõ ràng hơn, giống văn phong giải thích học thuật.

## 5. Conclusion ve Rank Trade-off
Với tập dữ liệu 200 mẫu, r=64 cho perplexity tốt nhất (4.379) nhưng tăng params và VRAM đáng kể. Chênh lệch perplexity từ r=16 sang r=64 chỉ giảm ~0.18, trong khi trainable params tăng 4x và peak VRAM lên ~8 GB. r=8 nhanh và nhẹ hơn, nhưng perplexity kém hơn rõ (4.748), chất lượng câu trả lời không ổn định bằng r=16. Diminishing returns bắt đầu rõ từ r=16 -> r=64: gain nhỏ, chỉ hợp nếu mục tiêu chất lượng cao nhất và GPU đủ. Nếu deploy production với T4 và dataset nhỏ, r=16 là điểm cân bằng tốt nhất về chất lượng/chi phí/VRAM. r=64 phù hợp khi có nhu cầu chất lượng tối đa và có tài nguyên đủ, còn r=8 chỉ nên dùng cho demo nhanh hoặc hệ thống rất giới hạn tài nguyên.

## 6. What I Learned
- Chọn rank không chỉ dựa trên perplexity, mà cần nhìn VRAM và trainable params.
- Eval sau train là cần thiết vì T4 không đủ VRAM để eval during training.
- Chất lượng qualitative có thể lộ ra lỗi fact dù train loss giảm, cần có kiểm soát nội dung.
