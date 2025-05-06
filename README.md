# OTUS.Lesson44.Mysql
Домашнее задание:
1) Базу развернуть на мастере и настроить так, чтобы реплицировались таблицы:
| bookmaker          |
| competition        |
| market             |
| odds               |
| outcome            |
2) Настроить GTID репликацию

Выполнение домашнего задания:  
Тестовый стенд 2 VM ( 2CPU 2GB RAM 6GB HDD OS DEBIAN 12)  
В файле vars укажем root пароль в переменной 'root_pass'  
Запускаем playbook mysql, дожидаемся выполенения и проверям результаты:   
Проверим что на реплике есть база данных bet и что реплицируются нужные таблицы:  
```
mysql> SHOW DATABASES LIKE 'bet';
+----------------+
| Database (bet) |
+----------------+
| bet            |
+----------------+
1 row in set (0,02 sec)
mysql> USE bet;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> SHOW TABLES;
+---------------+
| Tables_in_bet |
+---------------+
| bookmaker     |
| competition   |
| market        |
| odds          |
| outcome       |
+---------------+
5 rows in set (0,01 sec)
mysql>  SHOW SLAVE STATUS\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 10.200.3.95
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 120137
               Relay_Log_File: deb12-relay-bin.000002
                Relay_Log_Pos: 120353
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: bet.events_on_demand,bet.v_same_event

```

Проверим что репликация работает. Добавим на матере данные в БД bet:  
```
mysql> USE bet;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> 
mysql> 
mysql> INSERT INTO bookmaker (id,bookmaker_name) VALUES(1,'1xbet');
Query OK, 1 row affected (0,05 sec)

mysql> SELECT * FROM bookmaker;
+----+----------------+
| id | bookmaker_name |
+----+----------------+
|  1 | 1xbet          |
|  4 | betway         |
|  5 | bwin           |
|  6 | ladbrokes      |
|  3 | unibet         |
+----+----------------+
5 rows in set (0,00 sec)
```

и проверим что на slave появились эти же строки:  
```
mysql> SELECT * FROM bookmaker;
+----+----------------+
| id | bookmaker_name |
+----+----------------+
|  1 | 1xbet          |
|  4 | betway         |
|  5 | bwin           |
|  6 | ladbrokes      |
|  3 | unibet         |
+----+----------------+
5 rows in set (0,00 sec)
```

Глянем binlog  
```
*************************** 97. row ***************************
   Log_name: mysql-bin.000001
        Pos: 114295
 Event_type: Gtid
  Server_id: 1
End_log_pos: 114379
       Info: SET @@SESSION.GTID_NEXT= '51777a12-2a57-11f0-859f-bc2411c518df:38'
*************************** 98. row ***************************
   Log_name: mysql-bin.000001
        Pos: 114379
 Event_type: Query
  Server_id: 1
End_log_pos: 114522
       Info: GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%' /* xid=57 */
*************************** 99. row ***************************
   Log_name: mysql-bin.000001
        Pos: 114522
 Event_type: Gtid
  Server_id: 1
End_log_pos: 114608
       Info: SET @@SESSION.GTID_NEXT= '51777a12-2a57-11f0-859f-bc2411c518df:39'
*************************** 100. row ***************************
   Log_name: mysql-bin.000001
        Pos: 114608
 Event_type: Query
  Server_id: 1
End_log_pos: 114684
       Info: BEGIN
*************************** 101. row ***************************
   Log_name: mysql-bin.000001
        Pos: 114684
 Event_type: Query
  Server_id: 1
End_log_pos: 114814
       Info: use `bet`; INSERT INTO bookmaker (id,bookmaker_name) VALUES(1,'1xbet')
*************************** 102. row ***************************
   Log_name: mysql-bin.000001
        Pos: 114814
 Event_type: Xid
  Server_id: 1
End_log_pos: 114845
       Info: COMMIT /* xid=78 */
102 rows in set (0,00 sec)
```
