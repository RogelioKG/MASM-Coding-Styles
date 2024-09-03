# MASM Coding Style Guides

<!-- link -->
[0]:#️-大綱
[1]:#️-highlighting
[2]:#️-命名
[3]:#️-書寫習慣
[4]:#️-對齊
[5]:#️-建議
[6]:#️-子程序
[7]:#️-模組化


## [▶️][0] **大綱**
+ [**Highlighting**][1]
+ [**命名**][2]
+ [**書寫習慣**][3]
+ [**對齊**][4]
+ [**建議**][5]
+ [**子程序**][6]
+ [**模組化**][7]


## [⬆️][0] **Highlighting** 
使用 AsmDude2022 (v1.9.8.0) 作為 highlighting 與代碼提示的工具，以下是語法高亮顏色規範：

![highlighting-1](https://raw.githubusercontent.com/RogelioKG/MASM-Coding-Styles/main/image/highlighting-1.png)

![highlighting-2](https://raw.githubusercontent.com/RogelioKG/MASM-Coding-Styles/main/image/highlighting-2.png)


## [⬆️][0] **命名** 

- **Mnemonic**
    > 全小寫
- **Directive**
    > 有點全小寫 (`.data` / `.code` / `.while` / `.if` …)，其他全大寫 (`DWORD` / `INVOKE`)
- **變數名稱 / 跳躍標籤**
    > `snake_case` (單字間底線區隔，單字首字母小寫)
- **子程序名稱**
    > `PascalCase` (單字首字母大寫)
- **巨集名稱**
    > `mPascalCase` (開頭必為m，單字首字母大寫)


## [⬆️][0] **書寫習慣** 

- **縮排為 4 個空格**
    ```nasm
    .if nChars <= ecx
        sub ecx, nChars
    .endif
    ```
    你可以使用 TAB 排版，但你須確保他們輸出的是多個空白字元，而非製表字元，你可以在這裡調整設定：
    > 選擇插入空格
    ![tabs1](https://github.com/RogelioKG/MASM-Coding-Styles?tab=readme-ov-file/blob/main/image/tabs1.png)
    > 不使用自適性格式化
    ![tabs2](https://github.com/RogelioKG/MASM-Coding-Styles?tab=readme-ov-file/blob/main/image/tabs2.png)
- **逗號後必有一個空格**
    ```nasm
    INVOKE StrRemove, OFFSET target1, 5
    ```
- **冒號後不能有空格**
    ```nasm
    StrRemove    PROTO, pStart:PTR BYTE, nChars:DWORD
    ```
- **註解分號後必有一個空格**
    ```nasm
    mov  eax, v1        ; 取出 v1
    cmp  v2, eax        ; 與 v2 做比較
    ```
- **struct 內須頭尾有空格鋪墊**
    ```nasm
    .data
    point   COORD < 1, 2 >
    ```
- **symbol 須統一置於 .data 上方**
    ```nasm
    ; symbol
    BoxWidth  = 7
    BoxHeight = 7
    
    .data
    boxTop    BYTE    0DAh, (BoxWidth - 2) DUP(0C4h), 0BFh
    boxBody   BYTE    0B3h, (BoxWidth - 2) DUP(' '), 0B3h
    boxBottom BYTE    0C0h, (BoxWidth - 2) DUP(0C4h), 0D9h
    ```


## [⬆️][0] **對齊**

- **名稱 / PROTO / 參數**
    ```nasm
    ExitProcess     PROTO, dwExitCode:DWORD
    DifferentInputs PROTO, v1:DWORD, v2:DWORD, v3:DWORD
    StrRemove       PROTO, pStart:PTR BYTE, nChars:DWORD
    ```
- **名稱 / 型態 / 值**
    ```nasm
    .data
    target1    BYTE    "111302502", 0
    num        SBYTE   -1
    value      DWORD   ?
    ```
- **命令 / 註解**
    > 僅需區域對齊 (在一個子程序裡皆對齊即可)
    ```nasm
                        ; @ 指令在左邊，註解在右邊
    mov  eax, v1        ; 取出 v1
    cmp  v2, eax        ; 與 v2 做比較
    je   label_equal    ; 若相等則跳到 label_equal
    cmp  v3, eax        ; 與 v3 做比較
    je   label_equal    ; 若相等則跳到 label_equal
    mov  eax, v2        ; 取出 v2
    cmp  v3, eax        ; 與 v3 做比較
    je   label_equal    ; 若相等則跳到 label_equal
    mov  eax, 1         ; return true
    jmp  label_exit     ; 跳到 label_exit 離開
    ```
- **指令 / 運算元**
    > 巢狀區塊的各區塊也須對齊，但條件假指令不在此限
    ```nasm
    FastPow PROC, x:DWORD, n:DWORD
        mov edx, 0
        mov eax, 1              ; eax = 1
        mov ebx, x              ; ebx = base
        .while n != 0
            .if n & 1           ; 如果最低位元為 1         (此時 eax = result，ebx = base)
                mul ebx         ; result = result * base  (此時 eax = result * base，ebx = base)
            .endif
            xchg eax, ebx       ; base 與 result 交換     (若有進入 .if，此時 eax = base，ebx = result * base)
            mul  eax            ; eax = base * base      (若有進入 .if，此時 eax = base * base，ebx = result * base)
            xchg eax, ebx       ; base 與 result 交換     (若有進入 .if，此時 eax = result * base，ebx = base * base)
            shr  n, 1           ; 吃掉最低位元
        .endw
        ret
    FastPow ENDP

    END main
    ```


## [⬆️][0] **建議**

- **命令分群**

    > 可在群的前一行註解，說明群的主要功能，並加上 `@` 符號以示區分。

    ```nasm
                                ; @ 設定迴圈次數
    mov ecx, eax                ; loop 次數 = 自 pStart 開始字串長度 (以 Example 為例，為 7)
    .if nChars <= ecx           ; 如果 (欲刪除字元數 < 自 pStart 開始的字串長度)
        sub ecx, nChars         ; loop 次數 = 自 pStart 開始字串，移除子字串後剩下的長度 (以 Example 為例，為 2)
    .endif                      ;
    ```

- **不須每行都有註解**

    > 但建議一個群中，沒有註解的行，仍須有 `;` 作為連續象徵。這避免一個群的註解過於破碎。

    ```nasm
                                ; @ 此處開始複製字串
    mov esi, pStart             ; 
    add esi, nChars             ; 來源位址 = pStart + nChars
    mov edi, pStart             ; 目標位址 = pStart
    cld                         ; 
    rep movsb                   ; BYTE 複製，複製 ecx 次
    mov BYTE PTR [edi], 0       ; 目標位址此時會在最後一個字元的後一個位置，我們補上空字元，以標示字串的結束
    ```


## [⬆️][0] **子程序** 

- **Docstring 項目**
    > 如果項目為空，可以使用 `...` 結束這回合
    - **Name**
      > 子程序名稱
    - **Brief**
      > 子程序功能 (簡短介紹)
    - **Uses**
      > 子程序使用到哪些暫存器 (不含回傳)
    - **Params**
      > 子程序使用到哪些參數 (對齊方式如下)
    - **Returns**
      > 子程序以哪個暫存器作為值的回傳
    - **Example** (optional)
      > 舉個栗子

- **範例**
    ```nasm
    ; ----------------------------
    ; Name:
    ;     FastPow
    ; Brief:
    ;     快速冪
    ; Uses:
    ;     ebx edx
    ; Params:
    ;     x = DWORD 底數 (base)
    ;     n = DWORD 指數 (power)
    ; Returns:
    ;     eax = base 的 power 次方
    ; Example:
    ;     INVOKE FastPow, 2, 5
    ;     eax 應為 32
    ; ----------------------------
    FastPow PROC, x:DWORD, n:DWORD
        mov edx, 0
        mov eax, 1              ; eax = 1
        mov ebx, x              ; ebx = base
        .while n != 0
            .if n & 1           ; 如果最低位元為 1         (此時 eax = result，ebx = base)
                mul ebx         ; result = result * base  (此時 eax = result * base，ebx = base)
            .endif
            xchg eax, ebx       ; base 與 result 交換     (若有進入 .if，此時 eax = base，ebx = result * base)
            mul  eax            ; eax = base * base      (若有進入 .if，此時 eax = base * base，ebx = result * base)
            xchg eax, ebx       ; base 與 result 交換     (若有進入 .if，此時 eax = result * base，ebx = base * base)
            shr  n, 1           ; 吃掉最低位元
        .endw
        ret
    FastPow ENDP

    END main
    ```

    
## [⬆️][0] **模組化**

- **規範**
    ```nasm
    ; third party library
    INCLUDE Irvine32.inc
    
    ; local library                 <- 如果是 library，根據 third party / local 分類即可
    INCLUDE library_def.inc

    ; from lab1.asm import          <- 如果是 extern，從哪裡 import 要寫清楚
    EXTERN Func1_1@0:PROC
    Func1_1 PROTO
    EXTERN FastPow@8:PROC            ; EXTERN / PROTO 都要寫
    FastPow PROTO, x:DWORD, n:DWORD  ; EXTERN / PROTO 都要寫

    ; from lab2.asm import
    EXTERN Func2_1@0:PROC
    Func2_1 PROTO

    ; from lab3.asm import
    EXTERN string3:BYTE
    EXTERN number3:ABS
    ```
