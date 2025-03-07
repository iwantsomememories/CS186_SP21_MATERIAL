# SQL, Disks and Files



## Question 1: SQL

**考虑以下图书馆书籍的模式：**

**Books：**

- **`bid` INTEGER，**
- **`title` TEXT，**
- **`library` REFERENCES Library，**
- **`genre` TEXT，**
- **`PRIMARY KEY (bid)`**

**Library：**

- **`lid` INTEGER，**
- **`lname` TEXT，**
- **`PRIMARY KEY (lid)`**

**Checkouts：**

- **`book` INTEGER REFERENCES Books，**
- **`day` DATETIME，**
- **`PRIMARY KEY (book, day)`**

**(a) 返回所有曾被借阅的书籍的 `bid` 和 `genre`，并去除任何具有相同 `bid` 和 `genre` 的重复行。**

```sql
SELECT DISTINCT b.bid, b.genre
FROM Books as b 
INNER JOIN Checkouts as c ON b.bid = c.book;
```

**(b) 查找所有被借阅过的奇幻（Fantasy）书籍的 `title` 及其借阅日期。即使某本书没有被借阅过，也仍然需要输出其 `title`（即结果行应为 `(title, NULL)`）。**

```sql
SELECT b.title, c.day
FROM Books as b 
LEFT JOIN Checkouts as c ON b.bid = c.book
WHERE b.genre = 'Fantasy';
```

**(c) 选择被借阅次数最多的书籍的名称以及相应的借阅次数。可以假设每本书的借阅次数都是唯一的，并且书籍的标题也是唯一的。**

```sql
SELECT b.title, COUNT(*)
FROM Books as b 
INNER JOIN Checkouts as c ON b.bid = c.book
GROUP BY b.title
ORDER BY COUNT(*)
LIMIT 1;
```

**(d) 选择所有书名相同的书籍所在的图书馆对，并包含两个图书馆的名称以及书籍的标题。结果中不应包含重复行，也不应包含仅图书馆顺序不同但内容相同的两行（例如 `('East', 'West', 'Of Mice and Men')` 和 `('West', 'East', 'Of Mice and Men')`）。为了确保这一点，第一个图书馆的名称应该在字母顺序上小于第二个图书馆的名称。可能存在零个、一个或多个正确答案。**

(A) 错误查询

```sql
SELECT DISTINCT l.lname, l.lname, b.title
FROM Library l, Books b
WHERE l.lname = b.library
AND b.library = l.lid
ORDER BY l.lname;
```

(B) 正确查询

```sql
SELECT DISTINCT l1.lname, l2.lname, b1.title
FROM Library l1, Library l2, Books b1, Books b2
WHERE l1.lname < l2.lname AND b1.title = b2.title
AND b1.library = l1.lid AND b2.library = l2.lid;
```

(C) 正确查询

```sql
SELECT DISTINCT first.l1, second.l2, b1
FROM
(SELECT lname l1, title b1
FROM Library l, Books b
WHERE b.library = l.lid) as first,
(SELECT lname l2, title b2
FROM Library l, Books b
WHERE b.library = l.lid) as second
WHERE first.l1 < second.l2
AND first.b1 = second.b2;
```



## Question 2: More SQL

**考虑以下用于城市间自行车骑行者的模式：**

**Locations（地点）：**

- **lid INTEGER PRIMARY KEY （地点ID，主键）**
- **city_name VARCHAR （城市名称）**

**Riders（骑行者）：**

- **rid INTEGER PRIMARY KEY （骑行者ID，主键）**
- **name VARCHAR （骑行者姓名）**
- **home INTEGER REFERENCES locations (lid) （骑行者的居住地，引用 Locations 表的 lid）**

**Bikes（自行车）：**

- **bid INTEGER PRIMARY KEY （自行车ID，主键）**
- **owner INTEGER REFERENCES riders (rid) （自行车的所有者，引用 Riders 表的 rid）**

**Rides（骑行记录）：**

- **rider INTEGER REFERENCES riders (rid) （骑行者ID，引用 Riders 表的 rid）**
- **bike INTEGER REFERENCES bikes (bid) （自行车ID，引用 Bikes 表的 bid）**
- **src INTEGER REFERENCES locations (lid) （出发地点，引用 Locations 表的 lid）**
- **dest INTEGER REFERENCES locations (lid) （目的地，引用 Locations 表的 lid）**

**(a) 选择所有返回拥有最多自行车的骑行者 rid 的查询。假设所有骑行者拥有的自行车数量是唯一的。**

**(A)**

```sql
SELECT owner FROM bikes GROUP BY owner ORDER BY COUNT(*) DESC LIMIT 1;
```

**(B)**

```sql
SELECT owner FROM bikes GROUP BY owner HAVING COUNT(*) >= ALL
(SELECT COUNT(*) FROM bikes GROUP BY owner);
```

**(C)**

```sql
SELECT owner FROM bikes GROUP BY owner HAVING COUNT(*) = MAX(bikes);
```

A、B

**(b) 选择所有从未被骑行过的自行车的 bid。**

**(A)**

```sql
SELECT bid FROM bikes b1 WHERE NOT EXISTS
(SELECT owner FROM bikes b2 WHERE b2.bid = b1.bid);
```

**(B)**

```sql
SELECT bid FROM bikes WHERE NOT EXISTS
(SELECT bike FROM rides WHERE bike = bid);
```

**(C)**

```sql
SELECT bid FROM bikes WHERE bid NOT IN
(SELECT bike FROM rides, bikes AS b2 WHERE bike = b2.bid);
```

B、C

**(c) 选择所有骑行者的姓名，以及他们所有骑行记录的出发地和目的地的城市名称。即使某个骑行者从未骑行过，我们仍然希望输出他们的姓名（即输出格式应为 (name, null, null)）。**

**(A)**

```sql
SELECT tmp.name, s.city_name AS src, d.city_name AS dst 
FROM locations s, locations d,
(riders r LEFT OUTER JOIN rides ON r.rid = rides.rider) AS tmp
WHERE s.lid = tmp.src AND d.lid = tmp.dest;
```

**(B)**

```sql
SELECT r.name, s.city_name AS src, d.city_name AS dst 
FROM riders r
LEFT OUTER JOIN rides ON r.rid = rides.rider
INNER JOIN locations s ON s.lid = rides.src
INNER JOIN locations d ON d.lid = rides.dest;
```

**(C)**

```sql
SELECT r.name, s.city_name AS src, d.city_name AS dst 
FROM rides
RIGHT OUTER JOIN riders r ON r.rid = rides.rider
INNER JOIN locations s ON s.lid = rides.src
INNER JOIN locations d ON d.lid = rides.dest;
```

B

<font color='red'>这些都不正确，因为后续的内连接或 WHERE 子句会过滤掉外连接中的空值行。</font>



## Question 3: Files, Pages, Records

**考虑以下关系：**

```sql
CREATE TABLE Cats (
    collar_id INTEGER PRIMARY KEY, -- 不能为 NULL！
    age INTEGER NOT NULL,
    name VARCHAR(20) NOT NULL,
    color VARCHAR(10) NOT NULL
);
```

**你可以假设：**

- **INTEGER 类型占 4 字节；**
- **VARCHAR(n) 最多可占 n 字节。**

**(a) 由于记录是可变长度的，我们需要在记录中包含一个记录头。记录头的大小是多少？你可以假设指针占 4 字节，并且记录头仅包含指针。**

8字节

**(b) 包括记录头部，在该模式下最小可能的记录大小是多少（以字节为单位）？（注意：NULL 在 SQL 中被视为特殊值，空字符串 VARCHAR 与 NULL 不同，就像 0 的 INTEGER 值也与 NULL 不同。如果类似问题出现在考试中，我们会提供必要的澄清。）**

16字节

**(c) 包括记录头部，在该模式下最大可能的记录大小是多少（以字节为单位）？**

46字节

**(d) 现在让我们来看页面。假设我们使用插槽页面布局存储这些记录，并且记录是可变长度的。页面页脚包含一个整数，用于存储记录数，以及一个指向空闲空间的指针。此外，还有一个插槽目录，其中每条记录存储一个指针和长度。那么，在一个 8KB 的页面上，我们最多可以存储多少条记录？（请记住，1KB 等于 1024 字节。）**
$$
(8 * 1024 - 8) / (16 + 8) = 341
$$
**(e) 假设我们在一个页面上存储了最大数量的记录，然后删除了一条记录。现在我们想插入另一条记录。我们是否可以保证能够插入？请解释原因。**

不一定，因为存储记录为变长记录，插入的记录长度可能大于被删除的记录长度。

**(f) 现在假设我们删除了 3 条记录。在不重新组织页面上任何记录的情况下，我们希望插入另一条记录。我们是否可以保证能够插入？请解释原因。**

可以保证插入成功；因为一条记录最小长度为16字节，最大长度为46字节，因此三条记录的最小总长度为48字节，大于一条记录的最大长度。

<font color='red'>不行；虽然有 48 个空闲字节，但它们可能是碎片化的——可能没有连续的 46 个字节。</font>



## Question 4: Files, Pages, Records

考虑以下关系：

``` sql
 CREATE TABLE Student (
 	student_id INTEGER PRIMARY KEY,
 	age INTEGER NOT NULL,
 	units_passed INTEGER NOT NULL
 );
```

**(a) Student 记录是固定长度还是可变长度？**

固定长度

**(b) 为了存储这些记录，我们将使用一个带有页面头的未打包表示。该页面头仅包含一个位图，并向上舍入到最接近的字节。在一个 4KB 的页面上，我们最多可以存储多少条记录？**

337条记录

**(c) 假设有 7 页的记录。我们想要执行以下查询：**

```sql
 SELECT * FROM Student WHERE student_id = 3034213355; -- 只是一个数字
```

 **假设这些页面存储在一个作为链表实现的堆文件中。回答该查询所需的最小和最大页面 I/O 数是多少？**

最小I/O数为8，最大I/O数为8（需要读取header页，且student_id可能重复）。

<font color='red'>最小I/O数为2，最大I/O数为8（需要读取header页）。</font>

**(d) 现在假设这些页面存储在一个按照 student_id 排序的文件中。你需要访问的最小和最大页数是多少？假设排序文件没有头部页面。**

最小I/O数为1，最大I/O数为3。



## Question 5: Files, Pages, Records

**(a) 假设我们在一个链表堆文件中存储可变长度记录。在“具有可用空间的页面”列表中，假设恰好有 5 个页面。插入一条记录所需的最大页面 I/O 次数是多少？**
**你可以假设至少有一个页面有足够的空间，并且在插入后不会变满。**

6 + 1 = 7

**(b) 延续 (a) 的情况，现在假设该页面在插入后确实变满了。**
**现在，我们需要将该页面移动到“已满页面”列表。**
**假设我们已经完成了 (a) 部分的最坏情况所需的所有页面读取（并且这些页面仍然在内存中），但尚未执行任何页面写入。**
**将该页面移动到“已满页面”列表还需要多少额外的页面 I/O？**

<font color='red'>5次</font>

<font color='red'>答案的核心思想如下：我们有两个双向链表，一个用于已满页面，另一个用于未满页面；它们都连接到头部页面。</font>

<font color='red'>我们需要将“未满页面”列表末尾的一个页面（来自(a)部分）移动到“已满页面”列表的头部（选择头部是因为这里的插入成本最低，并且我们可以这样做，因为这些页面并没有特定的顺序）。</font>

<font color='red'>因此，我们需要将该页面从一个链表的尾部移动到另一个链表的头部，这一过程通过指针更新完成。</font>

<font color='red'>在此过程中，我们需要更新被移动的页面、其旧的相邻页面，以及它在新列表中的两个新邻居（头部页面和“已满页面”列表中的旧首个页面）。</font>

<font color='red'>具体的 I/O 计算如下：
 (1) 写入更新后的数据页面。
 (1) 写入先前的未满页面（即旧的后向邻居）。
 (1) 读取旧的已满页面列表中的首个页面（即新的前向邻居）。
 (1) 写入旧的首个已满页面。
 (1) 写入头部页面（即新的后向邻居）。</font>

**(c) 现在假设记录是固定长度的；插入一条记录所需的最大页面 I/O 次数是多少？**
**假设我们插入的页面在插入后不会变满。**

3

**(d) 现在假设我们使用一个页面目录，并且只有一个目录页面。**
**插入一条记录所需的最大页面 I/O 次数是多少？**

4