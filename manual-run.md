# Manual run

For debugging purpose, each check code can be started manually, setting proper environment variables:

~~~
$ PREFIX=myserver: PREFIX2=logs: PATHLIST=/tmp/*.log  okerrupdate/mods-available/linecount/check 
NAME: myserver:logs:ansible.log
TAGS: ansible.log
METHOD: numerical|minlim=0
DETAILS: 230 lines
STATUS: 230

NAME: myserver:logs:mail.log
TAGS: mail.log
METHOD: numerical|minlim=0
DETAILS: 24 lines
STATUS: 24

~~~