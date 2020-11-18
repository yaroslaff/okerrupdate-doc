# module OK

OK is trivial check, mostly used to show how easy is to start writing okerrmod modules.

Full code of OK check is:
~~~
#!/usr/bin/python3

print("STATUS: OK")
~~~

It has no explicit configuration, it will update/create indicator with name hostname:ok of type `heartbeat`, `Default` policy and status `OK`.

Even such simple module is useful. If machine will crash, or network connection will go down, or something will happen to okerrmod (e.g. uninstalled one of required python modules) or cron will stop working - it will stop sending updates to okerr server. And, after indicator will expire, it will switch to `ERR` status and send Email/Telegram alerts.
