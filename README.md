# SRE-Questions

Unix Process

What is the difference between a process and a thread?

A thread is a lightweight process. Each process has a separate stack, text, data and heap. Threads have their own stack, but share text, data and heap with the process. Text is the actual program itself, data is the input to the program and heap is the memory which stores files, locks, sockets. Reference: https://computing.llnl.gov/tutorials/pthreads/#Thread

What is a zombie process?

A zombie process is a one which has completed execution, however it’s entry is still in the process table to allow the parent to read the child’s exit status. The reason the process is a zombie is because it is “dead” but not yet “reaped” by it’s parent. Parent processes normally issue the wait system call to read the child’s exit status whereupon the zombie is removed. The kill command does not work on zombie process. When a child dies the parent receives a SIGCHLD signal.
Reference: http://en.wikipedia.org/wiki/Zombie_process

Describe ways of process inter-communication

POSIX mmap, message queues, semaphores, and Shared memory
Anonymous pipes and named pipes
Unix domain sockets
RPC
For a complete list see http://en.wikipedia.org/wiki/Inter-process_communication
How to daemonize a process

The fork() call is used to create a separate process.
The setsid() call is used to detach the process from the parent (normally a shell).
The file mask should be reset.
The current directory should be changed to something benign.
The standard files (stdin,stdout and stderr) need to be reopened.
Describe how processes executes in a Unix shell

Let’s take the example of /bin/ls. When run ‘ls’ the shell searches in it’s path for an executable named ‘ls, when it finds it, the shell will forks off a copy of itself using the fork system call. If the fork succeeds, then in the child process the shell will run ‘exec /bin/ls’ which will replace the copy of the child shell wit itself. Any parameters that that are passed to ‘ls’ are done so by exec.

When you send a HUP signal to a process, you notice that it has no impact, what could have happened?

During critical section execution, some processes can setup signal blocking. The system call to mask signals is ‘sigprocmask’. When the kernel raises a blocked signal, it is not delivered. Such signals are called pending. When a pending signal is unblocked, the kernel passes it off to the process to handle. It is possible that the process was masking SIGHUP.

How do you end up with zombie processes?

Zombie processes are created when the parent does not reap the child. This can happen due to parent not executing the wait() system call after forking.

What are Unix Signals?

Signals are an inter process communication method. The default signal in Linux is SIG-TERM. SIG-KILL cannot be ignored and causes an application to be forcefully killed. Use the ‘kill’ command to send signals to a process. Another popular signal is the ‘HUP’ signal which is used to ‘reset’ or ‘hang up’ applications. A list of signals can be found here http://man7.org/linux/man-pages/man7/signal.7.html. A snipet from the man page is below.

Signal Value Action Comment
──────────────────────────────────────────────────────────────────────
SIGHUP 1 Term Hangup detected on controlling terminal
or death of controlling process
SIGINT 2 Term Interrupt from keyboard
SIGQUIT 3 Core Quit from keyboard
SIGILL 4 Core Illegal Instruction
SIGABRT 6 Core Abort signal from abort(3)
SIGFPE 8 Core Floating point exception
SIGKILL 9 Term Kill signal
SIGSEGV 11 Core Invalid memory reference
SIGPIPE 13 Term Broken pipe: write to pipe with no
readers
SIGALRM 14 Term Timer signal from alarm(2)
SIGTERM 15 Term Termination signal
SIGUSR1 30,10,16 Term User-defined signal 1
SIGUSR2 31,12,17 Term User-defined signal 2
SIGCHLD 20,17,18 Ign Child stopped or terminated
SIGCONT 19,18,25 Cont Continue if stopped
SIGSTOP 17,19,23 Stop Stop process
SIGTSTP 18,20,24 Stop Stop typed at terminal
SIGTTIN 21,21,26 Stop Terminal input for background process
SIGTTOU 22,22,27 Stop Terminal output for background process

The signals SIGKILL and SIGSTOP cannot be caught, blocked, or
ignored.

Networking

Define TCP slow start

Tcp slow start is a congestion control algorithm that starts by increasing the TCP congestion window each time an ACK is received, until an ACK is not received.

Reference:http://en.wikipedia.org/wiki/Slow-start

Name a few TCP connections states

1) LISTEN – Server is listening on a port, such as HTTP
2) SYNC-SENT – Sent a SYN request, waiting for a response
3) SYN-RECEIVED – (Server) Waiting for an ACK, occurs after sending an ACK from the server
4) ESTABLISHED – 3 way TCP handshake has completed

Define the various protocol states of DHCP

DHCPDISCOVER client->server : broadcast to locate server
DHCPOFFER server->client : offer to client with offer of configuration parameters
DHCPREQUEST client->server : requesting a dhcp config from server
DHCPACK server->client : actual configuration paramters
DHCPNAK server->client : indicating client’s notion of network address is incorrect
DHCPDECLINE client->server : address is already in use
DHCPRELEASE client->server : giving up of ip address
DHCPINFORM client->server : asking for local config parameters
Reference: https://www.ietf.org/rfc/rfc2131.txt
How do you figure out the network and broadcast address of a network given a netmask?

Describe a TCP packet fields

Difference between TCP/UDP

Reliable/Unreliable
Ordered/Unordered
Heavyweight/Lightweight
Streaming
Header size
Examples:
What are the different kind of NAT available?

There is SNAT and DNAT. SNAT stands for source network address translation. DNAT stands for destination network address translation. SNAT occurs when the source IP address if RFC 1918 and is changed to be non-RFC 1918. For instance if you are at home using your cable model and want to connect to and external site such as http://www.cnn.com, then your router will change the source address of the TCP packet to be it’s external public IP. This is called SNAT. DNAT is when the destination IP address is changed. For instance when your packet reaches the http://www.cnn.com router, and the web server behind the router is using RFC 1918 space, then the router might change the destination to be the RFC 1918 IP address of the web server. This is called DNAT.

DNS

Explain the SOA record in DNS

SOA stands for Start of Authority and it contains the following entries:

@ IN SOA nameserver.mycomaind.com. postmaster.mydomain.com. (
1 ; serial number
3600 ; refresh [1h]
600 ; retry [10m]
86400 ; expire [1d]
3600 ) ; min TTL [1h]fire

Serial number should be refreshed each time a change is made to the zone file. This is how slave DNS servers know to pull a change from the master.
Refresh is the amount of time a slave DNS server should wait before pulling from the master.
Retry is how long a slave should wait before retrying to get a zone file if the initial retry fails.
Expire is how long a secondary server will keep trying to get a zone from the master. If this time expires before a successful zone transfer, the secondary will stop answering queries.
TTL is how long to keep the data in a zone file.

Filesystems

List open file handles

lsof -p process-id
Or ls /proc/process-id/fd
What is an inode?

An inode is a data structure in Unix that contains metadata about a file. Some of the items contained in an inode are:
1) mode
2) owner (UID, GID)
3) size
4) atime, ctime, mtime
5) acl’s
6) blocks list of where the data is

The filename is present in the parent directory’s inode structure.

What is the difference between a soft link and a hard link?

1) Hardlink shares the same inode number as the source link. Softlink has a different inode number. Example:

1
2
3
4
5
6
7
$ touch a
$ ln a b
$ ls -i a b
24 a  24 b
$ ln -s a c
$ ls -i a c
24 a  25 c
2) In the data portion of the softlink is the name of the source file
3) Hardlinks are only valid in the same filesystem, softlinks can be across filesystems

When would you use a hardlink over a softlink?

A hardlink is useful when the source file is getting moved around, because renaming the source does not remove the hardlink connection. On the other hand, if you rename the source of a softlink, the softlink is broken. This is because hardlink’s share the same inode, and softlink uses the source filename in it’s data portion.

Describe LVM and how it can be helpful

LVM stands for logical volume manager and it is a way of grouping disks into logical units. The basic unit of LVM is a PE or a physical extent. One disk may be divided into one or more PE’s. One or more PE’s are contained in a VG or a volume group. Or or more LV or logical volumes are created out of a VG. For instance, if we have a server with 2x1TB disk drives, we can create 4xPE’s on it, each one being 500GB. On disk 1 let’s say we name the PE’s PE1 and PE3 and on disk 2 we name the PE’s PE2 and PE4. We can then create VG0 out of PE1 and PE2, and VG1 out of PE3 and PE4. After that we can create a LV called /root and another one called swap on VG0.

An advantage of using LVM is that we can create ‘software’ RAID, i.e., we can join multiple disks into one bigger disk. We cannot select the RAID level with LVM, for instance we cannot say that a VG is of RAID 5 type, however we are able to pick and chose the different PE’s we want in a VG. Also LVM allows for dynamically growing a disk.

What is ‘md’ and how do you use it?

MD is Linux software RAID. RAID can be done either in hardware wherein there is a RAID controller that does RAID and presents a logical volume to the OS, or RAID can be done in software wherein the kernel has a RAID driver which takes one or more disks can does RAID across them. ‘MD’ refers to the software RAID component of Linux.

What are some reasons to consider one filesystem type over another, such as XFS, ext?

What is RAID, and define a few RAID levels

Wikipedia has a very well written on RAID here https://en.wikipedia.org/wiki/RAID.

If a filesystem is full, and you see a large file that is taking up a lot of space, how do make space on the filesystem?

1) If no process has the filehandle open, you can delete the file
2) If a process has the filehandle open, it is better if you do not delete the file, instead you can ‘cp /dev/null’ on the file, which will reduce it’s size to 0.
3) A filesystem has a reserve, you can reduce the size of this reserve to create more space using tunefs.

What is the difference between character device and block device?

Block devices are generally buffered and are read/written to in fixed sizes, for instance hard drives, cd-roms. Characters devices read/writes are one character at a time, such as from a keyboard or a tty, and are not buffered.

Algorithms

Binary tree complexity

O(log n)

Hash table complexity
O(1)

HTTP

Common Http response codes

200 OK The request has succeeded
500 Internal Server Error (Server Error)
403 Forbidden
301 Permanent Redirect
302 Temporary Redirect
http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html
What is a http cookie

Http cookie is a small piece of data that a server sends to a browser, which a browser usually stores in it’s cookie cache. Cookie can be used to maintain session information since HTTP is stateless, and also for user preferences at a given site. Cookies can also be used to store encrypted password. Browsers send cookies back to the server when they make a connection’
http://en.wikipedia.org/wiki/HTTP_cookie
Http methods

Http methods are ways of communicating between server and client. Common examples are http get and http put which is used by http forms for data exchange. Other methods include, post, head, and connect.
http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html
Http headers

Http header fields are common components of HTTP requests and responses. Headers are colon separated name-value pairs in clear text. Some common headers are: Cache-control which specifies where to cache or not the contents of a page, Accept, which can be text/plain, Content-length which specifies the size of the content, Host, which is the domain name of the server.
http://en.wikipedia.org/wiki/List_of_HTTP_header_fields
Databases

What is the difference between MyISAM and InnoDB?

MyISAM:
1. Supports table level locking
2. Is Faster than InnoDB
3. Does not support foreign keys
4. Stores table, data and index in different files
5. Does not support transactions (no commit with rollback)
6. Useful for more selects with fewer updates
InnoDB:
1. Support row level locking
2. Is slower than MyISAM
3. Supports foreign keys
4. Stores table and index in table space
5. Supports transactions

What are some things to check on a slow database?

MySQL is fairly popular, so let’s look at some basic MySQL debugging. First off, check the OS to make sure the system is running fine, specially check CPU, memory, SWAP space and disk I/O. Assuming those are all ok, then log into MySQL and check the running queries, you can do so by running the command ‘show full processlist’. This will give you a list of queries running on the server. If you see a query that has been running for an excessively long time, you should investigate that query. See https://dev.mysql.com/doc/refman/5.1/en/show-processlist.html for additional details.

1
2
3
4
5
6
7
mysql&amp;gt; show full processlist;
+-----+------+-----------+-----------+---------+------+-------+-----------------------+
| Id  | User | Host      | db        | Command | Time | State | Info                  |
+-----+------+-----------+-----------+---------+------+-------+-----------------------+
| 865 | root | localhost | wordpress | Query   |   -1 | NULL  | show full processlist |
+-----+------+-----------+-----------+---------+------+-------+-----------------------+
1 row in set (0.00 sec)
To investigate queries use the command ‘explain. When investigating queries, if you notice the lack of a primary key you should investigate if having a primary key for that particular table makes sense. Having a key in general improves performance of a table. See https://dev.mysql.com/doc/refman/5.0/en/explain.html for additional details.

1
2
3
4
5
6
7
mysql&amp;gt; explain select * from wp_posts;
+----+-------------+----------+------+---------------+------+---------+------+------+-------+
| id | select_type | table    | type | possible_keys | key  | key_len | ref  | rows | Extra |
+----+-------------+----------+------+---------------+------+---------+------+------+-------+
|  1 | SIMPLE      | wp_posts | ALL  | NULL          | NULL | NULL    | NULL |   19 |       |
+----+-------------+----------+------+---------------+------+---------+------+------+-------+
1 row in set (0.01 sec)
Another item you should investigate is the slow query log file. If you look in /etc/mysql/my.cnf, you will notice 2 lines that relate to slow queries, make sure you uncomment them and restart MySQL. The long_query_time can be adjusted to say 10 seconds, so that any query running longer than 10 seconds is logged. See https://dev.mysql.com/doc/refman/5.1/en/slow-query-log.html for additional details.

1
2
#log_slow_queries       = /var/log/mysql/mysql-slow.log
#long_query_time = 2
Another thing you can do is enable logging for queries that are not using indexes. As mentioned above using indexes speeds up performance. In /etc/mysql/my.cnf uncomment the below line and restart MySQL. The log will be in the same place as mysql-slow.log.

1
#log-queries-not-using-indexes
Query cache is another item to check. MySQL caches queries and returns results from this cache if the table has not changed. This has a performance improvement of over 200%. You should check the query cache to ensure that there is no memory for the cache and that the cache is not having to be cleared for new items. Additional information can be found here https://dev.mysql.com/doc/refman/5.1/en/query-cache.html.

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
mysql&amp;gt; SHOW VARIABLES LIKE 'have_query_cache';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| have_query_cache | YES   |
+------------------+-------+
1 row in set (0.00 sec)
 
mysql&amp;gt; SHOW STATUS LIKE 'Qcache%';
+-------------------------+----------+
| Variable_name           | Value    |
+-------------------------+----------+
| Qcache_free_blocks      | 12       |
| Qcache_free_memory      | 16491184 |
| Qcache_hits             | 7645     |
| Qcache_inserts          | 5539     |
| Qcache_lowmem_prunes    | 0        |
| Qcache_not_cached       | 277      |
| Qcache_queries_in_cache | 156      |
| Qcache_total_blocks     | 334      |
+-------------------------+----------+
8 rows in set (0.00 sec)
How do you allow *only* SSL connections from remote users in MySQL?

mysql> UPDATE mysql.user SET ssl_type=’ANY’ WHERE user='<your-username>’

Linux Systems

Define the boot process of a Linux system

Once you power a system on, the first thing that happens is the BIOS loads and performs POST or a power on self test, to ensure that the components needed for a boot are ok. For instance if the CPU is defective, the system will give an error that POST has failed. (BIOS stands for Basic Input/Output system)
After POST the BIOS looks at the MBR or master book record and executes the boot loader. In case of a Linux system that might be GRUB or Grand Unified BootLoader. GRUB’s job is to give you the choice of loading a Linux kernel or other OS that you may be running
Once you ask GRUB to load a kernel, usually an initial ramdisk kernel is loaded, which is a small kernel that understands filesystem. This will in turn mount the filesystem and will start the Linux kernel from the filesystem
The kernel will then start init, which is the very first process, usually having PID 1. Init will look at /etc/inittab and will switch to the default run-level which on Linux servers tends to be 3.
There are different run level scripts in /etc/rc.d/rc[0-6].d/ which are then executed based on the runlevel the system needs to be in.
And that’s about it!
How do you make changes to kernel parameters that you want to persist across boot?

/etc/sysctl.conf contains kernel parameters that can be modified. You can also use the sysctl command to make changes at runtime.

Security

How does SSL work?

SSL stands for secure socket layer. It has been superseded by TLS or transport layer security. TLS is a secure way of communicating through a network. A majority of secure HTTP communication on the web takes place using TLS. TLS works at session layer and presentation layer of the OSI model. Initially at the session layer asymmetric encryption takes place, after that at the presentation later symmetric cipher and session key are used. The basic principle behind TLS is to encrypt data going across the network using public key encryption first, followed by using a shared key. Also the other component of TLS is server certificate authentication which is done through a certificate authority. Clients contain a list of certificate authorities, and it uses the public key of the CA in the certificate to verify the certificate being authentic. A good reference for TLS is here https://en.wikipedia.org/wiki/Secure_Socket_Layer.

Miscellaneous

How does Apache worker.c compare to prefork?

Worker.c uses threads. Prefork uses forks. Prefork is by default in Apache. Worker.c uses less resources, but is more complex.

How do you update packages on Fedora/Ubuntu?

1
2
3
4
5
#Ubuntu
sudo apt-get update -y
 
#Fedora
sudo yum update -y
How do you upgrade to a new release on Fedora/Ubuntu?

1
2
3
4
5
#Ubuntu
sudo apt-get upgrade -y
 
#Fedora
Use FedUP https://fedoraproject.org/wiki/FedUp
How do you use SSH proxy to connect to a remote host?

In your $HOME/.ssh/config use:

1
2
3
4
5
TCPKeepAlive=yes
ServerAliveInterval=15
 
Host finaldestinationhost
   ProxyCommand ssh user@jumphost nc finaldestinationhost %p
To ssh use ssh user@finaldestinationhost.

How do you use SSH to create a dynamic tunnel?

Let’s say there are 3 hosts, one is source, the other is destination and you cannot get to the destination from the source.
In the middle is a gateway that can reach both the source and the destination.
One possible solution to get from source to destination using SSH dynamic tunnel, is to create a dynamic tunnel.
The way it would work is

How do you use change MySQL root password?

1
2
3
4
$mysql -u root -p
&amp;gt;use mysql;
&amp;gt;update user set password=PASSWORD(&amp;quot;NEWPASSWORD&amp;quot;) where User='root';
&amp;gt;flush privileges;
How do you redirect console of a Linux host to serial interface?

Make sure in BIOS serial console port redirection is set.
Secondly in the Grub menu, append the following to the boot line ‘console=tty0 console=ttyS1,57600n8’.

How do you VNC server without any authentication?

Xvnc :2 -nevershared -depth 16 -br IdleTimeout=0 -auth /dev/null -once DisconnectClients=false desktop=”MyDesktop” SecurityTypes=None rfbauth=0

How do you install CentOS via HTTP if not using Kickstart?

One option is to boot from the network using PXE or using a USB drive which has Unetbootin installed.
Once you start installation, go to the main menu, select the ‘Start Installation’ option, choose ‘Network’ as the source, choose ‘HTTP’ as the protocol, enter ‘mirrors.kernel.org’ when prompted for a server, and enter ‘/centos/6/os/x86_64’ when asked for the folder.
