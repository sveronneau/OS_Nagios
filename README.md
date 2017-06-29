# OS_Nagios
OpenStack Monitoring with Nagios

# Pre-requisite
On the Nagios server the OpenStack client needs to be installed.
https://docs.openstack.org/user-guide/common/cli-install-openstack-command-line-clients.html 

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

# INSTANCE CHECK (example)
 
**MANDATORY** your openrc file must be copied in /usr/lib/nagios/plugins

sudo vi /usr/lib/nagios/plugins/OS_server-list

```
> #!/bin/bash
> #
> source openrc
> # 
> data=$(openstack server list --all-projects 2>&1)
> rv=$?
> #
> if [ "$rv" != "0" ] ; then
>     echo $data
>     exit $rv
> fi
> #
> echo "$data" | grep -v -e '--------' -e '| ID' -e '^$' | wc -l
> #
```

sudo chmod u+x /usr/lib/nagios/plugins/OS_server-list

sudo mkdir /etc/nagios3/conf.d/openstack

sudo vi /etc/nagios3/conf.d/openstack/openstack.cfg

```
> #########################################
> # OPENSTACK
> #########################################
> ##### HOSTS
> #########################################
> define host{
>         use                     generic-host            ; Name of host template to use
>         host_name               OpenStack
>         alias                   OpenStack
>         address                 10.178.148.59
>         }
> #########################################
> ##### COMMANDS
> #########################################
> define command {
>         command_line            /usr/lib/nagios/plugins/OS_server-list
>         command_name            OS_server-list
>         name                    OS_server-list
> }
> #
> define command{
>         command_name            check_nrpe_os
>         command_line            /usr/lib/nagios/plugins/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
> }
> #
> #######################################
> ##### SERVICES
> #######################################
> define service {
>         check_command           OS_server-list
>         host_name               OpenStack
>         name                    OS_server-list
>         normal_check_interval   5
>         service_description     Number of instances
>         use                     generic-service
>         }
> #
```

sudo nagios3 -v /etc/nagios3/nagios.cfg (shoudl return OK)

sudo service nagios3 restart

http://you_public_ip/nagios3


# NODE MONITORING VIA NRPE AGENT (example)
 
On each bare metal server / instance or container you want to monitor:

```
> sudo apt-get update
> sudo apt-get -y install nagios-nrpe-server nagios-plugins-standard
> sudo apt-get upgrade -y
> sudo vi /etc/nagios/nrpe.cfg
>   allowed_hosts=127.0.0.1, NagiosServerIP
> sudo vi /etc/nagios/nrpe.d/openstack.cfg
>   command[OS_keystone]=/usr/lib/nagios/plugins/OS_check_keystone_procs
> sudo vi /usr/lib/nagios/plugins/OS_check_keystone_procs
>   #!/bin/bash
>   #
>   data=$(ps -def | grep 'wsgi:keystone' | grep -v grep | wc -l)
>   #
>   if [ "$data" -eq "0" ] ; then
>       echo "CRITICAL - $data Keystone Processes"
>       exit 2
>   fi
>   #
>   if [ "$data" -lt "4" ] ; then
>       if [ "$data" -gt "1" ] ; then
>           echo "WARNING - $data Keystone Processes"
>           exit 1
>       fi
>   fi
>   #
>   if [ "$data" -eq "4" ] ; then
>       echo "OK - $data Keystone Processes"
>       exit 0
>   fi
>   #
> iptables -I INPUT -p tcp --dport 5666 -j ACCEPT
> iptables-save
```

service nagios-nrpe-server restart

# SERVICE CHECKS (example)

From the NAGIOS server: 

sudo vi /etc/nagios3/conf.d/openstack/openstack_keystone.cfg

```
> #########################################
> # OPENSTACK
> #########################################
> ##### HOSTS
> #########################################
> define host{
>         use                     generic-host
>         host_name               aio1_keystone_container-dc6b299f
>         alias                   Keystone
>         address                 172.29.236.196
>         }
> #
> #######################################
> ##### SERVICES
> #######################################
> define service{
>         use                     generic-service
>         host_name               aio1_keystone_container-dc6b299f
>         service_description     PING
>         check_command           check_ping!100.0,20%!500.0,60%
> }
> #
> define service{
>         use                     generic-service
>         host_name               aio1_keystone_container-dc6b299f
>         service_description     Logged in users
>         check_command           check_nrpe_os!check_users
> }
> #
> define service{
>         use                     generic-service
>         host_name               aio1_keystone_container-dc6b299f
>         service_description     Load Average
>         check_command           check_nrpe_os!check_load
> }
> #
> define service{
>         use                     generic-service
>         host_name               aio1_keystone_container-dc6b299f
>         service_description     Zombie Processes
>         check_command           check_nrpe_os!check_zombie_procs
> }
> #
> define service{
>         use                     generic-service
>         host_name               aio1_keystone_container-dc6b299f
>         service_description     Total Processes
>         check_command           check_nrpe_os!check_total_procs
> }
> #
> define service{
>         use                     generic-service
>         host_name               aio1_keystone_container-dc6b299f
>         service_description     Root Partition
>         check_command           check_nrpe_os!check_root
> }
> #
> define service{
>         use                     generic-service
>         host_name               aio1_keystone_container-dc6b299f
>         service_description     Log Partition
>         check_command           check_nrpe_os!check_log
> }
> #
> define service{
>         use                     generic-service
>         host_name               aio1_keystone_container-dc6b299f
>         service_description     Identity Service Processes
>         check_command           check_nrpe_os!OS_keystone
> }
> #
```

sudo nagios3 -v /etc/nagios3/nagios.cfg (shoudl return OK)

sudo service nagios3 restart

http://you_public_ip/nagios3
