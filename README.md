# extend Nagios by custom plugin in Hortonworks HDP
## checks
check hadoop-hosts.cfg if all nodes are included

check/verify hadoop-hostgroups.cfg and hadoop-servicegroups.cfg if the mappings are complete
## integrate your own plugin
copy your script to /usr/lib64/nagios/plugins/
```
cp <sourcedir>/<scriptname> /usr/lib64/nagios/plugins
chmod +x /usr/lib64/nagios/plugins/<scriptname>
```

add command to /etc/nagios/objects/hadoop-commands.cfg
```
define command{
       command_name    check_<whatever-your-check-does>
       command_line    $USER1$/<scriptname> $ARG1$ $ARG2$
       }
```

e.g. let's add the command check_hdfs_fsck which will at the end execute /usr/lib64/nagios/plugins/check_hadoop_hdfs_state (without further parameters)

```
define command{
        command_name    check_hdfs_fsck
        command_line    $USER1$/check_hadoop_hdfs_state
       }

```

add service to /etc/nagios/objects/hadoop-services.cfg

```
define service {
    host_name   <hostname>
    service_description <text description of your check>
    check_command   <command to be executed, parameters separated by "!">
    max_check_attempts  <no.of retries in case of non-ok state>
    check_interval  <interval between two checks in ok-state. Interval length = interval*interval_length(default 60s)>
    retry_interval  <interval between two checks in non-ok-state. interval base is defined by interval_length (default 60s)>
    [use         <service-template name>] #to preconfigure common settings
}
```

```
e.g. assign the previously defined command to a service check (corresponding to the example command from above):
define service {
  host_name               sandbox.hortonworks.com
  use                     hadoop-service
  service_description     check if HDFS is healthy
  servicegroups           HDFS
  check_command           check_hdfs_fsck
  _host_component         NAMENODE
  normal_check_interval   0.5
  retry_check_interval    0.5
  max_check_attempts      3
}
```

## restart nagios
```
/etc/init.d/nagios restart
```
