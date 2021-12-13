## Deletion Anomalies
When you have a redundant database and you delete all tuples that contain a certain value for a certain attribute. All data on that attribute is lost, making it impossible to create new tuples with that attribute.

For instance, in the table:

| student_id | student_name | class |
|------------|--------------|-------|
| 1 | yigit | Java|
| 2 | ipek| Web and Database|

When you delete the student yigit, all information on the class Java is lost.