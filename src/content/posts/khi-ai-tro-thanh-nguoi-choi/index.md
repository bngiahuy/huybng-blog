---
title: "Khi các AI trở thành những người chơi: Lý thuyết trò chơi giúp bảo vệ dữ liệu như thế nào?"
published: 2026-10-29T08:00:00+07:00
description: "Mô hình hóa các node trong học liên kết phi tập trung như những người chơi chọn ngân sách riêng tư ε: cân bằng Nash, ESS và replicator dynamics."
image: ""
tags: ["Game Theory", "Evolutionary Game Theory", "Nash Equilibrium", "Federated Learning"]
category: "Research"
draft: false
lang: "vi"
---

5 bệnh viện cùng huấn luyện một mô hình AI. Bệnh viện A muốn bảo vệ dữ liệu tối đa, B muốn mô hình chính xác nhất, C muốn tốn ít tài nguyên tính toán. Mục tiêu khác nhau, nhưng ai cũng muốn dùng mô hình chung. **Liệu họ có thể tự điều chỉnh hành vi mà không cần người quản lý trung tâm?**

Đây là bài thứ tư trong series *When AI Learns Together: Privacy, Trust & Game Theory*. Sau [bài về ngân sách riêng tư ε](/posts/quyen-rieng-tu-co-gia-bao-nhieu/), câu hỏi còn lại là: **mỗi node nên chọn mức ε nào?** Công cụ hữu ích ở đây là lý thuyết trò chơi (Game Theory).

## Thế lưỡng nan của người tù

Lý thuyết trò chơi nghiên cứu các tình huống mà kết quả của mỗi bên phụ thuộc cả vào lựa chọn của người khác. Ví dụ kinh điển: **Thế lưỡng nan của người tù (Prisoner's Dilemma)**: hai nghi phạm bị thẩm vấn riêng, mỗi người chọn *Hợp tác* (im lặng) hoặc *Phản bội* (khai ra người kia). Bảng ghi lợi ích (payoff) dạng (người 1, người 2), số lớn hơn là tốt hơn:

| | Người 2: Hợp tác | Người 2: Phản bội |
|---|---|---|
| **Người 1: Hợp tác** | (3, 3) | (0, 5) |
| **Người 1: Phản bội** | (5, 0) | (1, 1) |

Dù đối phương làm gì, phản bội luôn cho người 1 nhiều điểm hơn (5 so với 3, hoặc 1 so với 0). Người 2 suy luận y hệt, nên cả hai nhận (1, 1), tệ hơn (3, 3).

Bài học: **lựa chọn hợp lý cho từng cá nhân có thể dẫn tới kết quả tệ cho cả nhóm.**

## Các node DFL như những người chơi

Trong học liên kết phi tập trung (DFL), mỗi node trao đổi bản cập nhật mô hình với hàng xóm, có áp dụng quyền riêng tư vi phân (Differential Privacy, DP) với ngân sách riêng tư ε. Ánh xạ sang trò chơi:

| Khái niệm trò chơi | Trong DFL với DP |
|---|---|
| Người chơi | Mỗi node (bệnh viện A–E) |
| Chiến lược | Mức ε node chọn, ví dụ {chặt, vừa, lỏng} |
| Lợi ích (payoff) | Lợi từ mô hình chung − mất mát riêng tư − chi phí |
| Tương tác | Trao đổi cập nhật với hàng xóm trên đồ thị |

Nhắc lại: ε nhỏ hơn là riêng tư mạnh hơn và, khi các yếu tố khác giữ nguyên, cần nhiều nhiễu hơn. ε không phải "lượng nhiễu"; nó còn phụ thuộc số vòng, tỷ lệ lấy mẫu và kế toán riêng tư (privacy accounting).

### Lợi ích của một node

Một cách viết trực quan cho node $i$, với $\varepsilon_{-i}$ là lựa chọn của các node khác:

$$
u_i(\varepsilon_i, \varepsilon_{-i}) = G_i(\varepsilon_i, \varepsilon_{-i}) - L_i(\varepsilon_i) - C_i(\varepsilon_i)
$$

$G_i$ là lợi từ mô hình chung, $L_i$ là mất mát riêng tư, $C_i$ là chi phí. Điểm mấu chốt: $G_i$ phụ thuộc vào **mọi người**, vì mô hình chung tổng hợp cập nhật của hàng xóm; hàng xóm thêm nhiều nhiễu thì node $i$ chịu thiệt.

Đây là hương vị của bài toán **hàng hóa công cộng (public goods)**: chất lượng mô hình là của chung, rủi ro riêng tư là của riêng. Mỗi node có động cơ chọn ε nhỏ để bảo vệ mình và hưởng lợi từ cập nhật ít nhiễu của người khác: **ăn theo (free-riding)**, rất giống "phản bội" ở trên. Chỉ là "hương vị": trò chơi thật có đúng cấu trúc Prisoner's Dilemma hay không tùy cách định nghĩa $G_i$, $L_i$, $C_i$.

## Cân bằng Nash

Nash (1950) định nghĩa trạng thái mà **không người chơi nào có lợi khi đơn phương đổi chiến lược**: cân bằng Nash (Nash Equilibrium). Bộ $(\varepsilon_1^*, \dots, \varepsilon_n^*)$ là cân bằng Nash nếu với mọi node $i$ và mọi $\varepsilon_i$:

$$
u_i(\varepsilon_i^*, \varepsilon_{-i}^*) \ge u_i(\varepsilon_i, \varepsilon_{-i}^*)
$$

Trong Prisoner's Dilemma, (Phản bội, Phản bội) là cân bằng Nash duy nhất dù không tốt cho cả nhóm. Vì vậy cần phân biệt lợi ích cá nhân với **lợi ích toàn mạng (social welfare)**.

Nhưng các node DFL không suy nghĩ hay thương lượng như con người; chúng chạy quy tắc chọn và cập nhật chiến lược được lập trình sẵn. Vì thế cần thêm một góc nhìn khác.

## Lý thuyết trò chơi tiến hóa

Lý thuyết trò chơi tiến hóa (Evolutionary Game Theory, EGT) bắt nguồn từ sinh học. EGT hỏi: **trong một quần thể, chiến lược nào cho lợi ích cao sẽ lan rộng theo thời gian?**

### Chiến lược ổn định tiến hóa (ESS)

Maynard Smith và Price (1973) đưa ra khái niệm chiến lược ổn định tiến hóa (Evolutionarily Stable Strategy, ESS): khi cả quần thể dùng $s^*$, một nhóm nhỏ "đột biến" dùng $s$ khác không thể xâm chiếm. Với mọi $s \ne s^*$, hoặc $u(s^*, s^*) > u(s, s^*)$, hoặc $u(s^*, s^*) = u(s, s^*)$ và $u(s^*, s) > u(s, s)$.

### Động lực học sao chép (replicator dynamics)

Taylor và Jonker (1978) mô tả cách tỷ lệ các chiến lược thay đổi bằng động lực học sao chép (replicator dynamics). Gọi $x_i$ là tỷ lệ quần thể dùng chiến lược $i$, $f_i(x)$ là lợi ích của chiến lược đó, $\bar f(x) = \sum_j x_j f_j(x)$ là lợi ích trung bình:

$$
\dot{x}_i = x_i\,\big(f_i(x) - \bar f(x)\big)
$$

Chiến lược tốt hơn trung bình thì tỷ lệ tăng, kém hơn thì giảm. Trong DFL, $i$ có thể là các mức ε và $x_i$ là tỷ lệ node dùng mức đó. Công thức này dành cho quần thể lớn, trộn đều; với 5 node trên đồ thị, quy tắc "bắt chước hàng xóm làm tốt hơn" chỉ cùng tinh thần, không trùng khớp về toán học.

## Mô phỏng nhỏ: 5 node chọn ε

5 bệnh viện A–E trên một đồ thị nhỏ, mỗi node chọn một trong ba mức ε. Mỗi vòng, node quan sát một hàng xóm ngẫu nhiên; nếu hàng xóm có lợi ích cao hơn, node bắt chước với xác suất 0,5.

:::warning[Đây là mô hình đồ chơi]
Hàm lợi ích dưới đây chỉ để minh họa, **không phải mô hình từ bất kỳ nghiên cứu nào**: không huấn luyện mô hình thật, không tính ε bằng kế toán riêng tư, mọi tham số đều giả định.
:::

```python
import random

random.seed(42)

# Đồ thị: vòng A-B-C-D-E-A, thêm cạnh A-C
NEIGHBORS = {"A": ["B", "E", "C"], "B": ["A", "C"], "C": ["B", "D", "A"],
             "D": ["C", "E"], "E": ["D", "A"]}
EPS_LEVELS = [0.5, 1.0, 4.0]                       # chặt / vừa / lỏng
SENSITIVITY = {"A": 0.20, "B": 0.05, "C": 0.10, "D": 0.10, "E": 0.15}
COST = {0.5: 0.02, 1.0: 0.01, 4.0: 0.0}           # giả định

def quality(eps):            # ε lớn -> ít nhiễu -> đóng góp tốt hơn (đồ chơi)
    return eps / (eps + 1.0)

def payoff(node, strat):     # LỢI ÍCH ĐỒ CHƠI, CHỈ ĐỂ MINH HỌA
    group = [node] + NEIGHBORS[node]
    shared = sum(quality(strat[n]) for n in group) / len(group)
    return shared - SENSITIVITY[node] * strat[node] - COST[strat[node]]

strat = {n: random.choice(EPS_LEVELS) for n in NEIGHBORS}

for t in range(1, 51):
    pay = {n: payoff(n, strat) for n in strat}
    new = dict(strat)
    for n in strat:
        m = random.choice(NEIGHBORS[n])                # quan sát 1 hàng xóm
        if pay[m] > pay[n] and random.random() < 0.5:  # bắt chước nếu tốt hơn
            new[n] = strat[m]
        if random.random() < 0.02:                     # thỉnh thoảng thử ngẫu nhiên
            new[n] = random.choice(EPS_LEVELS)
    strat = new
    if t == 1 or t % 10 == 0:
        row = "  ".join(f"{n}:ε={strat[n]:<3}" for n in strat)
        welfare = sum(payoff(n, strat) for n in strat)
        print(f"vòng {t:2d} | {row} | tổng lợi ích = {welfare:.3f}")
```

Code chỉ dùng thư viện chuẩn (`python3 sim.py`). Kết quả chỉ là hành vi của mô hình đồ chơi, nhưng hãy để ý:

- **Lan truyền:** một mức ε có dần chiếm cả mạng, hay các node vẫn dùng các mức khác nhau?
- **Cá nhân và tập thể:** trạng thái các node đi tới có phải trạng thái có "tổng lợi ích" cao nhất không?
- **Độ nhạy:** đổi `random.seed`, `SENSITIVITY`, `quality` hay các cạnh, rồi so sánh.

:::tip[Demo tương tác]
Phiên bản tương tác của mô phỏng này đang được lên kế hoạch.
:::

:::caution[Không có gì được bảo đảm]
Các node **không chắc chắn** đạt cân bằng Nash hay ESS. Cân bằng có tồn tại, có ổn định, và quy tắc cập nhật có hội tụ về đó hay không đều phụ thuộc thiết kế trò chơi (hàm lợi ích, tập chiến lược, đồ thị) và quy tắc cập nhật. Có trò chơi không có ESS, có nhiều cân bằng, hoặc dao động mãi.
:::

## Lựa chọn ε như một tương tác chiến lược

Ghép lại: chọn ε là một **tương tác chiến lược**, khác với lên lịch tăng giảm ε theo thời gian, vì lựa chọn tốt nhất của mỗi node phụ thuộc vào lợi ích của nó và lựa chọn của node khác. Cân bằng Nash mô tả điểm dừng, còn EGT mô tả cách các quy tắc cập nhật đơn giản có thể (hoặc không) đưa mạng tới đó.

:::note[Câu hỏi còn mở]
- Thiết kế hàm lợi ích thế nào để cân bằng không rơi vào cảnh "ai cũng thêm thật nhiều nhiễu"?
- Dữ liệu không đồng nhất (Non-IID) làm lợi ích của từng node khác nhau ra sao?
- Nếu có node khai gian về chiến lược hoặc cập nhật của mình thì sao?
:::

## Lời kết

Thay vì áp cùng một mức bảo vệ cho mọi người, chúng ta có thể nghiên cứu cách các node chọn chiến lược riêng tư dựa trên lợi ích của chính mình và hành vi của hàng xóm.

### Trong series này

Bài này khép lại series *When AI Learns Together: Privacy, Trust & Game Theory*. Xem lại [bài tổng quan](/posts/ai-cung-hoc-ma-khong-tin-nhau/) và các bài trước:

- [AI có thể học từ dữ liệu của bạn mà không cần nhìn thấy?](/posts/ai-hoc-tu-du-lieu-ma-khong-nhin-thay/)
- [Không chia sẻ dữ liệu gốc đã an toàn chưa?](/posts/khong-chia-se-du-lieu-da-an-toan-chua/)
- [Quyền riêng tư có giá bao nhiêu?](/posts/quyen-rieng-tu-co-gia-bao-nhieu/)

## Tài liệu tham khảo

- Nash, J. (1950). *Equilibrium Points in n-Person Games*.
- Maynard Smith, J., & Price, G. R. (1973). *The Logic of Animal Conflict*.
- Taylor, P. D., & Jonker, L. B. (1978). *Evolutionary Stable Strategies and Game Dynamics*.
- Dwork, C., & Roth, A. (2014). *The Algorithmic Foundations of Differential Privacy*.
- Kairouz, P., et al. (2021). *Advances and Open Problems in Federated Learning*.
