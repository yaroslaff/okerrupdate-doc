# Virtual modules

Sometimes module can check one parameter (for example, `la` module can check load average, for last 1, 5 or 15 minutes, but only one of three) and suppose we need to check two of it. 

Virtual module - is module which uses same code as 'real module' but uses different configuration.

Lets clone module `la`, but with name 'la5'. We will see how it's linked for enabled directory to real path and link it again with other name 

~~~
# okerrmod --clone la la5
Cloned /usr/local/lib/python3.7/dist-packages/okerrupdate/mods-available/la as check la5
Config: /etc/okerr/mods-env/la5
~~~

It will not be listed in `okerrmod --list` output (because it's 'virtual', in fact, this is still same `la` module) but we can run it and it enabled (okerrmod without arguments will run it along with other modules):
~~~
# okerrmod la la5
okerr updated (200 OK) pi:la@okerr = 0.1
okerr updated (200 OK) pi:la5@okerr = 0.1

~~~

And now edit `la5` config. While `la` module has `PERIOD=15`, we will set `PERIOD=5` for `la5`.

Now, lets run it:
~~~
# okerrmod la la5
okerr updated (200 OK) pi:la@okerr = 0.1
okerr updated (200 OK) pi:la5@okerr = 0.15

# uptime
07:58:09 up 42 days, 19:17,  2 users,  load average: 0.27, 0.15, 0.10
~~~

Done! Now, we're using module which can monitor just one parameter to monitor many parameters.