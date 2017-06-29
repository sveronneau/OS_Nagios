# OS_Nagios
OpenStack Monitoring with Nagios

# Pre-requisite
On the Nagios server the OpenStack client needs to be installed (https://docs.openstack.org/user-guide/common/cli-install-openstack-command-line-clients.html)

# NAGIOS BASE INSTALL
 
Ubuntu instance/server:

```
> sudo apt-get update
> sudo apt-get install -y nagios*
> sudo vi /etc/nagios3/nagios.cfg
> check_external_commands=1
> sudo vi /etc/group
> - nagios:x:???:
> + nagios:x:???:www-data
> sudo chmod g+x /var/lib/nagios3/rw
> sudo apt-get upgrade -y
```

If Apache user was not created during Nagios installation:

```
> sudo htpasswd -c /etc/nagios3/htpasswd.users nagiosadmin
> #provide password when prompted
```

sudo nagios3 -v /etc/nagios3/nagios.cfg (shoudl return OK)

sudo service nagios3 restart

http://you_public_ip/nagios3


# NODE MONITORING VIA NRPE AGENT
 
On each bare metal server / instance or container you want to monitor:

```
> sudo apt-get update
> sudo apt-get -y install nagios-nrpe-server nagios-plugins-standard
> sudo apt-get upgrade -y
> sudo vi /etc/nagios/nrpe.cfg
>   allowed_hosts=127.0.0.1, **NagiosServerIP**
> iptables -I INPUT -p tcp --dport 5666 -j ACCEPT
> iptables-save
```

service nagios-nrpe-server restart
