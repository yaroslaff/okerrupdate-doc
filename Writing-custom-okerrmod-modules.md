# Writing custom okerrmod modules



## Task
We are small company called Google (but we have our humble website https://google.com/) and we want to monitor it and alert us if it's down.

## Step 1: Not writing module
As quick solution we can just use standard [runstatus](checks/runstatus) module:

enable it:
~~~
# okerrmod --enable runstatus
~~~

edit config fille `/etc/okerr/mods-env/runstatus`
and make sure we have line like this (not connected):
~~~
google_RUN = curl --silent --head --fail --output /dev/null http://google.com/
~~~

Now run it:
~~~
# okerrmod runstatus
okerr updated (200 OK) pi:google@okerr = 0
~~~

## Step 2: Writing small useful custom module

We decided to control not only fact that page is loaded or not, but also if it loaded fast enough. And we will create small module for it. We will use Python language for module, but modules can be written in any language, it just should use same format of output as okerrmod expect.

Create module directory:
~~~
mkdir /etc/okerr/mods-available/pageload
~~~

Edit `/etc/okerr/mods-available/pageload/check` :

~~~python
#!/usr/bin/python3

import requests

url = 'http://google.com/'
max_time = 0.5

print("# Loading", url)
r = requests.get(url)
load_time = r.elapsed.total_seconds()
print("# Loaded in {}s".format(load_time))

if load_time > max_time:
    status="ERR"
else:
    status="OK"

print("STATUS: {}".format(status))
~~~

Make it executable:
~~~
chmod +x /etc/okerr/mods-available/pageload/check
~~~

Now... module is ready! `okerrmod --list` will list it. And we can run it for debugging.
~~~
$ okerrmod --dump pageload
# Loading http://google.com/
# Loaded in 0.184669s
STATUS: OK

~~~
Okerrmod modules can print any debug info starting with '#' sign. Okerrmod just ignores this line but they are useful for debugging.

Now you can enable module: `okerrmod --enable pageload` and then simple `okerrmod` command will run it.

## Step 3: Using environment configuration

We used hardcoded values for URL and max_time in our module. But better if user can override it easily. So, we will create config file for it `/etc/okerr/mods-env/pageload` :
~~~
URL=http://google.com
MAX_TIME=0.5
~~~

and modify first part of our script a little:
~~~python
import requests
import os

url = os.getenv('URL','http://google.com/')
max_time = float(os.getenv('MAX_TIME','3'))
...
~~~

Now, if we will make changes to env file (e.g. set MAX_TIME=0.01) we will see result:
~~~
$ okerrmod --dump pageload
# Loading http://google.com
# Loaded in 0.185091s
STATUS: ERR
~~~

We want to use this current config as default, so we copy it with name `_config` to pageload directory:
`cp /etc/okerr/mods-env/pageload /etc/okerr/mods-available/pageload/_config`

Now, if we will disable module ( `okerrmod --disable pageload` ) and delete config ( 'rm /etc/okerr/mods-env/pageload' ) and then enable it again, we will see it will start with default config:
~~~
$ sudo okerrmod --enable pageload
enable /etc/okerr/mods-available/pageload
make default config file: /etc/okerr/mods-env/pageload
~~~

## Step 4: Use numbers!
Using `MAX_LIM` variable is not nice way. If we will want to adjust value, we will need to log in to server. Better to use [Okerr](https://okerr.com/) numerical indicators. If we will use it, we will be able to set maximal value right in UI, and they have good feature to use relative `diffmin` and `diffmax` and we can set alert if page load time is more then 0.5s higher then it was before, or use relative value like 20%.

Our last version of module outputs just one valueable line: `STATUS: OK` (or `ERR`). We want to use numbers, same was as `df` or `maxfilesz` modules does. Lets peek how they do it!
~~~shell
$ okerrmod --dump df
NAME: pi:df-/
TAGS: la
METHOD: numerical|maxlim=90
DETAILS: 48.61%, 13.7G/28.2G used, 13.3G free
STATUS: 48.61

...
~~~

Heh, it was easy! Now, lets adjust our script for this:
~~~python
#!/usr/bin/python3

import requests
import os

url = os.getenv('URL','http://google.com/')
max_time = float(os.getenv('MAX_TIME','3'))

print("# Loading", url)
r = requests.get(url)
load_time = r.elapsed.total_seconds()
print("# Loaded in {}s".format(load_time))

print("METHOD: numerical|maxlim={}".format(max_time))
print("DETAILS: URL {} loaded in {:.2f} seconds".format(url, load_time))
print("STATUS: {}".format(load_time))
~~~

Our script became even shorter and simpler, we do not make decision inside script, okerr server will compare status (load_time) against maxlim. Lets verify it with `okerrmod --dump pageload` and delete old indicator in okerr UI, because old indicator has 'heartbeat' type (not 'numerical' as we need). Now, lets run it!

~~~
$ okerrmod pageload
okerr updated (200 OK) pi:pageload@okerr = 0.22991
~~~
Great! Indicator created, it has numerical type, indicator option 'maxlim' initialized with default value from config file. And we can adjust maxlim or other options right from UI. If we will set maxlim 0.1 in UI, next okerrmod run will switch indicator to `ERR` and trigger alert.

## Step 4: Update many indicators at once

Our module is doing great for checking our main page. But we want to check many pages. One of solution is to use our module many times as [virtual](Virtual modules) but better if module is smart enough to check many parameters.

And this is actual task for us because we are going launch two new small projects - youtube and gmail (and they should be reliable as our main google.com project!).

But how we can report many indicators? Lets try to steal that secret technology from other module which can do this:
~~~
root@pi:/etc/okerr/mods-enabled# okerrmod la3
okerr updated (200 OK) pi:la1@okerr = 0.22
okerr updated (200 OK) pi:la5@okerr = 0.09
okerr updated (200 OK) pi:la15@okerr = 0.07
~~~
Yesss! `la3` module is good for this, it can update many indicators. How? Lets see:
~~~
root@pi:/etc/okerr/mods-enabled# okerrmod --dump la3
NAME: pi:la1
TAGS: la
METHOD: numerical|maxlim=2
STATUS: 0.26

NAME: pi:la5
TAGS: la
METHOD: numerical|maxlim=2
STATUS: 0.11

NAME: pi:la15
TAGS: la
METHOD: numerical|maxlim=2
STATUS: 0.08

~~~
It just repeats update information separated by empty lines. So simple!

We will allow to specify many URLs in URL variable (separated by space). 
Config:
~~~
URLS=http://google.com http://youtube.com http://gmail.com
MAX_TIME=0.5
~~~

Module:
~~~python
#!/usr/bin/python3

import requests
import os

prefix = os.getenv('PREFIX')
urls = os.getenv('URLS','http://google.com/')
max_time = float(os.getenv('MAX_TIME','3'))

for url in urls.split(' '):
    name = url.split('//')[1]

    print("# Loading", url)
    r = requests.get(url)
    load_time = r.elapsed.total_seconds()
    print("# Loaded in {}s".format(load_time))

    print("NAME: {}{}".format(prefix, name))
    print("METHOD: numerical|maxlim={}".format(max_time))
    print("DETAILS: URL {} loaded in {:.2f} seconds".format(url, load_time))
    print("STATUS: {}".format(load_time))
    print()
~~~

Changes:
- we use NAME directive (otherwise okerrmod will update one indicator with three different values) and use PREFIX env variable for this. No need to use default value for PREFIX, because it's system variable and okerrmod always provides it.
- we split 'protocol://' part from URL to be used as name of indicator. We must do this, because indicator names cannot contain duplicate slash (//)
- renamed config variable URL to URLS, and script variable url to urls
- perform checks in loop (for each url in urls)
- after each indicator report, we print empty line - okerrmod will understand this as separator

also, lets add 'PREFIX2=pageload:' to our config to group all indicators together and we're done:
~~~
# okerrmod pageload
okerr updated (200 OK) pi:pageload:google.com@okerr = 0.180459
okerr updated (200 OK) pi:pageload:youtube.com@okerr = 0.324318
okerr updated (200 OK) pi:pageload:gmail.com@okerr = 1.362101
~~~

## Step 5: Further improvements
Actually, our module is already working and doing all that we need. But we may improve it a little for sense of perfections.

### Set description
Our module is not looking good in list:
~~~
$ okerrmod --list
+ pageload:
+ backups: Check freshness for backup files
+ df: Free disk space
...
~~~

No description at all. Okay, just steal `_info` file from any other module and edit it:
~~~
# locate /_info
/usr/local/lib/python3.7/dist-packages/okerrupdate/mods-available/backups/_info
/usr/local/lib/python3.7/dist-packages/okerrupdate/mods-available/empty/_info
...
# cp /usr/local/lib/python3.7/dist-packages/okerrupdate/mods-available/empty/_info /etc/okerr/mods-available/pageload/
~~~

Now it will looks like this:
~~~
root@pi:/etc/okerr/mods-enabled# okerrmod --list
+ pageload: Report web page load time
~~~

### Add better exceptions handling
For sake of simplicity, we did not used try/except in basic examples, but lets add it. To make error, add wrong domain to URLS and see what happen... check crashed.

Thanks to okerr phylosophy - this is not a big deal. If module crashed it will not send positive confirmation, indicator will expire, switch to ERR and send alert. You will know about problem. 

But better to handle exceptions anyway. 

Minimal - just use requests.get in try/except with continue:
~~~python
    try:
        r = requests.get(url)
    except requests.RequestException as e:
        continue
~~~
Now, if url will fail, we just will not update indicator and let it expire.

More accurate:
~~~python
#!/usr/bin/python3

import requests
import os

prefix = os.getenv('PREFIX')
urls = os.getenv('URLS','http://google.com/')
max_time = float(os.getenv('MAX_TIME','3'))

for url in urls.split(' '):

    print("# Loading", url)

    try:
        name = url.split('//')[1]
    except IndexError:
        continue

    try:
        r = requests.get(url)
    except requests.RequestException as e:
        print("# Exception: {}".format(e))
        details = str(e)
        status = -1
    else:
        load_time = r.elapsed.total_seconds()
        print("# Loaded in {}s".format(load_time))
        details = "URL {} loaded in {:.2f} seconds".format(url, load_time)
        status = load_time

    print("NAME: {}{}".format(prefix, name))
    print("METHOD: numerical|maxlim={}|minlim=0".format(max_time))
    print("DETAILS: {}".format(details))
    print("STATUS: {}".format(status))
    print()

~~~

Now, indicators with problem will have numerical value -1. We can manually edit each indicator in okerr UI to use minlim=0 (then -1 will trigger ERR). But we modified 'METHOD' here, and we can delete old indicators. Run okerrmod again and it will re-create it with minlim=0.

~~~
# okerrmod pageload
okerr updated (200 OK) pi:pageload:http://google.com/@okerr = 0.175678
okerr updated (200 OK) pi:pageload:ZZZ@okerr = -1
~~~

### Pre-enable check
Our module uses `requests` python module. Most likely it's already installed (because it's used by okerrmod itself) but lets suppose we have a chance that user can miss it. Obviously, our module will not work. Would be good to check for this situation. Okerrmod already has [sql](checks/sql) module which has similar check and we can use it as template. Copy `preenable` script from sql:
~~~
# locate preenable
/usr/local/lib/python3.7/dist-packages/okerrupdate/mods-available/sql/preenable
# cp /usr/local/lib/python3.7/dist-packages/okerrupdate/mods-available/sql/preenable /etc/okerr/mods-available/pageload/
~~~
And modify it a little:
~~~
#!/usr/bin/python3
import sys

try:
    import requests
except ImportError as e:
    print("{}. To install module: sudo pip3 install requests".format(e))
    sys.exit(1)
else:
    sys.exit()
~~~

preenable script should just either exit with code 0 if everything is fine, or any other code and print info for user.

Now our pageload module is ready for production and we can share it and maybe even include in okerrmod default distribution!
