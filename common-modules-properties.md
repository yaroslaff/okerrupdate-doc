# Common module properties

# Location
When `okerrmod` is invoked without options, it runs modules following symlinks from /etc/okerr/mods-enabled. Each symlink points to directory with module (where `check` executable file is located). Actual module directory on disk is inside python package location, which could be different (e.g. `/usr/local/lib/python3.7/dist-packages/okerrupdate/mods-available/runstatus`). You can find it using `-v` option to okerrmod when enable/disable or run  modules.

Custom modules could be located anywhere (but make symlink to it from `/etc/okerr/mods-enabled/`), but recommended place is `/etc/okerr/mods-available`, this is empty directory for users custom modules and this is where okerrmod will find it for --list command).

# Configuration
Module configuration is located in `/etc/okerr/mods-env/<module>` 

Unless module will overwrite it, these values are used:

`PREFIX2`- 2nd part of prefix, usually empty '' but can be specified if module creates many indicators (such as la3 module) and you want all of them to be nested in common sub-tree (in `host:la` for example) to avoid cluttering of dashboard.

`NAME` - name of indicator (default: PREFIX+PREFIX2+name of module). Module can overwrite it (especially, when module creates more then one indicator)

`POLICY`- policy (only when create new indicator) 'Default' by default, but some modules may set other policy (e.g. Daily).

# Enable/Disable
When module is enabled:
- if `preenable` executable exists, it's executed and exit code checked (should be 0 if successful). If not - module not enabled. See `sql` module.
- `_config` template file from module directory copied to `/etc/okerr/mods-env/<module>`

