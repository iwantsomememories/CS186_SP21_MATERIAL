# DIS 13



##  1 OLTPvs OLAP

For each workloads, choose whether it’s better characterized as Online Transaction Processing (OLTP) or Online Analytical Processing (OLAP). 

(a) A social media site with millions of users needs to track all the "likes" and "dislikes" that each post receives. 

**OLTP**

(b) An online book store needs to aggregate and analyze its users book purchases by genre over the last eight months. 

**OLAP**

(c) An multiplayer online game has added updated areas to its map and now wants to assess how users behave in those areas, and how user playtime has changed as a result.

**OLAP**

##  2 Scaling

(a) A small startup realizes that its current database can’t sustain their growing workloads. Given that these workloads involve lot of write but few reads, should it invest in more partitioning or more replication?

**应该更多进行分片**

(b) A mechanical failure causes some of the startup’s database machines to permanently crash, losing data in the process. If the startup wants to prevent similar losses in the future, should it invest more in partitioning or more replication?

**复制技术**

## 3 BASE

 (a) Database designer Doug is annoyed with his distributed database because for some time af ter issuing a write, all his reads return different values. Does this violate any of the BASE properties? 

**并没有违反，保持了较弱的最终一致性**

Which properties of BASE do these scenarios violate?

(b) All reads and writes always have the same views of data, but they sometimes respond to valid inputs with timeout errors.

**可用性**

(c) Writes only propagate to 3 replicas, but the system has 5 replicas of each piece of data.

**一致性** 错误：最终一致性

(d) An empty database that has never been populated responds to a read query on some specific key with the message "Error: key nonexistent!"

**不违反任何性质**

## 4 Key-Value Stores

Database Doug now has the following tables: 

​	Sales (sid, date, quantity, customer, product) 

​	Product (pid, name, price) 

​	Customer (cid, name, address) 

Sales data is stored with Key = sid, Value = entire Sales record, partitioned on hash function h and replicated across 3 servers.

(a) Describe how the operation get(sid1) would be executed. (Assume a Sale with sid1 exists in the data). 

**首先根据主键以及哈希函数计算该数据存储在哪几台机器上，然后从任意一个副本查询得到数据并返回。**

(b) Describe how the operation put(sid2, saleRecord) would be executed. 

**首先根据主键以及哈希函数计算该数据存储在哪几台机器上，然后在所有机器上进行相应更新，返回结果给用户。**

(c) After put(sid2, saleRecord) is executed, is it guaranteed that every app will be able to access that new Sale data?

**不一定，有可能发生网络隔离。** 错误：不一定，因为只提供了最终一致性。



##  5 JSON

1. Convert the following relational table into a JSON document.

   ```json
   {
       "player": [
           {"name": "Tony",
           "debut": "10/12/09",
           "goals": 43},
           {"name": "Katy",
           "debut": "1/20/14",
           "goals": 22}
       ]
   }
   ```

   

2. Convert the following JSON document into two relational tables, Players(name, debut) and Goals(name, goals). 

   {“players”: [ 

   ​	{“name”: “Abby”, “debut”: “10/12/09”, “goals”: 43}, 

   ​	{“name”: “Babby”, “debut”: “1/20/14”, “goals”: 22}, 

   ​	{“name”: “Cabby”, “debut”: “1/21/14”, “goals”: 23} 

   ​	] 

   }

**Players**	

| name  | debut    |
| ----- | -------- |
| Abby  | 10/12/09 |
| Babby | 1/20/14  |
| Cabby | 1/21/14  |

**Goals**

| name  | goal |
| ----- | ---- |
| Abby  | 43   |
| Babby | 22   |
| Cabby | 23   |



## 6 Mongo Query Language (MQL)

For the entire question, consider the MongoDB collection teams with the following fields: 

• teamId (int) 

• divisionId (int) 

• stadiumCapacity (int) 

• wins (int) 

• losses (int) 

• coach (string) 

• captain (string)

1. Using MQL, write a query to fetch the following: Find the coach and captain of all teams from division 1 with at least 10 wins, sorted by coach DESC and ties broken by captain ASC.

   ```
   db.teams.find({divisionId: 1, wins: {$gte: 10}}, {coach: true, captain: true}).sort({coach: -1, captain: 1})
   ```

   **错误，正确查询如下：**

   ```
   db.team.aggregate([
        {$match: {
            wins: {$gte: 10},
            divisionId: 1
        }},
        {$sort: {
            "coach":-1,
            "captain": 1
        }},
        {$project: {
            "coach": 1,
            "captain": 1,
            "_id": 0
        }}
    ])
   ```

   

2. Translate the following SQL query into an MQL query: 

   SELECT divisionId AS div, MAX(wins) AS maxWins 

   FROM team WHERE stadiumCapacity >= 20000 

   GROUP BY divisionId 

   SORT BY MAX(wins), COUNT(*) DESC;

   ```
   db.teams.find({stadiumCapacity: {$gte: 20000}})
   ```

   **错误，正确查询如下：**

   ```
   db.teams.aggregate([
        { $match: {
            stadiumCapacity: {$gte: 20000}
        }},
        { $group: {
            _id: "$divisionId",
            maxWins: {$max: "$wins"},
            count: {$sum: 1}
        }},
        { $sort: {
            "maxWins": 1,
            "count":-1
        }},
        { $project: {
            div: "$_id",
            _id: 0,
            maxWins: 1
        }}
    ])
   ```

   

