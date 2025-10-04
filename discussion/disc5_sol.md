# DIS 5



## 1 General External Merge Sort

(a) You have 4 buffer pages and your file has a total of 108 pages of records to sort. How many passes would it take to sort the file?

**4**

(b) How many runs would each pass produce?

**27, 9, 3, 1**

(c) What is the total cost for this sort process in terms of I/O?

**864**

(d) If the pages were already sorted individually, how many passes would it take to sort the file and how many I/Os would it be instead?

**4, 864**

(e) If we wanted to sort N pages in at most p total passes, write an expression relating the minimum # of buffer pages B needed with N and p. What do you notice about B when p = 1?

**B满足$(B - 1) ^{p - 1} * B = N$,**

**当$p = 1$时$B = N$。**



## 2 Hashing
(a) What are some use-cases in which hashing is preferred over sorting?

**对顺序并不关心，只希望按照值分组的场合，例如GROUP BY, DISTINCT**

(b) Suppose we have B buffer pages and can process B(B − 1) pages of data with External Hashing in two passes. For this case, fill in the blanks with the appropriate # of pages.
**1** input buffer(s)
**B - 1** partitions after pass 1
__B__ pages per partition

(c) If you are processing exactly B(B − 1) pages of data with external hashing, is it likely that you’ll have to perform recursive external hashing? Why or why not?

**有可能，这取决于选择的哈希函数，初始分区很可能无法使得各个分区页面数均匀分配**

(d) We want to hash N = 100 pages using B = 10 buffer pages. Suppose in the initial partitioning pass, the pages are unevenly hashed into partitions of 10, 20, 20, and 50 pages. Assuming uniform hash functions are used for every partitioning pass after this pass, what is the total I/O cost for External Hashing?

**100 + 100  + 20 + 27 + 20 + 27 + 50 + 54  + 118 * 2 = 634**