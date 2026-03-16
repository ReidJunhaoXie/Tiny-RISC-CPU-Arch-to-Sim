# 11_重構CPU（加入暫存器 B）


目前系統中的暫存器只有一個，相當於寫程式的時候只有一個變數可以用，新增一個 `register_B` 。
這造成兩個問題
    - 依賴記憶體存取
    - 指令不靈活

---

## 一、系統電路

<img width="700" height="500" alt="Image" src="https://github.com/user-attachments/assets/63ecf3f8-c333-4247-9555-6e300628d9a1" />


為了新功能:

1. 複製一組A暫存器
2. 利用 `enable` 控制暫存器載入
3. 透過 MUX 決定輸入來自 A 還是 B 暫存器

---

## 二、Data Flow

### 2.1 原始設計

```text
RAM → Dout → ALU B → ALU 運算 → S → ALU A → Register_A
```

- ALU B 端固定接 RAM 資料，意思就是RAM吐的資料必定會送到ALU B 端
- ALU 運算結果存入 Register_A
- 更合理的做法是，ALU的A、B端不應該有任何差別，都可以載入RAM的資料、暫存器資料，而任何暫存器也都可以存入ALU的計算結果

### 2.2 增加暫存器後的新設計

```text
// 可選擇來源
RAM → Dout → MUXA/B → ALU A/B → S → Register_A/B
```

- RAM 的輸出可送至 ALU A 或 B 端
- ALU A/B 端可由任意暫存器（A/B）或 RAM 資料供給
- 這就是為什麼一開始sel_a、sel_b控制位是編2bits

> 這樣能讓兩個暫存器（A、B）互相賦值，或與 RAM 交換資料

---

## 三、指令對應時序調整

- **原本**：
  1. 指令讀取 → 指令暫存器(IR) → operand → 高位補零 → immediate → execute → ALU B 端
  2. ALU B 僅接受 RAM 資料，`RAM.ld` 長期保持開啟

- **新增暫存器後**：
  - immediate 資料可直接拉至 MUXA／MUXB，但切記，這時需要關閉 `RAM.ld`
  - MUX 根據 `sel_a`、`sel_b` 選擇輸入來源 - RA、RB、0、Dout(immediate)

---

## 四、LUT 更新

<img width="700" height="500" alt="Image" src="https://github.com/user-attachments/assets/4d69d331-22ee-4b6d-b581-c7ec85ea4624" />

- 加入控制位en_b，來判斷 `Register_B` 的enable

---

## 五、新的系統電路

<img width="700" height="400" alt="Image" src="https://github.com/user-attachments/assets/887ab4e1-2ddd-40f0-a6ae-b53979c9d7d2" />

- 暫存器A/B : ALU計算後立即存在這
- ALU : A (+/-/or/and) B
- RAM : 存指令跟資料
- Controller : 所有控制位都在這
- State logic : 決定這時用的是PC的地址(指令)還是operand(資料)
- PC : 指向RAM的某處
- Immediate logic : 決定是否啟用immediate
- Cmpare Reg : 放比較結果
- Instruction Reg : 指令暫存器

---

## 六、再談Data Flow

```text
// 原始 Data Flow
RAM.Dout → ALU B → Register_A

// 更新後 Data Flow
// RAM 資料
RAM.Dout → MUXA/B → ALU A/B → Register_A/B

// 暫存器互傳
Register_A/B → MUXA/B → ALU A/B → Register_A/B
```

經過immediate、暫存器B重構之後，真的可以開始編指令了 !