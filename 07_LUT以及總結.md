
# 07_Lookup Table（LUT）以及系統總結

## 1. LUT
- LUT是一個輸入為opcode（5bits），輸出為對應控制信號編碼
- 在Digital裡可以直接編輯，如下：

<img width="1241" height="600" alt="Image" src="https://github.com/user-attachments/assets/713d9e24-810e-461b-b3a5-04f9abe3a797" />

- 實際運用：
    - 指令RAM 16bits → **只取高5bits作為opcode**  
    - opcode → LUT → 所有控制訊號

<img width="898" height="500" alt="Image" src="https://github.com/user-attachments/assets/1c953800-9f89-492b-9109-b0605642325d" />

- 修改原本的電路結構，如下：

<img width="1879" height="400" alt="Image" src="https://github.com/user-attachments/assets/0c279c23-33e3-48e7-b96f-e65f104a9c3f" />

- 可以把LUT看成一個控制中心，我們把所有的訊號都放在LUT裡面，統一在LUT做控制以及調整。

---

## 2. 合併資料RAM與指令RAM

- 目前的設計，指令RAM與資料RAM為兩套獨立記憶體，程序計數器(PC)同時作為兩者的地址來源，這一點都不靈活，指令跟資料只能接受同樣的地址，同時資料只能被循序的使用，為何要這樣?
- 合併後，**一個記憶體同時存放指令與資料**，根據不同操作型態(取指或取數據)切換狀態
- 這正是**馮諾伊曼(Von Neumann)架構**的基礎：**程式與資料共用同一個記憶體體系**
- 一個完整的指令除了指令本身(==opcode==)，還需要包含指令想要操作的對象地址，我們稱為「==operand==」

---

## 3. 現有系統運作回顧

1. **程序計數器(PC)**  
   4位元，產生當前位址給記憶體

2. **同地址雙用途**  
   - PC同時尋址指令RAM與資料RAM

3. **指令解碼與控制訊號**  
   - PC尋址，從記憶體讀出指令opcode，經LUT展開為所有控制信號，執行對應操作。

4. **指令驅動資料執行**  
   - 指令本身直接控制ALU、暫存器、資料RAM行為。

5. **資料RAM輸出用途**  
   資料RAM的輸出有以下兩個主要用途：  
   - 作為資料輸出使用。  
   - 當執行跳轉指令（如 `jmp`、`je`）時，作為跳轉目標地址

6. **算術邏輯單元（ALU）輸入控制**  
   - ALU 的輸入A來自暫存器A或常數0，透過信號 `selA` 控制。  
   - 輸入B來自資料RAM或常數0，透過信號 `selB` 控制。
  
7. **比較暫存器的功能**  
   比較暫存器存儲 ALU 輸入 A 與 B 的比較結果：  
   - 相等時存入1，  
   - 不相等時存入0。  
   該比較結果用於條件跳轉指令 `je` 的判斷依據。

---

## 4. 指令編碼說明與操作總覽

### 4.1 Opcode對應操作表

| 操作名稱 | Opcode（二進位） | 操作說明                          |
|----------|------------------|---------------------------------|
| halt     | 00000            | 停止clock                        |
| ld_a     | 00001            | 將資料RAM中的資料載入暫存器A    |
| add      | 00010            | 將資料RAM的資料與暫存器A相加，存回A |
| sub      | 00011            | A減去資料RAM的資料，存回A       |
| or       | 00100            | 對A與資料RAM做位元或，存回A     |
| and      | 00101            | 對A與資料RAM做位元且，存回A     |
| str      | 00110            | 將A存回資料RAM                  |
| jmp      | 00111            | 無條件跳轉至指定地址            |
| je       | 01000            | ZF=1時條件跳轉至指定地址        |

---
## 5. 系統彈性與優化思考

- **現有方式限制**：PC指到哪，指令與資料RAM只能對應操作同一地址，前面說過，jmp是用來支援function call的，但在現在的系統中，如果資料不想循序操作，想對某處的資料做操作，就會需要用jmp，這不合理
- **改良1**：希望指令可對任意RAM地址操作，也就是「指令需自帶目標資料地址」（operands）
    - 現在是不是跟寫組語越來越像了?拿C語言做舉例:
    c = a + b ;
    - 當你使用指令+時(假設他是基本指令)，a、b很顯然是指向某個RAM地址，裡面存了一個值。
    - 也就是說，指令原本只含: 指令RAM地址、opcode(操作本身)
    - 現在改成: 指令暫存器地址、opcode、資料RAM地址
    - 這個資料RAM地址，我們稱為==operands== 
    - 操作數(operand)位元利用opcode以外的11bits表示
    - 執行流程：先fetch指令，再根據指令自帶地址讀寫資料
- **改良2**：兩塊RAM合併
---

## 6. 狀態機與時序優化

- 將CPU運作劃分為「取指」(fetch)與「執行」(execute)兩個狀態。
- 原因 : RAM的輸出線只有一根，也就是同時間沒辦法輸出指令跟資料，我們先取指再執行，每做完一組週期，PC才+1，取指是把指令拿出來放到某個地方，要再加一個Instruction Register(==IR==)，取指時RAM輸出指令，執行時RAM輸出資料。
- 狀態設計 : 利用一個反向flipflop，在clock的每個邊沿交替切換狀態，實現兩階段運作，==兩個clock才是一次完整的行為==

<img width="1259" height="200" alt="Image" src="https://github.com/user-attachments/assets/b629ab03-680e-4752-91e8-b642a3fd6f2b" />

---

## 7. 馮諾伊曼模型

- 合併記憶體設計後，架構向經典的**Von Neumann（馮諾伊曼）模型**靠近：
    - **指令與資料同存於一個記憶體空間**
    - **不同的CPU狀態**
    
<img width="1118" height="450" alt="Image" src="https://github.com/user-attachments/assets/d0bc5b05-c726-43e8-8a4b-c99d78aa8345" />

- **取指與執行時序圖**：
    
<img width="900" height="400" alt="Image" src="https://github.com/user-attachments/assets/010f9018-7da0-42ae-8a9b-704a2c7fb436" />

<img width="900" height="400" alt="Image" src="https://github.com/user-attachments/assets/f93c74ad-0bb9-492f-ac03-75d3059144b3" />

---

**小結：**
目前我們有PC、IR、Control Unit(LUT)、ALU、Register、RAM，跟典型的CPU處理器架構已經非常接近了，繼續加油。