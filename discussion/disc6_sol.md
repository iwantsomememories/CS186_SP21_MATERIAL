# DIS 6



## 1 Assorted Joins

- Companies: (company_id, industry, ipo_date)
- NYSE: (company_id, date, trade, quantity)

We have 20 pages of memory, and we want to join two tables Companies and NYSE on C.company_id
= N.company_id. Attribute company_id is the primary key for Companies. For every tuple in Companies, assume there are 4 matching tuples in NYSE.
NYSE contains [N] = 100 pages, NYSE holds pN = 100 tuples per page.
Companies contains [C] = 50 pages, C holds pC = 50 tuples per page.
There are unclustered B+ tree indexes of height 1 on C.company_id and N.company_id.

(a) How many disk I/Os are needed to perform a simple nested loops join?

**250050**

(b) How many disk I/Os are needed to perform a block nested loops join?

**350**

(c) How many disk I/Os are needed to perform an index nested loops join?

**15050**

(d) For this part only, assume the index on NYSE.company_id is clustered. What is the cost of an
index nested loops join using companies as the outer relation?

**7550**

(e) How many disk I/Os are needed to perform a sort merge join without optimization? If we can
perform the sort merge join optimization, how many disk I/Os are needed with optimization?

**750, 450**

(f) How many disk I/Os are needed to perform a hash join? Assume uniform partitioning.

**492**



## 2 Grace Hash Join

We have 2 tables – Catalog and Transactions.
Catalog has a total of 100 pages and 20 tuples per page. Transactions has a total of 50 pages and
50 tuples per page. Assume the hash functions uniformly distribute the data for both tables.
(a) If we had 10 buffer pages, how many partitioning phases would we require for grace hash join?Consider which table we should build the hash table in the probing phase on.

**1**

(b) What is the I/O cost for the grace hash join then?

**474**

(c) For the above question, if we only had 8 buffer pages, how many partitioning phases would
there be?

**2**

(d) What will be the I/O cost?

**962**