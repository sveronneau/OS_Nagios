# file location
Content of the openstack folder must be put in /etc/nagios3/conf.d/openstack /           (create the openstack folder)

Content of the plugins folder must be put in /usr/lib/nagios/plugins/
```
> chown root:root /usr/lib/nagios/plugins/OS_*
> chmod 755 /usr/lib/nagios/plugins/OS_*
```

# openstack/openstack.cfg (pre-requisite)
you must edit the openstack.cfg file to reflect the IPs of your environment
