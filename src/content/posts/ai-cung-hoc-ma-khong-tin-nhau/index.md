---
title: "Khi các AI cùng học nhưng không hoàn toàn tin nhau: bức tranh tổng thể về DFL, Differential Privacy và Game Theory"
published: 2026-10-29T12:00:00+07:00
description: "Bức tranh tổng thể: học liên kết phi tập trung, Differential Privacy và lý thuyết trò chơi khớp với nhau thế nào khi các bên không tin nhau."
image: ""
tags: ["Federated Learning", "Differential Privacy", "Game Theory"]
category: "Research"
draft: false
lang: "vi"
---

**Khi nhiều hệ thống AI cùng học với nhau nhưng không ai hoàn toàn tin ai, thì mỗi bên nên chọn mức bảo vệ dữ liệu thế nào, mà việc học chung vẫn hiệu quả?** Càng tìm hiểu, tôi càng thấy câu hỏi này nằm ở giao điểm của ba lĩnh vực, mỗi lĩnh vực trả lời một phần: học liên kết phi tập trung (DFL), quyền riêng tư vi phân (Differential Privacy, DP) và lý thuyết trò chơi (Game Theory).

Đây là bài mở đầu, cũng là tấm bản đồ, của series *When AI Learns Together: Privacy, Trust & Game Theory*. Tôi là một kỹ sư, và series này là những gì tôi đã tìm hiểu được, viết lại cho những người làm phần mềm chưa từng đụng tới DP hay lý thuyết trò chơi.

:::note[Về chữ "thương lượng"]
Trong bài tôi đôi khi nói các node (nút) "thương lượng" về mức riêng tư. Đó chỉ là **phép ẩn dụ**. Các node không bàn bạc hay "quyết định" như con người; chúng chỉ thực thi các quy tắc chọn chiến lược và cập nhật đã được định nghĩa trước. Điểm "giống thương lượng" là lựa chọn của mỗi bên phụ thuộc vào lựa chọn của các bên khác.
:::

## Phần I: Ba thách thức

Ví dụ xuyên suốt series: 5 bệnh viện (Bệnh viện A–E) cùng huấn luyện một mô hình phát hiện bệnh mà không gửi hồ sơ bệnh nhân ra ngoài.

### DFL không xóa bỏ rủi ro rò rỉ

Học liên kết (Federated Learning) cho mỗi bên huấn luyện cục bộ rồi chỉ chia sẻ bản cập nhật mô hình. FedAvg (McMahan et al., 2017) là ví dụ kinh điển: server lấy trung bình các bản cập nhật. DFL bỏ luôn server, các node trao đổi trực tiếp với hàng xóm (Lian et al., 2017).

Nhưng không gửi dữ liệu gốc chưa có nghĩa là an toàn. Tấn công đảo ngược gradient (gradient inversion) cho thấy trong một số điều kiện, người nhận gradient có thể tái tạo khá sát dữ liệu đầu vào (Zhu, Liu & Han, 2019; Geiping et al., 2020). Mức thành công phụ thuộc vào mô hình, thứ được chia sẻ (gradient một bước hay cập nhật nhiều bước), kích thước batch và giả định tấn công. Trong DFL, mọi hàng xóm đều thấy bản cập nhật của bạn, nên chỉ cần một hàng xóm trung thực nhưng tò mò (honest-but-curious) là đã có rủi ro.

### DP tạo ra một sự đánh đổi

DP (Dwork et al., 2006; Dwork & Roth, 2014) cho một bảo đảm **định lượng**: đầu ra gần như không đổi dù có hay không một bản ghi cụ thể. Trong học sâu, DP-SGD (Abadi et al., 2016) cắt gradient của từng mẫu về ngưỡng $C$ rồi cộng nhiễu Gauss với độ lệch chuẩn $\sigma C$.

Ngân sách riêng tư ε càng nhỏ thì bảo đảm càng mạnh và, nếu các yếu tố khác giữ nguyên, cần càng nhiều nhiễu. Nhiều nhiễu thường làm mô hình học kém đi: đó là đánh đổi giữa riêng tư và độ hữu ích.

:::warning[Cộng nhiễu không phải an toàn tuyệt đối]
Bảo đảm của DP chỉ có nghĩa khi dùng đúng cơ chế kèm kế toán riêng tư (privacy accounting). Hệ số nhiễu σ và ε cuối cùng không phải ánh xạ một-một: số vòng, tỉ lệ lấy mẫu và loại accountant (ví dụ Rényi DP, Mironov 2017) đều ảnh hưởng tới ε.
:::

### Các bên không giống nhau

Hãy giả sử A ít dữ liệu nhưng rất nhạy cảm, B nhiều dữ liệu và muốn mô hình chính xác nhất, C có máy yếu nên muốn giảm chi phí. Dữ liệu giữa các bên lại thường không đồng nhất (Non-IID) (Kairouz et al., 2021). Một ε cố định cho tất cả, ở mọi tình huống, có vẻ không hợp lý.

Hơn nữa, mô hình chung tổng hợp từ bản cập nhật của mọi người: nếu A cộng thật nhiều nhiễu, B và C cũng chịu ảnh hưởng. Lợi ích của mỗi node phụ thuộc vào lựa chọn của node khác, và đó là lúc lý thuyết trò chơi xuất hiện.

## Phần II: Ba ý tưởng khớp với nhau thế nào

Khi áp dụng lý thuyết trò chơi vào quyền riêng tư trong học cộng tác, có một khuôn mô hình hóa chung khá hay gặp. Tôi xem nó như một **lăng kính tư duy**, không phải phương pháp cụ thể của ai:

| Thành phần | Trong hệ thống DFL |
|---|---|
| Người chơi | Các node trong mạng DFL |
| Chiến lược | Các mức ngân sách riêng tư ε mà node có thể chọn |
| Lợi ích (payoff) | Lợi ích từ mô hình chung, trừ tổn thất riêng tư và chi phí |
| Phúc lợi xã hội | Lợi ích toàn mạng |
| Động lực | Cách chiến lược thay đổi qua tương tác giữa các node |

Cân bằng Nash (Nash, 1950) là trạng thái không node nào muốn đơn phương đổi chiến lược. Lý thuyết trò chơi tiến hóa (EGT) nhìn theo hướng khác: chiến lược có lợi ích cao hơn trung bình sẽ lan rộng dần. Một mô hình phổ biến là động lực học sao chép (replicator dynamics) (Taylor & Jonker, 1978):

$$
\dot{x}_i = x_i \left( f_i(x) - \bar{f}(x) \right)
$$

với $x_i$ là tỉ lệ dùng chiến lược $i$, $f_i(x)$ là lợi ích của nó, $\bar{f}(x)$ là lợi ích trung bình. Chiến lược ổn định tiến hóa (ESS) (Maynard Smith & Price, 1973) là chiến lược mà khi đa số đã dùng, một nhóm nhỏ "đột biến" không thể xâm chiếm.

Ghép ba mảnh lại, một vòng học nhìn từ node A trông như sau:

```text
 vòng t:  chọn ε_A theo chiến lược hiện tại          (Game Theory)
            │
            ▼
          huấn luyện cục bộ + clip, cộng nhiễu       (DP)
            │
            ▼
          trao đổi bản cập nhật với hàng xóm B, E    (DFL)
            │
            ▼
          lợi ích = độ chính xác − tổn thất riêng tư − chi phí
            │
            ▼
          cập nhật chiến lược theo lợi ích của mình
          và hành vi của hàng xóm  ───►  vòng t+1
```

Điểm mấu chốt: đây là **tương tác chiến lược** (strategic interaction), không phải một lịch trình đổi ε theo thời gian kiểu "vòng đầu ε lớn, vòng sau ε nhỏ". Lựa chọn tốt nhất của mỗi node phụ thuộc vào lợi ích của chính nó **và** lựa chọn của các node khác.

Cũng cần nói rõ: không có gì bảo đảm các node sẽ đạt cân bằng Nash hay ESS. Cân bằng có tồn tại, có ổn định, hệ có hội tụ hay không đều phụ thuộc vào thiết kế trò chơi và quy tắc cập nhật.

## Phần III: Những ý tưởng như vậy thường được đánh giá thế nào?

Khi đọc các công trình theo hướng này, tôi thấy cách đánh giá thường xoay quanh bốn nhóm chỉ số:

| Nhóm chỉ số | Câu hỏi |
|---|---|
| Chất lượng mô hình | Mô hình chung chính xác đến đâu? |
| Bảo đảm riêng tư | ε (và δ) thực sự đạt được là bao nhiêu, tính qua kế toán riêng tư? |
| Chi phí | Tốn bao nhiêu tính toán và truyền thông? |
| Hành vi chiến lược | Chiến lược thay đổi ra sao, có ổn định hay hội tụ không? |

Điều đáng chú ý là không thể chỉ nhìn một chỉ số: một mô hình rất chính xác nhưng ε lớn, hay ε rất nhỏ nhưng mô hình gần như vô dụng, đều chưa phải câu trả lời tốt.

## Phần IV: Những câu hỏi còn mở

:::note[Câu hỏi còn mở]
- **Chiến lược có hội tụ không**, hay các node dao động mãi giữa các mức ε?
- **Lợi ích cá nhân có mâu thuẫn với lợi ích toàn mạng không?** Một node có thể cộng thật nhiều nhiễu để tự bảo vệ mà vẫn hưởng mô hình chung do người khác đóng góp (free-riding).
- **Dữ liệu không đồng nhất (Non-IID)** làm lợi ích của từng node lệch nhau thế nào?
- **Nếu có node không trung thực thì sao?** (ngoài trọng tâm series, nhưng rất thực tế)
- **Một cách chọn ε có tốt hơn cách khác một cách bền vững không**, hay chỉ trong vài cấu hình?
:::

## Lộ trình series

1. [AI có thể học từ dữ liệu của bạn mà không cần nhìn thấy dữ liệu đó không?](/posts/ai-hoc-tu-du-lieu-ma-khong-nhin-thay/): từ học tập trung đến học liên kết và DFL.
2. [Không chia sẻ dữ liệu gốc, liệu thông tin cá nhân đã thực sự an toàn?](/posts/khong-chia-se-du-lieu-da-an-toan-chua/): gradient rò rỉ thông tin thế nào, DP giúp gì.
3. [Quyền riêng tư có giá bao nhiêu?](/posts/quyen-rieng-tu-co-gia-bao-nhieu/): ngân sách ε và đánh đổi riêng tư – độ hữu ích.
4. [Khi các AI trở thành những người chơi](/posts/khi-ai-tro-thanh-nguoi-choi/): cân bằng Nash, EGT và động lực học sao chép.

## Lời kết

Lý thuyết trò chơi không tự động tốt hơn các cách tiếp cận khác. Nhưng nó là một **lăng kính hữu ích** để suy nghĩ về quyền riêng tư khi mỗi bên tham gia có lợi ích và điều kiện khác nhau, và khi lựa chọn của bên này ảnh hưởng tới tất cả các bên còn lại.

**Bài tiếp theo** trong series *When AI Learns Together: Privacy, Trust & Game Theory*: [bài 1 về học liên kết và DFL](/posts/ai-hoc-tu-du-lieu-ma-khong-nhin-thay/).

## Tài liệu tham khảo

- Abadi, M., et al. (2016). *Deep Learning with Differential Privacy*.
- Dwork, C., McSherry, F., Nissim, K., & Smith, A. (2006). *Calibrating Noise to Sensitivity in Private Data Analysis*.
- Dwork, C., & Roth, A. (2014). *The Algorithmic Foundations of Differential Privacy*.
- Geiping, J., et al. (2020). *Inverting Gradients – How easy is it to break privacy in federated learning?*
- Kairouz, P., et al. (2021). *Advances and Open Problems in Federated Learning*.
- Lian, X., et al. (2017). *Can Decentralized Algorithms Outperform Centralized Algorithms? A Case Study for Decentralized Parallel Stochastic Gradient Descent*.
- Maynard Smith, J., & Price, G. R. (1973). *The Logic of Animal Conflict*.
- McMahan, H. B., et al. (2017). *Communication-Efficient Learning of Deep Networks from Decentralized Data*.
- Mironov, I. (2017). *Rényi Differential Privacy*.
- Nash, J. (1950). *Equilibrium Points in n-Person Games*.
- Taylor, P. D., & Jonker, L. B. (1978). *Evolutionary Stable Strategies and Game Dynamics*.
- Zhu, L., Liu, Z., & Han, S. (2019). *Deep Leakage from Gradients*.
