# Configuration
okerrupdate, okerrmod and okerrapi are configuraed via CLI arguments, or environment or config files. Config files are in .env format, e.g.:
~~~
  PREFIX=myserver:
  OKERR_TEXTID=MyTextID
  OKERR_SECRET=MySecretSecret
  # OKERR_URL=http://localhost.okerr.com/
  # OKERR_DIRECT=1
  OKERR_MOD_AVAIL=

  OKERR_API_KEY=...
~~~

Config file is named okerrupdate.conf and searched in following order:
- ~/okerr/
- ~/.okerr/
- /usr/local/etc/okerr/
- /etc/okerr/
