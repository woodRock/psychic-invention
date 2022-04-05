The software utility `cron <https://en.wikipedia.org/wiki/Cron>`_ was developed at AT&T and Bell Labs. 
It is often referred to as a cron job. 
This utility provides a time-based scheduler on Linux systems. 
It can be used to run a script at specific times, repeatedly. 
For example, we could check for updates on the for the `apt` package _once_ a week \[1\]. 

Cron is most suitable for scheduling repetitive tasks. In our case, we ingest data from the moorings once a day. We need to check the central databases, which contains mooring data and update the existing database on NZODN to include the new record. 

crontab
~~~~~~~

The frequency at which a cronjob is run depends on the `crontab` (cron table) file. 
This is a configuration file that specifies scripts to be run on a schedule. 
All the `crontab` files are stored in a directory where the cron `daemon` is kept. 
Users can have their own crontab files, and there is usually a system-wide contrab file located in the `/etc` or a subdirectory of `/etc`. 
This file can only be edited by system administrators \[1\].

.. code-block:: text 

    # ┌───────────── minute (0 - 59)
    # │ ┌───────────── hour (0 - 23)
    # │ │ ┌───────────── day of the month (1 - 31)
    # │ │ │ ┌───────────── month (1 - 12)
    # │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
    # │ │ │ │ │                                   7 is also Sunday on some systems)
    # │ │ │ │ │
    # │ │ │ │ │
    # * * * * * <command to execute>


Each line of a crontab starts with five fields. These fields are used to represent the time schedule for the command to be run.

`Online <https://crontab.guru/>`_ tools exist to easily calculate this seemingly cryptic language, much like regex, sometimes recall vs. recognition is important, we must recognise this is a niche skill and outsource knowledge when convenient.

Examples
~~~~~~~~

The example below clears the Apache error log at one minute past midnight (00:01) every day.

.. code-block:: bash

    $ 1 0 * * * printf "" > /var/log/apache/error_log

This example runs a shell script called export_dump.sh at 23:45 (11:45 pm) every Saturday.

.. code-block:: bash

    $ 45 23 * * 6 /home/oracle/scripts/export_dump.sh

Note: It is also possible to specify `*/n` t run for every _n_-th interval of time.

The output below would write "hello world" to standard output every 5th minute of every first, second, and third hour (i.e., 01:00, 01:05, 01:10, up until 03:55).

.. code-block:: bash

    $ */5 1,2,3 * * * echo hello world

Marcos
~~~~~~

Some cron implementations support non-standard macro definitions. 
For example, `@reboot` configures a job to run once the daemon is started. 
Cron is rarely restarted, so this macro corresponds to when the machine is rebooted. 
In some Debian distributions this behaviour is enforced, where restarting cron will not run the `@reboot` jobs.

Some other examples include

|Entry|Description|Equivalent to|
|---|---|---|
|`@yearly (or @annually)`|Run once a year at midnight of 1 January|`0 0 1 1 *`|
|`@monthly`|Run once a month at midnight of the first day of the month|`0 0 1 * *`|
|`@weekly`|Run once a week at midnight on Sunday morning|`0 0 * * 0`|
|`@daily (or @midnight)`|Run once a day at midnight|`0 0 * * *`|
|`@hourly`|Run once an hour at the beginning of the hour|`0 * * * *`|
|`@reboot`|Run at startup| `n\a`|

Permission
~~~~~~~~~~

There are two files that affect the permissions for executing cron jobs.
    * `/etc/cron.allow` - if file exists, it contains a list of users allowed to use cron jobs
    * `/etc/cron.deny` - if `cron.allow` does not exist, this becomes a blacklist, all other users are permitted

If neither of these files exists, either only the superuser can use cron jobs, or every user can.

`Tutorial <https://ostechnix.com/a-beginners-guide-to-cron-jobs/>`_

We can edit CRON jobs for our user using this command:

.. code-block:: bash

    $ crontab -e

Take the cronjob below, which appends the string "Hello World" to a log file in our `/tmp` directory, once a every minute.

.. code-block:: bash
    $     * *    * * *    echo "Hello World!" >> /tmp/test.log

We then wait a couple of minutes for these messages to build up. Then check the log file.

.. code-block:: bash

    $ cat /tmp/test.log

After 4 minutes have passed, the output should be something similar to this:

.. code-block:: text

    Hello World
    Hello World
    Hello World
    Hello World

This verifies that our cronjob is working for our user.

Finally, to stop the cronjob from running forever, we execute the command below. This clears all of the cronjobs for the current user.

.. code-block:: bash

    $ crontab -r

Data Flow
~~~~~~~~~ 

This is the flow of the data for research projects such as CTD and Mooring data. The information is recorded on a regular interval by sensors, for example, a Mooring. This is stored on a database on a local machine. That is then transferred to Niwa's central database, which aggregates all the field data into a single place. We then construct a cronjob, which ensures the NZODN Postgres database is updated each day, with the most recent data.

.. image:: https://user-images.githubusercontent.com/18411037/106079425-10c23700-617a-11eb-8cd9-c2f215e4569e.png) 

NZODN Example
~~~~~~~~~~~~~

This is the crontab from the NZODN server. This is on the crontab of the root user of Wellimos. 

.. code-block:: bash

    #Schedule a daily import for mooring data. Get don't expect new data
    #most days, but when there is data we want it imported as soon as possible
    2 10 * * * robot /home/robot/aodn/import/mooring/bin/loadAllMooringData.sh -c /home/robot/aodn/import/mooring/config/prod.ini >> /tmp/nzodn-mooring-import.log

It runs everyday, at 2 minutes past 10. 
The 2 minute offset is to ensure that not all cron jobs are run at the same time on a server. 
That would put unnecessary strain on the existing server, and require additional compute for a very short amount of time, and no need for that power the rest of the time. 

The crontab executes a bash script with the `robot` user. 
This is a service account used by NZODN for automated processes. 
It has permission to read and write to directories, such as the `/data/niwa/publish` that exposes files to `NZODN <https://github.com/woodRock/psychic-invention/wiki/NZODN>`_ FTP server.

This script takes in a configuration file `prod.ini` which specifies that parameters to the bash script `loadAllMooringData.sh`. 
Using a configuration file makes avoids hard-coding, and makes this script re-usable in the future for other purposes. 
A non-technical user can easily adjust the configuration, without needing to have an understanding of the implementation details of the program. 
The appendix gives an example a configuration file.

The output of the bash script is appended to a log file `/tmp/nzodn-mooring-import.log`. Such that any runtime errors, executions, or unintended side affects can be reviewed in future. We use the `tmp` directory, such that we don't keep erroneous log files that are no longer relevant for extended amounts of time. These would be a waste of disk space. We use the append command `>>`, since we do not want to overwrite the existing file entirely each time an ingestion is performed. In the event of a catastrophic error, we would have four days to review the log files.


Configuration File example
~~~~~~~~~~~~~~~~~~~~~~~~~~

This is an example of the kind of configuration file `prod.ini` which is passed as an argument to our bash script that loads the mooring data. Sensitive details are obfuscated for obvious security reasons.

.. code-block:: text
    [source]
    type=ftp
    server=ftp.niwa.co.nz
    directory=incoming/AODN/Moorings_data
    fileglob=*\.DAT3
    user=FAKE_NAME_HERE
    password=NOT_SO_FAST_BUCKO

    [aodn]
    directory=/data/niwa/publish/mooring
    server=localhost
    database=database
    user=FAKE_NAME_HERE
    password=NOT_SO_FAST_BUCKO

Log Files
~~~~~~~~~

By running `cat /tmp/nzodn-mooring-import.log` we can see the contents of the ingestion log files:

.. code-block:: text

    [2021/02/08 10:02:01] INFO    : Fetching data
    [2021/02/09 10:02:01] INFO    : Fetching data
    [2021/02/10 10:02:02] INFO    : Fetching data
    [2021/02/11 10:02:01] INFO    : Fetching data

Here we see that the ingestion process has been completed for the past consecutive four days, with no further previous records. 
This is because the `/tmp` directory is cleared every four days (I assume).