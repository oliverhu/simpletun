simpletun, a (too) simple tunnelling program.

-------

To compile the program, just do

$ gcc simpletun.c -o simpletun

If you have GNU make, you can also exploit implicit targets and do

$ make simpletun

-------

Usage:
simpletun -i <ifacename> [-s|-c <serverIP>] [-p <port>] [-u|-a] [-d]
simpletun -h

-i <ifacename>: Name of interface to use (mandatory)
-s|-c <serverIP>: run in server mode (-s), or specify server address (-c <serverIP>) (mandatory)
-p <port>: port to listen on (if run in server mode) or to connect to (in client mode), default 55555
-u|-a: use TUN (-u, default) or TAP (-a)
-d: outputs debug information while running
-h: prints this help text

-------

Refer to http://backreference.org/2010/03/26/tuntap-interface-tutorial/ for
more information on tun/tap interfaces in Linux in general, and on this
program in particular.
The program must be run at one end as a server, and as client at the other
end. The tun/tap interface must already exist, be up and configured with an IP
address, and owned by the user who runs simpletun. That user must also have
read/write permission on /dev/net/tun. (Alternatively, you can run the
program as root, and configure the transient interfaces manually before
starting to exchange packets. This is not recommended)

Use is straightforward. On one end just run

[server]$ openvpn --mktun --dev tun3 --user pi // create a tunnel device
[server]$ ip link set tun3 up
[server]$ ip addr add 10.0.0.2/24 dev tun3 // assign 10.0.0.2/24 to interface tun3, note that don't overlap with your physical address..
[server]$ ./simpletun -i tun13 -s

at the other end run

[client]$ openvpn --mktun --dev tun11 --user pi
[client]$ ip link set tun11 up
[client]$ ip addr add 10.0.0.1/24 dev tun11
[client]$ ./simpletun -i tun0 -c 192.168.0.10

where 192.168.0.10 is the remote server's IP address, and tun13 and tun0 must be
replaced with the names of the actual tun interfaces used on the computers.

In another terminal in client, try:

[client]$ ping 10.0.0.1
[client]$ ssh pi@10.0.0.1

By default it assumes a tun device is being used (use -u to be explicit), and
-a can be used to tell the program that the interface is tap.
By default it uses TCP port 55555, but you can change that by using -p (the
value you use must match on the client and the server, of course). Use -d to
add some debug information. Press ctrl-c on either side to exit (the other end
will exit too).

The program is very limited, so expect to be disappointed.
