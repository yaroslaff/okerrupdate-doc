# okerrupdate module

You can use okerrupdate script itself as simple example (it's small).

## Very basic example

```python
#!/usr/bin/python
import okerrupdate

op = okerrupdate.OkerrProject('MyTextID', secret='MySecret')
i = op.indicator('temp', method='numerical|maxlim=37', policy='Daily')
i.update('36.6')
```

## Configuration
If OkerrProject created without textid or secret, it reads it from environment variables OKERR_TEXTID and OKERR_SECRET. You can pre-load it from /etc/okerr/okerrupdate file using python_dotenv:
```python
from okerrupdate import OkerrProject
from dotenv import load_dotenv

conf_file = '/etc/okerr/okerrupdate'
load_dotenv(dotenv_path = conf_file)
op = OkerrProject()
i = op.indicator('temp2', method='numerical|maxlim=37', policy='Daily')
i.update(36.6)
```

## Exceptions
All Exceptions derives from OkerrExc class:
~~~python
from okerrupdate import OkerrProject, OkerrExc

op = OkerrProject()
op.verbose()
i = op.indicator('testindicator', method='numerical|minlim=0|maxlim=100')

try:
    i.update(80)
except OkerrExc as e:
    print("Got OkerrExc: {}".format(e))
~~~

## Throttling
It's not recommented to send updates much more often then policy requires (For policy with period 1h, it's ok to send updates every 20 minutes). OkerrIndicator has built-in `throttle` parameter, default value is 300 seconds (5min). You can override it with `i.throttle = 30`. Updates will not be send to server if status not changed. Updates will be send if either status changed or throttle time passed since last update.

## Using specific okerr server
If you run your own okerr server, use `url` parameter to OkerrProject constructor to work with it. Also, you may want to use `direct=True` if you have just one server (not a cluster) to make things faster.
~~~python
op = OkerrProject(url='http://localhost/', direct=True)
~~~
