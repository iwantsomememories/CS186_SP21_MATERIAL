# DIS 4



## 1 Buffer Management

**(a) 请填写以下表格，以反映给定的缓冲区替换策略。你有 4 个缓冲页，访问模式如下：**
 **A B C D A F A D G D G E D F**

- **LRU**

  | **A** |       |       |       | A    |       | A    |       |       |       |       |       |       | F    |
  | ----- | ----- | ----- | ----- | ---- | ----- | ---- | ----- | ----- | ----- | ----- | ----- | ----- | ---- |
  |       | **B** |       |       |      | **F** |      |       |       |       |       | **E** |       |      |
  |       |       | **C** |       |      |       |      |       | **G** |       | **G** |       |       |      |
  |       |       |       | **D** |      |       |      | **D** |       | **D** |       |       | **D** |      |

  **Hit Rate =** 6 / 14

- **MRU**

  | **A** |       |       |       | A    | F    | A    |       |       |       |       |       |       |       |
  | ----- | ----- | ----- | ----- | ---- | ---- | ---- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
  |       | **B** |       |       |      |      |      |       |       |       |       |       |       |       |
  |       |       | **C** |       |      |      |      |       |       |       |       |       |       |       |
  |       |       |       | **D** |      |      |      | **D** | **G** | **D** | **G** | **E** | **D** | **F** |

  **Hit Rate =** 2 / 14 

- **CLOCK**

  | **A** |       |       |       | A    | <font color='red'>F</font> | A                              |                                |       |                                |       |                                | <font color='red'>**D**</font> |                                |
  | ----- | ----- | ----- | ----- | ---- | -------------------------- | ------------------------------ | ------------------------------ | ----- | ------------------------------ | ----- | ------------------------------ | ------------------------------ | ------------------------------ |
  |       | **B** |       |       |      |                            | <font color='red'>**A**</font> | **D**                          |       | **D**                          |       |                                | **D**                          | <font color='red'>**F**</font> |
  |       |       | **C** |       |      |                            |                                |                                | **G** |                                | **G** | **E**                          |                                |                                |
  |       |       |       | **D** |      | **F**                      |                                | <font color='red'>**D**</font> |       | <font color='red'>**D**</font> |       | <font color='red'>**E**</font> |                                | **F**                          |

  **Hit Rate =** 4 / 14

  

**(b) 请填写以下表格，以反映给定的缓冲区替换策略。你有 4 个缓冲页，访问模式如下：**
 	**A、B、C、D、A、F（保持固定）、D、G、D、解除固定 F、G、E、D、F**
	**请记住，解除固定（unpin）不会计入命中次数！**

- **LRU**

  | **A** |       |       |       | A    |       |       |       |       |       | E    |       |       |
  | ----- | ----- | ----- | ----- | ---- | ----- | ----- | ----- | ----- | ----- | ---- | ----- | ----- |
  |       | **B** |       |       |      | **F** |       |       |       |       |      |       | **F** |
  |       |       | **C** |       |      |       |       | **G** |       | **G** |      |       |       |
  |       |       |       | **D** |      |       | **D** |       | **D** |       |      | **D** |       |

  **Hit Rate =** 6 / 13

- **MRU**

  | **A** |       |       |       | A    | F    |       |       |       | <font color='red'>**G**</font> | <font color='red'>**E**</font> |       | F                              |
  | ----- | ----- | ----- | ----- | ---- | ---- | ----- | ----- | ----- | ------------------------------ | ------------------------------ | ----- | ------------------------------ |
  |       | **B** |       |       |      |      |       |       |       |                                |                                |       |                                |
  |       |       | **C** |       |      |      |       |       |       |                                |                                |       |                                |
  |       |       |       | **D** |      |      | **D** | **G** | **D** | **G**                          | **E**                          | **D** | <font color='red'>**F**</font> |

  **Hit Rate =** 3 / 13

**(c) MRU 是否比 LRU 更好？**

在某些场景下MRU优于LRU——当循环读取一组页面且页面的数量大于缓冲区的数量时。

**(d) 为什么我们会使用时钟（Clock）替换策略而不是 LRU？**

LRU的实现更加复杂，需要存储各个页面的最近访问时间。<font color='red'>效率更高。</font>

**(e) 为什么数据库管理系统需要实现自己的缓冲区替换策略？为什么不能仅依赖操作系统？**

数据库的访问模式可能与操作系统的访问模式存在差异。<font color='red'>数据库系统可以根据数据访问模式优化缓存替换策略。</font>



## 2 Relational Algebra

**考虑以下模式：**

```sql
Songs (song_id, song_name, album_id, weeks_in_top_40)
Artists (artist_id, artist_name, first_year_active)
Albums (album_id, album_name, artist_id, year_released, genre)
```

**请为以下查询写出关系代数表达式：**

**(a) 找出具有“pop”或“rock”类型专辑的艺术家姓名。**
$$
\pi_{Artists.artist_name}(Artists \Join (\sigma _{genre = 'pop' \vee  genre = 'rock'}Albums))
$$
**(b) 找出同时具有“pop”和“rock”类型专辑的艺术家姓名。**
$$
\pi_{Artists.artist_name}(Artists \Join (\sigma _{genre = 'pop'}Albums)) \cap \pi_{Artists.artist_name}(Artists \Join (\sigma _{genre = 'rock'}Albums))
$$
**(c) 找出具有“pop”类型专辑或在前 40 名榜单中停留超过 10 周的艺术家的 ID。**
$$
\pi_{Artists.artist_id}(\sigma _{genre = 'pop'}(Albums)) \cup \pi_{Artists.artist_id} (Albums \Join \sigma_{weeks_in_top_40 > 10}(Songs))
$$
**(d) 找出没有发表任何专辑的艺术家的姓名。**
$$
\pi_{Artists.artist_name}(Artists \Join (\pi_{artist_id}(Artists) - \pi_{artist_id}(Albums)))
$$