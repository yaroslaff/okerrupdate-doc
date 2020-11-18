
# okerrmod usage

## init

After install, run `okerrmod --init` once. This will prepare directory structire inside /etc/okerr (will not overwrite files which exists), will enable few modules, prepare /etc/okerr/okerrupdate config file template and make /etc/cron.d/okerrmod cron job.

Do not forget to edit /etc/okerr/okerrupdate config file and set your textid (required) and secret (optional).

## list modules
`okerrmod --list` lists all known modules. If line starts with `+` - module enabled, if starts with `-` - disabled.

# enable/disable modules
`okerrmod --enable mod1 mod2 ...` enables one or more modules. When module is enabled:
- symlink created from /etc/okerr/mods-enabled to actual module location
- default module config file `_config` copied to /etc/okerr/mods-env/<module> (not overwriting)

module specified either by name (la, df, client_ip) or as path to module directory

When disabling - symlink deleted. Config files are not deleted.

## Running
Usually runs from cron job. If you need to run manually, usually simple `okerrmod` command without any arguments is enough to run all enabled modules.

### Configuration order
Configuration is taken from command line arguments. Arguments default values are often read from environment variables. And file /etc/okerr/okerrupdate is processed to set environment. So, any method will work, but CLI arguments has highest priority, and env variables in okerrupdate config file has lowest priority.

### Important options

- `-p PREFIX` - prefix to indicator names ($PREFIX). If not specified anywhere, generated from hostname (prefix for server1.example.com is 'server1:')
- `-i <textid>` - project textid (required) ($OKERR_TEXTID)
- `-S <secret>` - policy secret (matching server-side value for secret in 'Default' or other policy) ($OKERR_SECRET).

Followint options are only needed if you run your own okerr server:
- `--url <URL>` - okerr server URL ($OKERR_URL)
- `--direct` - use URL directly, do not use 'director' feature. ($OKERR_DIRECT is 0 or 1. default 0). To use private server, you need to use `--url <URL>` and `--direct`.


## Tips and tricks
### Just run all enabled modules
~~~
okerrmod
~~~

### Run just one module
~~~
okerrmod client_ip
~~~

### Debugging
Run without really updating indicator
~~~
# okerrmod --dry -v client_ip
load env for client_ip from /etc/okerr/mods-env/client_ip
... Run check /usr/local/lib/python3.7/dist-packages/okerrupdate/mods-available/client_ip/check
POLICY= NAME=bravo:ip NAME=bravo:ip TAGS=ip METHOD=string|options=reinit dynamic DETAILS=37.59.102.26 STATUS=37.59.102.26
~~~

or by full path:
~~~
# okerrmod --dry -v /etc/okerr/mods-enabled/client_ip
...
# okerrmod --dry -v /usr/local/lib/python3.7/dist-packages/okerrupdate/mods-available/client_ip
...
~~~

All the details (--dump never send update to server):
~~~
root@bravo:~# okerrmod --dump client_ip
NAME: bravo:ip
TAGS: ip
METHOD: string|options=reinit dynamic
DETAILS: 37.59.102.26
STATUS: 37.59.102.26
~~~