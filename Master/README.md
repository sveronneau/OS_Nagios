# MANDATORY
Your admin openrc file must be copied in /usr/lib/nagios/plugins/ on the Nagios server and be made readable by all.

# contacts/contacts_nagios2.cfg (pre-requisite)
you must edit the contacts_nagios2.cfg file and put your email adress. Also, if you don't want the slack integration part, just remove the ',slack' after root in the contactgroup members section.

# openstack/openstack.cfg (pre-requisite)
you must edit the openstack.cfg file to reflect the IPs of your environment

# openstack/slack_nagios.cfg (if you want to use slack inetgration)
edit slack_nagios.cfg and insert the channel you want to post on. 
```
> command_line /usr/lib/nagios/plugins/slack_nagios.pl -field slack_channel=#MY_SLACK_CHANNEL -field.............
```

# file location on the Nagios server
Content of the **contacts folder** must be put in /etc/nagios3/conf.d/

Content of the **openstack folder** must be put in /etc/nagios3/conf.d/openstack/           (create the openstack folder)

Content of the **plugins** folder must be put in /usr/lib/nagios/plugins/
```
> chown root:root /usr/lib/nagios/plugins/OS_*
> chmod 755 /usr/lib/nagios/plugins/OS_*
```

# Post-Copy requirement
You must restart the nagios server service to load the configs
```
> sudo service nagios3 restart
```
