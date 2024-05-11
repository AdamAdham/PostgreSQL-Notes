# PostgreSQL-Notes

## Multiple Column Queries
When multiple conditions are supplied in the query, the planner assumes that the columns (or the where clause conditions) are independent of each other. This doesn’t hold true when columns are correlated or dependant on each other and that leads the planner to under or over-estimate the number of rows which will be returned by these conditions.<br>

## Query Time Taken
```SQL
EXPLAIN ANALYZE SELECT * FROM tbl where col1 = 1;
```
#### Result:
```SQL
Seq Scan on tbl  (cost=0.00..169247.80 rows=9584 width=8) (actual time=0.641..622.851 rows=10000 loops=1)
   Filter: (col1 = 1)
   Rows Removed by Filter: 9990000
 Planning time: 0.051 ms
 Execution time: 623.185 ms
(5 rows)
```
## Create Statistics
### Dependencies
```SQL
CREATE STATISTICS s1 (dependencies) on col1, col2 from tbl;
ANALYZE tbl;
```
### Combination Distinct Values
```SQL
CREATE STATISTICS s2 (ndistinct) on col1, col2 from tbl;
ANALYZE tbl;
```

## Check for dependecies
```SQL
SELECT stxname, stxkeys, stxdependencies
  FROM pg_statistic_ext
  WHERE stxname = 's1';
```
#### Result
``` SQL
stxname | stxkeys |   stxdependencies
---------+---------+----------------------
 s1      | 1 2     | {"1 => 2": 1.000000}
(1 row)
```

## Link: https://www.citusdata.com/blog/2018/03/06/postgres-planner-and-its-usage-of-statistics/
