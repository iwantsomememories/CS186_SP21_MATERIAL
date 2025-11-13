# DIS 11



## 1 Parallel Query Processing

1. What is the difference between inter- and intra- query parallelism?

   **处于不同粒度的并行，一个是并行执行多个查询，一个是并行执行一个查询的不同部分**

2. What are the advantages and disadvantages of organizing data by keys?

   **优点：因为数据是按键组织的，所以查找和更新会更有效率。**

   **缺点：为了维持这种组织，需要付出额外开销。**

3. Assume for parts (a) and (b) that we have m=3 machines with B=5 buffer pages each, along
   with N=63 pages of data that don’t contain duplicates.

   (a) In the best case, what is the number of passes needed to sort the data?

   **4**

   (b) What is the number of passes needed to hash the data (once)? Find the best case, assum-
   ing that somehow the data will be uniformly distributed under the given hash function.

   **4**

   (c) Assume that relation R has R pages of data, and relation S has S pages of data. If we have
   m machines with B buffer pages each, what is the number of passes in order to perform
   sort merge join (in terms of R, S, m, and B)?
   Consider reading over either relation to be a pass.

   **6 + $\left \lceil \log_{B - 1}{\frac{R}{Bm} }  \right \rceil $ + $\left \lceil \log_{B - 1}{\frac{S}{Bm} }  \right \rceil $**

   (d) Can you use pipeline parallelism to implement this join?

   **无法实现流水线并行的排序联结。**

4. All of the data for a relation with N pages starts on one machine, and we would like to
   partition the data onto M machines. Assume that the size of each page is S (in KB).

   (a) How much data (in KB) would be sent over the network to partition the data through each
   of the following: range, hash, and round-robin partitioning? Assume we use uniform
   hash functions and are able to construct ranges that have the same number of values in
   them.

   **均为$\frac{NS(M - 1)}{M}$KB**

   (b) If there are no assumptions about hash functions or data ranges, what is the best and
   worst case in terms of network I/O cost for the three partitioning schemes?

   **range，最好：0，最差：NS**

   **hash，最好：0，最差：NS**

   **round-robin，最好：$\frac{NS(M - 1)}{M}$，最差：$\frac{NS(M - 1)}{M}$**

5. Relation R has 10,000 pages, round-robin partitioned across 4 machines (M1, M2, M3, M4).
   Relation S has 10 pages, all of which are only stored on M1. We want to join R and S on the
   condition R.col = C.col.

   Assume the size of each page is 1 KB.
   (a) What type of join would be best in this scenario, and why?

   **broadcast join, 因为表R与表S的大小相差悬殊，且表R已均匀分布在四台机器上。**

   (b) How many KB of data must be sent over the network to join R and S?

   **30KB**

   (c) Would the amount of data sent over the network change if R was hash partitioned among
   the 4 machines rather than round-robin partitioned? What about range partitioned?

​		**不会发生变化，因为只需要将表S发送到3台机器上即可。**