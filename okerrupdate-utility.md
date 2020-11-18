# okerrupdate utility

## Basic examples

```shell
$ okerrupdate myindicator OK
okerr updated (200 OK) myindicator@okerr = OK
```

```shell
$ okerrupdate -p Daily -m 'numerical|maxlim=37' -d 'everything is good' temp 36.6
okerr updated (200 OK) temp@okerr = 36.6
```

Note: `policy` (`-p`) and `method` (`-m`) are used only if no such indicator exists and server creates new indicator. Otherwise it's ignored (but not a problem to send it with every update).


## Running your own okerr server
If you work with your own server, you can use --url parameter and --direct (to skip redirection stage, which is used only in clusters)

~~~shell
okerrupdate --url http://localhost/ --direct -d "just testing" myindicator OK
~~~