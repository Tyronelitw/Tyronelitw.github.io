---
title: ansible安装和基本使用
date: 2018-04-08 17:31:37
categories: 
- 技术
tags: 
- 技术 
- ansible 
- 自动化 
- 文档 
---
## 安装配置

# 1、安装
```bash
[root@litw-docker ~]# yum install ansible -y
```
# 2、配置秘钥免密认证
```bash
[root@litw-docker ~]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
63:0f:67:df:2e:9d:3c:9b:41:b5:f4:85:03:ad:db:78 root@litw-docker.novalocal
The key's randomart image is:
+--[ RSA 2048]----+
|            ..   |
|             ... |
|             .o.o|
|            . .o+|
|        S o  +...|
|       . * .ooE  |
|          . .+o. |
|            ..=o |
|             .+o |
+-----------------+

[root@litw-docker ~]# ssh-copy-id root@192.168.101.6
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.101.6's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@192.168.101.6'"
and check to make sure that only the key(s) you wanted were added.

[root@litw-docker ~]# ssh-copy-id root@192.168.101.11
The authenticity of host '192.168.101.11 (192.168.101.11)' can't be established.
ECDSA key fingerprint is dd:a8:4a:aa:a3:09:92:50:50:53:e8:de:76:83:0c:05.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.101.11's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@192.168.101.11'"
and check to make sure that only the key(s) you wanted were added.
```
<!--read more-->

# 3、修改ansible配置hosts文件
```bash
vim /etc/ansible/hosts
[test]
192.168.101.6
192.168.101.11
```
## ansible基本使用（模块）

# 1、ping
```bash
[root@litw-docker ~]# ansible test -m ping
192.168.101.6 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
192.168.101.11 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```
# 2、shell/command
```bash
[root@litw-docker ~]# ansible test -m shell -a "ps aux | grep -c sshd"
192.168.101.6 | SUCCESS | rc=0 >>
5

192.168.101.11 | SUCCESS | rc=0 >>
5

[root@litw-docker ~]# ansible test -m command -a "ps aux | grep -c sshd"
192.168.101.6 | FAILED | rc=1 >>
error: garbage option

Usage:
 ps [options]

 Try 'ps --help <simple|list|output|threads|misc|all>'
  or 'ps --help <s|l|o|t|m|a>'
 for additional help text.

For more details see ps(1).non-zero return code

192.168.101.11 | FAILED | rc=1 >>
error: garbage option

Usage:
 ps [options]

 Try 'ps --help <simple|list|output|threads|misc|all>'
  or 'ps --help <s|l|o|t|m|a>'
 for additional help text.

For more details see ps(1).non-zero return code

[root@litw-docker ~]# ansible test -m command -a "uptime"
192.168.101.6 | SUCCESS | rc=0 >>
 17:48:14 up 11 days, 54 min,  2 users,  load average: 0.07, 0.04, 0.05

192.168.101.11 | SUCCESS | rc=0 >>
 17:48:14 up 11 days, 54 min,  2 users,  load average: 0.00, 0.01, 0.05
```
可以看到command和shell模块最明显的区别是shell执行命令可以使用管道，command不可以。

# 3、copy
```bash
[root@litw-docker ~]# ansible test -m copy -a "src=/root/ip.txt dest=/tmp/test owner=root group=root mode=600"
192.168.101.11 | SUCCESS => {
    "changed": true, 
    "checksum": "1ec0d33dd7cfee811a9d0fc833d8d0b861084c95", 
    "dest": "/tmp/test", 
    "gid": 0, 
    "group": "root", 
    "md5sum": "885a3db6120c0857ecd8a5e71a3c8400", 
    "mode": "0600", 
    "owner": "root", 
    "size": 57, 
    "src": "/root/.ansible/tmp/ansible-tmp-1523181044.35-147146951272504/source", 
    "state": "file", 
    "uid": 0
}
192.168.101.6 | SUCCESS => {
    "changed": true, 
    "checksum": "1ec0d33dd7cfee811a9d0fc833d8d0b861084c95", 
    "dest": "/tmp/test", 
    "gid": 0, 
    "group": "root", 
    "md5sum": "885a3db6120c0857ecd8a5e71a3c8400", 
    "mode": "0600", 
    "owner": "root", 
    "size": 57, 
    "src": "/root/.ansible/tmp/ansible-tmp-1523181044.34-260315740643744/source", 
    "state": "file", 
    "uid": 0
}
```
# 4、service
```bash
[root@litw-docker ~]# ansible test -m service -a "name=sshd state=started"
192.168.101.11 | SUCCESS => {
    "changed": false, 
    "name": "sshd", 
    "state": "started", 
    "status": {
        "ActiveEnterTimestamp": "Wed 2018-03-28 16:54:21 CST", 
        "ActiveEnterTimestampMonotonic": "39061736", 
        "ActiveExitTimestamp": "Wed 2018-03-28 16:54:21 CST", 
        "ActiveExitTimestampMonotonic": "39051401", 
        "ActiveState": "active", 
        "After": "network.target system.slice systemd-journald.socket syslog.target auditd.service basic.target cloud-init.service", 
        "AllowIsolate": "no", 
        "AmbientCapabilities": "0", 
        "AssertResult": "yes", 
        "AssertTimestamp": "Wed 2018-03-28 16:54:21 CST", 
        "AssertTimestampMonotonic": "39053249", 
        "Before": "multi-user.target shutdown.target", 
        "BlockIOAccounting": "no", 
        "BlockIOWeight": "18446744073709551615", 
        "CPUAccounting": "no", 
        "CPUQuotaPerSecUSec": "infinity", 
        "CPUSchedulingPolicy": "0", 
        "CPUSchedulingPriority": "0", 
        "CPUSchedulingResetOnFork": "no", 
        "CPUShares": "18446744073709551615", 
        "CanIsolate": "no", 
        "CanReload": "yes", 
        "CanStart": "yes", 
        "CanStop": "yes", 
        "CapabilityBoundingSet": "18446744073709551615", 
        "ConditionResult": "yes", 
        "ConditionTimestamp": "Wed 2018-03-28 16:54:21 CST", 
        "ConditionTimestampMonotonic": "39053249", 
        "Conflicts": "shutdown.target", 
        "ControlGroup": "/system.slice/sshd.service", 
        "ControlPID": "0", 
        "DefaultDependencies": "yes", 
        "Delegate": "no", 
        "Description": "OpenSSH server daemon", 
        "DevicePolicy": "auto", 
        "EnvironmentFile": "/etc/sysconfig/sshd (ignore_errors=no)", 
        "ExecMainCode": "0", 
        "ExecMainExitTimestampMonotonic": "0", 
        "ExecMainPID": "946", 
        "ExecMainStartTimestamp": "Wed 2018-03-28 16:54:21 CST", 
        "ExecMainStartTimestampMonotonic": "39061689", 
        "ExecMainStatus": "0", 
        "ExecReload": "{ path=/bin/kill ; argv[]=/bin/kill -HUP $MAINPID ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }", 
        "ExecStart": "{ path=/usr/sbin/sshd ; argv[]=/usr/sbin/sshd -D $OPTIONS ; ignore_errors=no ; start_time=[Wed 2018-03-28 16:54:21 CST] ; stop_time=[n/a] ; pid=946 ; code=(null) ; status=0/0 }", 
        "ExecStartPre": "{ path=/usr/sbin/sshd-keygen ; argv[]=/usr/sbin/sshd-keygen ; ignore_errors=no ; start_time=[Wed 2018-03-28 16:54:21 CST] ; stop_time=[Wed 2018-03-28 16:54:21 CST] ; pid=943 ; code=exited ; status=0 }", 
        "FailureAction": "none", 
        "FileDescriptorStoreMax": "0", 
        "FragmentPath": "/usr/lib/systemd/system/sshd.service", 
        "GuessMainPID": "yes", 
        "IOScheduling": "0", 
        "Id": "sshd.service", 
        "IgnoreOnIsolate": "no", 
        "IgnoreOnSnapshot": "no", 
        "IgnoreSIGPIPE": "yes", 
        "InactiveEnterTimestamp": "Wed 2018-03-28 16:54:21 CST", 
        "InactiveEnterTimestampMonotonic": "39052998", 
        "InactiveExitTimestamp": "Wed 2018-03-28 16:54:21 CST", 
        "InactiveExitTimestampMonotonic": "39053607", 
        "JobTimeoutAction": "none", 
        "JobTimeoutUSec": "0", 
        "KillMode": "process", 
        "KillSignal": "15", 
        "LimitAS": "18446744073709551615", 
        "LimitCORE": "18446744073709551615", 
        "LimitCPU": "18446744073709551615", 
        "LimitDATA": "18446744073709551615", 
        "LimitFSIZE": "18446744073709551615", 
        "LimitLOCKS": "18446744073709551615", 
        "LimitMEMLOCK": "65536", 
        "LimitMSGQUEUE": "819200", 
        "LimitNICE": "0", 
        "LimitNOFILE": "4096", 
        "LimitNPROC": "14644", 
        "LimitRSS": "18446744073709551615", 
        "LimitRTPRIO": "0", 
        "LimitRTTIME": "18446744073709551615", 
        "LimitSIGPENDING": "14644", 
        "LimitSTACK": "18446744073709551615", 
        "LoadState": "loaded", 
        "MainPID": "946", 
        "MemoryAccounting": "no", 
        "MemoryCurrent": "18446744073709551615", 
        "MemoryLimit": "18446744073709551615", 
        "MountFlags": "0", 
        "Names": "sshd.service", 
        "NeedDaemonReload": "no", 
        "Nice": "0", 
        "NoNewPrivileges": "no", 
        "NonBlocking": "no", 
        "NotifyAccess": "none", 
        "OOMScoreAdjust": "0", 
        "OnFailureJobMode": "replace", 
        "PermissionsStartOnly": "no", 
        "PrivateDevices": "no", 
        "PrivateNetwork": "no", 
        "PrivateTmp": "no", 
        "ProtectHome": "no", 
        "ProtectSystem": "no", 
        "RefuseManualStart": "no", 
        "RefuseManualStop": "no", 
        "RemainAfterExit": "no", 
        "Requires": "basic.target", 
        "Restart": "on-failure", 
        "RestartUSec": "42s", 
        "Result": "success", 
        "RootDirectoryStartOnly": "no", 
        "RuntimeDirectoryMode": "0755", 
        "SameProcessGroup": "no", 
        "SecureBits": "0", 
        "SendSIGHUP": "no", 
        "SendSIGKILL": "yes", 
        "Slice": "system.slice", 
        "StandardError": "inherit", 
        "StandardInput": "null", 
        "StandardOutput": "journal", 
        "StartLimitAction": "none", 
        "StartLimitBurst": "5", 
        "StartLimitInterval": "10000000", 
        "StartupBlockIOWeight": "18446744073709551615", 
        "StartupCPUShares": "18446744073709551615", 
        "StatusErrno": "0", 
        "StopWhenUnneeded": "no", 
        "SubState": "running", 
        "SyslogLevelPrefix": "yes", 
        "SyslogPriority": "30", 
        "SystemCallErrorNumber": "0", 
        "TTYReset": "no", 
        "TTYVHangup": "no", 
        "TTYVTDisallocate": "no", 
        "TasksAccounting": "no", 
        "TasksCurrent": "18446744073709551615", 
        "TasksMax": "18446744073709551615", 
        "TimeoutStartUSec": "1min 30s", 
        "TimeoutStopUSec": "1min 30s", 
        "TimerSlackNSec": "50000", 
        "Transient": "no", 
        "Type": "simple", 
        "UMask": "0022", 
        "UnitFilePreset": "enabled", 
        "UnitFileState": "enabled", 
        "WantedBy": "cloud-init.service multi-user.target", 
        "Wants": "system.slice", 
        "WatchdogTimestamp": "Wed 2018-03-28 16:54:21 CST", 
        "WatchdogTimestampMonotonic": "39061722", 
        "WatchdogUSec": "0"
    }
}
192.168.101.6 | SUCCESS => {
    "changed": false, 
    "name": "sshd", 
    "state": "started", 
    "status": {
        "ActiveEnterTimestamp": "Wed 2018-03-28 16:54:25 CST", 
        "ActiveEnterTimestampMonotonic": "30808691", 
        "ActiveExitTimestamp": "Wed 2018-03-28 16:54:25 CST", 
        "ActiveExitTimestampMonotonic": "30789169", 
        "ActiveState": "active", 
        "After": "cloud-init.service basic.target network.target syslog.target systemd-journald.socket auditd.service system.slice", 
        "AllowIsolate": "no", 
        "AmbientCapabilities": "0", 
        "AssertResult": "yes", 
        "AssertTimestamp": "Wed 2018-03-28 16:54:25 CST", 
        "AssertTimestampMonotonic": "30792622", 
        "Before": "shutdown.target multi-user.target", 
        "BlockIOAccounting": "no", 
        "BlockIOWeight": "18446744073709551615", 
        "CPUAccounting": "no", 
        "CPUQuotaPerSecUSec": "infinity", 
        "CPUSchedulingPolicy": "0", 
        "CPUSchedulingPriority": "0", 
        "CPUSchedulingResetOnFork": "no", 
        "CPUShares": "18446744073709551615", 
        "CanIsolate": "no", 
        "CanReload": "yes", 
        "CanStart": "yes", 
        "CanStop": "yes", 
        "CapabilityBoundingSet": "18446744073709551615", 
        "ConditionResult": "yes", 
        "ConditionTimestamp": "Wed 2018-03-28 16:54:25 CST", 
        "ConditionTimestampMonotonic": "30792621", 
        "Conflicts": "shutdown.target", 
        "ControlGroup": "/system.slice/sshd.service", 
        "ControlPID": "0", 
        "DefaultDependencies": "yes", 
        "Delegate": "no", 
        "Description": "OpenSSH server daemon", 
        "DevicePolicy": "auto", 
        "EnvironmentFile": "/etc/sysconfig/sshd (ignore_errors=no)", 
        "ExecMainCode": "0", 
        "ExecMainExitTimestampMonotonic": "0", 
        "ExecMainPID": "944", 
        "ExecMainStartTimestamp": "Wed 2018-03-28 16:54:25 CST", 
        "ExecMainStartTimestampMonotonic": "30808580", 
        "ExecMainStatus": "0", 
        "ExecReload": "{ path=/bin/kill ; argv[]=/bin/kill -HUP $MAINPID ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }", 
        "ExecStart": "{ path=/usr/sbin/sshd ; argv[]=/usr/sbin/sshd -D $OPTIONS ; ignore_errors=no ; start_time=[Wed 2018-03-28 16:54:25 CST] ; stop_time=[n/a] ; pid=944 ; code=(null) ; status=0/0 }", 
        "ExecStartPre": "{ path=/usr/sbin/sshd-keygen ; argv[]=/usr/sbin/sshd-keygen ; ignore_errors=no ; start_time=[Wed 2018-03-28 16:54:25 CST] ; stop_time=[Wed 2018-03-28 16:54:25 CST] ; pid=941 ; code=exited ; status=0 }", 
        "FailureAction": "none", 
        "FileDescriptorStoreMax": "0", 
        "FragmentPath": "/usr/lib/systemd/system/sshd.service", 
        "GuessMainPID": "yes", 
        "IOScheduling": "0", 
        "Id": "sshd.service", 
        "IgnoreOnIsolate": "no", 
        "IgnoreOnSnapshot": "no", 
        "IgnoreSIGPIPE": "yes", 
        "InactiveEnterTimestamp": "Wed 2018-03-28 16:54:25 CST", 
        "InactiveEnterTimestampMonotonic": "30792148", 
        "InactiveExitTimestamp": "Wed 2018-03-28 16:54:25 CST", 
        "InactiveExitTimestampMonotonic": "30793292", 
        "JobTimeoutAction": "none", 
        "JobTimeoutUSec": "0", 
        "KillMode": "process", 
        "KillSignal": "15", 
        "LimitAS": "18446744073709551615", 
        "LimitCORE": "18446744073709551615", 
        "LimitCPU": "18446744073709551615", 
        "LimitDATA": "18446744073709551615", 
        "LimitFSIZE": "18446744073709551615", 
        "LimitLOCKS": "18446744073709551615", 
        "LimitMEMLOCK": "65536", 
        "LimitMSGQUEUE": "819200", 
        "LimitNICE": "0", 
        "LimitNOFILE": "4096", 
        "LimitNPROC": "14644", 
        "LimitRSS": "18446744073709551615", 
        "LimitRTPRIO": "0", 
        "LimitRTTIME": "18446744073709551615", 
        "LimitSIGPENDING": "14644", 
        "LimitSTACK": "18446744073709551615", 
        "LoadState": "loaded", 
        "MainPID": "944", 
        "MemoryAccounting": "no", 
        "MemoryCurrent": "18446744073709551615", 
        "MemoryLimit": "18446744073709551615", 
        "MountFlags": "0", 
        "Names": "sshd.service", 
        "NeedDaemonReload": "no", 
        "Nice": "0", 
        "NoNewPrivileges": "no", 
        "NonBlocking": "no", 
        "NotifyAccess": "none", 
        "OOMScoreAdjust": "0", 
        "OnFailureJobMode": "replace", 
        "PermissionsStartOnly": "no", 
        "PrivateDevices": "no", 
        "PrivateNetwork": "no", 
        "PrivateTmp": "no", 
        "ProtectHome": "no", 
        "ProtectSystem": "no", 
        "RefuseManualStart": "no", 
        "RefuseManualStop": "no", 
        "RemainAfterExit": "no", 
        "Requires": "basic.target", 
        "Restart": "on-failure", 
        "RestartUSec": "42s", 
        "Result": "success", 
        "RootDirectoryStartOnly": "no", 
        "RuntimeDirectoryMode": "0755", 
        "SameProcessGroup": "no", 
        "SecureBits": "0", 
        "SendSIGHUP": "no", 
        "SendSIGKILL": "yes", 
        "Slice": "system.slice", 
        "StandardError": "inherit", 
        "StandardInput": "null", 
        "StandardOutput": "journal", 
        "StartLimitAction": "none", 
        "StartLimitBurst": "5", 
        "StartLimitInterval": "10000000", 
        "StartupBlockIOWeight": "18446744073709551615", 
        "StartupCPUShares": "18446744073709551615", 
        "StatusErrno": "0", 
        "StopWhenUnneeded": "no", 
        "SubState": "running", 
        "SyslogLevelPrefix": "yes", 
        "SyslogPriority": "30", 
        "SystemCallErrorNumber": "0", 
        "TTYReset": "no", 
        "TTYVHangup": "no", 
        "TTYVTDisallocate": "no", 
        "TasksAccounting": "no", 
        "TasksCurrent": "18446744073709551615", 
        "TasksMax": "18446744073709551615", 
        "TimeoutStartUSec": "1min 30s", 
        "TimeoutStopUSec": "1min 30s", 
        "TimerSlackNSec": "50000", 
        "Transient": "no", 
        "Type": "simple", 
        "UMask": "0022", 
        "UnitFilePreset": "enabled", 
        "UnitFileState": "enabled", 
        "WantedBy": "multi-user.target cloud-init.service", 
        "Wants": "system.slice", 
        "WatchdogTimestamp": "Wed 2018-03-28 16:54:25 CST", 
        "WatchdogTimestampMonotonic": "30808640", 
        "WatchdogUSec": "0"
    }
}

```
# 5、yum
```bash
[root@litw-docker ~]# ansible test -m yum -a "name=vim state=present"
192.168.101.6 | SUCCESS => {
    "changed": true, 
    "msg": "", 
    "rc": 0, 
    "results": [
        "Loaded plugins: fastestmirror\nLoading mirror speeds from cached hostfile\n * base: mirrors.aliyun.com\n * extras: mirrors.aliyun.com\n * updates: mirrors.cn99.com\nResolving Dependencies\n--> Running transaction check\n---> Package vim-enhanced.x86_64 2:7.4.160-2.el7 will be installed\n-->

内容太多，只摘取部分。
```
# 6、script
```bash
[root@litw-docker ~]# ansible test -m script -a "/root/ping.sh"
192.168.101.6 | SUCCESS => {
    "changed": true, 
    "rc": 0, 
    "stderr": "Shared connection to 192.168.101.6 closed.\r\n", 
    "stdout": "PING 192.168.101.1 (192.168.101.1) 56(84) bytes of data.\r\n64 bytes from 192.168.101.1: icmp_seq=1 ttl=64 time=1.20 ms\r\n64 bytes from 192.168.101.1: icmp_seq=2 ttl=64 time=0.383 ms\r\n64 bytes from 192.168.101.1: icmp_seq=3 ttl=64 time=0.359 ms\r\n64 bytes from 192.168.101.1: icmp_seq=4 ttl=64 time=0.388 ms\r\n64 bytes from 192.168.101.1: icmp_seq=5 ttl=64 time=1.48 ms\r\n\r\n--- 192.168.101.1 ping statistics ---\r\n5 packets transmitted, 5 received, 0% packet loss, time 4001ms\r\nrtt min/avg/max/mdev = 0.359/0.764/1.488/0.485 ms\r\n", 
    "stdout_lines": [
        "PING 192.168.101.1 (192.168.101.1) 56(84) bytes of data.", 
        "64 bytes from 192.168.101.1: icmp_seq=1 ttl=64 time=1.20 ms", 
        "64 bytes from 192.168.101.1: icmp_seq=2 ttl=64 time=0.383 ms", 
        "64 bytes from 192.168.101.1: icmp_seq=3 ttl=64 time=0.359 ms", 
        "64 bytes from 192.168.101.1: icmp_seq=4 ttl=64 time=0.388 ms", 
        "64 bytes from 192.168.101.1: icmp_seq=5 ttl=64 time=1.48 ms", 
        "", 
        "--- 192.168.101.1 ping statistics ---", 
        "5 packets transmitted, 5 received, 0% packet loss, time 4001ms", 
        "rtt min/avg/max/mdev = 0.359/0.764/1.488/0.485 ms"
    ]
}
192.168.101.11 | SUCCESS => {
    "changed": true, 
    "rc": 0, 
    "stderr": "Shared connection to 192.168.101.11 closed.\r\n", 
    "stdout": "PING 192.168.101.1 (192.168.101.1) 56(84) bytes of data.\r\n64 bytes from 192.168.101.1: icmp_seq=1 ttl=64 time=0.810 ms\r\n64 bytes from 192.168.101.1: icmp_seq=2 ttl=64 time=0.517 ms\r\n64 bytes from 192.168.101.1: icmp_seq=3 ttl=64 time=0.544 ms\r\n64 bytes from 192.168.101.1: icmp_seq=4 ttl=64 time=0.563 ms\r\n64 bytes from 192.168.101.1: icmp_seq=5 ttl=64 time=1.59 ms\r\n\r\n--- 192.168.101.1 ping statistics ---\r\n5 packets transmitted, 5 received, 0% packet loss, time 4000ms\r\nrtt min/avg/max/mdev = 0.517/0.806/1.599/0.411 ms\r\n", 
    "stdout_lines": [
        "PING 192.168.101.1 (192.168.101.1) 56(84) bytes of data.", 
        "64 bytes from 192.168.101.1: icmp_seq=1 ttl=64 time=0.810 ms", 
        "64 bytes from 192.168.101.1: icmp_seq=2 ttl=64 time=0.517 ms", 
        "64 bytes from 192.168.101.1: icmp_seq=3 ttl=64 time=0.544 ms", 
        "64 bytes from 192.168.101.1: icmp_seq=4 ttl=64 time=0.563 ms", 
        "64 bytes from 192.168.101.1: icmp_seq=5 ttl=64 time=1.59 ms", 
        "", 
        "--- 192.168.101.1 ping statistics ---", 
        "5 packets transmitted, 5 received, 0% packet loss, time 4000ms", 
        "rtt min/avg/max/mdev = 0.517/0.806/1.599/0.411 ms"
    ]
}
```
