---
layout: post
title:  "Crontab: Setting Scheduled Tasks in Linux"
date:   2021-09-10 10:00:40
blurb: "How to use crontab command to schedule your tasks."
og_image: /assets/img/content/post-example/Banner.jpg
---

In Linux, periodic tasks are generally handled by **cron**, a daemon. Cron's configuration file is called **"crontab"**, which is short for "cron table". 

With the crontab command, we can execute specified system commands or shell scripts at fixed intervals. The time interval can be in any combination of minutes, hours, days, months, weeks and more. 

This command is ideal for tasks such as periodic log analysis or data backup.
<br />

#### Table of Contents

* toc
{:toc}

#### Overview

##### 1. Crontab Command

The **crontab** command is used to view or edit the table of commands to be run by **cron**.

```
crontab [-u user] [-l | -r | -e] [-i] [-s] 
```

`-u user` : Specifies the user whose crontab is to be viewed or modified.

`-e` : Edits a copy of the user's crontab file or creates an empty file to edit if the crontab file does not exist for a valid UserName. 

`-l` : Lists the user's crontab file.

`-r` : Removes the user's crontab file from the crontab directory.

`-i` : Reminds users before removing a crontab file.

`-s` : Adds SELINUX security to crontab file. 

<br />

##### 2. Crontab Files

`/etc/crontab` : main system crontab file.

`/etc/cron.d/`: a directory for storing system crontabs.

`/var/spool/cron/` : a directory for storing crontabs defined by users.  

<br />

##### 3. Cron command entries

Each **cron** command entry in the crontab file has **five time and date fields** (followed by a **username**, only if it's the system crontab file), followed by **a command**. As below :
```
 *  *  *  *  * user-name  command to be executed
```

###### 3.1. Time and date fields

```
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |    
# *  *  *  *  * user-name  command to be executed
```

- Asterisk (`*`) : Specifies all possible values for a field, e.g. every hour or every day. 
- Comma (`,`) operator specifies a list of values, for example, "1,2,5,7,8,9"
- Middle bar (`-`) : Specifies a range of values, e.g. "2-6" means "2,3,4,5,6"
- Forward slash (`/`) : Specifies the frequency of time intervals, for example, "0-23/2" means once every two hours. Also the forward slash can be used together with an asterisk, for example */10, if used in the minute field, means once every ten minutes.

**Example:**

The command below is expected to run the **calendar** command every 5 minutes during 09:00 and 16:00 from Monday through Friday, between January and May and between September and December.

```
*/5 9-16 * 1-5,9-12 1-5 /usr/bin/calendar
```

###### 3.2. Environment settings

If **a command is executed manually but fail to work in the crontab**, it is usually a problem with the environment variables. 
>
When we execute a task manually, the process is done in the current environment and the system automatically executes task scheduling without loading any environment variables. In order that the system can execute the task scheduling correctly, it is necessary to specify all the environment variables required for the task to run in the crontab file.

Four ways of specifying environment variables are listed below.

*1) Use the absolute path if a file is involed*

```
0 * * * * /root/anaconda3/envs/my_env/bin/python3 /home/test_python/test_2.py
```

*2) Specify the interpreter path in the file*

To replace the first line of the following:

```
#!/usr/bin/python
```

with:

```
#!/root/anaconda3/envs/my_env/bin/python3
```

*3) Add the environment variable to* `/etc/crontab`

```
0 * * * * . /etc/profile;/bin/sh /var/www/java/audit_no_count/bin/restart_audit.sh
```

*4) Use* **`source`**

```
!/bin/sh
source /etc/profile
export RUN_CONF=/home/d139/conf/platform/cbp/cbp_jboss.conf
/usr/local/jboss-4.0.5/bin/run.sh -c mev &
```

###### 3.3. Redirect loggings

Cron will email to the user (`/var/spool/mail/`) all output of the commands it runs, so the log information will be very large day by day and may affect the normal operation of the system. To silence this, we can redirect the output to a log file or to `/dev/null`.

**Example:**

```
0 */3 * * * /usr/local/apache2/apachectl restart >/dev/null 2>&1
```

<br />

#### How to

First enter the terminal as follow:

```
# enter the crontab file
crontab -e

# add an entry
0 1 * * * /root/script/scp.sh >>/tmp/scp.log 2 >& 1

# or remove an entry
#0 1 * * * /root/script/scp.sh >>/tmp/scp.log 2 >& 1

# check the setting
crontab -l

# view the loggings
tail -f /var/log/cron
```
If the EDITOR environment variable exists, the command invokes the editor it specifies. Otherwise, the crontab command uses the **vi editor**.

Below are some commands in vi editor:

`i` – Insert at cursor (goes into insert mode)

`:w` – Save the file but keep it open

`:q` – Quit without saving

`:wq` – Save the file and quit

`ESC` – Terminate insert mode (goes into view mode)

`dd` – Delete line

#### Reference

<https://man7.org/linux/man-pages/man5/crontab.5.html1>

<https://www.ibm.com/docs/en/aix/7.1?topic=c-crontab-command>