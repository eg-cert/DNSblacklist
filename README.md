#DNSBlacklist project.

##Introduction.
	This project aims to be an assisting tool to setup your own DNS blacklisting in your
	enterprise environment. While the DNS provide blacklisting service, it will provide
	a good and fast caching for the enterprise DNS use. This tool will retrieve latest known
	malicious domains, and generate configuration file for BIND or UNBOUND DNS server.

	This script utilizing unbound as local recursive DNS server for your environment.

##How it works

	The script will pull malicious domains from various sources, to be configured in an
	unbound/bind DNS server. This server will be your internal DNS server in your environment.
	Any DNS request to malicious domain by any user in your environment will be handled by 
	Unbound/BIND by returning a specified IP, usually 127.0.0.1, or any 'blackhole' IP. You 
	can point to another server to monitor the malicious request

This include domain parser from various malicious domain provider
- http://www.malwaredomains.com/
- https://spyeyetracker.abuse.ch/blocklist.php?download=domainblocklist
- http://www.abuse.ch/zeustracker/blocklist.php?download=domainblocklist
- https://palevotracker.abuse.ch/blocklists.php?download=domainblocklist
- https://isc.sans.edu/suspicious_domains.html#lists
- http://malc0de.com/bl/ZONES
- http://labs.sucuri.net/?malware
- www.malwareblacklist.com/mbl.xml
- http://www.malwarepatrol.net/cgi/submit?action=list_bind
- http://mtc.sri.com/live_data/malware_dns/
- http://exposure.iseclab.org/malware_domains.txt
- http://support.clean-mx.de/clean-mx/xmlviruses?format=xml&fields=review,url&response=alive
- http://www.nictasoft.com/ace/malware-urls/
- http://mirror1.malwaredomains.com/files/spywaredomains.zones

##Main features
- Configurables of which domain sources to be used.
- Option for output format, Unbound or Bind DNS server (Unbound by default)
- Domain permanent whitelisting and blacklisting

The main script is preparation.sh, which generate a configuration 
file for unbound DNS server. You can choose BIND format output as well

##How to use 
### General Configurations
- Edit blackhole/run.sh
- Insert 1 or 0 for the desired list to be downloaded from
~~~
SAGADC=0
SPYEYE=1
ZEUSTRACKER=1
~~~
- Change ` DOWNLOAD_FILES=0` to `DOWNLOAD_FILES=1` in order to tell the script to download the lists , you can later disable this if you don't want to update the lists

- Edit `/etc/resolv.conf` insert this line in the top of the file to resolve from yourself  **for testing**
```
nameserver 127.0.0.1
```

### Configuring it to be used with bind server

- Install bind9 packages
```bash 
apt-get install bind9
   ```
- Install dos2unix packages
```bash
apt-get install dos2unix
```
- Clone the repo
```bash 
git clone  https://github.com/aabed/DNSblacklist.git 
   ```
- cd into the directory
```bash
cd DNSblacklist/blackhole
```
- Copy the file ` ./master.list.hosts` to `/etc/bind`
```bash 
cp ../master.list.hosts /etc/bind
```

- Edit `/etc/bind/named.conf` and append this line
~~~
include "/etc/bind/master.list.zones";
~~~
- Edit run.sh 

    modify  ` DNSSERVER="unbound" `  to ` DNSSERVER="bind" ` 

- Run run.sh in run.sh
```bash
bash run.sh
```
- Copy the file ` ./master.list.zones` to `/etc/bind`
```bash 
cp master.list.zones /etc/bind
```
- Restart bind service
``` bash 
service bind9 restart
```
- Test the results
```bash
nslookup  nslookup zzzjsh.com
```

### Configuring it to be used with unbound server
- Clone the repo
```bash
git clone https://github.com/aabed/DNSblacklist.git 
```
- Move repo contents to /etc/unbound
```bash
mv DNSblacklist/* /etc/unbound
```
- Edit /etc/unbound/unbound.conf according to your server environment
i.e
```
interface: 192.168.0.1
```
```
forward-addr: 8.8.8.8
```

- cd into the directory
```bash
cd /etc/unbound/blackhole
```
- Edit run.sh 

make sure that  ` DNSSERVER="unbound" ` 
change `SINK_HOLE_IP=0.0.0.0` to your desired ip

- Run run.sh in /etc/unbound/blackhole/
```bash
bash /etc/unbound/blackhole/run.sh
```

- "/etc/unbound/blackhole/blacklisted_domains.conf" will be created automatically.
- Run `unbound-checkconf` to verify the config file
- Restart unbound for the config file to be effective.
```bash
service unbound restart
```
- Test the results
```bash
nslookup zzzjsh.com
```
