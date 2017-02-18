Setting up MySQL on xenial
===

久々にUbuntu 16.04 (xenial)にMySQLをinstallしたら,root userの利用に`sudo`が必要になっていて不便だったので`sudo`不要にする設定メモ.

```bash
# install
$ sudo apt-get install mysql-server

# connect mysql as a sudo user
$ sudo mysql -uroot -p

mysql> SELECT User, Host FROM mysql.user;
+------------------+-----------+
| User             | Host      |
+------------------+-----------+
| debian-sys-maint | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
3 rows in set (0.00 sec)

mysql> DROP USER 'root'@'localhost';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT User, Host FROM mysql.user;
+------------------+-----------+
| User             | Host      |
+------------------+-----------+
| debian-sys-maint | localhost |
| mysql.sys        | localhost |
+------------------+-----------+
2 rows in set (0.00 sec)

mysql> CREATE USER 'root'@'%' IDENTIFIED BY '';
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';
Query OK, 0 rows affected (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
```

これで`sudo`なしでMySQL consoleを起動できる.

### Credits

- http://askubuntu.com/questions/766334/cant-login-as-mysql-user-root-from-normal-user-account-in-ubuntu-16-04
