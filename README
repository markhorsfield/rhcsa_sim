setup node with centos/8 or rhel/8

launch two nodes
# vagrant up node1 node2 --provider=libvirt

destroy all nodes and remove any files
# vagrant destroy -f

ssh to VM node1
ignore the rubygems log messages here. it's a cosmetic issue in Ubuntu 20.04. no functional impact from what I have seen. details at https://bugs.launchpad.net/ubuntu/+source/vagrant/+bug/1871685 
```
% vssh node1                                                                                                                                                                     (master ?)
/usr/share/rubygems-integration/all/gems/vagrant-2.2.6/plugins/kernel_v2/config/vm.rb:354: warning: Using the last argument as keyword parameters is deprecated; maybe ** should be added to the call
/usr/share/rubygems-integration/all/gems/vagrant-2.2.6/plugins/kernel_v2/config/vm_provisioner.rb:92: warning: The called method `add_config' is defined here
/usr/share/rubygems-integration/all/gems/vagrant-2.2.6/lib/vagrant/errors.rb:103: warning: Using the last argument as keyword parameters is deprecated; maybe ** should be added to the call
/usr/share/rubygems-integration/all/gems/i18n-1.8.2/lib/i18n.rb:195: warning: The called method `t' is defined here
Last login: Fri May 22 10:39:27 2020 from 192.168.121.1
[vagrant@node1 ~]$ ping node2
PING node2 (192.168.121.202) 56(84) bytes of data.
64 bytes from node2 (192.168.121.202): icmp_seq=1 ttl=64 time=0.633 ms
64 bytes from node2 (192.168.121.202): icmp_seq=2 ttl=64 time=0.419 ms
^C
--- node2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 2ms
rtt min/avg/max/mdev = 0.419/0.526/0.633/0.107 ms
```
copy ssh public key to remote node (need to automate this)
```[vagrant@node1 ~]$ ssh-copy-id vagrant@node2
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
The authenticity of host 'node2 (192.168.121.202)' can't be established.
ECDSA key fingerprint is SHA256:dEtNKBQ8J4wY0I4N+b3sk4eG+En6Y+Hl+Yn1iA0qiOs.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
vagrant@node2's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'vagrant@node2'"
and check to make sure that only the key(s) you wanted were added.
```
test key based authentication
```
[vagrant@node1 ~]$ ssh vagrant@node2
Last login: Fri May 22 10:39:25 2020 from 192.168.121.1
```
