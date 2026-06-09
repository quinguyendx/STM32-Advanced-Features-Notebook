# 📝 STM32F1 GPIO Register-Level Cheat Sheet

Tài liệu tóm tắt cấu trúc và cách cấu hình hệ thống ngoại vi GPIO của vi điều khiển STM32F1 (Cortex-M3) ở cấp độ thanh ghi. Hỗ trợ tra cứu nhanh công thức tính toán dịch bit và các thao tác bitwise phổ biến trong lập trình nhúng.

---

## 1. Cấu Hình Chân Với Thanh Ghi `CRL` và `CRH`

Mỗi Port (GPIOA, GPIOB,...) có hai thanh ghi 32-bit để cấu hình trạng thái chân:
* **`CRL` (Control Register Low):** Cấu hình cho các chân từ `Pin 0` $\rightarrow$ `Pin 7`.
* **`CRH` (Control Register High):** Cấu hình cho các chân từ `Pin 8` $\rightarrow$ `Pin 15`.

Mỗi chân vật lý sẽ chiếm dụng đúng **4 bit** trong thanh ghi để thiết lập trạng thái (`2 bit MODE` và `2 bit CNF`).

### 📊 Bảng Tra Cứu Trạng Thái Cấu Hình (CNF + MODE)

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

### 🧮 Công Thức Tính Vị Trí Dịch Bit Tổng Quát `(Y << X)`

* **`Y` (Trạng thái):** Giá trị nhị phân hợp nhất của `CNF` và `MODE` (4 bit) tra từ bảng trên.
  * *Ví dụ:* Input Pull-up/down = `0b1000` (0x8); Output Push-pull 2MHz = `0b0010` (0x2).
* **`X` (Vị trí Shift):** * Đối với thanh ghi **`CRL`** (Chân 0 đến 7): $X = \text{Số chân} \times 4$
  * Đối với thanh ghi **`CRH`** (Chân 8 đến 15): $X = (\text{Số chân} - 8) \times 4$

### 💻 Code Cấu Hình Mẫu (Thanh Ghi)

Sử dụng kỹ thuật Clear-bit trước khi Ghi-bit để tránh làm ảnh hưởng đến cấu hình cũ của thanh ghi:

```c
// Cấu hình chân PA0 làm Input Pull-up/down (0b1000)
// Shift = 0 * 4 = 0
GPIOA->CRL = (GPIOA->CRL & ~(0xF << 0)) | (0x8 << 0); 
GPIOA->BSRR = (1 << 0); // Kích hoạt Pull-up cho PA0

// Cấu hình chân PC13 làm Output Push-pull 2MHz (0b0010)
// Shift = (13 - 8) * 4 = 20
GPIOC->CRH = (GPIOC->CRH & ~(0xF << 20)) | (0x2 << 20);