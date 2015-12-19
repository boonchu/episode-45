## Episode #45 - Learning Ansible with Vagrant (Part 2/4)

Code for [Episode #45 - Learning Ansible with Vagrant (Part 2/4)](https://sysadmincasts.com/episodes/45-learning-ansible-with-vagrant-part-2-4).

## update ssh-key in know-hosts

```
$ ssh-keyscan web1 lb web2 >> ~/.ssh/known_hosts
```

## run ansible 

```
$ ansible all -a "/bin/echo hello" -u vagrant --ask-pass
SSH password:
web1 | success | rc=0 >>
hello

lb | success | rc=0 >>
hello

web2 | success | rc=0 >>
hello
```

## add Key

```
[vagrant@mgmt ~]$ cat e45-ssh-addkey.yml
---
- hosts: all
  sudo: yes
  gather_facts: no
  remote_user: vagrant

  tasks:

  - name: install ssh key
    authorized_key: user=vagrant
                    key="{{ lookup('file', '/home/vagrant/.ssh/id_rsa.pub') }}"
                    state=present
[vagrant@mgmt ~]$ ls -lt .ssh
total 8
-rw-r--r-- 1 vagrant vagrant 2171 Dec 19 17:26 known_hosts
-rw------- 1 vagrant vagrant  951 Dec 19 17:08 authorized_keys
[vagrant@mgmt ~]$ ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/vagrant/.ssh/id_rsa.
Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub.
The key fingerprint is:
af:65:81:78:d7:1e:ea:4d:45:03:8e:48:7f:c6:ce:cf vagrant@mgmt
The key's randomart image is:
+--[ RSA 4096]----+
|        .   .    |
|       . o + .   |
|        . o = o  |
|       . . * . . |
|      . S o = .  |
|       . o + =   |
|          = o E  |
|         = o     |
|        . . .    |
+-----------------+
```

## run first playbook to drop in public key

```
[vagrant@mgmt ~]$ ansible-playbook e45-ssh-addkey.yml -u vagrant --ask-pass
SSH password:

PLAY [all] ********************************************************************

TASK: [install ssh key] *******************************************************
changed: [web1]
changed: [web2]
changed: [lb]

PLAY RECAP ********************************************************************
lb                         : ok=1    changed=1    unreachable=0    failed=0
web1                       : ok=1    changed=1    unreachable=0    failed=0
web2                       : ok=1    changed=1    unreachable=0    failed=0
```

## test without --ask-pass

```
[vagrant@mgmt ~]$ ansible all -a "/bin/echo hello" -u vagrant
web1 | success | rc=0 >>
hello

web2 | success | rc=0 >>
hello

lb | success | rc=0 >>
hello
```

## Ensure Package/Configure/Service (similar to Puppet package resource)

```
[vagrant@mgmt ~]$ ansible web1 -m yum -s -a "name=ntp state=installed"
web1 | success >> {
    "changed": true,
    "msg": "",
    "rc": 0,
    "results": [
        "Loaded plugins: fastestmirror\nLoading mirror speeds from cached hostfile\n * base: mirrors.kernel.org\n * extras: mirror.web-ster.com\n * updates: mirrors.kernel.org\nResolving Dependencies\n--> Running transaction check\n---> Package ntp.x86_64 0:4.2.6p5-22.el7.centos will be installed\n--> Processing Dependency: ntpdate = 4.2.6p5-22.el7.centos for package: ntp-4.2.6p5-22.el7.centos.x86_64\n--> Processing Dependency: libopts.so.25()(64bit) for package: ntp-4.2.6p5-22.el7.centos.x86_64\n--> Running transaction check\n---> Package autogen-libopts.x86_64 0:5.18-5.el7 will be installed\n---> Package ntpdate.x86_64 0:4.2.6p5-22.el7.centos will be installed\n--> Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package              Arch        Version                       Repository\n                                                                           Size\n================================================================================\nInstalling:\n ntp                  x86_64      4.2.6p5-22.el7.centos         base      543 k\nInstalling for dependencies:\n autogen-libopts      x86_64      5.18-5.el7                    base       66 k\n ntpdate              x86_64      4.2.6p5-22.el7.centos         base       84 k\n\nTransaction Summary\n================================================================================\nInstall  1 Package (+2 Dependent packages)\n\nTotal download size: 693 k\nInstalled size: 1.6 M\nDownloading packages:\n--------------------------------------------------------------------------------\nTotal                                              674 kB/s | 693 kB  00:01     \nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  Installing : ntpdate-4.2.6p5-22.el7.centos.x86_64                         1/3 \n  Installing : autogen-libopts-5.18-5.el7.x86_64                            2/3 \n  Installing : ntp-4.2.6p5-22.el7.centos.x86_64                             3/3 \n  Verifying  : ntp-4.2.6p5-22.el7.centos.x86_64                             1/3 \n  Verifying  : autogen-libopts-5.18-5.el7.x86_64                            2/3 \n  Verifying  : ntpdate-4.2.6p5-22.el7.centos.x86_64                         3/3 \n\nInstalled:\n  ntp.x86_64 0:4.2.6p5-22.el7.centos                                            \n\nDependency Installed:\n  autogen-libopts.x86_64 0:5.18-5.el7   ntpdate.x86_64 0:4.2.6p5-22.el7.centos  \n\nComplete!\n"
    ]
}

[vagrant@mgmt ~]$ ansible web1 -m copy -a "src=/home/vagrant/files/ntp.conf dest=/etc/ntp.conf mode=644 owner=root group=root" --sudo
web1 | success >> {
    "changed": true,
    "checksum": "f1f51d84bd084c9acbc1a1827b70860db2117ae4",
    "dest": "/etc/ntp.conf",
    "gid": 0,
    "group": "root",
    "md5sum": "5b7b1e1e54f33c6948335335ab03f423",
    "mode": "0644",
    "owner": "root",
    "size": 504,
    "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1450548877.85-78035864742501/source",
    "state": "file",
    "uid": 0
}

[vagrant@mgmt ~]$ ansible web1 -m service -a "name=ntpd state=restarted" --sudo
web1 | success >> {
    "changed": true,
    "name": "ntpd",
    "state": "started"
}
```

## Node Operational Status

```
[vagrant@mgmt ~]$ ansible all -m shell -a 'uptime'
web1 | success | rc=0 >>
 18:18:09 up  1:08,  1 user,  load average: 0.01, 0.05, 0.05

lb | success | rc=0 >>
 18:18:10 up  1:09,  1 user,  load average: 0.00, 0.01, 0.04

web2 | success | rc=0 >>
 18:18:09 up  1:08,  1 user,  load average: 0.00, 0.01, 0.04
```

## Second Playbook NTP

```
[vagrant@mgmt ~]$ ansible-playbook e45-ntp-install.yml

PLAY [all] ********************************************************************

TASK: [install ntp] ***********************************************************
ok: [web1]
ok: [lb]
ok: [web2]

TASK: [write our ntp.conf] ****************************************************
ok: [web2]
ok: [web1]
ok: [lb]

TASK: [start ntpd] ************************************************************
ok: [web2]
ok: [web1]
ok: [lb]

PLAY RECAP ********************************************************************
lb                         : ok=3    changed=0    unreachable=0    failed=0
web1                       : ok=3    changed=0    unreachable=0    failed=0
web2                       : ok=3    changed=0    unreachable=0    failed=0

[vagrant@mgmt ~]$ ansible all -a "/usr/sbin/ntpq -p" -u vagrant
web1 | success | rc=0 >>
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*time-b.timefreq .ACTS.           1 u   10   64    3   38.729  -32.222  32.514
+100tx-f1-0.c720 216.218.192.202  2 u   10   64    3   21.660  -37.717  30.797
+time-c.nist.gov .ACTS.           1 u   12   64    3  151.892    1.395  21.770
+disorder.primat 204.123.2.72     2 u   14   64    3   14.823  -32.142  26.618
-golem.canonical 193.79.237.14    2 u    9   64    3  153.666  -32.824  24.327

web2 | success | rc=0 >>
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
-four10.gac.edu  18.26.4.105      2 u   44   64  377   61.699  -111.65  30.415
+vps3.cobryce.co 216.218.254.202  2 u   38   64  377   18.173  -72.699  17.997
*216.152.240.220 164.67.62.194    2 u   41   64  377   21.148  -78.027  13.265
+ntp-sansome.moc 128.101.101.101  3 u   38   64  377   12.808  -108.22  29.377
+golem.canonical 193.79.237.14    2 u   40   64  377  162.687  -75.327  45.259

lb | success | rc=0 >>
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
+pacific.latt.ne 44.24.199.34     3 u   28   64  377   28.342  -247.31 105.580
+resolver2.level 10.67.8.18       3 u   38   64  377   10.816  -248.85 106.958
+dragon506.start 17.253.34.253    2 u   42   64  375   62.149  -245.56 141.120
*hydrogen.consta 200.98.196.212   2 u   40   64  377   88.204  -240.64 107.229
+juniperberry.ca 140.203.204.77   2 u   41   64  377  149.864  -242.36 104.856
```

## Third playbook : Remove NTP service

```
[vagrant@mgmt ~]$ ansible-playbook e45-ntp-remove.yml

PLAY [all] ********************************************************************

TASK: [remove ntp] ************************************************************
changed: [lb]
changed: [web2]
changed: [web1]

PLAY RECAP ********************************************************************
lb                         : ok=1    changed=1    unreachable=0    failed=0
web1                       : ok=1    changed=1    unreachable=0    failed=0
web2                       : ok=1    changed=1    unreachable=0    failed=0
```

## Ansible facts

```
[vagrant@mgmt ~]$ ansible web1 -m setup

[vagrant@mgmt ~]$ ansible all -m setup -a 'filter=ansible_product_name'
web1 | success >> {
    "ansible_facts": {
        "ansible_product_name": "VirtualBox"
    },
    "changed": false
}

web2 | success >> {
    "ansible_facts": {
        "ansible_product_name": "VirtualBox"
    },
    "changed": false
}

lb | success >> {
    "ansible_facts": {
        "ansible_product_name": "VirtualBox"
    },
    "changed": false
}

```

## Note A: Vagrant User setup

  * It is nessessary for first time to run ping/pong and have 'ask-pass' with vagrant user
  * folk might be bumping into the issue with vagrant user - https://github.com/puphpet/puphpet/issues/1253
  * I ended up using echo 'vagrant' | passwd --stdin vagrant to reset 'vagrant' user's password

```

  # default VM settings
  config.ssh.username = 'vagrant'
  config.ssh.insert_key = 'true'

$ vagrant ssh-config mgmt
Host mgmt
  HostName 127.0.0.1
  User vagrant
  Port 2200
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /Users/bngampairoijpibul/Documents/src/ansible/episode-45/.vagrant/machines/mgmt/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL

```

## Note B: Its much improving security when use sftp (if not, use flag to disable on ansible.cfg)

  * posted in this chat 
    - http://stackoverflow.com/questions/23899028/ansible-failed-to-transfer-file-to
  * how to setup ansible control-path in sftp
    - http://docs.ansible.com/ansible/intro_configuration.html#control-path
