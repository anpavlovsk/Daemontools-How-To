# Daemontools-How-To

Daemontools Intro

With daemontools, if you want to write your own daemon, you can write it as a normal program, and then just incorporate it into daemontools to daemonize it, with logs and restarts and control and everything. You can daemonize pretty much any executable, even one never made to be a daemon. You can daemonize a shellscript. Once again, this isn't just running it in the background: It's controlling it with the svc command, examining it with svstat and svok, and having its every line to stdin and stdout sent to a log file whose format you control.

### daemontools Service Hello

Will'll make a shellscript that, once every second, writes the yyyymmdd:hhmmss time both to a file in the /tmp directory, and to stdout. This shellscript runs in the foreground and has no daemon-like properties. Then will'll call this shellscript, from a daemontools implementation, to make it into a daemon, controllable by the svc command, and observable with the svstat and svok commands. Will'll watch the file in the /tmp directory as you start and stop the daemon with the svc command.

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
