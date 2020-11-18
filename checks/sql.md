# Module SQL

To enable this module, you need to have installed `mysqlclient` python3 module.

## Pre-enable
~~~
apt install libmariadb-dev
pip3 install mysqlclient
~~~

## Configuration

If you want to create special user, use SQL command:
~~~
CREATE USER 'okerrtest'@'localhost' IDENTIFIED BY 'okerrtestpassword';
~~~

Set mysql user and password in config `/etc/okerr/mods-env/sql`. 
~~~
DBHOST=localhost
DBUSER=okerrtest
DBPASS=okerrtestpassword
DBNAME=
~~~


## Possible problems

Not installed mysqlclient:
```shell
# okerrmod --enable sql
2020/01/17 14:31:45 enable /usr/local/lib/python3.5/dist-packages/okerrupdate/mods-available/sql
2020/01/17 14:31:46 No module named 'MySQLdb'. To install module: sudo pip3 install mysqlclient
2020/01/17 14:31:46 Pre-enable check failed.
```

Not installed `libmariadb-dev` debian package. (or similar for your OS):
```shell
# pip3 install mysqlclient
Collecting mysqlclient
  Downloading https://files.pythonhosted.org/packages/d0/97/7326248ac8d5049968bf4ec708a5d3d4806e412a42e74160d7f266a3e03a/mysqlclient-1.4.6.tar.gz (85kB)
    100% |████████████████████████████████| 92kB 4.3MB/s 
    Complete output from command python setup.py egg_info:
    /bin/sh: 1: mysql_config: not found
    /bin/sh: 1: mariadb_config: not found
    /bin/sh: 1: mysql_config: not found
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/tmp/pip-build-_tal2qfd/mysqlclient/setup.py", line 16, in <module>
        metadata, options = get_config()
      File "/tmp/pip-build-_tal2qfd/mysqlclient/setup_posix.py", line 61, in get_config
        libs = mysql_config("libs")
      File "/tmp/pip-build-_tal2qfd/mysqlclient/setup_posix.py", line 29, in mysql_config
        raise EnvironmentError("%s not found" % (_mysql_config_path,))
    OSError: mysql_config not found
    
    ----------------------------------------
Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-_tal2qfd/mysqlclient/
```

In both cases, follow pre-enable instructions