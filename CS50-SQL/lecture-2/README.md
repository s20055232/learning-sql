# CS50 SQL 筆記

此份筆記延續[Lecture 1](../lecture-1/README.md)記得先去查看

## Lecture 2: Designing

這邊先使用 lecture 0 的 longlist.db

在 SQLite 中要查看 schema 的資訊

```sql
.schema
```

會看到關於欄位的名稱以及欄位所儲存的資料型態

```sql
CREATE TABLE IF NOT EXISTS "longlist" (
    "isbn" TEXT,
    "title" TEXT,
    "author" TEXT,
    "translator" TEXT,
    "format" TEXT,
    "pages" INTEGER,
    "publisher" TEXT,
    "published" TEXT,
    "year" INTEGER,
    "votes" INTEGER,
    "rating" REAL
);
```

接著我們到 lecture 1 的 longlist.db，一樣查看 schema 資訊，可以看到不同的 table 的資訊

```sql
CREATE TABLE IF NOT EXISTS "authors" (
    "id" INTEGER,
    "name" TEXT,
    "country" TEXT,
    "birth" INTEGER,
    PRIMARY KEY("id")
);
CREATE TABLE IF NOT EXISTS "authored" (
    "author_id" INTEGER,
    "book_id" INTEGER,
    FOREIGN KEY("author_id") REFERENCES "authors"("id"),
    FOREIGN KEY("book_id") REFERENCES "books"("id")
);

...
```

我們也可以查看特定 table 的資訊

```sql
.schema table_name
```

在稍微看過 schema 長怎樣之後，接著我們要設計自己的 schema，這邊有一個情境，假設我們今天要設計一個儲存地鐵資訊的 DB，該怎麼做？

首先我們應該會想到我們可以儲存使用者名稱跟站名

| name | station | 
|---|---|
| Charlie | Kendall/MIT | 

接著可能會想到可以儲存行為，以及儲存票價

| name | station | action | fare | 
|---|---|---|---|
| Charlie | Kendall/MIT | enter | 10 | 

在扣款之後，我們想要知道使用者的餘額，所以我們可以儲存餘額資訊

| name | station | action | fare | balance | 
|---|---|---|---|---|
| Charlie | Kendall/MIT | enter | 10 | 5 | 

上面是一筆入站資訊的記錄，我們也可以記錄出站資訊

| name | station | action | fare | balance | 
|---|---|---|---|---|
| Charlie | Kendall/MIT | enter | 10 | 5 | 
| Charlie | Jamaica Plain | exit | 5 | 0 | 

接著我們再多添加一些資訊

| name | station | action | fare | balance | 
|---|---|---|---|---|
| Charlie | Kendall/MIT | enter | 10 | 5 | 
| Charlie | Jamaica Plain | exit | 5 | 0 | 
| Alice | Harvard | enter | 10 | 20 | 
| Alice | Park Street | exit | 5 | 15 | 
| Bob | Alewife | enter | 10 | 30 | 
| Bob | Park Street | exit | 10 | 20 | 

但可以看得出來上面的表格有很多重複的資訊，我們接著來優化

```plain
# 填寫你的優化看法
name 應該可以拆出去變成 用戶資訊 的 table
fare 應該跟 station 有關，應該也可以拆出去
balance 應該跟 用戶資訊 有關，可以跟 name 一起放
action 可以獨立出去變成 事件記錄 的 table
station 也可以獨立出去單獨紀錄
```

Q: 如果名稱重複怎麼辦？我們是不是應該賦予每個人一個獨立的 ID

A: 我們可以將 name 拆出來獨立成一個搭乘人的 table

| id | name | 
|---|---|
| 1 | Charlie | 
| 2 | Alice | 
| 3 | Bob | 

Q: 站名未來要變更的話很麻煩

A: 將可能會變動的資訊獨立出來往往是一個好選擇，這可以讓我們修改一個欄位就讓相關聯的 table 都吃到更新，舉例來說，我們將“站名”獨立出去，然後原本的 table 記錄站名的部分改成 ID，未來若站名從 A 變成 B，我們只需要到獨立出去的 table 修改該數值，原本與他連結的 table 因為是使用 ID，所以不受影響，我們不需要將原本的 table 裡面的所有數值都更新一遍

| id | location | 
|---|---|
| 1 | Harvard | 
| 2 | Kendall/MIT | 
| 3 | Park Street | 

上述的步驟我們可以反覆的一直進行，而這個過程就被稱為「Normalizing」，而 Normalizing 的程度還可以細分為 First normal form、Second normal form、Third normal form，分別代表不同的拆分程度，3rd Normal Form 是最顆粒度最細的結果。

而一般來說你的每張 table 至少要達到 1st Normal Form，不然就算是一個很差勁的設計。[出處](https://www.youtube.com/watch?v=mUtAPbb1ECM)

每到正題，我們已經進行了兩次的拆分，分出了 rider 跟 stations 兩張 tables，接著我們來思考 ER Diagrams 會長怎樣。每個 rider 會造訪 1～多 的站點（不然怎麼被視為搭乘者），而 stations 會與 0～多 個站點有關（沒人造訪是合理的），圖呈現如下

![Screenshot 2024-05-08 at 2.49.43 PM.png](./assets/Screenshot%202024-05-08%20at%202.49.43 PM.png)

⚠️ 注意：雖然我們建立以上的關係，但這是映射設計師對於這些資料的解讀，有可能有人覺得 station 就應該要有 rider，而認為不該是 0～多，應該是 1～多，這都是有可能的

### Create table

在 SQLite 中要建立一個 DB 很簡單，直接連接即可

```plain
> sqlite3 mbta.db
```

就算當下我們沒有 DB，連接後會自動建立一個

建立 table 使用 CREATE TABLE 的指令，並給予欄位名稱

```sql
CREATE TABLE riders (
    id, 
    name
);
```

```sql
CREATE TABLE stations (
  id,
  name,
  line
);
```

接著我們要建立兩個 table 之間的關聯，我們透過建立一個關聯表

```sql
CREATE TABLE visits(
 rider_id,
 station_id
);
```

可以看到我們還沒使用到前面學過的 Primary Key（PK）、Foreign Key（FK），也還沒為這些欄位添加資料型態，接下來我們將一步步添加上去，來讓我們的 table 變得更好，先來認識一下 storage class 跟 type affinity

### Storage Class

SQLite 支持下面五種儲存類別

`NULL`：代表“空”，沒有任何東西的概念

`INTEGER`：整數，可以設定 0, 1, 2, 3, 4, 6, 8 個 byte 的空間，但 SQLite 會自動幫你決定，基本上不用自己調整

`REAL`：浮點數

`TEXT`：文字，

`BLOB`：Binary Large Object，直接儲存二進位資料

⚠️ 注意：儲存類別不等於 data types，一個儲存類別可以儲存多種 data types，就像 INTEGER 這個儲存類別可以儲存 0-byte integer、6-byte interger 等等 data types

### Type Affinities

當你對欄位設定 storage class 之後，當有資料被添加進去時都會被轉換資料型態成你設定的資料型態，這稱為「type affinity 」，舉例來說，你的欄位是 INTEGER，當你塞入 TEXT 的資料時，type affinity 會將你的 TEXT 轉換成 INTERGER。

SQLite 提供五種預定型態

1. `TEXT`

2. `INTEGER`

3. `NUMERIC`

4. `REAL`

5. `BLOB`

### 優化 table

我們先將剛剛建立的那些 table 刪除

```sql
DROP TABLE riders;
```

然後建立一個 schema.sql

```sql
CREATE TABLE riders (id INTEGER, name TEXT);
CREATE TABLE stations (id INTEGER, name TEXT, line TEXT);
CREATE TABLE visits (rider_id INTEGER, station_id INTEGER);
```

接著我們進到 sqlite 中，執行這個檔案

```sql
sqlite> .read schema.sql
```

可以看到我們的 table 成功建立。回過頭更新我們的 ER diagrams

![Screenshot 2024-05-08 at 3.52.43 PM.png](./assets/Screenshot%202024-05-08%20at%203.52.43 PM.png)

接著我們要添加 PK 跟 FK

### Table Constraints

諸如像是 PK 跟 FK 都算是一種 table constraint，也就是建立一個限制條件，約束我們的欄位或表格，PK 代表的是唯一值，而 FK 代表的是這個數值應該同時出現在其他 table。

### 繼續優化

我們來修改 schema.sql 並添加 table constraints

添加 riders 的 PK、stations 的 PK，但可能你會疑惑，我們的 visits 沒有 id，PK 要怎麼設定？ 我們可以建立 joint PK，也就是設定兩個欄位作為我們的 PK，兩個欄位的組合沒有重複就符合限制

```sql
CREATE TABLE riders (
  id INTEGER,
  name TEXT,
  PRIMARY KEY("id")
  );

CREATE TABLE stations (
  id INTEGER,
  name TEXT,
  line TEXT,
  PRIMARY KEY("id")
  );

CREATE TABLE visits (
  rider_id INTEGER,
  station_id INTEGER,
  PRIMARY KEY("rider_id", "station_id")
  );
```

但以這個情境來說有點奇怪，因為理論上是可以重複造訪的，所以我們需要再調整一下，或許可以添加 id 到 visits 上並設定 id 為 PK，合理，但不需要，SQLite 會自己幫你為每一列添加 row_id，雖然 schema 上面看不到，但我們可以 query 它，所以關於 PK，我們可以先跳過。

對於 visits 這張 table 來說，添加 FK 是相當合理的，畢竟理論上不該有不存在的 rider_id 跟 station_id 出現，我們修改一下

```sql
CREATE TABLE visits (
  rider_id INTEGER,
  station_id INTEGER,
  FOREIGN KEY("rider_id") REFERENCES "riders"("id"),
  FOREIGN KEY("station_id") REFERENCES "stations"("id")
  );
```

### Column Constraints

除了為整張表添加限制，我們也可以為個別欄位添加限制

SQLite 提供四種限制

- CHECK：檢查，例：設定此欄位數值必須大於 0

- DEFAULT：預設，例：設定欄位沒有給予數值時，預設數值為何

- NOT NULL：不為空，當設定不為空，沒有給予數值時會出錯

- UNIQUE：這個欄位的數值不應該重複

我們試著添加限制到我們的 stations

```sql
CREATE TABLE stations (
  id INTEGER,
  name TEXT NOT NULL UNIQUE,
  line TEXT NOT NULL,
  PRIMARY KEY("id")
  );
```

首先是 name，station 的名稱不該為空，且名稱不應該重複

line 不該為空，但 line 會重複，所以我們僅添加不為空的限制

可能會問，為什麼 id 不用添加限制？ 因為當你設定欄位是 PK 時，欄位會自動添加 NOT NULL 跟 UNIQUE 的限制，我們不需要重複設定。

### Altering Tables

現在情況不一樣了，我們有地鐵卡的發行，我們不在直接追蹤使用者的資訊，取而代之，我們改追蹤地鐵卡的資訊，並且因應地鐵卡，我們添加 Swipe table 來記錄刷卡進出站的資訊，ER diagram 如下

![Screenshot 2024-05-08 at 5.04.48 PM.png](./assets/Screenshot%202024-05-08%20at%205.04.48 PM.png)

接著來到實作，我們先將 riders table 刪除，因為我們不再需要

```sql
DROP TABLE riders;
```

我們也不再需要 visits table，但我們這次可以不直接刪除，取而代之，我們修改它。

首先是修改名稱

```sql
ALTER TABLE visits RENAME TO swipes;
```

接著添加欄位

```sql
ALTER TABLE swipes ADD ttpe TEXT;
```

這邊出了一個問題，我們要打 type，但打成 ttpe，我們可以修改欄位名稱

```sql
ALTER TABLE swipes RENAME COLUMN ttpe to type;
```

如果我們不需要某個欄位，我們也可以刪除它

```sql
ALTER TABLE swipes DROP COLUMN type;
```

因應變更，我們修改 schema.sql

```sql
CREATE TABLE "cards" ("id" INTEGER, PRIMARY KEY("id"));
CREATE TABLE "stations" (
    "id" INTEGER,
    "name" TEXT NOT NULL UNIQUE,
    "line" TEXT NOT NULL,
    PRIMARY KEY("id")
);
CREATE TABLE "swipes" (
    "id" INTEGER,
    "card_id" INTEGER,
    "station_id" INTEGER,
    "type" TEXT NOT NULL CHECK("type" IN ('enter', 'exit', 'deposit')),
    "datetime" NUMERIC NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "amount" NUMERIC NOT NULL CHECK("amount" != 0),
    PRIMARY KEY("id"),
    FOREIGN KEY("station_id") REFERENCES "stations"("id"),
    FOREIGN KEY("card_id") REFERENCES "cards"("id")
);
```

### 1st Normal Form

有 4 個規則

1. 每個欄位的數值應該只儲存一個數值，而不是多個，錯誤範例如下

   

   | Column1 | Column2 | 
   |---|---|
   | A | A, B | 
   | B | P, Q | 
   

2. 每個欄位儲存的數值應該是同一個資料型態，不允許混用，錯誤範例如下

   

   | Column1 | Column2 | 
   |---|---|
   | 1 | A | 
   | 2 | BAF | 
   | A | 45 | 
   

3. 欄位名稱不可以重複，錯誤範例如下

   

   | Name | Name | 
   |---|---|
   | A | A | 
   | B | C | 
   | A | K | 
   

4. 儲存順序不重要，以下是一個正常的情況

   

   | ID | Name | 
   |---|---|
   | 5 | A | 
   | 3 | C | 
   | 6 | K | 
   


