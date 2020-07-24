## IPTables

By Jack Szwergold

### Start, stop and control IPTables.

On Ubuntu/Debian systems you can start, stop, restart and flush IPTables this way:

	sudo service iptables-persistent start
	sudo service iptables-persistent stop
	sudo service iptables-persistent restart
	sudo service iptables-persistent flush

For Ubuntu 16.04, use `netfilter-persistent` instead:

	sudo service netfilter-persistent start
	sudo service netfilter-persistent stop
	sudo service netfilter-persistent restart
	sudo service netfilter-persistent flush

On CentOS/RedHat systems you can start, stop, restart and flush IPTables this way:

    sudo /etc/rc.d/init.d/iptables start
    sudo /etc/rc.d/init.d/iptables stop
    sudo /etc/rc.d/init.d/iptables restart
    sudo /etc/rc.d/init.d/iptables flush

Check what version of IPTables you have installed by running this command:

    iptables -V

The output should be something like this:

    iptables v1.4.12

### Different ways to showing IPTables rules.

List all rules (`-L`) in the selected table—default is `-t filter`—with hostname lookups.

	sudo iptables -L

List all rules (`-L`) in the selected table—default is `-t filter`—in numeric format (`-n`) without hostname lookups.

	sudo iptables -L -n

List all rules (`-L`) in the selected table—default is `-t filter`—in numeric format (`-n`) without hostname lookups and add line numbers (`--line-numbers`) to each line.

	sudo iptables --line-numbers -n -L

List all rules (`-L`) in the NAT table (`-t nat`) in numeric format (`-n`) without hostname lookups.

    sudo iptables -t nat -L -n

### Install and setup `iptables-persistent`.

By default, an Ubuntu/Debian systems lose IPTables rules on reboot. Installing `iptables-persistent` assures that rules are saved and reloaded on reboot:

    sudo aptitude install iptables-persistent

Or for Ubuntu 16.04 and higher:

    sudo aptitude install netfilter-persistent

If IPTables rules have already been set on the system, export the rules into a text file like this:

    sudo iptables-save > ~/rules.v4

Now create an `iptables-persistent` specific storage directory if it doesn’t exist already:

    sudo mkdir /etc/iptables

On Ubuntu/Debian you can copy the rules saved in the text file `~/rules.v4` into the `/etc/iptables/rules.v4` location so `iptables-persistent` can load them:

    sudo cp ~/rules.v4 /etc/iptables/rules.v4

On RedHat/CentOS you can copy the rules saved in the text file `~/rules.v4` into the `/etc/iptables/rules.v4` location so `iptables-persistent` can load them:

    sudo cp ~/rules.v4 /etc/sysconfig/iptables

And with that done, `iptables-persistent` should be all set.

### Manually importing IPTables rules.

To manually import IPTables rules, just use `iptables-restore` like this:

    sudo iptables-restore < ~/rules.v4

### Flushing IPTables rules.

To dump all IPTables rules, use the `-F` flag to flush all of the IPTables rules in all chains:

    sudo iptables -F

### Delete an IPTables chain.

To delete all non-core IPTables chains, just use the `-X` flag like so:

    sudo iptables -X

To delete a specific IPTables chain, just use the `-X` flag like so and be sure to indicate the name of the chain to be deleted:

    sudo iptables -X [chain name]

### A simple `PREROUTING` example.

To set port rerouting directly in the command line, run a command like this; this command assumes the source IP of `11.22.33.44` will connect on port `8000` and get traffic from port `80`:

    sudo iptables -A PREROUTING -t nat -s 11.22.33.44 --p tcp --dport 8000 -j REDIRECT --to-ports 80

Or you can add this rule to the `*nat` table into the `rules.v4` text file; this command assumes the source IP of `11.22.33.44` will connect on port `8000` and get traffic from port `80`:

    -A PREROUTING -s 11.22.33.44 -p tcp -m tcp --dport 8000 -j REDIRECT --to-ports 80

### A basic and solid IPTables rule set.

This is a basic, solid and relatively simple rule set I like to use with IPTables. The only adjustments to note are that one should adjust the `123.456.789.0` port `22` line to match any IP address you want to be allowed past standard SSH rules. Ports `80` and `443` are open in this config as well as they are standard web ports; add or remove them based on server needs.

	# NAT stuff.
	*nat
	:PREROUTING ACCEPT [2:80]
	:INPUT ACCEPT [2:80]
	:OUTPUT ACCEPT [3:198]
	:POSTROUTING ACCEPT [3:198]
	COMMIT
	
	# Mangle stuff.
	*mangle
	:PREROUTING ACCEPT [87:6395]
	:INPUT ACCEPT [87:6395]
	:FORWARD ACCEPT [0:0]
	:OUTPUT ACCEPT [50:4502]
	:POSTROUTING ACCEPT [50:4502]
	COMMIT
	
	# Filter stuff.
	*filter
	:INPUT ACCEPT [0:0]
	:FORWARD ACCEPT [0:0]
	:OUTPUT ACCEPT [50:4502]
	:BANNED_ACTIONS - [0:0]
	:DDOS_ACTIONS - [0:0]
	:DDOS_DETECT - [0:0]
	:SPOOF_ACTIONS - [0:0]
	:SPOOF_DETECT - [0:0]
	:TOR_ACTIONS - [0:0]
	:AWS_ACTIONS - [0:0]
	-A INPUT -i lo -j ACCEPT
	-A INPUT -p tcp -m set --match-set WHITELIST_IPS src -j ACCEPT
	-A INPUT -p tcp -m set --match-set BANNED_RANGES src -j BANNED_ACTIONS
	-A INPUT -p tcp -m set --match-set BANNED_IPS src -j BANNED_ACTIONS
	-A INPUT -p tcp -m set --match-set TOR_IPS src -j TOR_ACTIONS
	-A INPUT -p tcp -m set --match-set AWS_RANGES src -j AWS_ACTIONS
	-A INPUT -j DDOS_DETECT
	-A INPUT -j SPOOF_DETECT
	-A INPUT -p tcp -m state --state NEW -m tcp -m multiport --dports 80,443 -j ACCEPT
	-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
	-A INPUT -d 224.0.0.251/32 -p udp -m udp --dport 5353 -j ACCEPT
	-A INPUT -p icmp -m icmp --icmp-type any -j ACCEPT
	-A INPUT -p esp -j ACCEPT
	-A INPUT -p ah -j ACCEPT
	-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
	-A INPUT -j REJECT --reject-with icmp-host-prohibited
	
	# Define the banned actions.
	-A BANNED_ACTIONS -j REJECT --reject-with icmp-host-prohibited
	
	# Define the DDoS actions.
	-A DDOS_ACTIONS -p tcp -m limit --limit 3/min --limit-burst 10 -j LOG --log-prefix "IPTABLES_DENIED_TCP: "
	-A DDOS_ACTIONS -p udp -m limit --limit 3/min --limit-burst 10 -j LOG --log-prefix "IPTABLES_DENIED_UDP: "
	-A DDOS_ACTIONS -p icmp -m limit --limit 3/min --limit-burst 10 -j LOG --log-prefix "IPTABLES_DENIED_ICMP: "
	-A DDOS_ACTIONS -j REJECT --reject-with icmp-host-prohibited
	
	# Drop invalid SYN packets.
	-A DDOS_DETECT -p tcp -m tcp --tcp-flags ALL ACK,RST,SYN,FIN -j DDOS_ACTIONS
	-A DDOS_DETECT -p tcp -m tcp --tcp-flags SYN,FIN SYN,FIN -j DDOS_ACTIONS
	-A DDOS_DETECT -p tcp -m tcp --tcp-flags SYN,RST SYN,RST -j DDOS_ACTIONS
	
	# The combination of these TCP flags is not defined.
	-A DDOS_DETECT -p tcp -m tcp --tcp-flags FIN,SYN,RST,PSH,ACK,URG NONE -j DDOS_ACTIONS
	-A DDOS_DETECT -p tcp -m tcp --tcp-flags FIN,SYN,RST,PSH,ACK,URG FIN,SYN,RST,PSH,ACK,URG -j DDOS_ACTIONS
	-A DDOS_DETECT -p tcp -m tcp --tcp-flags FIN,SYN,RST,PSH,ACK,URG FIN,PSH,URG -j DDOS_ACTIONS
	-A DDOS_DETECT -p tcp -m tcp --tcp-flags FIN,RST FIN,RST -j DDOS_ACTIONS
	-A DDOS_DETECT -p tcp -m tcp --tcp-flags FIN,ACK FIN -j DDOS_ACTIONS
	-A DDOS_DETECT -p tcp -m tcp --tcp-flags PSH,ACK PSH -j DDOS_ACTIONS
	-A DDOS_DETECT -p tcp -m tcp --tcp-flags ACK,URG URG -j DDOS_ACTIONS
	
	# Drop new incoming TCP connections are not SYN packets.
	-A DDOS_DETECT -p tcp -m tcp ! --syn -m state --state NEW -j DDOS_ACTIONS
	
	# Drop packets with incoming fragments.
	-A DDOS_DETECT -p tcp -m tcp --tcp-flags ALL ALL -j DDOS_ACTIONS
	
	# Define the spoof actions.
	-A SPOOF_ACTIONS -j ACCEPT
	# -A SPOOF_ACTIONS -m limit --limit 3/min --limit-burst 10 -j LOG --log-prefix "IPTABLES_DENIED_SPOOF: "
	# -A SPOOF_ACTIONS -j REJECT --reject-with icmp-host-prohibited
	
	# One batch of spoof detection addresses.
	-A SPOOF_DETECT -s 10.0.0.0/8 -j SPOOF_ACTIONS
	# -A SPOOF_DETECT -s 169.254.0.0/16 -j SPOOF_ACTIONS
	# -A SPOOF_DETECT -s 172.16.0.0/12 -j SPOOF_ACTIONS
	-A SPOOF_DETECT -s 127.0.0.0/8 -j SPOOF_ACTIONS
	
	# Another batch of spoof detection addresses.
	-A SPOOF_DETECT -s 224.0.0.0/4 -j SPOOF_ACTIONS
	-A SPOOF_DETECT -d 224.0.0.0/4 -j SPOOF_ACTIONS
	-A SPOOF_DETECT -s 240.0.0.0/5 -j SPOOF_ACTIONS
	-A SPOOF_DETECT -d 240.0.0.0/5 -j SPOOF_ACTIONS
	-A SPOOF_DETECT -s 0.0.0.0/8 -j SPOOF_ACTIONS
	-A SPOOF_DETECT -d 0.0.0.0/8 -j SPOOF_ACTIONS
	-A SPOOF_DETECT -d 239.255.255.0/24 -j SPOOF_ACTIONS
	-A SPOOF_DETECT -d 255.255.255.255/32 -j SPOOF_ACTIONS
	
	# Define the TOR actions.
	-A TOR_ACTIONS -j REJECT --reject-with icmp-host-prohibited
	
	# Define the AWS actions.
	-A AWS_ACTIONS -j REJECT --reject-with icmp-host-prohibited
	
	# Commit it.
	COMMIT

### Logging rejected packets.

These are IPTables entries to log entries in the Kernel log (`kern.log`). They would be placed before the final `-A INPUT -j REJECT` in the chain. In the above example these would be placed before this line:

    -A INPUT -j REJECT --reject-with icmp-host-prohibited

Setting a generic log entry:

    -A INPUT -m limit --limit 3/min -j LOG --log-prefix "IPTABLES_DENIED: " --log-level 4

Setting log entries based on TCP, UDP or ICMP requests:

	-A INPUT -p tcp -m limit --limit 3/min --limit-burst 10 -j LOG --log-prefix "IPTABLES_DENIED_TCP: " --log-level 4
	-A INPUT -p udp -m limit --limit 3/min --limit-burst 10 -j LOG --log-prefix "IPTABLES_DENIED_UDP: " --log-level 4
	-A INPUT -p icmp -m limit --limit 3/min --limit-burst 10 -j LOG --log-prefix "IPTABLES_DENIED_ICMP: " --log-level 4

Finding the IPTables specific log entries in the Kernel log (`kern.log`):

    sudo tail -f -n 500 /var/log/kern.log | grep "IPTABLES_DENIED"

This Grep command parses out the source (`SRC`) IP address from `IPTABLES_DENIED` entries based on delimeter position:

    grep "IPTABLES_DENIED" kern.log | cut -d = -f5 | cut -d " " -f1

This Grep command parses out the source (`SRC`) IP address from `IPTABLES_DENIED` entries based on the value of `SRC=`:

    grep "IPTABLES_DENIED" kern.log | grep -Eo "SRC\=[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" | cut -d"=" -f2

Not reliable, but this Awk command parses out the source (`SRC`) IP address from “IPTABLES_DENIED” entries:

    sudo awk '/IPTABLES_DENIED/ { split($11,split_11,"="); printf "%s\n", split_11[2] }' /var/log/kern.log

This variant will let you know date, time and whether the dropped packet was TCP, UDP or ICMP:

    sudo awk '/IPTABLES_DENIED/ { split($11,split_11,"="); printf "(%s %s %s) %s %s\n", $1, $2, $3, $7, split_11[2] }' /var/log/kern.log

### Some ideas that seem to work.

	# Reject packets from RFC1918 class networks (i.e., spoofed)
	
	sudo iptables -N SPOOFING
	sudo iptables -N SPOOF_ACTIONS
	
	sudo iptables -A INPUT -j SPOOFING
	
	sudo iptables -A SPOOFING -s 10.0.0.0/8 -j SPOOF_ACTIONS
	sudo iptables -A SPOOFING -s 169.254.0.0/16 -j SPOOF_ACTIONS
	sudo iptables -A SPOOFING -s 172.16.0.0/12 -j SPOOF_ACTIONS
	sudo iptables -A SPOOFING -s 127.0.0.0/8 -j SPOOF_ACTIONS
	
	sudo iptables -A SPOOFING -s 224.0.0.0/4 -j SPOOF_ACTIONS
	sudo iptables -A SPOOFING -d 224.0.0.0/4 -j SPOOF_ACTIONS
	sudo iptables -A SPOOFING -s 240.0.0.0/5 -j SPOOF_ACTIONS
	sudo iptables -A SPOOFING -d 240.0.0.0/5 -j SPOOF_ACTIONS
	sudo iptables -A SPOOFING -s 0.0.0.0/8 -j SPOOF_ACTIONS
	sudo iptables -A SPOOFING -d 0.0.0.0/8 -j SPOOF_ACTIONS
	sudo iptables -A SPOOFING -d 239.255.255.0/24 -j SPOOF_ACTIONS
	sudo iptables -A SPOOFING -d 255.255.255.255 -j SPOOF_ACTIONS
	
	sudo iptables -A SPOOF_ACTIONS -p tcp -m limit --limit 3/min --limit-burst 10 -j LOG --log-prefix "IPTABLES_DENIED_SPOOF: " --log-level 4
	sudo iptables -A SPOOF_ACTIONS -j REJECT --reject-with icmp-host-prohibited

### Some ideas that are not ready for prime time.

#### Block all UDP traffic except on port 53.

    sudo iptables -A INPUT -p udp --sport 53 -j ACCEPT
    sudo iptables -A INPUT -p udp --dport 53 -j ACCEPT

    sudo iptables -A OUTPUT -p udp --sport 53 -j ACCEPT
    sudo iptables -A OUTPUT -p udp --dport 53 -j ACCEPT

    sudo iptables -A INPUT -p udp -j REJECT
    sudo iptables -A OUTPUT -p udp -j REJECT

#### Block outbound UDP on port 53 but allow port 53 with Google’s DNS servers.

    sudo iptables -A OUTPUT -p udp -j REJECT --reject-with icmp-host-prohibited
    sudo iptables -A OUTPUT -p udp --dport 53 -d 8.8.8.8 -j ACCEPT
    sudo iptables -A OUTPUT -p udp --dport 53 -d 8.8.4.4 -j ACCEPT

#### UDP throttling rules.

    sudo iptables -N UDP_OUT_FLOOD
    sudo iptables -A OUTPUT -p udp -j UDP_OUT_FLOOD
    sudo iptables -A UDP_OUT_FLOOD -p udp -m limit --limit 3/min --limit-burst 10 -j LOG --log-prefix "IPTABLES_DENIED_UDP_OUT: " --log-level 4
    sudo iptables -A UDP_OUT_FLOOD -j REJECT --reject-with icmp-host-prohibited

***

*IPTables (c) by Jack Szwergold; written on September 11, 2015. This work is licensed under a Creative Commons Attribution-NonCommercial 4.0 International License (CC-BY-NC-4.0).*