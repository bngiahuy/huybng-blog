---
title: "Quyền riêng tư có giá bao nhiêu? Bài toán đánh đổi giữa Privacy và Accuracy"
published: 2026-10-29T09:00:00+07:00
description: "Ngân sách riêng tư ε là gì, nhiễu Gaussian và DP-SGD ảnh hưởng đến việc học ra sao, và vì sao một mức ε cố định cho mọi node là chưa đủ."
image: ""
tags: ["Differential Privacy", "Privacy Budget", "Privacy-Utility Trade-off"]
category: "Research"
draft: false
lang: "vi"
---

Hãy tưởng tượng một thanh trượt quyết định AI được phép "để lộ" bao nhiêu về dữ liệu của bạn. Kéo về một phía, dữ liệu được bảo vệ chặt hơn nhưng AI học kém đi. Kéo về phía kia, AI học tốt hơn nhưng thông tin của bạn dễ bị suy ra hơn. Bạn sẽ đặt nó ở đâu?

Đây là bài 3 trong series *When AI Learns Together: Privacy, Trust & Game Theory*. Ở [bài trước](/posts/khong-chia-se-du-lieu-da-an-toan-chua/), tôi đã cho thấy gradient có thể làm lộ dữ liệu, và quyền riêng tư vi phân (Differential Privacy, DP) giải quyết bằng cách cắt (clip) và thêm nhiễu được hiệu chỉnh. Nhưng thêm bao nhiêu nhiễu? Câu trả lời nằm ở con số ε. Nếu bạn mới vào series, hãy đọc bài [tổng quan](/posts/ai-cung-hoc-ma-khong-tin-nhau/) trước.

Ví dụ quen thuộc: năm bệnh viện A–E cùng huấn luyện một mô hình phát hiện bệnh theo kiểu học liên kết phi tập trung (DFL), mỗi bệnh viện là một node (nút) chỉ trao đổi bản cập nhật mô hình với hàng xóm.

## Ngân sách riêng tư ε

Định nghĩa DP (Dwork, McSherry, Nissim & Smith, 2006; Dwork & Roth, 2014) xét hai tập dữ liệu "hàng xóm" $D$ và $D'$ chỉ khác nhau một bản ghi, chẳng hạn hồ sơ của một bệnh nhân. Cơ chế ngẫu nhiên $M$ thỏa mãn $(\varepsilon, \delta)$-DP nếu với mọi tập kết quả $S$:

$$
\Pr[M(D) \in S] \;\le\; e^{\varepsilon}\,\Pr[M(D') \in S] + \delta
$$

Nói cách khác: nhìn vào đầu ra, kẻ quan sát gần như không phân biệt được bệnh nhân đó có trong dữ liệu hay không. ε đo mức "gần như" ấy.

- **ε nhỏ**: hai phân phối đầu ra rất giống nhau, đảm bảo riêng tư **mạnh**.
- **ε lớn**: đảm bảo **yếu** hơn.
- **δ**: xác suất nhỏ mà đảm bảo trên được phép "thất bại"; thường chọn nhỏ hơn nhiều so với $1/n$ ($n$ là số bản ghi).

Gọi là **ngân sách riêng tư ε** vì mỗi lần cơ chế "chạm" vào dữ liệu, ta tiêu một phần ngân sách. Huấn luyện nhiều vòng thì mức rò rỉ cộng dồn, và việc tính tổng đó gọi là kế toán riêng tư (privacy accounting).

:::warning[ε không phải là "lượng nhiễu"]
ε là **mức đảm bảo**; nhiễu là **công cụ** để đạt nó. Với cơ chế Gaussian, khi các yếu tố khác giữ nguyên, muốn ε nhỏ hơn (riêng tư mạnh hơn) thì phải thêm nhiều nhiễu hơn.
:::

## Cơ chế Gaussian

Cách đơn giản nhất để đạt DP là cộng nhiễu Gaussian vào kết quả của hàm $f$. Lượng nhiễu phụ thuộc **độ nhạy** $\Delta_2 f$: kết quả thay đổi tối đa bao nhiêu (chuẩn L2) khi thêm hay bớt một bản ghi. Theo Dwork & Roth (2014), với $\varepsilon \in (0, 1)$, cơ chế

$$
M(D) = f(D) + \mathcal{N}(0, s^2 I), \qquad s \;\ge\; \frac{\Delta_2 f \,\sqrt{2\ln(1.25/\delta)}}{\varepsilon}
$$

thỏa mãn $(\varepsilon, \delta)$-DP. Trực giác: **ε giảm thì nhiễu tăng**, **độ nhạy lớn thì nhiễu cũng phải lớn**. Vì vậy ta cần chặn độ nhạy, và đó là việc của clipping.

## DP-SGD: nhiễu trong quá trình huấn luyện

Abadi et al. (2016) đề xuất DP-SGD. Ở mỗi bước, với batch $B$ được lấy mẫu ngẫu nhiên theo tỉ lệ $q$:

1. Tính **gradient riêng cho từng mẫu** $g_i$.
2. **Clip** từng gradient về chuẩn tối đa $C$: $\bar g_i = g_i \cdot \min\left(1, C / \lVert g_i \rVert_2\right)$. Mỗi bệnh nhân chỉ ảnh hưởng tối đa $C$ lên tổng.
3. **Cộng nhiễu** tỉ lệ với $C$, với hệ số nhiễu σ, rồi cập nhật mô hình:

$$
\tilde g = \frac{1}{|B|}\left(\sum_{i \in B} \bar g_i + \mathcal{N}(0, \sigma^2 C^2 I)\right)
$$

Bài báo cũng giới thiệu *moments accountant* để theo dõi tổng ε qua rất nhiều bước; ý tưởng này sau được hệ thống hóa qua Rényi Differential Privacy (Mironov, 2017), nền tảng của các "RDP accountant" hiện nay.

Vì sao mô hình học kém đi? Clipping làm méo gradient của các mẫu "khác thường", còn nhiễu làm hướng đi mỗi bước kém chính xác. Mô hình vẫn học được, nhưng thường chậm hơn và kém chính xác hơn. Đó là **đánh đổi giữa riêng tư và độ hữu ích** (privacy–utility trade-off).

## ε và σ không phải quan hệ một-đối-một

:::caution[Lưu ý quan trọng]
Cùng một hệ số nhiễu σ có thể cho ra ε cuối cùng rất khác nhau. ε phụ thuộc đồng thời vào σ, **tỉ lệ lấy mẫu** $q$, **số bước/số vòng**, δ, và **phương pháp kế toán riêng tư** (các accountant khác nhau cho cận khác nhau). Không thể nói "σ = X thì ε = Y" nếu không nêu đủ các yếu tố này.
:::

Đoạn code dưới đây dùng `RDPAccountant` của Opacus: giữ nguyên σ, chỉ đổi số bước và tỉ lệ lấy mẫu. Bạn có thể tự chạy sau khi `pip install opacus`; tôi cố ý không ghi kết quả để bạn tự quan sát.

```python
from opacus.accountants import RDPAccountant

def epsilon_after(noise_multiplier, sample_rate, steps, delta=1e-5):
    acc = RDPAccountant()
    for _ in range(steps):
        acc.step(noise_multiplier=noise_multiplier, sample_rate=sample_rate)
    return acc.get_epsilon(delta=delta)

sigma = 1.0  # giữ nguyên hệ số nhiễu
for q in (0.01, 0.05):
    for steps in (100, 1000, 5000):
        print(q, steps, epsilon_after(sigma, q, steps))
```

Điều nên để ý: với σ cố định, càng nhiều bước hoặc $q$ càng lớn thì ε càng tăng, tức ngân sách bị tiêu nhiều hơn.

## Tam giác đánh đổi

Thanh trượt ở đầu bài thực ra có ba đỉnh:

```text
              Riêng tư (ε nhỏ)
                   /\
                  /  \
                 /____\
   Độ chính xác          Chi phí tính toán
                         và truyền thông
```

Riêng tư hơn thì thêm nhiễu, mô hình kém chính xác hơn. Muốn bù độ chính xác thì huấn luyện nhiều vòng hơn, tốn chi phí hơn, và nhiều vòng lại tiêu thêm ngân sách. Clipping theo từng mẫu cũng tốn thêm tính toán.

### Playground ba thanh trượt (bản tĩnh)

:::warning[Chỉ mang tính minh họa]
Bảng dưới **không phải số liệu đo đạc**, chỉ là xu hướng định tính khi kéo một thanh trượt và giữ nguyên các yếu tố khác.
:::

| Thanh trượt | Riêng tư | Độ chính xác | Chi phí |
|---|---|---|---|
| ε mục tiêu ↓ | ↑ cao | ↓ thường giảm | ↑ có thể cần thêm vòng để bù |
| ε mục tiêu ↑ | ↓ thấp | ↑ thường tăng | ↓ thấp |
| σ ↑ (số bước, $q$ giữ nguyên) | ↑ cao (ε giảm) | ↓ thường giảm | gần như không đổi |
| σ ↓ | ↓ thấp (ε tăng) | ↑ thường tăng | gần như không đổi |
| Số node ↑ | ε mỗi node gần như không đổi | ↑ có thể tăng, nhiễu độc lập được trung bình bớt | ↑ trung bình → cao |
| Số node ↓ | ε mỗi node gần như không đổi | ↓ có thể giảm | ↓ thấp |

:::tip[Demo tương tác]
Một phiên bản tương tác với ba thanh trượt (ε, σ, số node) đang được lên kế hoạch.
:::

## Vì sao một ε cho tất cả là chưa ổn?

Cách phổ biến là chọn một cấu hình DP chung cho mọi node. Đơn giản, dễ kiểm toán. Nhưng hãy nhìn năm bệnh viện:

- **Kích thước dữ liệu khác nhau.** A có rất nhiều hồ sơ, E chỉ vài trăm. Cùng σ nhưng $q$ và số bước khác nhau thì ε thực tế cũng khác.
- **Độ nhạy cảm khác nhau.** B chuyên một bệnh hiếm, một bản ghi có thể rất dễ nhận diện.
- **Năng lực tính toán khác nhau.** D có cụm GPU, E chỉ có một máy chủ nhỏ, khó chạy thêm vòng để bù nhiễu.
- **Mức hưởng lợi khác nhau.** Node ít dữ liệu được lợi nhiều từ mô hình chung; node lớn có thể ít cần hơn.

Ép một ε chung giống bắt mọi người mặc cùng một cỡ áo.

### ε cố định và ε như một chiến lược

1. **ε cố định:** tham số đặt trước, không phụ thuộc các node khác.
2. **ε như một chiến lược:** mỗi node chọn mức riêng tư dựa trên lợi ích (payoff) của chính nó, và lợi ích đó **phụ thuộc vào lựa chọn của các node khác**: hàng xóm thêm nhiều nhiễu thì mô hình tổng hợp kém đi, và tôi cũng chịu ảnh hưởng.

Cách thứ hai không phải "đổi ε theo một lịch thời gian". Cốt lõi là **tương tác chiến lược**: lựa chọn tốt nhất của mỗi node phụ thuộc vào lợi ích của nó và vào lựa chọn của các node khác. Cũng cần nói rõ: các node không "quyết định" như con người, chúng chỉ thực thi các quy tắc chọn và cập nhật chiến lược được định nghĩa sẵn. Đây chính là góc nhìn mà lý thuyết trò chơi mang lại, chủ đề của bài sau.

:::note[Câu hỏi còn mở]
Nên định nghĩa lợi ích của một node thế nào để cân đúng độ chính xác, mức rò rỉ riêng tư và chi phí? Khi mỗi node tự chọn ε, lợi ích riêng có xung đột với lợi ích toàn mạng không, và có node nào "ăn theo" (free-riding) bằng cách thêm thật nhiều nhiễu trong khi vẫn hưởng mô hình chung? Với dữ liệu không đồng nhất (Non-IID), bức tranh đánh đổi thay đổi ra sao? Đây là những câu hỏi cộng đồng nghiên cứu vẫn đang tìm lời giải, chưa có câu trả lời chung cho mọi trường hợp.
:::

## Kết

Quyền riêng tư có giá, trả bằng độ chính xác và chi phí. Nhưng câu hỏi không chỉ là *làm sao bảo vệ dữ liệu tốt nhất*, mà là *làm sao chọn mức bảo vệ phù hợp khi mỗi bên có lợi ích và điều kiện khác nhau*.

**Bài tiếp theo:** khi mỗi node trở thành một "người chơi", lý thuyết trò chơi giúp gì? Đón đọc [Khi các AI trở thành những người chơi](/posts/khi-ai-tro-thanh-nguoi-choi/).

**Trong series này:** [Tổng quan](/posts/ai-cung-hoc-ma-khong-tin-nhau/) · [Bài 1](/posts/ai-hoc-tu-du-lieu-ma-khong-nhin-thay/) · [Bài 2](/posts/khong-chia-se-du-lieu-da-an-toan-chua/) · Bài 3 (bài này) · [Bài 4](/posts/khi-ai-tro-thanh-nguoi-choi/)

## Tài liệu tham khảo

- Dwork, C., McSherry, F., Nissim, K., & Smith, A. (2006). *Calibrating Noise to Sensitivity in Private Data Analysis*.
- Dwork, C., & Roth, A. (2014). *The Algorithmic Foundations of Differential Privacy*.
- Abadi, M., et al. (2016). *Deep Learning with Differential Privacy*.
- Mironov, I. (2017). *Rényi Differential Privacy*.
- Opacus, thư viện PyTorch cho huấn luyện với Differential Privacy (module `opacus.accountants`).
