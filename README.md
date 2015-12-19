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

## Note A: Vagrant User setup

  * folk might be bumping into the issue with vagrant user
    - https://github.com/puphpet/puphpet/issues/1253

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
