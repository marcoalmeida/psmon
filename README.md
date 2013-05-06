Copyright (C) 2009-2013 Marco Almeida marcoafalmeida@gmail.com

This file is part of psmon.

psmon is free software; you can redistribute it and/or modify it
under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or
(at your option) any later version.

psmon is distributed in the hope that it will be useful, but WITHOUT
ANY WARRANTY; without even the implied warranty of MERCHANTABILITY
or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public
License for more details.

You should have received a copy of the GNU General Public License
along with psmon; if not, write to the Free Software Foundation,
Inc., 51 Franklin St, Fifth Floor, Boston, MA 02110-1301 USA


===========
DESCRIPTION
===========

'psmon' is a process monitoring utility with customizable email
notification facilities. It may run as a stand-alone program or a
background daemon.

It is implemented as a Python script and rellies only on some usual
*NIX command line tools such as 'ps' and 'grep'.

It should work fine with python 2.4 - 2.7 on any OS with a POSIX
compliant version of 'ps' and GNU 'grep' (I use it everyday on several
different environments - GNU/Linux, Solaris, OpenSolaris, and FreeBSD
- with Python 2.4, 2.5, and 2.6).

The user defines a set of running processes (pid, name, command line
args, etc.) to be monitorized (possibly with some filters) and some
action to take when a process is matched. Limitations on maximum
memory usage, time, etc. may be imposed.

The report may be sent to standard output, a file, or by email.

There is no documentation except for the usage description and
examples bellow. I will try to include a man page soon.

I wrote it with the purpose of solving some very specific problems of
my own, so it's not actually "production ready" and there are
certainly some bugs laying around. If you have any question,
sugestions, or comments, please email me: malmeida@netc.pt


=======
USAGE
=======
psmon [options]

Options:
  -h, --help            show the help message and exit
  -p PID, --pid=PID     monitor process PID
  -m CMDLINE, --match=CMDLINE
                        monitor all processes where CMDLINE matches the
                        command line
  -R, --reverse         report only when there is no output (no matching
                        processes or all matched processes were filtered)
  -f FILTER, --filter=FILTER
                        monitor processes which pass the filter (predicate
                        list)
  -a ACTION, --action=ACTION
                        action to execute (shell command) on each of the
                        monitored processes
  -r FILE, --report=FILE
                        write report to FILE
  --append              append reports instead of overwriting FILE
  -M ADDRESS, --mail-to=ADDRESS
                        send report to ADDRESS
  -o OUTPUT, --output=OUTPUT
                        information to include on the report (for each
                        monitored process)
  -s SLEEP, --sleep=SLEEP
                        pause between monitorizations (in seconds)
  -d COUNT, --displays=COUNT
                        monitor COUNT times and quit
  -Q, --auto-quit       stops monitoring as soon as the matching produces no
                        output
  -t TITLE, --title=TITLE
                        title of the report
  -D, --daemon          run as a daemon
  -l, --list            list currently running psmon instances
  -L LEVEL, --log=LEVEL
                        log level: debug,info,warning,critical,error,none; log
                        file is /home/marco/.psmon.log
  --mail-server=SERVER  user SERVER as the SMTP server
  --mail-port=PORT      connect to non-standard port PORT
  --mail-from=FROM      use FROM as source address
  --mail-user=USER      authenticate at SERVER with the login USER
  --mail-passwd=PASSWD  authenticate at SERVER with the password PASSWD
  --mail-ssl            use SSL when connecting to the SMTP server


Filters (see EXAMPLES) are strings of the form "ELEMENT OP VALUE".
ELEMENT is one of the POSIX names available with the -o option of the
`ps` tool: count, user, ruser, group, rgroup, pid, ppid, pgid, pcpu,
vsz, nice, etime, time, tty, comm, args.
OP is one of <, >, ==, !=, and VALUE is an arbitrary string or integer
to form the condition (predicate).

For example, "nice > 10" will filter out all processes running with a
"nice" value smaller or equal to 10. In the same way, "time > 1:00:00"
will only match processes that have used more that 1 hour of CPU
time. Time limits may be given in seconds or the usual `ps` format
[dd-]hh:mm:ss.

Memory related values may be contain the usual sufixes --- b, K, M, G,
and T --- designating bytes, kilobytes, megabytes, gigabytes, and
terabytes, respectivelly. These suffixes are case-insensitive.

The shell commands in the action string (see EXAMPLES) may include the
values of the same POSIX names/variables used in the filters. The
reference to the variable name should start with "%" and be enclosed
in "{}".

Suppose you want to keep the pid of all matched processes in file
/home/foo/X. You could use an action string such as "echo %{pid} >>
/home/foo/X". The EXAMPLES section has a few examples in which some
processes are killed (under certain conditions) using this approach.


========
EXAMPLES
========

monitor process 456, showing command line arguments, cpu time,
> psmon -p 456 -o args -o time 

same as above, but producing a report only every 60 seconds
> psmon -p 456 -o args -o time -s 60

show all processes matching 'foo' in any of the arguments printing
pid, cpu time, virtual memory
> psmon -m foo -o pid -o time -o vsz -o count 

same as above, but run in the background and email reports every 10
minutes to bar@example.com
> psmon -m foo -o pid -o time -o vsz -o count -s 600 -M bar@example.com -D

kill all 'foo' processes running for more that 48 hours; check every
minute, running in the background, and email a report when some
process gets killed
> psmon -mfoo -opid -f"time > 48:00:00" -a"kill -9 %{pid}" -s60 -t"some foo process killed" -M bar@example.com -D

send a report (by email) whenever there are less than 4 instances of
the 'foo' process running; runs in the background awaking every 10
minutes; the report will include the process count, pid and full
command line of each process
> psmon -mfoo -f"count < 4" -t"less than 4 foo processes running" -o count -o pid -o args -s600 -M bar@example.com -D

show currently running psmon instances with the corresponding pids and
command line args
> psmon -l -opid -oargs -d1

send a report (by email) when some process exceeds 0.8GB of RAM; runs
in the background awaking every 10 minutes; the report will include
the process count, pid and full command line of each process
> psmon -f"vsz > 0.8G" -o count -o pid -o vsz -o args -s300 -M bar@example.com -D
