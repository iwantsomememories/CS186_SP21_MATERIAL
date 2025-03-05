# DIS 3



## 1 Indices (B+ Trees)

**假设我们有以下阶数为 1 的 B+ 树。每个索引节点必须包含 1 或 2 个键（即 2 或 3 个指针），而叶子节点最多可以容纳 2 个条目。**

**(a) 在不改变树的高度的情况下，最多可以执行多少次插入操作？**

12

**(b) 需要插入的最少键数是多少才能使树的高度发生变化？**

3



## 2 Indices

**两组术语：**

**聚簇（Clustered） vs. 非聚簇（Unclustered）**

- **在聚簇索引（Clustered Index）中，数据页按照构建B+ 树 的相同索引进行排序。这意味着键值的大致顺序与数据页的顺序相同，因此每获取一页记录大约需要 1 次 I/O。**
- **在非聚簇索引（Unclustered Index）中，数据页是无序的，这意味着每获取一条记录大约需要 1 次 I/O。**

**存储底层数据的三种替代方案：**

- **替代方案 1（按值存储）： 整个记录直接存储在叶子页中。**
- **替代方案 2（按引用存储）： 叶子页存储 `(key: 指向 rid 的指针)` 对，每个键值可能不唯一。**
- **替代方案 3（按引用存储）： 叶子页存储 `(key: [指向 rid1 的指针, 指向 rid2 的指针, ...])` 对，每个键值唯一。**

**(a) 是否可能在不同的列上创建两个聚簇索引？**

如果不同的列的键值之间成正比，则可能，否则不可能。

<font color='red'>是，但是需要存储数据的两份拷贝。</font>

**假设我们在 `(assignment_id, student_id)` 上有一个替代方案 2 的非聚簇索引，其高度为 3（需要遍历 3 个索引页才能到达任何叶子页）。**

**以下是表模式：**

```sql
CREATE TABLE Submissions (
    record_id integer UNIQUE,
    assignment_id integer,
    student_id integer,
    time_submitted integer,
    grade_received byte,
    comment text,
    regrade_request text,
    PRIMARY KEY (assignment_id, student_id)
);
CREATE INDEX SubmissionLookupIndex ON Submissions (
    assignment_id, student_id
);
```

**假设表及其相关数据占用 12MB 磁盘空间（1MB = 1024KB），每个页大小为 64KB（包括为未来插入预留的额外空间）。**

**(b) 我们想要扫描 `Submissions` 表中的所有记录。此操作需要多少次 I/O？**

192

**(c) 执行以下 `UPDATE` 语句需要多少次 I/O？**

```sql
UPDATE Students 
SET grade_received = 85 
WHERE assignment_id = 20 
AND student_id = 12345;
```

5

<font color='red'>6 = 4 + 1 + 1</font>

**(d) 在最坏情况下，对 `grade_received` 进行等值搜索需要多少次 I/O？**

192



## 3 Bulk-Loading

**假设我们使用批量加载方式创建阶数 d=2 的 B+ 树，并设定填充因子为 $\frac{3}{4}$。**

**其中，填充因子仅适用于叶子节点；内部节点应完全填满，并在分裂时平均分割。**

**我们按顺序插入整数键 1-16，请画出最终的 B+ 树结构，并回答以下问题：**

- **最终的树高是多少？**

最终树高为3。

<font color='red'>最终树高为2。</font>