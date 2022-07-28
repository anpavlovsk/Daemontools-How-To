# Daemontools-How-To

Daemontools Intro

With daemontools, if you want to write your own daemon, you can write it as a normal program, and then just incorporate it into daemontools to daemonize it, with logs and restarts and control and everything. You can daemonize pretty much any executable, even one never made to be a daemon. You can daemonize a shellscript. Once again, this isn't just running it in the background: It's controlling it with the svc command, examining it with svstat and svok, and having its every line to stdin and stdout sent to a log file whose format you control.

### daemontools Service Hello

Will'll make a shellscript that, once every second, writes the yyyymmdd:hhmmss time both to a file in the /tmp directory, and to stdout. This shellscript runs in the foreground and has no daemon-like properties. Then will'll call this shellscript, from a daemontools implementation, to make it into a daemon, controllable by the svc command, and observable with the svstat and svok commands. Will'll watch the file in the /tmp directory as you start and stop the daemon with the svc command.
