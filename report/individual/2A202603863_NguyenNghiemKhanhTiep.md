# Báo cáo cá nhân — Day 01: Lọc tin nhắn rác bằng ML, DL, Embedding và LLM

> Đổi tên file này thành `MSSV_HoTen.md` của bạn, ví dụ `2A20260000_NguyenVanAn.md`, rồi điền từng mục. Xoá các dòng hướng dẫn in nghiêng khi xong.

- **MSSV:**
- **Họ và tên:**
- **SEED** (in ở ô đầu notebook):457

## 1. Bằng chứng

| Tệp | Có? |
| --- | --- |
| [`day01_lab.ipynb`](day01_lab.ipynb) — notebook đã chạy, còn output | ☑ |
| [`lab1_report.json`](lab1_report.json) — do ô "Báo cáo cuối" sinh ra | ☑ |
| [`hand_calc.jpg`](hand_calc.jpg) — ảnh tính tay có các bước | ☑ |

## 2. Kết quả chính

| Cách | F1 test | F1 SUB 40 | p95 ms/tin | USD/1k tin |
| --- | --- | --- | --- | --- |
| 1. BoW + LogReg | 0.908 | 0.974 | 0.08 | 0 |
| 2. BoW + MLP | 0.960 | 1.000 | 0.42 | 0 |
| 3a. Emb + kNN | ____ | 0.976 | ____ | 0 |
| 3b. Emb + LogReg | ____ | 1.000 | ____ | 0 |
| 3c. Emb + MLP head | ____ | 0.974 | ____ | 0 |
| 4a. LLM nhỏ zero-shot | — | 0.976 | ____ | ____ |
| 4b. LLM lớn zero-shot | — | ____ | ____ | ____ |
| 4c. LLM nhỏ few-shot | — | ____ | ____ | ____ |

Trên đường Pareto (1 − F1 so với p95): **BoW + LogReg** và **BoW + MLP**. Các LLM đều vượt SLA 50 ms.

## 3. Dự đoán và thực tế

1. **MLP so với LogReg (2.A).** Tôi đoán MLP xấp xỉ LogReg vì BoW đã gần như tách được tuyến tính. Thực tế MLP cao hơn rõ: F1 0.96 so với 0.908, recall 0.95 so với 0.83. Lý do lệch: lớp ẩn với ReLU kết hợp được nhiều từ yếu thành tín hiệu spam, và LogReg ở Phần 1 chưa hội tụ hết (loss cuối 0.085).
2. **lr = 50 (1.B).** Tôi đoán đường loss răng cưa và không hội tụ. Thực tế loss chỉ vọt lên khoảng 2.5 ở epoch đầu, sau đó hội tụ về 0.005, thấp nhất trong ba trường hợp. Lý do lệch: gradient của logistic regression bị chặn (|p − y| ≤ 1), BCE là hàm lồi, và dữ liệu gần như tách được, nên w lớn chỉ làm loss nhỏ hơn.
3. **Few-shot (4.B).** Tôi đoán F1 tăng nhẹ. Thực tế LLM nhỏ few-shot có 1 − F1 khoảng 4.8%, kém hơn zero-shot (khoảng 2.4%), trong khi chi phí lại tăng. Lý do lệch: 4 ví dụ ít ỏi có thể làm model bám theo bề mặt của ví dụ. Ngoài ra, SUB chỉ có 40 tin nên chênh 1 tin đã là khoảng 2.5 điểm %.


## 4. Phân tích 50 case khó

**BoW + LogReg** tụt nhiều nhất: F1 từ 0.974 (SUB) xuống 0.333. Nó yếu nhất ở nhóm *spam giọng tự nhiên* (đúng 10%) và *thao túng / lừa đảo* (đúng 12%). LLM tụt ít nhất (0.903–0.935).

- **Case 1:** *"Exclusive: get a brand new phone at no cost, just reply with your address"* là spam nhưng BoW đoán ham. Tin không có từ khóa spam nào như "free", "call", "txt", mà BoW chỉ cộng trọng số của từng token rời rạc. Encoder hiểu "at no cost" có nghĩa là "free" nên đoán đúng.
- **Case 2:** *"FREE pizza in the office kitchen right now, come quick before it's gone"* là ham nhưng LLM (kể cả có prompt phòng thủ) đoán spam. LLM bị mồi bởi "FREE" và giọng giục giã, không nhận ra đây là tin nội bộ vô hại.

## 5. Thử thách cá nhân

*Đề theo SEED của bạn, cách làm, kết quả, giải thích.*

## 6. Phương án cho sản phẩm thật

Tôi chọn **router hai tầng**. **BoW + MLP** xử lý mọi tin (F1 test 0.96, p95 khoảng 0.4 ms, khoảng 24 000 tin/giây, dư sức cho khoảng 116 tin/giây): p < 0.3 là ham, p > 0.7 là spam. Chỉ các tin lưỡng lự (p ∈ [0.3, 0.7]) mới được gửi sang LLM kèm prompt phòng thủ.

Nếu dùng LLM cho tất cả: 10 triệu tin × 10.50 USD / 1 triệu tin = 105 USD/ngày, khoảng **3 150 USD/tháng**, và không đạt SLA. Với router, giả sử khoảng 5% số tin sang LLM, chi phí còn khoảng **160 USD/tháng** và p95 vẫn dưới 50 ms.

Đánh đổi chấp nhận: các tin lưỡng lự sẽ chậm hơn, và BoW vẫn yếu với spam giọng tự nhiên (F1 case khó chỉ khoảng 0.44) nếu mô hình quá tự tin mà đoán sai.

## 7. AI log và điều đã học

*Mỗi lần dùng AI: hỏi gì, nhận gì, bạn sửa gì. Sau đó 2–3 câu: điều bạn hiểu ra hôm nay mà trước đó chưa hiểu.*
