# Learning SQL

本書採用 MySQL，因為作者覺得 MySQL simple to download and install，不過這本書並不局限於 MySQL，就算你不會使用到 MySQL，也可以閱讀並嘗試看看

## 前置步驟

1. 啟動 Mysql 服務，並將 [sakila-db](https://dev.mysql.com/doc/index-other.html) 掛載到 container 的 tmp 之中

   ```docker
   docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=passwd --volume ./sakila-db:/tmp -d mysql:8.0
   ```

2. 進到 container，啟動 mysql command line

    ```sh
    mysql -u root -ppasswd
    ```

3. 執行以下指令，會看到很多執行結果，此時測試用資料庫就建立完成了

    ```sh
    source /tmp/sakila-schema.sql
    source /tmp/sakila-data.sql
    ```

## Introduction to Databases

資料庫簡單來說就是用來存放一堆**彼此之間都緊密相關的資料**的地方。

你會經常看到 “transaction” 這個名詞，這是因為過去資料庫是用來處理金融場景的，所以就順水推舟的繼續使用 transaction 來代表**一組不可分割的操作序列**。

## **Nonrelational Database Systems**

### hierarchical database system

![Screenshot 2024-04-22 at 2.47.56 PM.png](./Learning%20SQL-assets/Screenshot%202024-04-22%20at%202.47.56 PM.png)

資料以樹狀結構或 parent-child 結構儲存。例子：*single-parent hierarchy*，其中每個樹的 node 有 0 或 1 個 parent node，並且有 0\~N 個 child node。

hierarchical database systems 一個實際的使用場景是 [directory services](https://zh.wikipedia.org/zh-tw/%E7%9B%AE%E5%BD%95%E6%9C%8D%E5%8A%A1) 領域，像是 Microsoft’s Active Directory 或是 Apache Directory Server，而 [directory services](https://zh.wikipedia.org/zh-tw/%E7%9B%AE%E5%BD%95%E6%9C%8D%E5%8A%A1) 跟我們比較貼近的使用場景是 [DNS](https://zh.wikipedia.org/wiki/DNS "DNS") 服務。

### network database system

![Screenshot 2024-04-19 at 10.01.17 AM.png](./Learning%20SQL-assets/Screenshot%202024-04-19%20at%2010.01.17%20AM.png)

有很多的 records 跟 links 用來描述 records 之間的關係。每個 record 可以由多個 record 連接而來，形成網路般錯綜復雜的關係，也被稱為「*multi‐ parent hierarchy*」。

## **The Relational Model**

![Screenshot 2024-04-19 at 10.24.45 AM.png](./Learning%20SQL-assets/Screenshot%202024-04-19%20at%2010.24.45%20AM.png)

資料由多個 table 來定義與描述，每個 table 都會對其所可以乘載的資料做出定義，而每一筆（row）資料都會有**唯一的**識別符（primary key）。

你可能會想 唯一的識別符要怎麼生成？我不知道怎麼做？ 這點現代的資料庫都已經幫你考慮好了，他會幫你追蹤並且生成不重複的 ID。

一般來說會選擇 table 的其中一個欄位作為 primary key，但有時候你可能會需要選擇兩個或多個欄位來作為你的唯一識別符，這種由多個欄位組合而成 priimary key 稱為「compound key」。

在選擇 primary key 時若選用一些現實世界常用的用素或屬性來代表，像是 姓名、身分證，稱為「natural key」，而使用現實世界不具意義的數字、文字來作為 primary key 則稱為「surrogate key」，選用哪種作為 primary key 必須根據情況，沒有對錯。

要留意的是，**primary key 在賦予之後應該永遠都不會被更動**。

Table 中用來象徵每一筆資料的是 primary key，而用來象徵其他 table 中的資料的是 foreign key。

在設計 table 上我們會學到一個概念稱為 「***normalization***」（正規化），包含的概念如下：

- 資料修改應該只發生在一處，而不是需要到處修改，這衍生的概念是是我們不應該有重複的資料

- 一個欄位代表一個小部分的資料，不要將所有資料都塞在一起，能拆就盡量拆

後來還有衍生一個概念是**反正規化**，不過該概念是建立在**資料庫正規化之後**，透過打破部分規則來換取性能。因此資料庫正規化仍是必知必會的重點之一。

## What is SQL?

SQL 原本用來描述關聯式資料庫的操作語言，但現在已經成為資料庫操作語言的技術代稱。

## SQL Statement Classes

SQL 語言由多種**陳述式部分**組合而成，包括：

- SQL schema statements: 用來定義我們儲存在資料庫的資料所要遵循的資料結構

- SQL data statements：用來操作我們透過 SQL schema statement 定義的資料

- SQL transaction statements：用來開始、結束、回溯交易

所有透過 SQL schema statements 生成的資料庫的元素都被儲存在一組特殊的 tables 裡面，稱為「data dictionary」，這些資料是關於資料庫的資料，也就是所謂「metadata」

data dictionary tables 可以透過 select statement 去檢索，讓我們儘管在運行時也可以知道當前資料庫中所定義的資料結構。

## **SQL: A Nonprocedural Language**

**procedural language**(程序式語言) 將重點放在**過程**，使用這種語言會定義流程跟預期得到的結果，但 **Non-procedural** 相反，我們將重點放在**結果**，至於過程是怎麼做到的我們依賴於 external agent 來自動完成。

而 SQL 正是一門 **Non-procedural language**，我們不自行處理流程細節，只設定輸入跟輸出，剩下的交由 database 的 engine 來完成，而這類 engine 稱為「optimizer」。

這些 optimizer 的工作是接收你撰寫的 SQL statements，然後查看你的 table 是怎麼設定的、有沒有 indexes 可以用，最終自動生成高效率的執行路徑來完成任務。

大多數 database engines 仍然保留讓使用者介入的空間，我們可以用 *optimizer hints* 來介入，像是強制 optimizer 使用某個特定 index 之類的。

### SQL Examples

在 SQL 中要添加文字或註解，需要加上 /\* 跟 \*/ 來標記

```sql
/* 我是註解 */
SELECT * FROM ABC;
```

通常我們會使用一門 procedural language 並搭配 toolkits 來連線到 DB 互動，而 DB 執行完SQL 之後通常會出現提示訊息來表示執行結果，通常來說，一個好的使用方式是**接收這些回傳的提示訊息並檢查結果是否符合預期**。

## Creatingt and Populating a Database

### Character Data

Character Data 可以用兩種方式儲存

- 固定長度

- 變數長度

兩者的差異是面對多餘的空間會怎麼處理，固定長度會幫你添加空格，並將你設定的空間用好用滿，但變數長度反過來，不會添加空格也不會一定要把空間用完。但一個共通點是，我們都必須設定 Character 可以存放的空間上限。

一般來說，我們使用固定長度的 Character Data 來存放長度固定的文字，例如：狀態的縮寫。

使用變數長度來存放長度不固定的文字。

但根據我的調查，[MySQL](https://stackoverflow.com/questions/68670136/does-having-fixed-size-rows-in-mysql-improve-query-performance) 跟 [PostgreSQL](https://docs.postgresql.tw/the-sql-language/data-types/character-types) 中使用 Fixed-size 都沒有比較快，**一般情況下都選擇 varchar type 會是比較好的選擇**。

SQL 提供多種編碼，過去 MySQL 都是預設使用 latin1，但在 version 8 之後預設使用 utf8mb4，編碼帶來的差異除了對語言的支援之外，還有體現在佔用的空間，latin1 支援的文字較少，但每個字佔用的空間也比較少（1 byte），而 utf8mb4 支援多國語系，但每個字都需要 4 bytes，而這些佔用較多 byte 的語言也稱為「multibyte character sets」。

雖然預設是使用 utf8mb4，但你還是可以針對 DB、table、field 指定自己想要的編碼，這可以在 type definition 時設定

```sql
varchar(20) character set latin1
```

或是建立 DB 時設定

```sql
create database european_sales character set latin1;
```

### **Text data**

![Screenshot 2024-04-22 at 11.41.45 AM.png](./Learning%20SQL-assets/Screenshot%202024-04-22%20at%2011.41.45 AM.png)

Text data 讓我們可以儲存大於 varchar type 所提供的最大上限 64 KB 以上大小的文字資料。

在儲存小於 64 KB 的資料時，繼續使用 VARCHAR(0 \~ 65535) 即可，大於時再使用 TEXT，tinytext 則不需要使用。

在使用時 Text data 時要注意：

- 如果存入的資料大於設定的資料上限，不會出現錯誤，但你的文字超出上限的會被裁切掉

- 文字前後的空白（trailing spaces）是不會被自動踢掉的，記得自行將這些多餘的空白去掉

- 當使用這些 text 欄位排序（sorting）或分組（grouping），最多只會使用到 1024 bytes，雖然這限制有需要可以調整



若你的文字是像 document 那種的，TEXT 可能依然沒辦法滿足你的需求，這時可以使用 mediumtext 或 longtext。註：[PostgreSQL](https://docs.postgresql.tw/the-sql-language/data-types/character-types) 沒有區別，都使用 TEXT 即可。

### Numeric Data

![Screenshot 2024-04-22 at 11.48.26 AM.png](./Learning%20SQL-assets/Screenshot%202024-04-22%20at%2011.48.26 AM.png)

Int 類型的資料有提供多種 type，通常**選擇可以涵括你的資料的數值範圍的類型**即可，不需要浪費空間設定過大的類型

Int 類型可以設定 signed 或 unsigned，除了控制數值是否可以為負之外，還可以操縱數值的範圍，若今天你的數值不會為負，像是戶頭餘額，那就可以設定 unsigned，我們的數值範圍可以變得更大，使用上有更多彈性。 

![Screenshot 2024-04-22 at 11.49.47 AM.png](./Learning%20SQL-assets/Screenshot%202024-04-22%20at%2011.49.47 AM.png)

使用 floating-point type 的資料類型，我們可以設定 precision 跟 scale

- precision 代表精度，描述我們可以用多少個數字來表示這個數值

- scale 代表我們的小數點要取到第幾位

上述的設定不是必要的，不過在像是機器學習模型分數的儲存上可以設定 scale 來避免儲存的分數過於精細。

超出設定限制的 digits 會被四捨五入，這點需要留意。

就像 Int 類型一樣，floating-point type 也可以設定 signed 或 unsigned，但只能控制是否為負，不會調整數值的範圍。

### Temporal Data

儲存像是 dates 或 times 的資料，我們通常稱他們為「temporal」

![Screenshot 2024-04-22 at 11.58.43 AM.png](./Learning%20SQL-assets/Screenshot%202024-04-22%20at%2011.58.43 AM.png)

![Screenshot 2024-04-22 at 11.58.52 AM.png](./Learning%20SQL-assets/Screenshot%202024-04-22%20at%2011.58.52 AM.png)

可以看到有很多種不同類型的字串格式，這些格式看起來雷同，似乎有點沒意義，但他其實是有其象徵的意義的。

字串格式通常也象徵 temporal 資料是如何被獲取、添加、修改的。

但若是想儲存時間但不是像上面的格式的話，就必須改使用 string。

datetime、timestamp、time 可以讓我們儲存到小數點六位的精度，但很多時候我們不會需要這麼詳細的時間點，我們可以設定來降低精度，更符合使用的場景。

```sql
datetime(2)
```

## Table Creation

如何設計 table？這邊提供簡單的步驟供參考

**步驟 1：設計（Design）**

- 思考什麼資料儲存下來是有幫助的

- 這些資料應該使用什麼 data type 來儲存

**步驟 2：提煉（Refinement）**

- 正規化，將重複的、複合的欄位拆開

- 設定 primary key 以確保**唯一性**

**步驟 3：建立 SQL Schema Statements**

- 撰寫 SQL Schema Statements，並設定 constraint，像是：primary constraint、foreign key 之類的

- Check constraint 是一種特殊的 contraint，設定這個資料在添加時會被檢查

   ```sql
   eye_color CHAR(2) CHECK (eye_color IN ('BR','BL','GR')),
   ```

   不過在 MySQL 中如果要強制啟用需要用另外一個關鍵字 `ENUM` 

   ```sql
   eye_color ENUM('BR','BL','GR'),
   ```

   ```sql
   CREATE TABLE person
        (person_id SMALLINT UNSIGNED,
         fname VARCHAR(20),
         lname VARCHAR(20),
         eye_color ENUM('BR','BL','GR'),
         birth_date DATE,
         street VARCHAR(30),
         city VARCHAR(20),
         state VARCHAR(20),
         country VARCHAR(20),
         postal_code VARCHAR(20),
         CONSTRAINT pk_person PRIMARY KEY (person_id)
   );
   ```

Null 在 SQL 中是到處可見，用來表示某個數值當前沒有有效值，會在下列情境中使用：

- **Not applicable**：不適用，像是“配偶姓名”這個字段，對於未婚的人來說，這個字段就是不適用的，因此可以被設置為**`null`**。

- **Unknown**：未知，如果某人的出生日期未知，那麼在相應的數據庫字段中可以填入**`null`**來表示這一信息目前不可獲得。

- **Empty set**：為空，例如，在數據庫查詢中，如果一個查詢的結果預期應該返回一組值，但實際上沒有任何值匹配查詢條件，則可以使用**`null`**來表示沒有結果。

### Populating and Modifying Tables

讓 Mysql 幫你建立數值型 ID，只需要在 primary key 上添加 *auto-increment* 讓他自動調整即可

```sql
/* 如果有外鍵導致不能更新，可以先將它關掉之後再打開 */
set foreign_key_checks=0;
ALTER TABLE person
 MODIFY person_id SMALLINT UNSIGNED AUTO_INCREMENT;
set foreign_key_checks=1;

```

若提供的資料符合格式，MySQL 會把字串自動轉變成日期

### Updating Data

```sql
UPDATE `table` SET `column` = 'value' WHERE `column` = '1'
```

### **Deleting Data**

```sql
DELETE FROM `table` WHERE `column` = '1'
```

### 常見的錯誤

- **Nonunique Primary Key：使用到重複的 primary key**

- **Nonexistent Foreign Key：使用到不存在的 foreign key。**Foreign key constraints are enforced only if your tables are created using the InnoDB storage engine.

- **Column Value Violations：若欄位有限制合法的數值，而使用時用到非法的欄位數值**

- **Invalid Date Conversions：輸入資料不能被正確轉型成欄位的資料型態**

   ```sql
   // 一個好的實踐是說明字串應該被怎麼解析
   UPDATE person SET birth_date = str_to_date('DEC-21-1980', '%b-%d-%Y') WHERE person_id = 1;
   ```

   以下是常見的代號

   ```plain
   %a The short weekday name, such as Sun, Mon, ...
   %b The short month name, such as Jan, Feb, ...
   %c The numeric month (0..12)
   %d The numeric day of the month (00..31)
   %f The number of microseconds (000000..999999)
   %H The hour of the day, in 24-hour format (00..23)
   %h The hour of the day, in 12-hour format (01..12)
   %i The minutes within the hour (00..59)
   %j The day of year (001..366)
   %M The full month name (January..December)
   %m The numeric month
   %p AM or PM
   %s The number of seconds (00..59)
   %W The full weekday name (Sunday..Saturday)
   %w The numeric day of the week (0=Sunday..6=Saturday)
   %Y The four-digit year
   ```

## Query Primer

### Query Mechanics

當輸入正確的 username 跟 password 之後，我們會跟 database 建立一個連線來進行互動，這個連線稱為「database connection」。

這個 connection 會由需求方控制，直到需求方 release 或 server(DB) 關閉這個 connection 才會關閉，而建立的每個 connection，MySQL 都會提供一個 identifier 來識別他。

建立 connection 之後，我們就可以傳送指令並執行，每次傳送過去再實際執行之前都會經過幾個檢查：

- 你有權限執行嗎？

- 你有權限訪問資料嗎？

- 你的指令對嗎？

通過檢查之後，指令會交由 query optimizer 處理，他會決定 query 該怎麼執行才會有效率。

optimizer 會查看我們定義在 `from` 之後的  table 有哪些，其中有哪些 indexes 可以用，然後會選擇一個 *excution plan* 來執行。

執行完成後，server 會回傳 *result set* 給需求方，result set 大概會長下面那樣，像是一個表格的樣子。

![Screenshot 2024-04-22 at 5.21.35 PM.png](./Learning%20SQL-assets/Screenshot%202024-04-22%20at%205.21.35 PM.png)

### Query Clauses

一個 select 的 statement 會由多個部分組合而成，以下是常見的語句

![Screenshot 2024-04-22 at 5.23.54 PM.png](./Learning%20SQL-assets/Screenshot%202024-04-22%20at%205.23.54 PM.png)

### The Select Clause

雖然我們都是先寫 select，但 select 其實最後才執行，這是因為我們必須先知道最終的來源資料有哪些欄位可以被檢索，我們才可以進行檢索。

除了檢索特定的欄位之外，我們可以做一些不一樣的操作：

- Literals（固定值），數字、字串都可以

- Expression，像是 transaction.amount \* −1

- 內建函數，ROUND(transaction.amount, 2)

- 使用者自訂的函數

以下是一個範例

```sql
mysql> SELECT language_id,
        ->   'COMMON' language_usage,
        ->   language_id * 3.1415927 lang_pi_value,
        ->   upper(name) language_name
        -> FROM language;
+-------------+----------------+---------------+---------------+
| language_id | language_usage | lang_pi_value | language_name |
+-------------+----------------+---------------+---------------+
| 1           | COMMON         |     3.1415927 | ENGLISH       |
| 2           | COMMON         |     6.2831854 | ITALIAN       |
| 3           | COMMON         |     9.4247781 | JAPANESE      |
| 4           | COMMON         |    12.5663708 | MANDARIN      |
| 5           | COMMON         |    15.7079635 | FRENCH        |
| 6           | COMMON         |    18.8495562 | GERMAN        |
+-------------+----------------+---------------+---------------+
6 rows in set (0.04 sec)
```

如果我們只是想要執行內建函數或是跑一些簡單的 expression 也可以，像是

```sql
mysql> SELECT version(),
       ->   user(),
       ->   database();
    +-----------+----------------+------------+
    | version() | user()         | database() |
    +-----------+----------------+------------+
    | 8.0.15    | root@localhost | sakila     |
    +-----------+----------------+------------+
    1 row in set (0.00 sec)

```

像是上面的情境我們就不需要使用到 `from` 語句。

### Column Aliases

有時候我們會想要將搜尋出來的欄位名稱更換名稱，換成我們更好理解跟操作的名稱，指令如下

```sql
SELECT language_id, 
       'COMMON' language_usage,
       language_id * 3.1415927 lang_pi_value,
       upper(name) language_name 
  FROM language;
```

只要在搜尋後加上想要更換的名稱就可以，但這樣可能會比較不好閱讀，我們可以加上 `AS` 區隔開

```sql
SELECT language_id, 
       'COMMON' AS language_usage,
       language_id * 3.1415927 AS lang_pi_value,
       upper(name) AS language_name 
  FROM language;
```

### **Removing Duplicates**

```sql
SELECT DISTINCT actor_id FROM film_actor ORDER BY actor_id;
```

## The from clause

### Tables

大概有四種 table：

- Permanent tables：我們使用 create table 建立的屬於這種

- Derived tables：subquery 產生並回傳的，並儲存在記憶體中

- Temporary tables：儲存在記憶體中的 volatile data（揮發性資料）

- Virtual tables：透過 create view 建立的

### Derived(subquery-generated) Table

subquery 是 query 當中的另一個 query，subquery 會以括號包起來，可以出現在 select 語句的不同部分。當子查詢位於 FROM 子句中時，它會生成一個衍生表，這個衍生表可以被查詢的其他部分看見，並且可以與 FROM 子句中提到的其他表進行交互。

```sql
SELECT concat(cust.last_name, ', ', cust.first_name) full_name
FROM
(SELECT first_name, last_name, email 
  FROM customer 
  WHERE first_name = 'JESSIE') cust;
```

### **Temporary tables**

暫時存在的 table，儲存在記憶體中，通常在 transaction 結束或是 database session 關閉後會消失

```sql
CREATE TEMPORARY TABLE actors_j (actor_id smallint(5),
  first_name varchar(45),
  last_name varchar(45)
);
```

個人認為在用資料庫做數據分析時應該會頻繁用到，先把資料撈出來暫存之類的，但直接用程式語言連接到資料庫撈回用程式語言做分析似乎更常見，暫存表的使用場景比較少。

### **Views**

View 是一種儲存在 data dictionary 的 query，他看起來像是 table 一樣，但沒有任何資料實際與 View 關聯著，當你對 View 進行 query，系統背後其實是將你的 query 跟 view 的 definition 去做合併，產生一個最終的 query 並執行。

建立一個 view definition

```sql
CREATE VIEW cust_vw AS SELECT customer_id, first_name, last_name, active FROM customer;
```

對這個 view 進行搜尋

```sql
SELECT first_name, last_name FROM cust_vw WHERE active = 0;
```

看起來很沒必要，但在某些場景很實用，像是隱藏 table 實際的欄位不讓 user 知道或是簡化複雜的資料庫設計。

### Table Links

```sql
SELECT customer.first_name,
  customer.last_name,
  time(rental.rental_date) rental_time 
  FROM customer 
  INNER JOIN rental ON customer.customer_id = rental.customer_id
  WHERE date(rental.rental_date) = '2005-06-14';
```

### **Defining Table Aliases**

有兩種方式可以使用 Table，一種是使用全名來調用，一種是取別名來使用

```sql
/* 使用全名 */
employee.emp_id
```

```sql
/* 使用別名 */
From employee AS e
```

### **The where Clause**

```sql
SELECT title, rating, rental_duration
FROM film
WHERE (rating = 'G' AND rental_duration >= 7)
OR (rating = 'PG-13' AND rental_duration < 4);
```

### **The group by and having Clauses**

```sql
SELECT c.first_name, c.last_name, count(*)
FROM customer c
INNER JOIN rental r
ON c.customer_id = r.customer_id
GROUP BY c.first_name, c.last_name
HAVING count(*) >= 40;
```

### **The order by Clause**

```sql
SELECT c.first_name,
  c.last_name,
  time(r.rental_date) rental_time
FROM customer c
INNER JOIN rental r
ON c.customer_id = r.customer_id
WHERE date(r.rental_date) = '2005-06-14'
ORDER BY c.last_name, c.first_name;
```

排序有兩種選項 *ascending* or *descending* 分別對應關鍵字 *asc* 和 *desc*

```sql

SELECT c.first_name,
  c.last_name,
  time(r.rental_date) rental_time
FROM customer c
INNER JOIN rental r
ON c.customer_id = r.customer_id
WHERE date(r.rental_date) = '2005-06-14'
ORDER BY time(r.rental_date) desc;
```

你也可以檢索搜尋出來的表的欄位的位置，這在對 expression 排序特別有用，但大多時候還是建議用名稱來指定會比較好

```sql
SELECT c.first_name, 
  c.last_name,
  time(r.rental_date) rental_time
FROM customer c
INNER JOIN rental r
ON c.customer_id = r.customer_id
WHERE date(r.rental_date) = '2005-06-14'
ORDER BY 3 desc;
```

## **Filtering**

這部分很簡單，查看 SQL 必知必會即可

## **Querying Multiple Tables**

foreign key 對於 JOIN 操作來說不是必備的，但對於確保該記錄同時存在於預期出現的 table 來說很重要

### **Cartesian Product**

```sql
/* 這會把所有 a 跟 c 的排列組合都 show 出來 */
SELECT c.first_name, c.last_name, a.address FROM customer c JOIN address a;
```

*Cartesian product（笛卡爾積），也稱為「Cross Join」*，這會把所有可能性都組合出來，理論上很少用到

### Inner Joins

![image.png](./Learning%20SQL-assets/image.png)

```sql
SELECT c.first_name, c.last_name, a.address 
  FROM customer c 
  JOIN address a 
ON c.address_id = a.address_id;
```

`Inner Join`：這會將要建立連結的 table ，依照指定欄位連結在一起，如果兩邊的指定欄位沒有對應的數值，則視為連接失敗，該筆紀錄就不會被顯示出來

雖然 `JOIN` 就代表 `INNER JOIN`，但正統來說應該要完整使用 `INNER JOIN` 的指令。

如果你 JOIN 的兩張表，用來 JOIN 的欄位名稱是一致的，可以使用 `using`

```sql
SELECT c.first_name, c.last_name, a.address
    FROM customer c 
  INNER JOIN address a
  USING (address_id);
```

但最好還是都使用 `ON` 來避免誤會。

### **Joining Three or More Tables**

跟 JOIN 兩張表雷同

```sql
 SELECT c.first_name, c.last_name, ct.city
    FROM address a
      INNER JOIN city ct
      ON a.city_id = ct.city_id
      INNER JOIN customer c
      ON c.address_id = a.address_id;
```

順序不重要，SQL 的 optimizer 會自行決定如何進行查詢。

### **Using Subqueries as Tables**

雖然下面的範例直接 JOIN 三張表也可以實現，但有時候為了性能或可讀性依然會這麼做

```sql
SELECT c.first_name, c.last_name, addr.address, addr.city 
 FROM customer c
INNER JOIN
(
  SELECT a.address_id, a.address, ct.city
  FROM address a
    INNER JOIN city ct
    ON a.city_id = ct.city_id
  WHERE a.district = 'California'
) addr ON c.address_id = addr.address_id;
```

### **Using the Same Table Twice**

你可以 JOIN 一張表多次，透過多次篩選確保你所查詢的資料符合你的限制

```sql
SELECT f.title
FROM film f
INNER JOIN film_actor fa1
ON f.film_id = fa1.film_id
INNER JOIN actor a1
ON fa1.actor_id = a1.actor_id
INNER JOIN film_actor fa2
ON f.film_id = fa2.film_id
INNER JOIN actor a2
ON fa2.actor_id = a2.actor_id
WHERE (a1.first_name = 'CATE' AND a1.last_name = 'MCQUEEN')
  AND (a2.first_name = 'CUBA' AND a2.last_name = 'BIRCH');
```

### **Self-Joins**

有時候你會需要自己篩選自己，JOIN 不只用來連接其他表，也可以用來連接自己

```sql
SELECT f.title, f_prnt.title prequel
FROM film f
INNER JOIN film f_prnt
ON f_prnt.film_id = f.prequel_film_id
WHERE f.prequel_film_id IS NOT NULL;
```

## Terminology

### Computerized Database

A computerized database is **a structured collection of data organized and saved in an electronic format on a computer system**. You can store, retrieve, and deploy vast amounts of data efficiently and effectively by using a computerized database.

### Entity

Something of interest to the database user community. Examples include customers, parts, geographic locations, etc.

### Column

An individual piece of data stored in a table.

### Row

A set of columns that together completely describe an entity or some action on an entity. Also called a record.

### Table

A set of rows, held either in memory (nonpersistent) or on permanent storage (persistent).

### Result set

Another name for a nonpersistent table, generally the result of an SQL query.

### Primary key

One or more columns that can be used as a unique identifier for each row in a table.

### Foreign key

One or more columns that can be used together to identify a single row in another table.


