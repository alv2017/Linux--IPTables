# Linux--IPTables

### What is IPTables

Every Linux distro has a built-in firewall called netfilter. IPTables is a command line
utility that is used to maintain netfilter.

### Checking Current IPTables Rules

IPv4:
$ sudo iptables -L

This command should return a list of rules for the IPv4

IPv6:
$ sudo ip6tables -L

This command should return a list of rules for the IPv6

### Attention Ubuntu Users

Initially Ubuntu systems do not have any IPTables rule set, so the machine is wide open,
and it is Ubuntu user's responsibility to set the rules up.

### IPv4 Basic Rules Explained

a) Allow traffic for the loopback interface:

**-A INPUT -i lo -j ACCEPT**

In Ubuntu in order to enter IPTables rule you need to use the following command:

$ sudo iptables -A INPUT -i lo -j ACCEPT

Explanation: -A INPUT means that we want to place the rule at the end of the INPUT chain (i.e. incoming traffic chain); -i is interface; lo translates to loopback; -j means jump; ACCEPT means that we want to allow the traffic.

b) Allow to pass all the incoming packets from servers that our host has requested a connection to:

**-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT**

Explanation: A couple of new things: -m calls for an iptables module, in our case for the conntrack module; --ctstate option tells conntrack module to look for two things: first, it is looking for a connection that client has established with a server; next, it looks for related connection that is coming back from the server in order to allow it to connect to the client.

c) Suppose you need to connect to machine using SSH, so you need to allow incoming SSH traffic:

**-A INPUT -p tcp --dport ssh -j ACCEPT**

Explanation: -p tcp means TCP protocol; --dport ssh stands for destination port ssh.

d) Suppose you want the machine to serve as DNS server, then add the following rules:

**-A INPUT -p tcp --dport 53 -j ACCEPT**

**-A INPUT -p udp --dport 53 -j ACCEPT**

e) Block everything that is not allowed:

**-A INPUT -j DROP**

### IPv4: Blocking ICMP with IPTables

The conventional wisdom says that we need to block all the packets
from the Internet Control Message Protocol (ICMP). This is not quite
true, blocking certain types of ICMP packets is a good practice,
however blocking all the ICMP packets is a bad idea. Certain types
of ICMP messages are necessary for proper network functioning. We are going
to allow the following types of ICMP messages:

type 3: destination unreachable messages.

type 4: source quench message is a request to decrease the traffic rate of data messages sent to an internet destination. 

type 11: time exceeded messages.

type 12: parameter problem messages indicate that the server had sent a packet with a bad IP header. 

This is a set of rules that we have set so far:

**-A INPUT -i lo -j ACCEPT**

**-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT**

**-A INPUT -p tcp --dport ssh -j ACCEPT**

**-A INPUT -p tcp --dport 53 -j ACCEPT**

**-A INPUT -p udp --dport 53 -j ACCEPT**

**-A INPUT -j DROP**

... and the ICMP rules are listed below

**-I INPUT 6 -p icmp -m conntrack --ctstate NEW,RELATED,ESTABLISHED -m icmp --icmp-type 3 -j ACCEPT**

**-I INPUT 7 -p icmp -m conntrack --ctstate NEW,RELATED,ESTABLISHED -m icmp --icmp-type 4 -j ACCEPT**

**-I INPUT 8 -p icmp -m conntrack --ctstate NEW,RELATED,ESTABLISHED -m icmp --icmp-type 11 -j ACCEPT**

**-I INPUT 9 -p icmp -m conntrack --ctstate NEW,RELATED,ESTABLISHED -m icmp --icmp-type 12 -j ACCEPT**

Explanation: ...and a couple of new things here:
-I INPUT 5 tells to insert the rule into the INPUT chain at the position 6;
-p icmp - tells that the rule is related to ICMP traffic;
-m icpmp - tells to use icmp module; --icmp-type 3 - tells that rule regulates ICMP messages of type 3.

This is it about IPv4 rules.