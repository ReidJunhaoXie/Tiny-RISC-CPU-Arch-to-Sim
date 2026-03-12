# 09_初次CPU模擬
在前一章完成後，CPU的雛型已經基本完成，我們一整章來跑一次模擬 !

---
## 1. 程式規劃

| clock | instruction | Instruction Description       | opcode | operand(操作目標地址) | data(操作值) | whole instruction(system control)  |                   |
|------|-------------|-------------------------------|--------|-----------------|-----------|--------------------|-------------------|
| 0    | Fetch       | Fetch Instruction to IR       |        |                 |           | opcode+operand     |                   |
| 1    | ld_a        | load data to ALU reg          | 00001  | 00000001000     | 10        | 0b0000100000001000 | 10->ALU           |
| 2    | Fetch       | Fetch Instruction to IR       |        |                 |           | 0b                 |                   |
| 3    | add         | ALU A(reg)+B(data by operand) | 00010  | 00000001001     | 10        | 0b0001000000001001 | 10+10->ALU        |
| 4    | Fetch       | Fetch Instruction to IR       |        |                 |           |                    |                   |
| 5    | jmp         | Jump to somewhere             | 00111  | 00000000110     |           | 0b0011100000000110 | jump to addr(6)   |
| 6    | Fetch       | Fetch Instruction to IR       |        |                 |           |                    |                   |
| 7    | str         | strore value to RAM           | 00110  | 00000001010     |           | 0b0011000000001010 | store to addr(10) |
| 8    | Fetch       | Fetch Instruction to IR       |        |                 |           |                    |                   |
| 9    |             |                               |        |                 |           |                    |                   |

---

## 2. RAM 指令與資料初始化

- 在Digital裡編輯RAM，**依照指令流程填入指令與對應的資料**：
  
  <img width="1020" height="300" alt="Image" src="https://github.com/user-attachments/assets/fff6e17e-d930-4a38-a439-872350075fc6" />
  
- 例如：0x808 表示 `ld_a` 指令、0x1009 表示 `add` 指令等，後方存放操作數據。

---

## 3. CPU工作時序

> 上電（模擬啟動）後，CPU會**自動交替進行取指（Fetch）與執行（Execute）狀態**，每一個clock狀態切換一次。

### 基本規則
- **active high訊號**：一開始所有enable類控制訊號皆預設為1。
- **狀態循環**：fetch → execute → fetch → execute ...（每一個clock交替）
- **流程關鍵字**：
    - **Fetch**：PC指向RAM讀出指令，指令在Dout，在所有的Fetch階段，system control code都是一樣的。
    - **Execute**：指令進IR，內容被解碼，根據opcode、state、cmp啟動LUT控制行為，如資料運算、跳轉、寫回。

---

## 4. 實際模擬流程（每個clock行為）

    
### 剛上電 - 模擬開始
- clock 0
- Fetch state
- 初始化這個時間點IR裡面存的是0，因此input code(op+cmp_q+state)會輸出0b0000000，電路行為halt、en_ir、ld、en_pc為1其他為零。
- PC=0，且en_pc為1，指向RAM第一條指令，0x808(ld_a指令)在RAM輸出位置，未進IR。

<img width="1670" height="500" alt="Image" src="https://github.com/user-attachments/assets/b3ae8f17-2c2b-4024-a743-59879ce0a9d2" />

### clock 1
- Execute state
- 指令0x808進入IR，並分解成operand跟opcode，opcode此時是執行ld_a，operand指向RAM第八個位置，裡面存的是預先存進去的資料10。
- 資料10(A)目前在ALU reg的D，隨下一個clock進入。

<img width="1607" height="500" alt="Image" src="https://github.com/user-attachments/assets/28538d32-96c6-4f2f-b598-4637314c0d0f" />

### clock2
- Fetch state
- PC = 1，指向RAM的第二個位置
- 指令0x1009(add)在RAM的輸出位置
- 目前IR存的是上一個指令，但任何指令Fetch state行為都一致，halt、en_ir、ld、en_pc為1其他為零。

<img width="1727" height="500" alt="Image" src="https://github.com/user-attachments/assets/09e0a935-17b4-471e-8eaa-efa34d4cbe03" />
    
### clock3
- Execute state
- 指令0x1009進入IR，並分解成operand跟opcode，opcode此時是執行add，operand指向RAM第九個位置，裡面存的是預先存進去的資料10(A)。
- 目前ALU Reg已經有10，RAM load出來的值與ALU Reg的值相加(0x14；20)，會隨下一個clock存進ALU Reg。

<img width="1539" height="500" alt="Image" src="https://github.com/user-attachments/assets/a1536e42-15ab-4e6d-b9bf-906f7c35ee19" />
    
### clock4
- Fetch state
- PC = 2，指向RAM的第二個位置
- 指令0x3806(jmp)在RAM的輸出位置
- Fetch state行為一致，halt、en_ir、ld、en_pc為1其他為零。

<img width="1442" height="500" alt="Image" src="https://github.com/user-attachments/assets/0c3b14f5-67c6-49d0-829e-09433e3024b3" />


### clock5
- Execute state
- 指令0x3806進入IR，並分解成operand跟opcode，opcode此時是執行jmp，ld_pc被打開，operand直接修改PC的值為6，這個位置是想跳去的指令位置。

<img width="1591" height="500" alt="Image" src="https://github.com/user-attachments/assets/73072f3d-74c5-41d6-beee-c10fb63089f0" />

### clock6
- Fetch state
- PC = 6，指向RAM的第六個位置
- 指令300A(opcode=00110；str)在RAM的輸出位置
- Fetch state行為一致，halt、en_ir、ld、en_pc為1其他為零。

<img width="1494" height="500" alt="Image" src="https://github.com/user-attachments/assets/e9e72a2c-1cdc-419d-a9c0-fee606151777" />

### clock7
- Execute state
- 指令300A(str)進入IR，並分解成operand跟opcode，opcode此時是執行str，operand指向RAM中想存的格子A(10)。

<img width="1483" height="500" alt="Image" src="https://github.com/user-attachments/assets/d8ae0739-9450-4591-b030-c72ecacd5880" />

### clock8
- Fetch
- 下一個指令為0(halt)

<img width="1577" height="500" alt="Image" src="https://github.com/user-attachments/assets/cf02a66b-aa4f-4c63-882d-866106f778c3" />

### clock9
- Execute state
- 隨halt執行，CPU關機。

<img width="1534" height="500" alt="Image" src="https://github.com/user-attachments/assets/e9d70475-0c91-44a4-bf61-cf9d4683b626" />

---

## 4. 小結

- 硬體基礎模擬跟驗證完成，就要開始編組語了 ! 終於 ! 


