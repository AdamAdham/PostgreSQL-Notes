# PostgreSQL-Notes

## Multiple Column Queries
When multiple conditions are supplied in the query, the planner assumes that the columns (or the where clause conditions) are independent of each other. This doesn’t hold true when columns are correlated or dependant on each other and that leads the planner to under or over-estimate the number of rows which will be returned by these conditions.<br>

## Exists Clause
For most queries the total cost is what matters, but in contexts such as a subquery in EXISTS, the planner will choose the smallest start-up cost instead of the smallest total cost (since the executor will stop after getting one row, anyway). Also, if you limit the number of rows to return with a LIMIT clause, the planner makes an appropriate interpolation between the endpoint costs to estimate which plan is really the cheapest.

https://www.postgresql.org/docs/15/sql-explain.html (in description)

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




## Resources
- Introduction to SQL Queries performance measurement in PostgreSQL
https://www.citusdata.com/blog/2018/03/06/postgres-planner-and-its-usage-of-statistics/
https://wiki.postgresql.org/wiki/Tuning_Your_PostgreSQL_Server *(Did not learn anything)*
- Advanced Tuning 
https://www.postgresql.org/docs/15/static/runtime-config.html *(Not read)*
- Create Statistics
https://www.postgresql.org/docs/15/sql-createstatistics.html *(Informative)*
- How to use EXPLAIN command and understand its output & other stuff: 
https://www.postgresql.org/docs/15/performance-tips.html
http://www.postgresql.org/docs/15/static/sql-explain.html
- Creating an Index for a table:
http://www.postgresql.org/docs/15/static/sql-createindex.html
- Statistics collected by PostgreSQL:
https://www.postgresql.org/docs/15/static/planner-stats.html
- Trouble shooting an Index:
https://www.gojek.io/blog/the-case-s-of-postgres-not-using-index
https://www.pgmustard.com/blog/why-isnt-postgres-using-my-index
