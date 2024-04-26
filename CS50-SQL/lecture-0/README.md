# CS50 SQL 筆記

## 課前須知

本課是使用 SQLite 來進行練習，怎麼安裝可以參考[這裡](https://www.runoob.com/sqlite/sqlite-installation.html)

## **Lecture 0**

會使用到 `longlist.db` 這個檔案，我已經預先下載了，原本的下載網址在[這邊](https://cdn.cs50.net/sql/2023/x/lectures/0/src0/)



- 課程沒有先教，但我覺得應該要知道如何查看 table 的資訊

   ```sql
   PRAGMA table_info(longlist);
   ```

- 進入資料庫，由於資料庫名稱為 `longlist.db` ，所以指令是

   ```sql
   sqlite3 longlist
   ```

- 離開資料庫

   ```sql
   .quit
   ```

### **SELECT**

- 搜尋所有資料，已知 table name 是 longlist

   ```sql
   SELECT * FROM longlist;
   ```

- 檢索的特定欄位

   ```sql
    SELECT title FROM longlist;
   ```

- 搜尋多個特定欄位

   ```sql
   SELECT title, author FROM longlist; 
   ```

### **LIMIT**

- 限制回傳筆數

   ```sql
   SELECT title FROM longlist LIMIT 5;
   ```

- 限制並跳過前幾筆

   ```sql
   SELECT title FROM longlist LIMIT 5 OFFSET 3;
   ```

### **WHERE**

- 篩選相等

   ```sql
   SELECT title, author FROM longlist WHERE year = 2021;
   ```

- 篩選不相等

   ```sql
   SELECT title, author FROM longlist WHERE year != 2021;
   ```

   或是

   ```sql
   SELECT title, author FROM longlist WHERE year <> 2021;
   ```

- 篩選條件相反

   ```sql
   SELECT title, author FROM longlist WHERE NOT year = 2021;
   ```

- 篩選多個條件交集

   ```sql
   SELECT title, author FROM longlist WHERE year = 2021 AND year = 2022;
   ```

- 篩選多個條件聯集

   ```sql
   SELECT title, author FROM longlist WHERE year = 2021 OR year = 2022;
   ```

- 設定條件執行順序

   ```sql
   SELECT title, author FROM longlist WHERE (year = 2021 OR year = 2022) AND format != 'hardcover';
   ```

### **NULL**

> 用來代表空值，這是唯一值，要用 `is` 去做判斷

- 判斷是空值

   ```sql
   SELECT title FROM longlist WHERE translator IS NULL;
   ```

- 判斷不是空值

   ```sql
   SELECT title FROM longlist WHERE translator IS NOT NULL;
   ```

### **LIKE**

> 用來模式匹配， % 用來匹配多個字元，\_ 用來匹配一個字元

- 匹配結果不論大小寫

   ```sql
   SELECT title FROM longlist WHERE title LIKE 'pyre';
   ```

- 匹配中間有指定自串的內容

   ```sql
   SELECT title FROM longlist WHERE title LIKE '%LOVE%';
   ```

- 匹配開頭有指定自串的內容

   ```sql
   SELECT title FROM longlist WHERE title LIKE 'THE %';
   ```

- 匹配結尾有指定自串的內容

   ```sql
   SELECT title FROM longlist WHERE title LIKE '% Bed';
   ```

- 混合

   ```sql
   SELECT title FROM longlist WHERE title LIKE 'The %of% Language';
   ```

- 匹配字元

   ```sql
   SELECT title FROM longlist WHERE title LIKE 'W___e';
   ```

- 兩種混合

   ```sql
   SELECT title FROM longlist WHERE title LIKE 'Go%G___';
   ```

### **Range Conditions**

- 大於

   ```sql
   SELECT title FROM longlist WHERE year > 2011;
   ```

- 小於

   ```sql
   SELECT title FROM longlist WHERE year < 2011;
   ```

- 大於等於

   ```sql
   SELECT title FROM longlist WHERE year >= 2011;
   ```

- 小於等於

   ```sql
   SELECT title FROM longlist WHERE year <= 2011;
   ```

- 介於

   ```sql
   SELECT title FROM longlist WHERE year BETWEEN 2019 AND 2022;
   ```

### **Order By**

> 用欄位數值進行排序

- 排序，從小到大

   ```sql
   SELECT title, rating FROM longlist ORDER BY rating LIMIT 10;
   ```

- 排序，從大到小

   ```sql
   SELECT title, rating FROM longlist ORDER BY rating DESC LIMIT 10;
   ```

- 用多個欄位進行排序

   ```sql
   SELECT title, rating FROM longlist ORDER BY rating DESC,votes DESC LIMIT 10;
   ```

### Aggregate

將欄位的數值聚合變成單一的數值

- 計算數量

   ```sql
   SELECT COUNT(title) FROM longlist;
   ```

- 平均

   ```sql
   SELECT AVG(rating) FROM longlist;
   ```

- 取整

   ```sql
   SELECT ROUND(AVG(rating),2) FROM longlist;
   ```

- 最大值

   ```sql
   SELECT MAX(rating) FROM longlist;
   ```

- 最小值

   ```sql
   SELECT MIN(rating) FROM longlist;
   ```

- 加總

   ```sql
   SELECT SUM(votes) FROM longlist;
   ```

### Rename

```sql
SELECT ROUND(AVG(rating),2) AS round FROM longlist;
```

### 不重複

```sql
SELECT DISTINCT(publisher) FROM longlist;
```
