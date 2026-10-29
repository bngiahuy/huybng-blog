---
title: "AI có thể học từ dữ liệu của bạn mà không cần nhìn thấy dữ liệu đó không?"
published: 2026-10-29T11:00:00+07:00
description: "Từ học máy tập trung đến học liên kết (Federated Learning) và học liên kết phi tập trung (DFL), qua ví dụ 5 bệnh viện cùng huấn luyện một mô hình."
image: ""
tags: ["Federated Learning", "Decentralized Federated Learning", "Machine Learning"]
category: "Research"
draft: false
lang: "vi"
---

Hãy tưởng tượng 5 bệnh viện A, B, C, D và E cùng muốn xây dựng một mô hình AI phát hiện sớm một loại bệnh. Mỗi bệnh viện có quá ít hồ sơ để tự huấn luyện một mô hình tốt, nhưng gộp lại thì có thể đủ. Vấn đề là: không bệnh viện nào được phép gửi hồ sơ bệnh nhân sang bệnh viện khác, hay lên một máy chủ chung.

Vậy họ có thể hợp tác được không?

Đây là bài đầu tiên (sau [bài tổng quan](/posts/ai-cung-hoc-ma-khong-tin-nhau/)) trong series *When AI Learns Together: Privacy, Trust & Game Theory*, nơi tôi chia sẻ những gì mình tìm hiểu được về chủ đề này. Hôm nay ta sẽ đi qua ba cách để 5 bệnh viện "học cùng nhau".

## Cách truyền thống: gom hết dữ liệu về một chỗ

Cách quen thuộc là: cả 5 bệnh viện gửi hồ sơ bệnh nhân về một trung tâm dữ liệu, rồi huấn luyện mô hình ở đó.

Về kỹ thuật, đây là cách đơn giản nhất. Nhưng thực tế có nhiều rào cản:

- **Pháp lý:** dữ liệu y tế được bảo vệ chặt chẽ, việc chuyển ra ngoài tổ chức rất khó.
- **Niềm tin:** bệnh viện A có thực sự muốn giao toàn bộ dữ liệu cho một bên thứ ba?
- **Rủi ro tập trung:** một kho chứa hồ sơ của cả 5 bệnh viện là mục tiêu hấp dẫn cho kẻ tấn công. Lộ một lần là lộ tất cả.

Câu hỏi tự nhiên: thay vì mang dữ liệu đến mô hình, ta có thể mang mô hình đến dữ liệu không?

## Học liên kết: mang mô hình đến dữ liệu

Đó chính là ý tưởng của **học liên kết (Federated Learning)**, được McMahan và cộng sự giới thiệu năm 2017 cùng thuật toán FedAvg:

1. Máy chủ trung tâm gửi mô hình hiện tại cho các bệnh viện.
2. Mỗi bệnh viện huấn luyện mô hình **trên dữ liệu của chính mình**, ngay tại chỗ, trong vài bước.
3. Mỗi bệnh viện chỉ gửi lại **bản cập nhật mô hình** (tham số mới hoặc phần thay đổi), không gửi hồ sơ bệnh nhân.
4. Máy chủ lấy trung bình các bản cập nhật, có trọng số theo lượng dữ liệu của từng bên.
5. Lặp lại nhiều vòng.

Giống như các đầu bếp không gửi nguyên liệu bí mật về bếp trung tâm, mà tự nấu thử ở nhà rồi chỉ gửi "ghi chú chỉnh công thức" để bếp trưởng tổng hợp.

Bước tổng hợp của FedAvg có dạng:

$$
w^{(t+1)} = \sum_{k=1}^{K} \frac{n_k}{n}\, w_k^{(t+1)}
$$

trong đó $w_k^{(t+1)}$ là tham số mô hình sau khi node (nút) $k$ huấn luyện cục bộ, $n_k$ là số mẫu dữ liệu của node đó và $n$ là tổng số mẫu. Bài khảo sát của Kairouz và cộng sự (2021) là điểm khởi đầu tốt để đọc sâu hơn.

## Nhưng vẫn còn một "ông chủ" ở giữa

Học liên kết cổ điển giữ được dữ liệu thô tại bệnh viện, nhưng vẫn dựa vào **một máy chủ trung tâm**, kéo theo vài vấn đề:

- **Điểm lỗi duy nhất (single point of failure):** máy chủ sập thì cả hệ thống dừng học.
- **Niềm tin:** máy chủ nhìn thấy bản cập nhật của tất cả mọi người. Trong nhiều tình huống, không có bên nào đủ trung lập để cả 5 bệnh viện cùng tin.
- **Nút thắt cổ chai:** mọi bản cập nhật đổ về một chỗ. Khi số node tăng, băng thông và tải xử lý của máy chủ trở thành giới hạn.

Vậy nếu bỏ luôn máy chủ trung tâm thì sao?

## Học liên kết phi tập trung: không cần ai đứng giữa

**Học liên kết phi tập trung (DFL)** loại bỏ máy chủ trung tâm. Các node kết nối ngang hàng (peer-to-peer) thành một mạng lưới (topology), và mỗi node chỉ trao đổi với **các node láng giềng** của mình. Trong mỗi vòng, một node:

1. **Huấn luyện cục bộ** trên dữ liệu của mình.
2. **Trao đổi và lấy trung bình** tham số với các láng giềng.

Kiểu trao đổi này còn gọi là *gossip*: như tin đồn trong khu phố, mỗi người chỉ kể cho người bên cạnh, nhưng sau nhiều lượt cả khu phố đều biết. Tương tự, sau nhiều vòng, những gì mô hình của bệnh viện A học được dần lan (gián tiếp, qua tham số) tới bệnh viện D, dù A và D không bao giờ trao đổi trực tiếp.

Lian và cộng sự (2017) đã phân tích một thuật toán kiểu này (D-PSGD) và chỉ ra rằng, trong những điều kiện nhất định, huấn luyện phi tập trung có thể cạnh tranh với huấn luyện tập trung mà tránh được nút thắt ở máy chủ. Chữ "điều kiện nhất định" là quan trọng: tốc độ hội tụ phụ thuộc vào cấu trúc mạng, mức độ khác biệt dữ liệu giữa các node và nhiều yếu tố khác.

### Ba cách học, một bức tranh

```text
(1) TẬP TRUNG (Centralized): dữ liệu thô rời bệnh viện

   [A]   [B]   [C]   [D]   [E]
    │     │     │     │     │      dữ liệu thô
    ▼     ▼     ▼     ▼     ▼
  ┌─────────────────────────────┐
  │ Máy chủ: lưu toàn bộ dữ liệu │
  │ và huấn luyện mô hình        │
  └─────────────────────────────┘

(2) HỌC LIÊN KẾT (Federated): chỉ bản cập nhật rời bệnh viện

   [A]   [B]   [C]   [D]   [E]     (dữ liệu ở lại tại chỗ)
    ▲│    ▲│    ▲│    ▲│    ▲│      ▼ bản cập nhật gửi lên
    │▼    │▼    │▼    │▼    │▼      ▲ mô hình chung gửi xuống
  ┌─────────────────────────────┐
  │ Máy chủ: lấy trung bình     │
  │ các bản cập nhật            │
  └─────────────────────────────┘

(3) PHI TẬP TRUNG (Decentralized, DFL): không có máy chủ

              [A]
            ╱     ╲
         [E]       [B]
          │         │
         [D] ───── [C]

  Mạng vòng: A↔B, B↔C, C↔D, D↔E, E↔A
  Láng giềng của A là B và E; A không trao đổi trực tiếp với C, D.
```

Ở cả (2) và (3), dữ liệu thô không rời bệnh viện. Khác biệt nằm ở chỗ ai nhận bản cập nhật: một máy chủ duy nhất, hay các láng giềng.

### Một vòng DFL trông như thế nào?

Mã giả dưới đây theo lối đơn giản nhất: mỗi node huấn luyện vài bước, rồi lấy trung bình đều với láng giềng.

```python
# Mã giả: một vòng DFL
# neighbors[i]: danh sách láng giềng của node i

def dfl_round(models, datasets, neighbors, local_steps, lr):
    # Bước 1: huấn luyện cục bộ, dữ liệu không rời node
    local = {}
    for i in models:
        w = models[i].copy()
        for _ in range(local_steps):
            batch = datasets[i].sample_batch()
            w = w - lr * gradient(w, batch)
        local[i] = w

    # Bước 2: trao đổi tham số, lấy trung bình với láng giềng
    new_models = {}
    for i in models:
        group = [i] + neighbors[i]
        new_models[i] = sum(local[j] for j in group) / len(group)
    return new_models
```

Thực tế, trọng số trung bình thường được thiết kế theo cấu trúc mạng (một "ma trận trộn") để các node dần tiến về cùng một mô hình. Tinh thần vẫn là: **học tại chỗ, rồi hòa trộn với hàng xóm**.

## Nếu một node "rớt mạng" hoặc gửi sai?

Không có trung tâm thì không có điểm lỗi duy nhất: nếu bệnh viện C tạm mất kết nối, các bệnh viện còn lại vẫn tiếp tục học với những láng giềng còn hoạt động. Tuy nhiên, DFL cũng có thách thức riêng:

- Nếu mạng bị chia cắt (ví dụ cả B và D cùng rớt thì C bị cô lập), thông tin không lan được giữa các phần.
- Nếu một node gửi tham số sai, do lỗi hay cố ý, sai lệch đó có thể lan qua láng giềng. Các node cố ý gửi thông tin sai thường được gọi là node Byzantine, và có cả một hướng nghiên cứu riêng về cách chống chịu với chúng.

:::note
Vấn đề node Byzantine nằm ngoài trọng tâm của series này. Từ đây trở đi, series giả định các node tuân thủ giao thức, nhưng có thể *tò mò* về dữ liệu của người khác. Bài sau sẽ nói rõ hơn về giả định này.
:::

## Nền móng cho cả series

DFL với 5 bệnh viện ngang hàng, không có máy chủ, mỗi node chỉ trao đổi với láng giềng, là bối cảnh mà các bài sau trong series sẽ dựa vào.

Khi không ai đứng giữa điều phối, cũng không có bên nào áp đặt một quy tắc chung cho tất cả. Trong khi đó, mỗi bệnh viện lại có dữ liệu, tài nguyên và mức độ nhạy cảm khác nhau.

:::note[Câu hỏi còn mở]
Trong DFL, khi các node có dữ liệu không đồng nhất (Non-IID), tài nguyên và mục tiêu khác nhau, làm sao để việc học chung vẫn hiệu quả và công bằng với từng bên? Đây là những vấn đề mà bài khảo sát của Kairouz và cộng sự (2021) liệt kê là còn nhiều hướng chưa được giải quyết trọn vẹn.
:::

## Kết lại

DFL cho phép nhiều tổ chức cùng huấn luyện AI mà không cần gom dữ liệu về một chỗ, cũng không phụ thuộc vào một máy chủ duy nhất. Với 5 bệnh viện của chúng ta, câu trả lời là: **có, họ có thể hợp tác**, mỗi bên giữ dữ liệu tại chỗ và chỉ chia sẻ bản cập nhật mô hình với láng giềng.

Nhưng **không chia sẻ dữ liệu gốc không có nghĩa là quyền riêng tư đã được đảm bảo**. Những con số mà mô hình học được vẫn mang dấu vết của dữ liệu đã tạo ra chúng.

:::tip[Bài tiếp theo]
Trong [bài 2](/posts/khong-chia-se-du-lieu-da-an-toan-chua/), ta sẽ xem vì sao bản cập nhật mô hình có thể làm lộ thông tin, và quyền riêng tư vi phân (Differential Privacy, DP) giúp được gì.
:::

**Trong series *When AI Learns Together: Privacy, Trust & Game Theory*:**

- [Tổng quan: Khi các AI cùng học nhưng không hoàn toàn tin nhau](/posts/ai-cung-hoc-ma-khong-tin-nhau/)
- [Bài 2: Không chia sẻ dữ liệu gốc đã an toàn chưa?](/posts/khong-chia-se-du-lieu-da-an-toan-chua/)

## Tài liệu tham khảo

- McMahan, H. B., Moore, E., Ramage, D., Hampson, S., & Agüera y Arcas, B. (2017). *Communication-Efficient Learning of Deep Networks from Decentralized Data*. AISTATS.
- Kairouz, P., McMahan, H. B., et al. (2021). *Advances and Open Problems in Federated Learning*. Foundations and Trends in Machine Learning.
- Lian, X., Zhang, C., Zhang, H., Hsieh, C.-J., Zhang, W., & Liu, J. (2017). *Can Decentralized Algorithms Outperform Centralized Algorithms? A Case Study for Decentralized Parallel Stochastic Gradient Descent*. NeurIPS.
