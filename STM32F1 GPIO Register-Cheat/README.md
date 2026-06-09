# 📝 STM32F1 GPIO Register-Level Cheat Sheet

Tài liệu tóm tắt cấu trúc và cách cấu hình hệ thống ngoại vi GPIO của vi điều khiển STM32F1 (Cortex-M3) ở cấp độ thanh ghi. Hỗ trợ tra cứu nhanh công thức tính toán dịch bit và các thao tác bitwise phổ biến trong lập trình nhúng.

---

## 1. Cấu Hình Chân Với Thanh Ghi `CRL` và `CRH`

Mỗi Port (GPIOA, GPIOB,...) có hai thanh ghi 32-bit để cấu hình trạng thái chân:
* **`CRL` (Control Register Low):** Cấu hình cho các chân từ `Pin 0` $\rightarrow$ `Pin 7`.
* **`CRH` (Control Register High):** Cấu hình cho các chân từ `Pin 8` $\rightarrow$ `Pin 15`.

Mỗi chân vật lý sẽ chiếm dụng đúng **4 bit** trong thanh ghi để thiết lập trạng thái (`2 bit MODE` và `2 bit CNF`).

### Bảng Tra Cứu Trạng Thái Cấu Hình (CNF + MODE)

| Chế độ | CNF [1:0] | Thao tác / Tính năng | MODE [1:0] | Tốc độ tối đa |
| :--- | :---: | :--- | :---: | :--- |
| **INPUT** | `00` | Analog mode | `00` | Input mode (Reset state) |
| | `01` | Floating input (Lơ lửng) | | |
| | `10` | Input with Pull-up / Pull-down | | |
| | `11` | Reserved (Bị giữ lại) | | |
| **OUTPUT** | `00` | General purpose Output Push-pull | `01` | Max speed 10 MHz |
| | `01` | General purpose Output Open-drain | `10` | Max speed 2 MHz |
| | `10` | Alternate function Output Push-pull | `11` | Max speed 50 MHz |
| | `11` | Alternate function Output Open-drain | | |

### Công Thức Tính Vị Trí Dịch Bit Tổng Quát `(Y << X)`

* **`Y` (Trạng thái):** Giá trị nhị phân hợp nhất của `CNF` và `MODE` (4 bit) tra từ bảng trên.
  * *Ví dụ:* Input Pull-up/down = `0b1000` (0x8); Output Push-pull 2MHz = `0b0010` (0x2).
* **`X` (Vị trí Shift):**
  * Đối với thanh ghi **`CRL`** (Chân 0 đến 7): $X = \text{Số chân} \times 4$
  * Đối với thanh ghi **`CRH`** (Chân 8 đến 15): $X = (\text{Số chân} - 8) \times 4$

### Code Cấu Hình Mẫu (Thanh Ghi)

```c
// Cấu hình chân PA0 làm Input Pull-up/down (0b1000)
// Shift = 0 * 4 = 0
GPIOA->CRL = (GPIOA->CRL & ~(0xF << 0)) | (0x8 << 0); 
GPIOA->BSRR = (1 << 0); // Kích hoạt Pull-up cho PA0

// Cấu hình chân PC13 làm Output Push-pull 2MHz (0b0010)
// Shift = (13 - 8) * 4 = 20
GPIOC->CRH = (GPIOC->CRH & ~(0xF << 20)) | (0x2 << 20);
```
============================< IO: THANH GHI IDR/ODR >============================
- Hai thanh ghi 16bit lưu các giá trị đọc vào / xuất ra từng chân.

- ((GPIOA->IDR >> 5) & 1)        // Trả về giá trị 0/1 chân PA5
- if(GPIOA->IDR & (1 << 5))      // Trả về giá trị 0/32
- if(GPIOA->IDR & GPIO_IDR_IDR5) // Đơn giản hơn, dùng marco

 Đọc 4 nút hoặc vài nút cùng bấm:
-- if(((GPIOA->IDR >> 4) & 0xF) == 0xF)

Ví dụ bật LED PA5

GPIOA->ODR |= (1 << 5);

GPIOA->ODR = 0x0020;

Tắt LED:

GPIOA->ODR &= ~(1 << 5);

Ghi nhiều bit cùng lúc.

GPIOA->ODR = 0x00F0;

Đảo bit:

GPIOA->ODR ^= (1 << 5);

==========================< OUT: THANH GHI BRR/BSRR >==========================
- BRR: 16 bit, lưu các giá trị để reset 16 chân.
- BSRR: 32 bit, 0->15 lưu các giá trị set on của chân, 16->31 là reset chân.

Ví dụ bật LED PA5:

GPIOA->BSRR = (1 << 5);

Tắt LED:

GPIOA->BSRR = (1 << (5+16));

GPIOA->BRR = (1 << 5);

Ví dụ bật LED PC13:

GPIOC->BSRR = GPIO_PIN_13;


Tắt LED:

GPIOC->BRR  = GPIO_PIN_13;

Chuyển number thành nhị phân rồi xuất:

GPIOA->BRR  = 0x0F;  // clear

GPIOA->BSRR = (number & 0x0F);

Reset và set cùng một lần:

GPIOA->BSRR = (0x000F << 16) | (number & 0x0F);

Set PA3 và reset PA2 cùng lúc:

GPIOA->BSRR = (1 << 3) | (1 << (2 + 16));