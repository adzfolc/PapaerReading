# Access_Path_Selection_in_a_Relational_Database_Management_System

## Paper Structure
* the place of optimizer in SQL statement
* the storage component access paths that are available on a single physically stored table
* single table queries' optimizer cost formulas
* the joining of two or more tables, and their correspoding costs
* nested queries(queries in predicates)

## Chp2 Processing of an SQL Statement

1. 4 steps of statement processing
   * parsing
     * check for correct syntax
     * A `query block` is representated by a SELECT list, a FROM list, and a WHERE tree.
       * WHERE tree
         * the list of items to be retrieved
         * the table(s) referenced
         * the boolean combination of simple predicates specified by the user
   * optimization
     * accumulate the names of tables and columns referenced in the query
     * look up names in the catalog to verify existence and to retrieve information about them
     * 
   * code generation
   * execution

## Chp4 Costs for single relation access paths

1. COST = PAGE FETCHES + W * (RSI CALLS)
2. For each relation T
   - NCARD(T), the cardinality of relation T.
   - TCARD(T), the number of pages in the segment that hold tuples of relation T.
   - P(T), the fraction of data pages in the segment that hold tuples of relation T. P(T) = TCARD(T) / (no. of non-empty pages in the segment).
3. For each index I on relation T,
   - ICARD(I), number of distinct keys in index I.
   - NINDX(I), the number of pages in index I.
4. SELECTIVITY FACTORS
   1. column = value
      * 有索引 -> F = 1 / ICARD(column index)
      * 无索引 -> F = 1/10
   2. column1 = column2
      * 两列都有索引 -> F = 1/MAX(ICARD(column1 index), ICARD(column2 index))
      * 仅一列有索引 -> F = 1/ICARD(column-i index) if there is only an index on column-i
      * 都没有索引 -> F = 1/10
    3. column > value (or any other open-ended comparison)
      * value是数值类型+路径选择时value已知 -> F = (high key value - value) / (high key value - low key value)
      * 其他 -> F = 1/3
    4. column BETWEEN value1 AND value2
      * 列是数值类型 + 路径选择时(value1, value2)已知 -> F = (value2 - value1) / (high key value - low key value)
      * 其他 -> F = 1/4
    5. column IN (list of values)
       * F = (number of items in list) * (selectivity factor for column = value)
       * <= 1/2
    6. columnA IN subquery
       * F = (expected cardinality of the subquery result) / (product of the cardinalities of all the relations in the subquery's FROM-list).
       * <= 1/2

## Chp5 Access path selection for joins

1. COST FOTMULAS
   1. |SITUATION|COST(in pages)|
      |-|-|
      |Unique index matching an equal predicate|1+1+W|
      |Clustered index I matching one or more boolean factors|F(preds) * (NINDX(I) + TCARD) + W * RSICARD|
      |Non-clustered index I matching one or more boolean factors|F(preds) * (NINDX(I) + NCARD) + W * RSICARD or F(preds) * (NINDX(I) + TCARD) + W * RSICARD if this number fits in the System R buffer|
      |Clustered index I not matching any boolean factors|(NINDX(I) + TCARD) + W * RSICARD|
      |Non-clustered index I not matching any boolean factors|(NINDX(I) + NCARD) + W * RSICARD or (NINDX(I) + TCARD) + W * RSICARD if this number fits in the System R buffer|
      |Segment scan|TCARD/P + W * RSICARD|

2. Computation of costs
   1. C-outer(path1) -> cost of scanning the outer relation via path1
   2. N -> the cardinality of the outer relation tuples which satisfy the applicable predicates
   3. N = (product of the cardinalities of all relations T of the join so far) * (product of the selectivity factors of all applicable predicates)
   4. C-inner(path2) -> cost of scanning the inner relation, applying all applicable predicates
   5. C-nested-loop-join(path1, path2) = C-outer(path1) + N * C-inner(path2)
   6. C-merge(path1, path2) = C-outer(path1) + N * C-inner(path2)