# 10_加入immediate

前面說要編組語，但我錯了，還有很多功能要加。 
目前的情況，一條指令包含 opcode 以及 operand，也就是說，當做運算的時候，需要從 RAM 裡面找資料，再做運算。

- 有更好的做法：把某些常用的數，如 0、10 等，**直接當成一個數值放在 operand 的位置**，概念上就是常數，稱為「immediate」，意思是馬上得到的一個數。  
- 聽到這裡，馬上可以反應，既然 immediate 跟 operand 共用位置，自然需要一個 bit 去區分當下的操作，所以操作 immediate 跟操作 RAM 裡的值應該要是不同指令。

再次看一次電路：  

<img width="1413" height="500" alt="Image" src="https://github.com/user-attachments/assets/53bac97b-e288-4981-815e-934f0640d03b" />

當執行階段時，en_pc 為 0，會選擇 operand 指向 RAM 的某處。  
我的初步想法是：  
1. 假設指令為操作 immediate，此時 operand 就是某個數值，應該要連到 ALU 的 B 端。  
2. operand 不允許指向 RAM，因此中間可以用開關控制，記得訊號要到 LUT。  

## 修改 LUT：加入控制位 `en_i`（enable_immediate）  

<img width="1691" height="450" alt="Image" src="https://github.com/user-attachments/assets/575053e3-5ae9-4671-9f8a-87ab9ac6df6b" />

> **注意**：ALU 的 B 端是 16 bits，而 operand 只有 11 bits，所以需要高位補零。  
> 第二個問題：我們當初設計 `sel_b` 時有預留，可用來控制 MUX。  
> - 原本：`sel_b` 控制「從 RAM 取值」或「取 0」  
> - 新增：當 `sel_b=11` 時，再由 `en_i` 控制「取 immediate」。  
> 在原始 operand 和零擴展後 operand 間加tri-state buffer，由 `en_i` 驅動開關。

在編指令時有些技巧：  
- 先編 `state=1` 且 `cmp=0` 的情況，目標是 immediate 版本的 `ld`/`+`/`-`/`and`/`or`，如前面所述，`en_i` 表示是否為 immediate，此時 RAM 不需要 load（取 0），而 `sel_b=11` 選擇 immediate。  
- 編完後，可直接移植到 `cmp=1` 情況，fetch 階段行為一致，無需額外編寫。

## 進 LUT 編指令  

<img width="2005" height="500" alt="Image" src="https://github.com/user-attachments/assets/528f247a-713b-4b1b-b630-09ffe787e0b7" />

## 更新後的電路  

<img width="1578" height="470" alt="Image" src="https://github.com/user-attachments/assets/01a64e5f-cb05-4c0b-b408-10b208b564e2" />

<img width="1050" height="450" alt="Image" src="https://github.com/user-attachments/assets/f3c2cec2-0bae-4c03-bd2b-d957fa01579a" />

## 操作 immediate 的程式規劃

| time | instruction | Instruction Description    | opcode | operand (目標地址)      | data (操作值) | whole instruction       | 備註                   |
| ---- | ----------- | -------------------------- | ------ | ---------------------- | ------------ | ----------------------- | ---------------------- |
| 0    | Fetch       | Fetch Instruction to IR     |        |                        |              | opcode + operand       |                        |
| 1    | ld_ia       | load immediate to ALU reg   | 01001  | 00000010011 (19)       | 19           | 0b01001 00000010011     | 19 → ALU               |
| 2    | Fetch       | Fetch Instruction to IR     |        |                        |              |                         |                        |
| 3    | add_ia      | ALU A (reg) + B (immediate) | 01010  | 00000010100 (20)       | 20           | 0b01010 00000010100     | 10 + 10 → ALU          |
| 4    | Fetch       | Fetch Instruction to IR     |        |                        |              |                         |                        |
| 5    | sub_ia      | ALU A (reg) − B (immediate) | 01011  | 00000001000 (8)        | 8            | 0b01011 00000001000     | jump to addr (6)       |
| 6    | Fetch       | Fetch Instruction to IR     |        |                        |              |                         |                        |
| 7    | str         | store value to RAM          | 00110  | 00000001010 (10)       | 31           | 0b00110 00000001010     | store to addr (10)     |
| 8    | Fetch       | Fetch Instruction to IR     |        |                        |              |                         |                        |
| 9    |             |                            |        |                        |              |                         |                        |

> **前面有跑過模擬，這次不講得非常詳細：**

- **Clock 0**  
  - Fetch  
  - ld_ia：`01001 + 00000010011` (19) 從 RAM 取指令  
  
  <img width="1692" height="450" alt="Image" src="https://github.com/user-attachments/assets/b671aac1-401b-4ebc-a8d5-da68d094f3f5" />

- **Clock 1**  
  - Execute  
  - ld_ia：`en_i` 打開 → operand 零擴展 → immediate  

<img width="1488" height="500" alt="Image" src="https://github.com/user-attachments/assets/aae42d66-a4c0-402e-a8b1-b1972b57e28b" />

- **Clock 2**  
  - Fetch  
  - add_ia：`01010 + 00000010100` (20) 取指令  

<img width="1376" height="500" alt="Image" src="https://github.com/user-attachments/assets/c595b3a6-edc3-4989-b085-6c9bf8069b3d" />

- **Clock 3**  
  - Execute  
  - add_ia：`en_i` 打開，`sel_b=11` → ALU 執行 `A + 20`  

<img width="1418" height="500" alt="Image" src="https://github.com/user-attachments/assets/378ae7b5-0e31-489d-ac28-20c174ad869a" />

- **Clock 4**  
  - Fetch  
  - sub_ia：`01011 + 00000001000` (8) 取指令  
  
<img width="1360" height="500" alt="Image" src="https://github.com/user-attachments/assets/32f62e5f-3c91-4208-8bdb-cfc93a842110" />

- **Clock 5**  
  - Execute  
  - sub_ia：`en_i` 打開，`sel_b=11` → ALU 執行 `A − 8`  
  
<img width="1473" height="500" alt="Image" src="https://github.com/user-attachments/assets/60b5cf07-ca2a-4798-b4f9-59511ca86b6c" />

## 總結
個人覺得 immediate 的 data flow 簡單多了，沒啥好總結
