# 📝 STM32F401 GPIO Register-Level Cheat Sheet

Tài liệu tóm tắt cấu trúc và cách cấu hình hệ thống ngoại vi GPIO của vi điều khiển STM32F4 (Cortex-M4) ở cấp độ thanh ghi. Hỗ trợ tra cứu nhanh công thức tính toán dịch bit và các thao tác bitwise phổ biến trong lập trình nhúng.

---

### Bật clock cấu hình cho các port: (cấp điện cho các thanh ghi hoạt động)
RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;

RCC->AHB1ENR |= RCC_AHB1ENR_GPIOBEN;

RCC->AHB1ENR |= RCC_AHB1ENR_GPIOCEN;

### A. Cấu Hình Chân Với các Thanh Ghi `MODER, OTYPER, OSPEEDR, PUPDR, AFR, LCKR`

1. MODER - Chọn chức năng của chân. 32 bit, Mỗi chân dùng 2 bit.

MODER	Chế độ

00	Input

01	General purpose Output

10	Alternate Function

11	Analog

- Ví dụ PA5 CPU OUTPUT:
- 
  GPIOA->MODER &= ~(3 << (5*2));   // clear PA5
  
  GPIOA->MODER |=  (1 << (5*2));   // General purpose Output

---------------------------------------------------------------------------------
2. OTYPER - Kiểu Output. 16bit, Mỗi chân chỉ có 1 bit.

Bit	Chế độ

0	Push Pull

1	Open Drain

GPIOA->OTYPER &= ~(1<<5);     // Push Pull

GPIOA->OTYPER |= (1<<5);      // Open Drain

---------------------------------------------------------------------------------
3. OSPEEDR - Tốc độ Output. 32 bit, Mỗi chân dùng 2 bit.

OSPEEDR	Tốc độ

00	Low

01	Medium

10	High

11	Very High

GPIOA->OSPEEDR &= ~(3<<(5*2));     // clear PA5

GPIOA->OSPEEDR |=  (2<<(5*2));     // High speed PA5

---------------------------------------------------------------------------------
4. PUPDR - Điện trở kéo

PUPDR	Chế độ

00	No pull

01	Pull-up

10	Pull-down

11	Reserved

GPIOA->PUPDR &= ~(3<<(5*2));      // clear PA5

GPIOA->PUPDR |=  (1<<(5*2));      // Pull-up PA5

---------------------------------------------------------------------------------
5. AFR - Alternate Function (Cấu hình ngoại vi peripheral). Mỗi chân dùng 4 bit.
- Tra AF và chọn chân theo datasheet của F411, (Table 9. Alternate function mapping)
- AFR được chia làm hai thanh ghi:

AFRL : Pin 0 → 7.   GPIOx->AFR[0].  (0xF << (7*4))      // Px7

AFRH : Pin 8 → 15.  GPIOx->AFR[1].  (0xF << ((8-8)*4))  // Px8

- Ví dụ PA8 làm TIM1_CH1:
- 
GPIOA->AFR[1] &= ~(0xF << ((8-8)*4));   // clear PA8

GPIOA->AFR[1] |=  (1 << ((8-8)*4));     // AF1 PA8 TIM1_CH1

---------------------------------------------------------------------------------
 ### B. IO VỚI CÁC THANH GHI THANH GHI IDR/ODR, BSRR.
 1.  IDR/ODR. (Input register) / (outPut register)
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

---------------------------------------------------------------------------------
2. BSRR: Bit set reset register

GPIOA->BSRR = (1<<5);      // Set PA1

GPIOA->BSRR = (1<<(5+16)); // Reset PA1
