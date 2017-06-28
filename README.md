# OS_Nagios
OpenStack Monitoring with Nagios


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

# INSTANCE CHECK (example)
 
sudo vi /usr/lib/nagios/plugins/OS_server-list

```
> #!/bin/bash
> export OS_ENDPOINT_TYPE=internalURL
> export OS_INTERFACE=internalURL
> export OS_USERNAME=admin
> export OS_PASSWORD='my_super_secure_password'
> export OS_PROJECT_NAME=admin
> export OS_TENANT_NAME=admin
> export OS_AUTH_URL=http://public_endpoint:5000/v3
> export OS_NO_CACHE=1
> export OS_USER_DOMAIN_NAME=Default
> export OS_PROJECT_DOMAIN_NAME=Default
> export OS_REGION_NAME=RegionOne
> export OS_IDENTITY_API_VERSION="3"
> 
> data=$(openstack server list --all-projects 2>&1)
> rv=$?
> 
> if [ "$rv" != "0" ] ; then
>     echo $data
>     exit $rv
> fi
> 
> echo "$data" | grep -v -e '--------' -e '| ID' -e '^$' | wc -l
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
> }
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


# NODE MONITORING
 
On each bare metal server / instance or container you want to monitor:

```
> sudo apt-get update
> sudo apt-get -y install nagios-nrpe-server nagios-plugins-standard
> sudo apt-get upgrade -y
> sudo vi /etc/nagios/nrpe.cfg
>   allowed_hosts=127.0.0.1, NagiosServerIP
> vi /etc/nagios/nrpe.d/openstack.cfg
>   command[keystone]=/usr/lib/nagios/plugins/check_procs -c 1: -w 3: -C keystone-all
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
