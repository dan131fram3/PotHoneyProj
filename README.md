## IP Camera honeypot Guide

This guide will get you up and running on the Honeypot 'HoneyPy' running an IoT camera configuration with the ports 554, 2323, and 8080 running.

Firstly install a Linux Distro such as Ubuntu 16.04, Linux Mint, or another of your choice. Testing was conducted on Linux Mint 18 and Ubuntu 16.04 but installing the right dependancies should support the installation of many other distros (Distro installation won't be covered in here, but there are plenty availability online already).

Once your machine is running for the first time, you will want to update and install latest patches for your disto. Run the following commands:

`sudo apt-get update`

`sudo apt-get upgrade`

Then reboot.

After the restart, install the requirements:

`sudo apt-get install python`

`sudo apt-get install python-matplotlib`

`sudo apt-get install python-twisted`

Create new user for honeypot user

`sudo adduser honeypot-user`

Switch the the new user

`su - honeypot-user`

Download zip file for honeypot framework

`wget http://raw.githubusercontent.com/dan131fram3/PotHoneyProj/master/HoneyPy.tar.gz`

Now uncompress the files

`tar -zxf HoneyPy.tar.gz`

Now, change into the directory for Honeypy. To start the honeypot and the emulated services, Honeypy can be run either as a daemon, so that it could run in the background (not just in a single terminal), or  standard in a terminal. For more information, check out Honeypy’s ‘README.md’ file.
The following command will start and background the honeypot process for persistence:

`cd /home/honeypot-user/HoneyPy-0.4.8/`

`python Honey.py -d &`

This won’t survive server reboots, so bare that in mind after restarting, the command will need to be reran.
You can now confirm that the honeypot is up via the command ‘netstat -ano | head -n 20’ which should output various listening ports.

Now the honeypot is ready to record attacks and also give the fake banner responses, but only on unprivilaged ports (port 1024+), and the current configuration uses ports 2323, 8080, and port 5544, although port 554 will be needed instead of 5544, to advertise itself as RTSP properly. This can be done via a linux command line tool, **IPtables**, allowing to redirect any traffic from port 554 to port 5544 (The port which is actually being emulated) and using existing scripts by the author ‘foospidy’ called ‘ipt-kit’, can ease these changes for port redirection, which in this case will be 554 -> 5544.

First, change to the user with sudo access, and change directory to the ipt-kit directory (This was created from unzipping file from earlier).

`su - yourmainuser` (may need to type ‘exit’ if you have no Privileges)

`cd /home/honeypot-user/ipt-kit/`

Now the scripts for ipt-kit will be in here. To redirect any attacker requests from port 554 to 5544, the following command is used:

`sudo ./ipt_set_tcp 554 5544`

You can check the port forwarding rules with this command:

`sudo ./ipt`

To save these changes for restarts, use this command:

`sudo ./ipt_survive_reboot`

And enter ‘y’ to save.

The honeypot should now have the ports 2323, 8080, and 554 visible to the network, and the honeypot will record all the requests.

To open up the HoneyPot to the world, you will need to configure your router settings to allow traffic through ports 554, 2323 and 8080 (This is a security risk, so ensure you do so safely). Opening ports on routers is out of scope for this project, as all routers are different (See your local router firewall settings).

Now within just days, you will see many connection attempts if all is setup correctly. To check for requests before graphing, try:

`tail /home/honeypot-user/HoneyPy-0.4.8/log/honeypy.log`

This should show requests if any have been established. You can see the honeypot IP and port, and the attacking IP and port, unless 'services started' messages appear, in which case, no results have been logged.

You can also test logging is working by using 'telnet' or 'nc' to the honeypot and port, e.g.

`telnet 192.168.0.50 2323`

Use `ctrl + ]` to quit telnet

This will generate the log in the file '/home/honeypot-user/HoneyPy-0.4.8/log/honeypy.log'

Also note, HoneyPy will create a new log file each day, and append the date on the file e.g. honeypy.log.2017_2_13, so look out for multiple log files.

####Visualising the logs

Once enough time has passed and requests have been logged, you can run the visualisation script and view the number of requests against various ports, although you could just test for valid attempts this way too. For visualisation of logs once together, there is a script at HoneyPy-0.4.8/log/visualise.py which is built to produce a bar chart containing connection attempts for each port. 
Just execute the script to visualise any files in the current directory starting with ‘honeypy.log’.

`cd /home/honeypot-user/HoneyPy-0.4.8/log/`

`./visualise`

This will create the graph called ‘honeypot_port_results.png’. #####Note - there will need to be logged attacks against ports before the graph can be produced.

#### Credits

Credit to the author of HoneyPy, foospidy for the Honeypot, and all work is an adaptation of this (https://github.com/foospidy/HoneyPy).
