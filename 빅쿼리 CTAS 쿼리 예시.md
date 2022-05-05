.

Data_Engineering_TIL(20220505)

```sql
CREATE TABLE `minman_project.qa_dataset.test_table`
AS SELECT * FROM `minman_project.test_dataset.test_table_20*`
WHERE _TABLE_SUFFIX BETWEEN '210505' AND '220505';
```