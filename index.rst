okerrupdate documentation
#########################


.. toctree::
    :hidden:

    okerrupdate-utility
    okerrupdate-module
    okerrmod-usage
    common-modules-properties
    basic-okerrmod-modules
    Virtual-modules
    checks/runstatus
    checks/runline
    Writing-custom-okerrmod-modules
    Running-checks-with-different-periods
    manual-run
    checks/sql
    checks/ok 


Okerrupdate is client-side library and utilities to work with `okerr <https://github.com/yaroslaff/okerr-dev>`_ monitoring.
You need it if you want to monitor your server 'from inside' (e.g. watch free disk space, load average, CPU temperature 
and number of customers/purchases in web application database).


Installation
************

Install as python package (recommended way for all users):

.. code-block:: shell

    pip3 install okerrupdate

Or clone source code from `okerrupdate repository <https://github.com/yaroslaff/okerrupdate>`_ (for developers):

.. code-block:: shell

    git clone https://github.com/yaroslaff/okerrupdate.git


Configuration
*************

:doc:`configuration`

okerrupdate 
***********

:doc:`okerrupdate-utility`

:doc:`okerrupdate-module`


okerrmod
********

Okerrmod is utility to work with okerr check modules.

Check module (or simple 'check' or 'module') is small program that checks one parameter of system, such as CPU temperature, free space on partition, is apache running or not and similar things.

:doc:`okerrmod-usage`


Okerrmod modules
****************

:doc:`common-modules-properties`

:doc:`basic-okerrmod-modules`


Extending okerrmod
******************

:doc:`Using virtual modules <Virtual-modules>`

:doc:`checks/runstatus` and :doc:`checks/runline`

:doc:`Writing-custom-okerrmod-modules`


okerrapi
********

:doc:`okerrapi`


Tips and tricks
****************
:doc:`Running-checks-with-different-periods`

:doc:`manual-run`
