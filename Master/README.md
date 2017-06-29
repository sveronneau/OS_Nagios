# MANDATORY
Your admin openrc file must be copied in /usr/lib/nagios/plugins/ on the Nagios server and be made readable by all.

# openstack/openstack.cfg (pre-requisite)
you must edit the openstack.cfg file to reflect the IPs of your environment

# file location on the Nagios server
Content of the openstack folder must be put in /etc/nagios3/conf.d/openstack /           (create the openstack folder)

Content of the plugins folder must be put in /usr/lib/nagios/plugins/
```
> chown root:root /usr/lib/nagios/plugins/OS_*
> chmod 755 /usr/lib/nagios/plugins/OS_*
```

# Post-Copy requirement
You must restart the nagios server service to load the configs
```
> sudo service nagios3 restart
```
