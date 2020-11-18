# runline module

Checks all environment (configuration) variables ending with _RUN and process each of it. for each such variable, (for example `apache_RUN`), first part is considered as *name*. Specified program with arguments is executed. One line of output (stdout stream) is used as value for indicator.

if *name*_GREP variable is used, output will be 'grepped' and only lines which matches grep regex will be left.
if *name*_LINE variabe is used (default 0), this line number is used. Lines are numbered from 0.
if *name*_CASE is 1 - grep will be case-sensitive, if 0 - case-insensitive.

If indicator is created, it will use method string with options 'reinit' and 'dynamic' (as dynamic, it will never switch status to ERR status, but will send alert always when string changed).

## Example
Simple example:
~~~
uptimestring_RUN=uptime
~~~
This will create indicator PREFIX:uptime with value of uptime, e.g: 
16:30:34 up 64 days, 10:50,  1 user,  load average: 0.13, 0.12, 0.09
(Obviously, this line will change every run and alert on every update, unless you will set indicator as silent.

Other useful example:
~~~
passwd_RUN=wc -l /etc/passwd
~~~
This will report number of lines in /etc/passwd. (example: "41 /etc/passwd"). If any user will be added or deleted, this line will change and indicator will send alert.

More complex example:
~~~
apache_RUN="systemctl status apache2"
apache_GREP=error
apache_CASE=1
apache_LINE=0
~~~

This will run command `systemctl status apache2` (check status of apache2 systemd daemon) and first line with word 'error' (if any) will be set as status for PREFIX:apache. 

If you want to run complex command (e.g. use shell pipes), you can use `sh -c` or `bash -c`:
~~~
# NOT working
sensors_RUN="pidof sensor.process|wc -w"

# Good, working
sensors_RUN="/bin/sh -c 'pidof sensor.process|wc -w'"
~~~

## Using runline as proxy
You can write simple monitoring programs to be used with runline. Program exit code will be ignored, and (to be used easier) it should output just one line of result in stdout.

Indicator will send alert when output line is changed.

Example script (/tmp/num.sh):
~~~shell
#!/bin/bash

echo $(( RANDOM % 100 ))
~~~

To add it to okerr (in /etc/okerr/mods-env/runline):
~~~
num_RUN=/tmp/num.sh
~~~

this will create indicator PREFIX:num with random value between 0 and 99. On next update, it will get other value
and send alert. You can modify indicator in okerr UI and set it to type 'numerical' and set minlim or maxlim values, to get alert, for example, only if this random number of over 95.



## See also
See also [runstatus](runstatus) module.