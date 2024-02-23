This NCPA plugin is made to monitor SFTP server health.

Functionalities :

- Disk usage or dedicated partition disk usage for sftp service ;
- SFTP access availability (supports also SFTP service on non-default port) ;
- SFTP active connections ;
- Connected users using SFTP.

### Setup

To install this script, just put it in the plugins folder of the NCPA agent of target server and enable bash support in the NCPA config file. For this, you have to add the .bash extension management (Yes, it's needed, despite #!/bin/bash at the beginning of the script, NCPA will by default use sh :)). To add this, just go to the "Extensions for plugins" section under "Linux / Mac OS X" and write this line for example after .sh:
```text
.bash = /bin/bash $plugin_name $plugin_args
```

Save the file and restart NCPA agent:

```shell
sudo service ncpa restart
# Or... If you prefer...
sudo systemctl restart ncpa
```

You're done.

Please keep in mind the NCPA agent is needed of course (see here for installation: https://www.nagios.org/ncpa/#downloads).

> If you want to adapt the script to your own monitoring solution or to a cron job (very easy), I will not explain it but it's totally possible. This plugin is published with a public license (as all other plugins and scripts here), so feel free to fork or use it in your own scripts or programs!

### Usage

To test, use it like this:

```shell
./check_sftp -P <partition> -d <disk_usage_threshold> -i <check_interval> -p <sftp_port>
```

Full usage, options and explanations are accessible with the -h argument.

```shell
./check_sftp -h
```

To use it as NCPA plugin, create a command and an associated service in Nagios like this example:

```text
# Command
define command {
    command_name check_sftp
    command_line $USER1$/check_ncpa.py -H $HOSTADDRESS$ -t 'your_token' -M 'plugins/check_sftp.bash' -q 'args=-P /dev/sdb1,args=-d 85,args=-i 5,args=-n internal-sftp,args=-p 22'
}
```

```text
# Service
define service {
    use                     generic-service
    hostname                your-server ;(or hostgroup_name          servers-group for a group of servers !)
    service_description     SFTP health
    check_command           check_sftp
    contacts                mailadmin
    max_check_attempts      3
    check_interval          5
    retry_interval          1
    notification_interval   0
}
```

As always, check your nagios configuration:

```shell
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

Then restart Nagios service and that's all!

### Samples

Examples of status:
![alt text](https://github.com/John4887/check_sftp/blob/main/check-sftp_sample1.png)
![alt text](https://github.com/John4887/check_sftp/blob/main/check-sftp_sample2.png)
