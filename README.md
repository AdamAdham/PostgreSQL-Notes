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
### Ndistinct
```SQL
CREATE STATISTICS stts2 (ndistinct) ON city, state, zip FROM zipcodes;
```

It's advisable to create ndistinct statistics objects only on combinations of columns that are actually used for grouping, and for which misestimation of the number of groups is resulting in bad plans. Otherwise, the ANALYZE cycles are just wasted.

#### Query
```SQL
SELECT stxkeys AS k, stxdndistinct AS nd
  FROM pg_statistic_ext join pg_statistic_ext_data on (oid = stxoid)
  WHERE stxname = 'stts2';
```
#### Result
```SQL
-[ RECORD 1 ]------------------------------------------------------​--
k  | 1 2 5
nd | {"1, 2": 33178, "1, 5": 33178, "2, 5": 27435, "1, 2, 5": 33178}
```
Interpretation:

k: This column represents the combinations of columns for which distinct values are being counted. The numbers represent the column identifiers. **For example**, "1" might represent the ZIP code, "2" might represent the city, and "5" might represent the state.<br>
nd: This column represents the distinct values observed for each combination of columns specified in k.<br>
From the nd column:<br><br>

"1, 2": This indicates the combination of ZIP code and city, which has 33,178 distinct values.
"1, 5": This indicates the combination of ZIP code and state, which also has 33,178 distinct values.
"2, 5": This indicates the combination of city and state, which has 27,435 distinct values.
"1, 2, 5": This indicates the combination of ZIP code, city, and state, which, as expected, has the same number of distinct values as the individual combinations of ZIP code and city (33,178).

### MVC
```SQL
CREATE STATISTICS stts3 (mcv) ON city, state FROM zipcodes;
```

#### Query
```SQL
ANALYZE zipcodes;

SELECT m.* FROM pg_statistic_ext join pg_statistic_ext_data on (oid = stxoid),
                pg_mcv_list_items(stxdmcv) m WHERE stxname = 'stts3';
```
#### Result
```SQL
index |         values         | nulls | frequency | base_frequency
-------+------------------------+-------+-----------+----------------
     0 | {Washington, DC}       | {f,f} |  0.003467 |        2.7e-05
     1 | {Apo, AE}              | {f,f} |  0.003067 |        1.9e-05
     2 | {Houston, TX}          | {f,f} |  0.002167 |       0.000133
     3 | {El Paso, TX}          | {f,f} |     0.002 |       0.000113
     4 | {New York, NY}         | {f,f} |  0.001967 |       0.000114
     5 | {Atlanta, GA}          | {f,f} |  0.001633 |        3.3e-05
     6 | {Sacramento, CA}       | {f,f} |  0.001433 |        7.8e-05
     7 | {Miami, FL}            | {f,f} |    0.0014 |          6e-05
     8 | {Dallas, TX}           | {f,f} |  0.001367 |        8.8e-05
     9 | {Chicago, IL}          | {f,f} |  0.001333 |        5.1e-05
   ...
(99 rows)
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

https://www.citusdata.com/blog/2018/03/06/postgres-planner-and-its-usage-of-statistics/

## Index

```SQL
CREATE UNIQUE INDEX active_employee_per_department_idx ON employees (department) WHERE is_active = true;
```

A unique index with a WHERE clause combines the benefits of both an index and a unique constraint.

Index: It acts like a regular index, facilitating faster data retrieval by organizing the data in a structured way. This helps in speeding up queries that involve the indexed columns.

Unique Constraint: It enforces uniqueness on a subset of the data, similar to a unique constraint. This ensures data integrity by preventing duplicate values within the specified subset, as defined by the WHERE clause.

By combining these features, a unique index with a WHERE clause provides the efficiency of indexing with the data integrity enforcement of a unique constraint, making it a powerful tool for optimizing performance while maintaining data consistency.

https://www.postgresql.org/docs/15/sql-createindex.html

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
