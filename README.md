# PostgreSQL-Notes

## Multiple Column Queries
When multiple conditions are supplied in the query, the planner assumes that the columns (or the where clause conditions) are independent of each other. This doesn’t hold true when columns are correlated or dependant on each other and that leads the planner to under or over-estimate the number of rows which will be returned by these conditions.<br>
https://www.citusdata.com/blog/2018/03/06/postgres-planner-and-its-usage-of-statistics/

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
