# ⚖️ Legal Chatbot System

![Legal Chatbot Banner](https://github.com/user-attachments/assets/e3100fd2-cfee-44e7-b588-083f0e3fdd0f)

Hệ thống chatbot hỗ trợ tìm kiếm thông tin pháp luật bằng tiếng Việt, ứng dụng mô hình Bi-Encoder kết hợp Cross-Encoder để tối ưu khả năng truy vấn ngữ nghĩa.

---

## 📂 Datasets

| Tên Dataset | Mô tả |
|-------------|-------|
| `soict-dataset-2024` | Bộ dữ liệu gốc gồm các văn bản pháp luật. |
| `soict-dataset-2024-segmented` | Phiên bản đã được phân đoạn bằng thư viện [underthesea](https://github.com/undertheseanlp/underthesea). |
| `cross-encoder-dataset` | Được tạo bằng cách lấy ground truth + 3 hard negative từ Bi-Encoder + 1 random negative. Gồm hơn **500.000 mẫu train** và **100.000 mẫu validation**. |
| `cross-encoder-dataset-segmented` | Tương tự `cross-encoder-dataset` nhưng dùng Bi-Encoder mới và áp dụng segmentation. |

---

## 🧠 Models

| Tên mô hình | Mô tả |
|-------------|-------|
| `bi_encoder` | Bi-Encoder sử dụng model gốc từ [bkai/vietnamese-bi-encoder](https://huggingface.co/bkai-foundation-models/vietnamese-bi-encoder), được fine-tune 2 epoch trên `soict-dataset-2024-segmented`, có tích hợp hard negative mining. |
| `bi_encoder_embedding` | FAISS vector database chứa các embedding vector từ toàn bộ corpus (`corpus_segmented.csv`) sử dụng encoder của `bi_encoder`. |
| `cross_encoder_new` | Mô hình Cross-Encoder dùng thư viện `sentence-transformers`, huấn luyện trên `soict-dataset-2024-segmented`, dùng để **rerank** kết quả từ `bi_encoder`. |

---

## ⚙️ Kiến trúc hệ thống (Tổng quan)

1. **Bi-Encoder**: Trích xuất các embedding vector từ câu hỏi và văn bản → tìm top-k kết quả gần nhất.
2. **Cross-Encoder**: Rerank lại các kết quả từ bước 1 để tăng độ chính xác.
3. **FAISS**: Tăng tốc truy vấn nhờ sử dụng indexing vector.
4. **Preprocessing**: Dữ liệu được segment để cải thiện chất lượng embedding.

---
## 📊 Kết quả đánh giá mô hình

### 🔍 Không xếp hạng lại (Bi-Encoder Only)

| Mô hình       | MAP    | MRR    | NDCG   | Recall@10 |
|--------------|--------|--------|--------|-----------|
| XLM-RoBERTa  | 0.2724 | 0.2839 | 0.2762 | 0.5018    |
| **PhoBERT**  | **0.4707** | **0.4900** | **0.4772** | **0.7223** |

> **Nhận xét**: PhoBERT vượt trội hơn XLM-RoBERTa trên tất cả các chỉ số, thể hiện lợi thế của việc sử dụng mô hình đơn ngôn ngữ được thiết kế riêng cho tiếng Việt. Recall@10 của PhoBERT đạt 0.7223, cho thấy tài liệu đúng được truy xuất trong top 10 kết quả cho 72.23% các truy vấn.

---

### 🔁 Có xếp hạng lại (Bi-Encoder + Cross-Encoder)

| Mô hình       | MAP    | MRR    | NDCG   | Recall@10 |
|--------------|--------|--------|--------|-----------|
| XLM-RoBERTa  | 0.3558 | 0.3613 | 0.3499 | 0.5545    |
| **PhoBERT**  | **0.5598** | **0.5766** | **0.5655** | **0.7714** |

> **Nhận xét**: Việc xếp hạng lại bằng Cross-Encoder cải thiện hiệu suất của cả hai mô hình. Đặc biệt với PhoBERT, Recall@10 tăng từ 0.7223 lên 0.7714, minh chứng rõ ràng cho hiệu quả của việc rerank kết quả truy xuất. Các chỉ số MAP, MRR và NDCG cũng tăng lên, cho thấy Cross-Encoder giúp phân biệt tốt hơn giữa các tài liệu có mức độ tương đồng cao.


## 🚀 Hướng phát triển tiếp theo

- Tích hợp giao diện người dùng (UI/UX) cho chatbot.
- Triển khai RESTful API hoặc gRPC cho backend.
- Huấn luyện thêm với dữ liệu thực tế từ người dùng.

---


