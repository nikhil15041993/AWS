## activate and monitor logs for an Amazon RDS MySQL DB instance

First, if you don't have a customer DB parameter group associated with your MySQL instance, create a custom DB parameter group and modify the parameter. Then, associate the parameter group with your MySQL instance.

If you already have a custom DB parameter group associated with the RDS instance, then proceed with modifying the required parameters.

Note: If you receive errors when running AWS CLI commands, make sure that you’re using the most recent version of the AWS CLI.

### Create a DB parameter group
```
* Open the Amazon RDS console, and then choose Parameter groups from the navigation pane.
* Choose Create parameter group.
* From the Parameter group family drop-down list, choose a DB parameter group family.
* For Type, choose DB Parameter Group.
* Enter the name in the Group name field.
* Enter a description in the Description field.
* Choose Create.
```

### Modify the new parameter group
```

* Open the Amazon RDS console, and then choose Parameter groups from the navigation pane.
* Choose the parameter group that you want to modify.
* Choose Parameter group actions, and then choose Edit.
* Choose Edit parameters, and set the following parameters to these values: ```General_log = 1``` (default value is 0 or no logging) ```Slow_query_log = 1``` (default value is 0 or no logging) ```Long_query_time = 2``` (to log queries that run longer than two seconds) ```log_output = FILE``` (writes both the general and the slow query logs to the file system, and allows viewing of logs from the Amazon RDS console) ```log_output =TABLE``` (writes both the general and the slow query logs to a table so you can view these logs with a SQL query)
* Choose Save Changes. Note: You can't modify the parameter settings of a default DB parameter group. You can modify the parameter in a custom DB parameter group if Is Modifiable is set to true.
```

### Associate the instance with the DB parameter group

```
* Open the Amazon RDS console, and then choose Databases from the navigation pane.
* Choose the instance that you want to associate with the DB parameter group, and then choose Modify.
* From the Database options section, choose the DB parameter group that you want to associate with the DB instance.
* Choose Continue.
```


### View the log

If log_output =TABLE, run the following command to query the log tables:

```
Select * from mysql.slow_log
Select * from mysql.general_log
```
Note: Enabling table logging can affect the database performance for high throughput workload. For more information about table-based MySQL logs, see Managing table-based MySQL logs.

If log_output =FILE, view database log files for your DB engine using the AWS Management Console.
