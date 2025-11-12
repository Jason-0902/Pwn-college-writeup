# Magic Numbers (C)

## Challenge Description 

程式 `/challenge/cimg` 是一個 C 語言編寫的影像解析器，會驗證輸入的 `.cimg` 檔案是否正確
若 magic number 正確，它會印出 flag；若錯誤，則輸出：


我們的任務：  
找出正確的 magic number（4 bytes），構造 `.cimg` 檔案並通過驗證。

---

## Walkthrough

1. 檢查檔案型態

```bash
file /challenge/cimg
```

結果：
```bash
/challenge/cimg: ELF 64-bit LSB executable, x86-64, dynamically linked
```

2. 搜尋字串

```bash
strings /challenge/cimg | grep -i error
```

結果：

```bash
ERROR: Invalid file extension!
ERROR: Failed to read header!
ERROR: Invalid magic number!
```

可以推測程式檢查三件事：

1. 副檔名是否為 .cimg

2. 是否成功讀取 header（前 4 bytes）

3. 是否符合正確的 magic number

### GDB 動態分析

#### 設斷點在`main()`

程式的檔案驗證邏輯幾乎一定在`main`裡：

```bash
gdb /challenge/cimg
(gdb) break main
(gdb) run test.cimg
```

程式停在`main`

#### 嘗試設斷點在`memcmp`

部分程式會用`memcmp()`直接比對magic number:

```bash
(gdb) break memcmp
(gdb) continue
```

但這題沒有觸發，代表它可能逐byte比對

#### 改斷在 puts

我們知道錯誤訊息是由`puts()`印出的：

```bash
(gdb) break puts
(gdb) run test.cimg
```

程式停下來:

```bash
Breakpoint 1, __GI__IO_puts (str=0x402135 "ERROR: Invalid magic number!") at ioputs.c:33
```

這說明程式正準備印出錯誤訊息

#### 查看呼叫堆疊(誰叫的`puts()`)

```
(gdb) backtrace
#0  puts("ERROR: Invalid magic number!")
#1  main ()
```

切換回上一層:

```bash
(gdb) frame 1
```

#### 反組譯`main()`

```bash
(gdb) disas
```

結果顯示:

```asm
0x00000000004015da <+218>:   movzbl -0x1c(%rbp),%eax
0x00000000004015de <+222>:   cmp    $0x3c,%al       ; byte 1
0x00000000004015e2 <+226>:   movzbl -0x1b(%rbp),%eax
0x00000000004015e6 <+230>:   cmp    $0x4f,%al       ; byte 2
0x00000000004015ea <+234>:   movzbl -0x1a(%rbp),%eax
0x00000000004015ee <+238>:   cmp    $0x25,%al       ; byte 3
0x00000000004015f2 <+242>:   movzbl -0x19(%rbp),%eax
0x00000000004015f6 <+246>:   cmp    $0x72,%al       ; byte 4
```

### 解析Magic Number

程式逐byte檢查輸入內容:

| Offset | Hex  | ASCII |
| ------ | ---- | ----- |
| 0      | 0x3C | `<`   |
| 1      | 0x4F | `O`   |
| 2      | 0x25 | `%`   |
| 3      | 0x72 | `r`   |


因此：

Magic number = `<O%r`
Hex = `\x3c\x4f\x25\x72`

### 建立`.cimg`檔案

```bash
echo -ne '\x3c\x4f\x25\x72' > valid.cimg
```

