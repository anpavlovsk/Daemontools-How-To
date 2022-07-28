# Daemontools-How-To

Daemontools Intro

With daemontools, if you want to write your own daemon, you can write it as a normal program, and then just incorporate it into daemontools to daemonize it, with logs and restarts and control and everything. You can daemonize pretty much any executable, even one never made to be a daemon. You can daemonize a shellscript. Once again, this isn't just running it in the background: It's controlling it with the svc command, examining it with svstat and svok, and having its every line to stdin and stdout sent to a log file whose format you control.

### daemontools Service Hello

Will'll make a shellscript that, once every second, writes the yyyymmdd:hhmmss time both to a file in the /tmp directory, and to stdout. This shellscript runs in the foreground and has no daemon-like properties. Then will'll call this shellscript, from a daemontools implementation, to make it into a daemon, controllable by the svc command, and observable with the svstat and svok commands. Will'll watch the file in the /tmp directory as you start and stop the daemon with the svc command.

In this Hello, assume the following:

Your username is root
Your hostname is ubuntu2004
The directory in which you assemble your services is /scratch/service, and that directory is owned and grouped root, and its permissions are 1755.
Your shellscript is at /root/daemhello/print_timestamps.sh, and is owned by user root and is executable.

### Make and test the shellscript
Make a directory directory called daemhello. Create the following shellscript, called print_timestamps.sh, in that newly created directory:
````
#!/bin/sh
log=/tmp/junklog.log
sleeptime=1
while /bin/true; do
  mydate=`date +%Y%m%d_%H:%M:%S`
  echo $mydate >> $log
  echo $mydate
  sleep $sleeptime
done
````
Set the file readable and executable by all, run it, and you'll see it count time second by second. Before trying to daemonize this shellscript, make sure it works right as a foreground program.

When your configuration is correct, you’ll see an output similar below.
````
root@ubuntu2004:~/daemhello# sh print_timestamps.sh
20220728_12:43:10
20220728_12:43:11
20220728_12:43:12
20220728_12:43:14
20220728_12:43:15
20220728_12:43:16
20220728_12:43:17
````
On a different terminal, perform a tail -fn0 /tmp/junklog.log, and you'll see that it's writing to that file also.
````
20220728_12:44:19
20220728_12:44:21
20220728_12:44:22
20220728_12:44:23
20220728_12:44:24
20220728_12:44:25

````
### Build the daemontools Service

Now lets build a daemontools service for the shellscript we just made. Do all this section's work logged in as root. Later we'll discuss daemontools running a daemon as a normal user, but the setup work should be done as user root, regardless of the user daemontools uses to run the program.

As you remember from earlier, all service directory trees are built under the /scratch/service directory.

* cd /scratch/service
* mkdir hello
* cd hello
* Create file run, which contains:
````
#!/bin/sh
echo Starting hello
exec /root/daemhello/print_timestamps.sh
````
* chmod u+x run
* ln -s /scratch/service/hello /etc/service/hello
Note that the preceding command "installs" service hello and actually starts it running.
* In another terminal, tail -fn0 /tmp/junklog.log
Note that the preceding command tests whether /root/daemhello/print_timestamp.sh is running and writing to tail -fn0 /tmp/junklog.log. It's how you test whether the daemon is working.
* Wait more than five seconds (and less than 12) for the tail command to produce output.

### Manipulate and Monitor Your New Service 

You manipulate your service with the svc command. You monitor your service with commands svstat and svok. Let's dispense with svok right now: It outputs nothing, and simply returns 0 if the service is working properly, or non-zero otherwise. 

The svc command is used to manipulate your daemon, by sending signals to it. This command is performed as user root, from within the /service directory. The following is an example:
````
svc -d hello
````
The preceding command downs (stops) the hello daemon. The following is a table of svc arguments, their meanings, and their signals:
```
Arg	Action	Signal
-u	Start (up)	-
-d	Stop (down)	TERM, then CONT
-t	Restart if running	TERM
```
The preceding are the commands I use all the time. Many, many more arguments to the command are explained at http://cr.yp.to/daemontools/svc.html.

With one terminal running a tail -f /tmp/junklog.log, use the three svc command previously listed and note their effect on the output. You can turn your daemon on and off at will.

### Elementary Troubleshooting

Here are some typical ps commands to see what's running and what's not:

* ps ax | grep svscan : One instance each of svscanboot and svscan is perfect. Zero instances means daemontools isn't running. More than one of either causes hard to solve problems, so it must be fixed right away. If you have more than one, try rebooting and see if you still have more than one. If you do, svscanboot is probably being run from more than one place. The proper grep commands should shed some light. Be aware that the standard djb provided daemontools install writes a line to start svscanboot in /etc/inittab, so if you manually enabled it anywhere else, you must disable one or the other, in order to boot with exactly one instance of svscanboot and one instance of svscan

* ps ax | grep supervise : This tells you all the services under daemontools supervision. If you don't see a supervise process for the service you're investigating, that tends to indicate a very basic failure in your service. investigate further
ps ax | grep myprogram, where myprogram is the program being exec'ed by your run command. For instance, in previous examples, it was print_timestamps.sh. If the ps command reveals no instance of myprogram, it's also probable that svstat /service/myserviced keeps showing the service up for zero or one seconds. It keeps stopping and restarting. Here are some avenues to investigate when this happens:
* The run failed to exec myprogram. This possibility can be confirmed by temporarily adding a first command of myprogram to append a timestamp to a chmod 777 file in the /tmp directory.
* myprogram runs but either aborts or fails to loop.
* Program myprogram was designed not to loop, depending on daemontools to restart it. This is a hideously defective design paradigm, and should be used only as a very temporary diagnostic, and then backed out immediately.
* ps ax | grep readproctitle : Although this isn't done to determine existence of a process, reading its output can tell a lot about what happened when the run script tried to exec myprogram.
* If you find yourself with no running svscanboot, you can run it as root from a terminal like this: csh -cf '/command/svscanboot &'

It should also be noted that, if svscanboot isn't being started at boot, you can put the preceding command in /etc/rc.local on non-systemd machines.

The next thing to do is use svstat and svc commands to investigate further. Perform the following command a few times, as user root, from the /service directory:
````
svstat hello
````
You’ll see an output similar below.
````
root@ubuntu2004:/etc/service# svc -d hello
root@ubuntu2004:/etc/service# svstat hello
hello: down 372 seconds, normally up
root@ubuntu2004:/etc/service# svstat hello
hello: down 405 seconds, normally up
root@ubuntu2004:/etc/service#
````
The preceding command tells you up to five pieces of information:

1. The service dir name (I guess in case you forgot what you typed)
2. The current state (either up or down)
3. The PID (skipped if the service is down)
4. The number of seconds it's been in its current state
5. Its normal state (skipped unless its current state is different from its normal state)

Check the results of svstat when you try to toggle its state with svc -u or svc -d.
````
root@ubuntu2004:/etc/service# svc -u hello
root@ubuntu2004:/etc/service# svstat hello
hello: up (pid 10804) 3 seconds
````
### Using Daemontools Logging

Our hello service writes its own log to /tmp/junklog.log, but sometimes you're daemonizing someone else's code and don't want to modify it to write a log. Daemontools can write a timestamped line to a log file every time a daemonized program writes to stdout or stderr. Lets adds logging to the hello service. Do the following, logged in as user root:

* cd /scratch/service/hello
* mkdir log
* cd log
* mkdir main
Create the following file, named run.new, in the log directory:
````
#!/bin/sh
exec 2>&1
exec multilog t ./main
````
* chmod u+x run.new
* mv run.new run
* cd /service
* svc -t hello
In another terminal that's visible, tail -fn0 /scratch/service/hello/main/current
````
root@ubuntu2004:~# tail -fn0 /scratch/service/hello/log/main/current
@4000000062e2a5c5283f564c 20220728_15:05:31
@4000000062e2a5c629021d94 20220728_15:05:32
@4000000062e2a5c72eaa6f1c 20220728_15:05:33
@4000000062e2a5c82f6a7b2c 20220728_15:05:34
@4000000062e2a5c9346cc4cc 20220728_15:05:35

````
