---
title: "Không chia sẻ dữ liệu gốc, liệu thông tin cá nhân đã thực sự an toàn?"
published: 2026-10-29T10:00:00+07:00
description: "Gradient có thể làm lộ dữ liệu huấn luyện. Bài viết giải thích tấn công đảo ngược gradient và cách Differential Privacy giúp giới hạn rủi ro."
image: ""
tags: ["Federated Learning", "Privacy", "Gradient Inversion", "Differential Privacy"]
category: "Research"
draft: false
lang: "vi"
---

Bạn không bao giờ gửi bức ảnh của mình đi, chỉ gửi vài con số mà AI đã "học" được từ nó. Nghe có vẻ an toàn. Nhưng nếu người nhận dùng chính những con số ấy để đoán ngược lại bức ảnh thì sao?

Đây là bài số 2 trong series *When AI Learns Together: Privacy, Trust & Game Theory*. Ở [bài 1](/posts/ai-hoc-tu-du-lieu-ma-khong-nhin-thay/), 5 bệnh viện (A–E) cùng huấn luyện một mô hình phát hiện bệnh bằng học liên kết phi tập trung (DFL) mà không ai gửi hồ sơ bệnh nhân cho ai. Bài này hỏi tiếp: chỉ chia sẻ bản cập nhật mô hình thì đã đủ riêng tư chưa?

## Bản cập nhật mô hình là gì?

Giả sử bệnh viện A có một mô hình tuyến tính nhỏ: nhận vector $x$ (vài chỉ số xét nghiệm, hay các pixel của một ảnh X-quang thu nhỏ) và trả ra $y = Wx + b$. Mức sai của dự đoán được đo bằng hàm mất mát $L$.

Gradient cho biết: "nhích mỗi tham số một chút thì mức sai thay đổi bao nhiêu?". Trong học liên kết (Federated Learning), mỗi node (nút) tính gradient, hoặc chạy vài bước huấn luyện, trên dữ liệu riêng rồi gửi bản cập nhật mô hình / gradient đi để tổng hợp (McMahan et al., 2017). Nhưng "hướng sửa sai" đó được tính **từ** dữ liệu, nên mang dấu vết của dữ liệu.

## Vì sao gradient làm lộ thông tin?

Với lớp fully connected $y = Wx + b$ và **một ví dụ duy nhất**, quy tắc chuỗi cho:

$$
\frac{\partial L}{\partial W} = \frac{\partial L}{\partial y}\, x^\top,
\qquad
\frac{\partial L}{\partial b} = \frac{\partial L}{\partial y}.
$$

Tức là mỗi hàng $i$ của gradient theo $W$ chính là $x$ nhân với con số $(\partial L/\partial b)_i$. Chỉ cần một phép chia:

$$
x = \frac{\text{hàng}_i\!\left(\partial L / \partial W\right)}{\left(\partial L / \partial b\right)_i}
\quad \left((\partial L/\partial b)_i \neq 0\right).
$$

Không cần đoán hay tối ưu gì: đầu vào được khôi phục chính xác. Giống như gửi đi "bản sao phóng to $k$ lần của bức ảnh" kèm luôn con số $k$.

Đoạn code sau minh họa điều đó, rồi thử cộng nhiễu Gaussian vào gradient trước khi "gửi đi" (cần Python và numpy):

```python
import numpy as np

rng = np.random.default_rng(0)
x = rng.uniform(0, 1, size=8)          # "dữ liệu bí mật": 8 pixel
W = rng.normal(size=(3, 8))            # lớp fully connected, 3 đầu ra
b = rng.normal(size=3)
t = np.array([1.0, 0.0, 0.0])          # nhãn

# Một ví dụ duy nhất, L = 0.5 * ||y - t||^2
y = W @ x + b
dL_dy = y - t
grad_W = np.outer(dL_dy, x)            # dL/dW = dL/dy · xᵀ
grad_b = dL_dy                         # dL/db = dL/dy

def recover(gW, gb):
    i = np.argmax(np.abs(gb))          # chia cho phần tử lớn nhất cho ổn định
    return gW[i] / gb[i]

print("x gốc       :", np.round(x, 3))
print("x khôi phục :", np.round(recover(grad_W, grad_b), 3))

for sigma in [0.01, 0.1, 1.0]:         # cộng nhiễu Gaussian vào gradient
    gW = grad_W + rng.normal(scale=sigma, size=grad_W.shape)
    gb = grad_b + rng.normal(scale=sigma, size=grad_b.shape)
    x_hat = recover(gW, gb)
    err = np.linalg.norm(x_hat - x) / np.linalg.norm(x)
    print(f"sigma={sigma:<5} sai số tương đối = {err:.3f}  x_hat = {np.round(x_hat, 2)}")
```

Khi tôi chạy, kết quả là:

```text
x gốc       : [0.637 0.27  0.041 0.017 0.813 0.913 0.607 0.729]
x khôi phục : [0.637 0.27  0.041 0.017 0.813 0.913 0.607 0.729]
sigma=0.01  sai số tương đối = 0.004  x_hat = [0.64 0.27 0.04 0.02 0.81 0.91 0.6  0.73]
sigma=0.1   sai số tương đối = 0.037  x_hat = [0.61 0.28 0.04 0.02 0.79 0.9  0.6  0.78]
sigma=1.0   sai số tương đối = 0.299  x_hat = [ 0.33  0.34 -0.26  0.1   0.99  0.9   0.43  0.73]
```

Không nhiễu: khôi phục trùng khớp. Nhiễu nhỏ: vẫn gần đúng. Nhiễu lớn hơn: lệch rõ, có cả "pixel" âm, nhưng vài giá trị vẫn gần giá trị gốc. Nhiễu làm mờ chứ không xóa sạch.

:::warning[Đây chỉ là ví dụ đồ chơi]
Một lớp tuyến tính, một ví dụ, một gradient. Cộng nhiễu tùy ý như trên **chưa phải** Differential Privacy: không có bước cắt (clipping), không có kế toán riêng tư, nên không có đảm bảo định lượng nào.
:::

:::tip[Demo tương tác]
Phiên bản tương tác (kéo thanh trượt σ) đang được lên kế hoạch.
:::

## Tấn công đảo ngược gradient (gradient inversion)

Mạng thật có nhiều lớp phi tuyến và gradient thường tính trên cả một lô (batch), nên phép chia trên không còn đủ. Nhưng tấn công vẫn làm được dưới dạng bài toán tối ưu.

Zhu, Liu & Han (2019) đề xuất *Deep Leakage from Gradients*: khởi tạo một đầu vào giả (dummy) ngẫu nhiên rồi liên tục chỉnh nó sao cho gradient của nó **khớp** với gradient quan sát được. Khi khớp, đầu vào giả thường trông giống dữ liệu thật. Geiping et al. (2020) cải tiến bằng cách so khớp theo hướng (cosine similarity) kèm ràng buộc để ảnh trông "tự nhiên", và cho thấy có thể khôi phục ảnh trong một số thiết lập thực tế hơn.

```text
Không bảo vệ:
  Ảnh gốc ─► Gradient ─► Tấn công đảo ngược ─► Ảnh khôi phục (có thể rất giống)

Có DP:
  Ảnh gốc ─► Gradient ─► Cắt + nhiễu ─► Tấn công đảo ngược ─► Ảnh khôi phục
                                                              (khó nhận ra hơn;
                                                               bảo vệ đo bằng ε, δ)
```

:::caution[Đừng phóng đại]
Mức thành công phụ thuộc vào kiến trúc mô hình; thứ được chia sẻ (gradient một bước hay cập nhật sau nhiều bước cục bộ); kích thước batch; và giả định về những gì kẻ tấn công biết. Rủi ro **có thật**, nhưng không phải hệ thống nào cũng dễ bị khôi phục dữ liệu.
:::

## Ai là kẻ tấn công trong DFL?

Mô hình đe dọa thường dùng là đối thủ **trung thực nhưng tò mò (honest-but-curious)**: tuân thủ đúng giao thức, nhưng âm thầm phân tích những gì nhận được (Kairouz et al., 2021).

Trong học liên kết có máy chủ, máy chủ thấy mọi bản cập nhật. Trong DFL, **mỗi láng giềng** đều nhận bản cập nhật của bạn: nếu bệnh viện A nối với B và C, cả hai đều có thể "tò mò". Bỏ máy chủ không xóa rủi ro rò rỉ, chỉ phân tán nó ra nhiều nơi.

## Differential Privacy: giới hạn những gì có thể suy ra

Quyền riêng tư vi phân (Differential Privacy, DP) không cố giấu dữ liệu. Nó hứa rằng **đầu ra gần như không đổi dù có hay không có dữ liệu của bất kỳ một cá nhân nào** (Dwork, McSherry, Nissim & Smith, 2006; Dwork & Roth, 2014).

Một cơ chế ngẫu nhiên $\mathcal{M}$ thỏa mãn $(\varepsilon, \delta)$-DP nếu với mọi cặp tập dữ liệu láng giềng $D, D'$ (khác nhau đúng một bản ghi) và mọi tập kết quả $S$:

$$
\Pr[\mathcal{M}(D) \in S] \;\le\; e^{\varepsilon}\, \Pr[\mathcal{M}(D') \in S] + \delta.
$$

Bằng lời thường:

- Hai thế giới: một có hồ sơ của bạn, một không. Người quan sát chỉ thấy đầu ra của $\mathcal{M}$.
- Ngân sách riêng tư ε giới hạn hai thế giới "trông khác nhau" đến mức nào. $\varepsilon$ càng nhỏ, càng khó phân biệt, bảo vệ càng mạnh.
- $\delta$ là xác suất nhỏ mà lời hứa trên được phép "trượt"; thường được chọn nhỏ hơn nhiều so với $1/n$ ($n$ là số bản ghi).

Lưu ý: $\varepsilon$ **không phải** lượng nhiễu, mà là thước đo mức đảm bảo. Với cơ chế Gaussian, muốn $\varepsilon$ nhỏ hơn thì, khi các yếu tố khác giữ nguyên, cần nhiều nhiễu hơn.

### Áp dụng vào huấn luyện

Cách phổ biến là DP-SGD (Abadi et al., 2016). Ở mỗi bước:

1. **Cắt (clip)** gradient của từng ví dụ để độ dài không vượt ngưỡng $C$, giới hạn ảnh hưởng tối đa của một cá nhân.
2. **Thêm nhiễu Gaussian** với độ lệch chuẩn $\sigma \cdot C$, trong đó $\sigma$ là hệ số nhiễu σ.

Sau đó, kế toán riêng tư (privacy accounting), như moments accountant (Abadi et al., 2016) hay dựa trên Rényi DP (Mironov, 2017), cộng dồn mức rò rỉ qua mọi vòng huấn luyện để ra $\varepsilon$ cuối cùng. Đoạn code ở trên chỉ có bước 2, nên chỉ cho trực giác, không phải lời hứa định lượng.

## Vì sao nhiễu không phải "an toàn tuyệt đối"?

:::important[Những điều DP không hứa]
- DP chỉ có ý nghĩa khi dùng **đúng cơ chế** (cắt, nhiễu hiệu chỉnh theo độ nhạy) **cùng kế toán riêng tư**. Thêm nhiễu tùy tiện thì không có đảm bảo.
- $\varepsilon$ và $\sigma$ không phải ánh xạ một-một: với cùng $\sigma$, số vòng, tỉ lệ lấy mẫu và loại accountant đều ảnh hưởng tới $\varepsilon$ cuối cùng.
- DP là đảm bảo **định lượng**, không phải lời hứa "không ai đoán được gì". $\varepsilon$ lớn thì đảm bảo yếu.
- Nhiễu có giá: mô hình thường học kém đi (đánh đổi giữa riêng tư và độ hữu ích, chủ đề bài sau).
:::

## Vì sao DFL cần thêm DP?

Khi mỗi node gửi cập nhật cho nhiều láng giềng có thể trung thực nhưng tò mò, chia sẻ gradient "trần" là không đủ. DP thêm một lớp bảo vệ có thể đo đếm lên trên DFL.

:::note[Câu hỏi còn mở]
Mỗi node nên chọn mức ε nào, khi dữ liệu và mức nhạy cảm của mỗi bên khác nhau, và nhiễu của một node ảnh hưởng tới mô hình chung? Chưa có câu trả lời chung cho mọi tình huống.
:::

## Kết

Không chia sẻ dữ liệu gốc mới chỉ là bước đầu; ta còn phải kiểm soát **những gì có thể suy ra** từ thứ mô hình chia sẻ. DP giúp làm điều đó một cách đo đếm được. Nhưng nhiễu có giá: thêm bao nhiêu là đủ?

**Bài tiếp theo:** [Quyền riêng tư có giá bao nhiêu? Bài toán đánh đổi giữa Privacy và Accuracy](/posts/quyen-rieng-tu-co-gia-bao-nhieu/).

**Trong series này:** [bài tổng quan](/posts/ai-cung-hoc-ma-khong-tin-nhau/) cho bức tranh toàn cảnh, và [bài 1 về học liên kết phi tập trung](/posts/ai-hoc-tu-du-lieu-ma-khong-nhin-thay/).

## Tài liệu tham khảo

- Abadi et al. (2016). *Deep Learning with Differential Privacy*.
- Dwork, McSherry, Nissim & Smith (2006). *Calibrating Noise to Sensitivity in Private Data Analysis*.
- Dwork & Roth (2014). *The Algorithmic Foundations of Differential Privacy*.
- Geiping et al. (2020). *Inverting Gradients – How easy is it to break privacy in federated learning?*
- Kairouz et al. (2021). *Advances and Open Problems in Federated Learning*.
- McMahan et al. (2017). *Communication-Efficient Learning of Deep Networks from Decentralized Data*.
- Mironov (2017). *Rényi Differential Privacy*.
- Zhu, Liu & Han (2019). *Deep Leakage from Gradients*.
