# file location in the server/instance/container running NRPE Server
Content of the openstack folder must be put in /etc/nagios/nrpe.d/openstack /           (create the openstack folder)

Content of the plugins folder must be put in /usr/lib/nagios/plugins/
```
> chown root:root /usr/lib/nagios/plugins/OS_*
> chmod 755 /usr/lib/nagios/plugins/OS_*
```
