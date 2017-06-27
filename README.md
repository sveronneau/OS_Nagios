# OS_Nagios
OpenStack Monitoring with Nagios


# NAGIOS BASE INSTALL
 
ubuntu instance/server:

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
 
sudo vi /usr/lib/nagios/plugins/nova-list
 
#!/bin/bash
export OS_ENDPOINT_TYPE=internalURL
export OS_INTERFACE=internalURL
export OS_USERNAME=admin
export OS_PASSWORD=''
export OS_PROJECT_NAME=admin
export OS_TENANT_NAME=admin
export OS_AUTH_URL=http://public_ip:5000/v3
export OS_NO_CACHE=1
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_REGION_NAME=RegionOne
#
data=$(nova list  2>&1)
rv=$?
#
if [ "$rv" != "0" ] ; then
    echo $data
    exit $rv
fi
#
echo "$data" | grep -v -e '--------' -e '| Status |' -e '^$' | wc -l
#
 
sudo chmod u+x /usr/lib/nagios/plugins/nova-list
sudo vi /etc/nagios3/conf.d/openstack.cfg
#########################################
# OPENSTACK
#########################################
##### HOSTS
#########################################
define host{
        use                     generic-host            ; Name of host template to use
        host_name               OpenStack
        alias                   OpenStack
        address                 127.0.0.1
        }
#########################################
##### COMMANDS
#########################################
define command {
        command_line            /usr/lib/nagios/plugins/nova-list
        command_name            nova-list
}
define command{
        command_name            check_nrpe_os
        command_line            /usr/lib/nagios/plugins/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}
#
#######################################
##### SERVICES
#######################################
define service {
        check_command           nova-list
        host_name               OpenStack
        name                    nova-list
        normal_check_interval   5
        service_description     Number of nova vm instances
        use                     generic-service
        }
#
sudo nagios3 -v /etc/nagios3/nagios.cfg
sudo service nagios3 restart
http://you_public_ip/nagios3


# NODE MONITORING
 
On each server / instance / container you want to monitor:


sudo apt-get update
sudo apt-get -y install nagios-nrpe-server nagios-plugins-standard
sudo apt-get upgrade -y
sudo vi /etc/nagios/nrpe.cfg
allowed_hosts=127.0.0.1, NagiosServerIP
vi /etc/nagios/nrpe.d/openstack.cfg
command[keystone]=/usr/lib/nagios/plugins/check_procs -c 1: -w 3: -C keystone-all
iptables -I INPUT -p tcp --dport 5666 -j ACCEPT
iptables-save
service nagios-nrpe-server restart


# SERVICE CHECKS (example)

From the NAGIOS server:
 
sudo vi /etc/nagios3/conf.d/openstack_keystone.cfg
