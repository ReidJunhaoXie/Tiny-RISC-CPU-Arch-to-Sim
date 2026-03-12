# 02_建立ALU


ALU（Arithmetic Logic Unit）是處理器的計算模組，可視為一個精簡的「電子計算機」!

---

## 1. 功能

- **運算位元**：16 bits
- **支援功能**：
    - 加法（add）
    - 減法（sub）
    - 位元與（and by bit）
    - 位元或（or by bit）
    - Zero Flag（ZF）：判斷輸出結果是否為零

---

## 2. 實作結構

### 2.1 結構
- 透過 MUX 選擇不同的運算模組：
    - 加法器
    - 減法器
    - AND 邏輯
    - OR 邏輯
- 利用比較器(也可以用減法器)，將輸出結果與 0 比較，得出 Zero Flag（ZF）。
- Mux的控制訊號sel是整個模組的輸入
- 電路

<img width="1486" height="500" alt="Image" src="https://github.com/user-attachments/assets/7f3cb2a1-db39-4b6d-84d4-e36ae6db200b" />

#### 2.1.1 運算

| sel   |    結果   |
| ----- | --------- |
| 0b00  | A + B     |
| 0b01  | A - B     |
| 0b10  | A & B     |
| 0b11  | A \| B    |

---
### 2.2 加入暫存器（Register）

- 將 ALU 輸出連接至 Register，儲存運算結果，實現累加器（Accumulator）的基本結構，平常用的簡易計算機可能就有相關設計。
- 「運算＋暫存」-> 完整的計算
- 電路
 
<img width="1282" height="350" alt="Image" src="https://github.com/user-attachments/assets/3509c88b-2a78-4c72-8201-0291666e6b57" />

---

## 3. 補充ZF訊號

- 在寫程式時大家都把判斷式用的很熟了，ZF訊號就是為了在底層支援邏輯判斷


