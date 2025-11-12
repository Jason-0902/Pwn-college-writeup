# File Formats & Magic Numbers (Python Version)

## Challenge Description 

在本題中，我們要從一個二進位可執行檔 /challenge/cimg 中，分析出自定義圖片格式 .cimg 的 magic number。這個程式會驗證輸入檔案的開頭是否包含正確的 magic number，如果正確，就會印出 flag

## Walkthrough

1. 初步偵查

執行 file 指令查看 /challenge/cimg 是什麼檔案：

````bash
$ file /challenge/cimg
/challenge/cimg: setuid Python script, ASCII text executable
````

結果顯示： 這不是 ELF 可執行檔，而是一個用 Python 撰寫的 setuid 腳本

2. 查看程式內容

因為是 Python 腳本，我們可以直接用`cat`或`less`觀看其程式碼：

```SHELL
$ cat /challenge/cimg
```

找到關鍵邏輯如下：

```Python
header = file.read1(4)
assert len(header) == 4, "ERROR: Failed to read header!"
assert header[:4] == b"(~m6", "ERROR: Invalid magic number!"
```

這表示程式會從檔案開頭讀 4 個 byte，並且要求完全等於 `b"(~m6"`

3. 解讀 magic number

Python 字串`b"(~m6"`對應的十六進位bytes為：

```
\x28\x7e\x6d\x36
```

4. 製作正確的 .cimg 檔案

使用`echo -ne`製作出正確的magic number二進位檔案：

```bash
$ echo -ne '\x28\x7e\x6d\x36' > valid.cimg
```

執行挑戰程式

將檔案作為參數傳入`/challenge/cimg`：

```bash
$ /challenge/cimg valid.cimg
```

成功印出：
```
pwn.college{XXXXXXXX...}
```