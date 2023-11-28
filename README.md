# MASM Coding Style Guides

---
## **大綱**
+ [**Highlighting**](https://github.com/RogelioKG/MASM-Coding-Styles#highlighting)
+ [**命名**](https://github.com/RogelioKG/MASM-Coding-Styles#%E5%91%BD%E5%90%8D)
+ [**書寫習慣**](https://github.com/RogelioKG/MASM-Coding-Styles#%E6%9B%B8%E5%AF%AB%E7%BF%92%E6%85%A3)
+ [**對齊**](https://github.com/RogelioKG/MASM-Coding-Styles#%E5%B0%8D%E9%BD%8A)
+ [**建議**](https://github.com/RogelioKG/MASM-Coding-Styles#%E5%BB%BA%E8%AD%B0)
+ [**子程序**](https://github.com/RogelioKG/MASM-Coding-Styles#%E5%AD%90%E7%A8%8B%E5%BA%8F)

---
## **Highlighting**
使用 AsmDude2022 (v1.9.8.0) 作為 highlighting 與代碼提示的工具，以下是語法高亮顏色規範：

![alt=highlighting-1](https://raw.githubusercontent.com/RogelioKG/MASM-Coding-Styles/main/highlighting-1.png)

![alt=highlighting-2](https://raw.githubusercontent.com/RogelioKG/MASM-Coding-Styles/main/highlighting-2.png)

---
## **命名**

- **Mnemonic**
    > 全小寫
- **Directive**
    > 有點全小寫 (`.data` / `.code` / `.while` / `.if` …)，其他全大寫 (`DWORD` / `INVOKE`)
- **變數名稱 / 跳躍標籤**
    > `snake_case` (單字間底線區隔，單字首字母小寫)
- **子程序標籤**
    > `PascalCase` (單字首字母大寫)

---
## **書寫習慣**

- **縮排為 4 個空格**
    ```nasm
    .if nChars <= ecx
        sub ecx, nChars
    .endif
    ```
    你可以使用 TAB 排版，但你須確保他們輸出的是多個空白字元，而非製表字元，你可以在這裡調整設定：
    ```
    Tools->Options->Text Editor->All Languages->Tabs
    ```
    ![alt=tabs](https://raw.githubusercontent.com/RogelioKG/MASM-Coding-Styles/main/tabs.png)
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
- **標籤須統一置於 .data 上方**
    ```nasm
    ; label
    BoxWidth  = 7
    BoxHeight = 7
    
    .data
    boxTop    BYTE    0DAh, (BoxWidth - 2) DUP(0C4h), 0BFh
    boxBody   BYTE    0B3h, (BoxWidth - 2) DUP(' '), 0B3h
    boxBottom BYTE    0C0h, (BoxWidth - 2) DUP(0C4h), 0D9h
    ```
---
## **對齊**

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
    target2    BYTE    "111303545", 0
    target3    BYTE    "999999999", 0
    num        SBYTE   -1
    value      DWORD   ?
    ```
- **命令列 / 註解**
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
    ; ----------------------------
    ; Name:
    ;     FastPow
    ; Brief:
    ;     快速冪
    ; Uses:
    ;     eax ebx edx
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
---
## **建議**

- **指令分群**

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

---
## **子程序**

- **Docstring 項目**
    > 如果項目為空，可以使用 `...` 結束這回合
    - **Name**
      > 子程序名稱
    - **Brief**
      > 子程序功能 (簡短介紹)
    - **Uses**
      > 子程序使用到哪些暫存器 (用 USES 關鍵字的暫存器也要列進來)
    - **Params**
      > 子程序使用到哪些參數 (對齊方式如下)
    - **Returns**
      > 子程序以哪個暫存器作為值的回傳
    - **Example** (optional)
      > 舉個栗子

- **範例1**
    ```nasm
    ; ------------------------------------------------
    ; Name:
    ;     DifferentInputs
    ; Brief:
    ;     比較三個數，若任兩個皆異則回傳 true，否則回傳 false
    ; Uses:
    ;     eax
    ; Params:
    ;     v1 = DWORD 數值1
    ;     v2 = DWORD 數值2
    ;     v3 = DWORD 數值3
    ; Returns:
    ;     eax = 任兩個皆異則回傳 true，否則回傳 false
    ; ------------------------------------------------
    DifferentInputs PROC, v1:DWORD, v2:DWORD, v3:DWORD

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

    label_equal:            ; 存在相等
        mov  eax, 0         ; return false

    label_exit:             ; 離開
        call DumpRegs       ; 印暫存器的值
        ret

    DifferentInputs ENDP
    ```

- **範例2**
    ```nasm
    ; -------------------------------------------
    ; Name:
    ;     StrRemove
    ; Brief:
    ;     移除一個字串中的子字串 (可跨字串)
    ; Uses:
    ;     eax ecx esi edi
    ; Params:
    ;     pStart = PTR BYTE 指向第一個要刪除的字元
    ;     nChars = DWORD    欲刪除字元數
    ; Returns:
    ;     ...
    ; Example:
    ;     字串 string 為 "012345678"
    ;     INVOKE StrRemove, OFFSET [string + 2], 5
    ;     意思是從 2 開始，移除 5 個字元
    ;     字串 string 應變為 "0178"
    ; -------------------------------------------
    StrRemove PROC, pStart:PTR BYTE, nChars:DWORD

        INVOKE Str_length, pStart   ; 獲得自 pStart 開始字串長度

                                    ; @ 設定迴圈次數
        mov ecx, eax                ; loop 次數 = 自 pStart 開始字串長度 (以 Example 為例，為 7)
        .if nChars <= ecx           ; 如果 (欲刪除字元數 < 自 pStart 開始的字串長度)
            sub ecx, nChars         ; loop 次數 = 自 pStart 開始字串，移除子字串後剩下的長度 (以 Example 為例，為 2)
        .endif                      ;

                                    ; @ 此處開始複製字串
        mov esi, pStart             ; 
        add esi, nChars             ; 來源位址 = pStart + nChars
        mov edi, pStart             ; 目標位址 = pStart
        cld                         ; 
        rep movsb                   ; BYTE 複製，複製 ecx 次
        mov BYTE PTR [edi], 0       ; 目標位址此時會在最後一個字元的後一個位置，我們補上空字元，以標示字串的結束

        ret

    StrRemove ENDP

    END main
    ```
