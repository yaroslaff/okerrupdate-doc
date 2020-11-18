# Basic okerrmod modules

[[_TOC_]]

These modules are included in okerrmod package right from install. But it's always possible to extend it with other modules or write your own module. All basic modules are open-source and written in Python, so you can easily adopt it for your needs or use as example for your module.

## backups
Check freshness for backup files. okerr will alert if recent backup missing.

Checks files by `PATHLIST` glob specification (can provide many globs separated by space, e.g. `/home/backups/*.tar.gz /var/www/backups/*/*gz`), ignores files older then `FRESH` (1d by default). For each other file, transforms file basename into indicator name (using `DATEFMT` and `DATESUB` variables), so mybackup-2020-01-21.tar.gz transformed into mybackup-DATE.tar.gz and this indicator updated. Use `PREFIX2` variable if you want to place all indicators into subtree (e.g. `PREFIX2=backups:`)

If today backup was not created for any reason, old backup file will be skipped (because of `FRESH` limitation), indicator will not be updated, so it will expire and switch to ERR status by server.

Nice trick: if you store many backups in one directory, you dont have to any special configuration when adding new backup there. e.g. if you stored serverA-...tar.gz and serverB-tar.gz backups, and then added serverC-...tar.gz file, this module automatically will detect it and create indicator for serverC backups and will alert if no new backups.

Also, indicators will send alert if `diffmin` restriction violated. For example, if diffmin=0 and old value was 100 and new is 99, this will trigger error. Usually for backups this makes sens (they should grow in size), but gzipped files could be little shorter, so backup uses relative 'DIFFMIN=-10%' restriction. It will alert if new size is less then old size -10%.

## client_ip
Detect and report server external IP address

Detect IP using any HTTP(s) service such as `URL=https://diagnostic.opendns.com/myip` (or https://ifconfig.me or https://ifconfig.io or any other). Uses `USER_AGENT` variable.


## df
Reports free/used disk space on mounted partitions. Skips partition if device matches `IGNORE_DEVICE` regex (default: ^/dev/loop) or if mount point matches `IGNORE_MOUNT` regex (default - empty). New created indicators will be numeric, and will use maxlim = `MAXLIM` (90%).


## dirsize
Reports total directory sizes for each directory (glob) from `PATHLIST` (default: `/root /home/*`). If glob expression matches more then one directory (like /home/*), one indicator will be created/updated for each directory.

if `FULLPATH` is 1, indicator name will include full path do directory, if 0, it will use only basename (may be not very reliable. if PATHLIST is "dir1 dir2" and both has subdir "x", their indicator names will overlap)

Option `USE_DU` (0 or 1) selects how to count directory size. if USE_DU=0, built-in method used (python, slower).  If USE_DU=1 (default), system du utility is used which is more then two times faster.

`SKIP` is space-separated list of globs to skip. For example, to watch LXC virtual machines sizes without snapshots we use this:
~~~
PATHLIST=/var/lib/lxc/*
SKIP=/var/lib/lxc/lxc-monitord.log /var/lib/lxc/*/snaps/*
~~~
First skip glob needed to skip logfile in /var/lib/lxc (otherwise, it will be used as indicator name). Second is needed to skip /var/lib/lxc/servername/snaps/* when we count dirsize for /var/lib/lxc/servername.


## empty
Report files which must be empty (e.g. error.log) but actually not empty. Useful to watch error logs (if record will appear in log - you will get alert).

PATHLIST is space-separated list of glob expresstions (default: `/var/log/apache2/error.log /var/log/mysql.err`)
File is skipped (even if matches PATHLIST glob) if it matches `SKIP_GLOB` (default: empty)

See also notempty module


## la
Very simple module, just report system load average. `PERIOD` could be either 1, 5 or 15 (only one of these three).
New created indicator will use `MAXLIM` limit

## la3
Very similar to la module, but creates one indicator for each of three types of Load Average.
Load average (3 indicators for 1, 5 and 15 minutes). Module mostly useful as simple example how to update few indiators at once. 

## linecount
Reports number of lines in files (usually: log files) as numerical indicators. Useful to detect simple anomalies (when log file suddenly starts to grow much faster then usual).

Example config:
~~~
PATHLIST=/var/log/okerr/*err.log /var/log/apache2/error.log
~~~

After indicators are created, it's recommended to adjust diffmax to set limit, when indicator will send alert.

By default, linecount does not report files which are not exists (so, their indicators will expire and switch to ERR). If missing file is ok, you can use option `MISSING_OK`, like this:
~~~
# space-separated list of files
MISSING_OK=/var/log/myapp/myapp-error.log
~~~
If any file from this list will be missing, module will report '0' for it.

## maxfilesz
Processes `PATHLIST` as space-separated list of directories to check (not glob!). Ignores files matching `SKIP` regex (e.g. `\.\d+$|\.gz$`). Reports size of largest file. If indicator created, `MAXLIM` value used.

After indicator created, you may adjust it's values (e.g. set diffmin, diffmax).

Module useful to detect abnormally large logs, or abnormally fast growing logs (which often sign of problem)

To skip specific file or pattern, add it to SKIP (separated by pipe char `|`):
~~~
SKIP=\.\d+$|\.gz$|/var/log/mail\..*
~~~
This regex skips all files ending by number (e.g. /var/log/syslog.1), all '.gz' files and /var/log/mail.*

## notempty

Report files which must be not empty, but actually empty (log files). Sometimes this problem may happen if logging is broken or any other error (e.g. no traffic reaches web or mail servers). 

Uses `PATHLIST` (as glob), `SKIP` regex and `MAXAGE` (default: 10m). File is not considered as problem, even if it's empty, if its mtime is very recent (e.g. it's just rotated)

See also empty module

## ok
[ok](checks/ok) trivial check. Just reports OK always.

## opentcp
Reports open network listening sockets
Port is reported if it's status matches regex `STATUS` (def: 'LISTEN'), local address matches `LADDR` regex (`0.0.0.0|::` which matches world-listening sockets, both IPv4 and IPv6). Port is skipped if it's listed in `SKIP_PORT` (e.g. "22 80") or if program which uses this port matches `SKIP_EXE` regex (e.g. '.*/sshd')

## runline
Run program and report line of output. Very powerful module, which could be used as 'proxy' module to any other diagnostic.

See detailed documentation page for [runline](checks/runline) module.

## runstatus
Run program and report exit code. (This is simpler then [runline](checks/runline) module).

See detailed documentation page for [runstatus](checks/runstatus) module.

## SQL
[sql](checks/sql) - perform Mariadb/MySQL numeric SQL query

## uptime 
Reports server uptime. New created indicator will use diffmin=0. If server will be rebooted, next update will violate diffmin restrictions, so indicator will switch to ERR and send alert.

## version
Reports okerrupdate version number (e.g. '1.2.23'). Changed only after okerrupdate upgraded. Useful to find out machines with old okerrupdate versions.
