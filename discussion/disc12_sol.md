# DIS 12



## 1 Two Phase Commit with Logging

1. Suppose we have one coordinator and three participants. It takes 30ms for a coordinator to send
  messages to all participants; 5, 10, and 15ms for participant 1, 2, and 3 to send a message to the
  coordinator respectively; and 10ms for each machine to generate and flush a record. Assume for
  the same message, each participant receives it from the coordinator at the same time.
  Under proper 2PC and logging protocols, how long does the whole 2PC process (from the begin-
  ning to the coordinator’s final log flush) take for a successful commit in the best case?

  **110ms** 错误：(30 + 10 + 15 + 10) * 2 = 130ms

2. Now in the two-phase commit protocol, describe what happens if a participant receives a PRE-
  PARE message, replies with a YES vote, crashes, and restarts (All other participants also voted
  YES and didn’t crash).

  **询问协调者是否已经commit，收到commit回复后在本地写入commit日志，并向协调者回复ACK。**

3. In the two-phase commit protocol, suppose that the coordinator sends PREPARE message to
  Participants 1 and 2. Participant 1 sends a "VOTE YES" message, and Participant 2 sends a "VOTE
  NO" message back to the coordinator.

  (a) Before receiving the result of the commit/abort vote from the coordinator, Participant 1 crashes. Upon recovery, what actions does Participant 1 take 1) if we were not using presumed abort,
  and 2) if we were using presumed abort?

  **w/o presumed abort：询问协调者是否commit，收到abort回复后在本地中止事务。**

  (b) Before receiving the result of the commit/abort vote from the coordinator, Participant 2 crashes.
  Upon recovery, what actions does Participant 2 take 1) if we were not using presumed abort,
  and 2) if we were using presumed abort?

  **without presumed abort：在本地中止事务并重新向协调者发送"VOTE NO"。**

  **with presumed abort：在本地中止事务，但不再发送投票。**

4. In the two-phase commit protocol, suppose that the coordinator sends PREPARE messages to
  the participants and crashes before receiving any votes for parts (a)-(c). Assuming that we are
  running 2PC with presumed abort, answer the following questions.

  (a) What sequence of operations does the coordinator take after it recovers?

  **在本地中止事务。**

  (b) What sequence of operations does a participant who received the message and replied NO be-
  fore the coordinator crashed take?

  **在本地中止事务。**

  (c) What sequence of operations does a participant who received the message and replied YES
  before the coordinator crashed take?

  **询问协调者是否commit，收到abort回复后终止事务。**

  (d) Let’s say that the coordinator instead crashes after successfully receiving votes from all par-
  ticipants, with all participants voting YES except for one NO vote. Assuming the coordinator
  sees no records for this transaction in its log after coming back online, how does this affect the
  answers to parts (a)-(c)?

​		**(a)：在本地中止事务。**

​		**(b)：在本地中止事务。**

​		**(c)：询问协调者是否commit，收到abort回复后终止事务。**

