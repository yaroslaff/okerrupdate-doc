# runstatus module

Checks all environment (configuration) variables ending with _RUN or _OK and process each of it. for each such variable, (for example `apache_RUN`), first part is considered as *name*. Specified program with arguments is executed. 

For *_RUN variables program exit code is used as value for *numerical* indicator. (exit code 0 is considered successful, any other exit code is considered as failure). If new indicator created, it will have numerical type with minline=0 and maxlim=0.

For *_OK variables exit code 0 is considered as 'OK' and any other code considered as 'ERR' status for *heartbeat* indicator.

If program writes anything to stderr first line of stderr is used as indicator details. If nothing in stderr, first line of stdout is used as details.


## Examples
~~~
true_RUN=/bin/true
~~~
Will create *numerical* indicator PREFIX:true and indicator will be OK always (unless /bin/true binary will be deleted).

Almost similar, but `_OK` instead of `_RUN`:

~~~
true_OK=/bin/true
~~~
Will create  *heartbeat* indicator PREFIX:true.

~~~
uptimestring_RUN=/usr/bin/uptime
~~~
Will create indicator prefix:uptimestring, always OK, details will be output of uptime program


~~~
google_RUN = curl --silent --head --fail --output /dev/null http://google.com/
~~~
Will try to fetch this page, if case of any problems (domain expired, no hostname, server is down, page is not found) will return non-zero code, indicator will switch to ERR and send alert.


## Using runstatus as proxy
You can write simple monitoring programs to be used with runline. Program should exit with code 0 to have indicator as OK, or any other code for ERR status. First line of output will be set to details.

This is example of *extremely logical* indicator program ( /tmp/girl.sh ):
~~~shell
#!/bin/bash

if (( RANDOM % 2 ))
then
	echo "I am happy!"
else
	echo "I am NOT happy!"
	exit 1
fi
~~~

If you will run it few times you will get output similar to this:
~~~shell
$ /tmp/girl.sh ; echo $?
I am NOT happy!
1
$ /tmp/girl.sh ; echo $?
I am happy!
0
~~~

Now, you can use it in runstatus:
~~~
girl_RUN = /tmp/girl.sh
~~~

Indicator will switch status OK/ERR depending on girl happines.

## See also
See also [runline](runline) module.