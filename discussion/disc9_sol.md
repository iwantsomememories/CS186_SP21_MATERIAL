# DIS 9



## 1 Undo Logging Q1

The following table gives the operations for Transaction T in the first column. Disk A = 7 and Disk B = 3 at the beginning.

| Operation     | Mem A  | Mem B | Disk A | Disk B | UNDO Log           |
| ------------- | ------ | ----- | ------ | ------ | ------------------ |
|               |        |       | **7**  | **3**  | **START T**        |
| READ(A)       | **7**  |       |        |        |                    |
| READ(B)       |        | **3** |        |        |                    |
| WRITE(A, A+B) | **10** |       |        |        | **UPDATE T, A, 7** |
| WRITE(B, A-B) |        | **7** |        |        | **UPDATE T, B, 3** |
| FLUSH(A)      |        |       | **10** |        |                    |
| FLUSH(B)      |        |       |        | **7**  |                    |
| COMMIT        |        |       |        |        | **COMMIT**         |

(a) Fill in the columns Mem A, Mem B, Disk A, Disk B.



(b) If the system crashes right before COMMIT, how will we recover? Fill in the UNDO column.

**从后向前扫描undo log，对于某个事务，若该事务没有提交则撤销更新**

(c) What happens if the system crashes again while we’re undoing? How do we recover?

**按照相同方式处理即可**



## 2 Redo Logging Q1

**The following table gives the operations for Transaction T in the first column. Disk A = 5 and Disk B = 4 at the beginning.**

| Operation     | Mem A | Mem B | Disk A | Disk B | REDO Log           |
| ------------- | ----- | ----- | ------ | ------ | ------------------ |
|               |       |       | **5**  | **4**  | **START T**        |
| READ(A)       | **5** |       |        |        |                    |
| READ(B)       |       | **4** |        |        |                    |
| WRITE(A, A+B) | **9** |       |        |        | **UPDATE T, A, 9** |
| WRITE(B, A-B) |       | **5** |        |        | **UPDATE T, B, 5** |
| COMMIT        |       |       |        |        | **COMMIT**         |
| FLUSH(A)      |       |       | **10** |        |                    |
| FLUSH(B)      |       |       |        | **7**  |                    |

(a) Fill in the columns Mem A, Mem B, Disk A, Disk B.



(b) If the system crashes right after COMMIT, how will we recover? Fill in the REDO column.

**从前向后扫描REDO Log，逐个重放已记录的操作。**



## 3 Recovery Q1

Consider the execution of the ARIES recovery algorithm given the following log (assume a checkpoint is completed before LSN 0 and the Dirty Page Table and Transaction Table for that checkpoint are empty):

(a) During Analysis, what log records are read? What are the contents of the transaction table and the dirty page table at the end of the analysis stage?

**会读取从10号开始的所有日志记录**

**Transaction Table：**

| Transaction | Status   | lastLSN |
| ----------- | -------- | ------- |
| T2          | aborting | 80      |
| T3          | aborting | 90      |

**Dirty Page Table：**

| Page ID | recLSN |
| ------- | ------ |
| P1      | 10     |
| P2      | 70     |
| P3      | 20     |
| P4      | 40     |

(b) During Redo, what log records are read? What data pages are read? What operations are redone (assuming no updates made it out to disk before the crash)?

**会读取从10号开始的所有日志记录**

**P1, P2, P3, P4**

**10, 20, 40, 50, 70**

(c) During Undo, what log records are read? What operations are undone? Show any new log records that are written for CLR’s. Start at LSN 100. Be sure to show the undoNextLSN.

**会读取从90号到10号的所有日志记录**

错误：会读取80, 70, 50, 40, 20（只有需要撤销的事务日志需要被读取）.

**20, 40, 50, 70号操作都会被undone**

| LSN  | Log Record          | undoNextLSN |
| ---- | ------------------- | ----------- |
| 100  | CLR Undo T3: LSN 70 | 40          |
| 130  | CLR Undo T2: LSN 50 | 20          |
| 110  | CLR Undo T3: LSN 40 | null        |
| 120  | END T3              |             |
| 140  | CLR Undo T2: LSN 20 | null        |
| 150  | END T2              |             |



## 4 Recovery Q2

Your database server has just crashed due to a power outage. You boot it up, find the following log and checkpoint information on disk, and begin the recovery process. Assume we use a STEAL/NO FORCE recovery policy.

(a) The log record at LSN 60 says that transaction 2 updated page 5. Was this update to page 5 successfully written to disk? The log record at LSN 70 says that transaction 1 updated page 2. Was this update to page 2 successfully written to disk?

**两者均不确定**

错误：60号不确定，70号已经落盘（因为P2没有出现在DPT中）

(b) At the end of the analysis phase, what transactions will be in the transaction table, and what pages will be in the dirty page table?

**T1, T3, T4, T5**

**P1, P2, P3, P5**

(c) At which LSN in the log should redo begin? Which log records will be redone (list their
LSNs)? All other log records will be skipped.

**40**

**40, 50, 60, 90, 110, 130, 160, 180**