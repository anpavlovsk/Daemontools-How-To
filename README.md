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
