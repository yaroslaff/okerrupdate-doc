# okerrapi

okerrapi is simple CLI tool to work with okerrupdate API. okerrapi needs API key to work. You can create/revoke API keys in projects settings.

API_KEY could be specified in config file (see **Configuration** on main page), or as environment variable `$OKERR_API_KEY` or as argument: `--key KEY`.

~~~shell
# get indicators, explicitly give textid and API-KEY
okerrapi -i textid -k MY-PROJECT-API-KEY indicators

# get list of indicators, textid and API-KEY taken from env or /etc/okerr/okerrupdate 
okerrapi indicators

okerrapi indicators prefix:

# get info about particular indicator (-n)
okerrapi -n srv1:myindicator indicator

# list indicators matching by filter (status=OK, has tag 'sslcert', host option is 'okerr.com')
okerrapi filter sslcert OK host=okerr.com

# set parameter days=20 and retest for indicator (-n)
okerrapi -n sslcert:okerr.com set days=20 retest=1
~~~
