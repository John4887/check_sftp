This Nagios plugin is able to get the health state of a SFTP service.

Functionalities :

- Disk usage or dedicated partition disk usage for sftp service ;
- SFTP access availability (supports also SFTP service on non-default port) ;
- SFTP active connections ;
- Connected users using SFTP.

Use it like this:

```shell
./check_sftp -P <partition> -d <disk_usage_threshold> -i <check_interval> -p <sftp_port>
```
