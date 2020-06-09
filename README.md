## setup node with centos/8 or rhel/8

#### vagrant alias
I use a zsh plugin "vagrant" (https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/vagrant)

to remove any ambiguity around what the alias might be doing, here is a list of vagrant commands:\
(Trey, don't be an animal. use the alias.). 
```
% alias | grep vagrant
vba='vagrant box add'
vbl='vagrant box list'
vbo='vagrant box outdated'
vbr='vagrant box remove'
vbu='vagrant box update'
vclos='vagrant up oob-mgmt-server oob-mgmt-switch leaf01 leaf02 leaf03 leaf04 spine01 spine02 server01 server02 server03 server04'
vd='vagrant destroy -f'
vdf='vagrant destroy -f'
vgi='vagrant init'
vgs='vagrant global-status'
vh='vagrant halt'
vminimal='vagrant up oob-mgmt-server oob-mgmt-switch leaf01'
voob='vagrant ssh oob-mgmt-server'
vp='vagrant push'
vpli='vagrant plugin install'
vpll='vagrant plugin list'
vplu='vagrant plugin update'
vplun='vagrant plugin uninstall'
vpr='vagrant provision'
vr='vagrant reload'
vrdp='vagrant rdp'
vre='vagrant resume'
vrp='vagrant reload --provision'
vsh='vagrant share'
vssh='vagrant ssh'
vsshc='vagrant ssh-config'
vssp='vagrant suspend'
vst='vagrant status'
vup='vagrant up'
```

#### provision / destroy 
launch two nodes\
`# vup node1 node2 --provider=libvirt`

destroy all nodes and remove any files\
`# vdf`

#### copy SSH public keys from VM1 to VM2 to allow key-based login
check VM is reachable from ansible control node\
```
% ansible -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory server -m ping                                                                       (allow_ssh_between_servers !?)
node2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
node1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```
fetch VM SSH public key\
```
% ansible -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory server -m fetch -a "src='/home/vagrant/.ssh/id_rsa.pub' dest='buffer/{{inventory_hostname}}-id_rsa.pub' flat='yes'" -b
node1 | CHANGED => {
    "changed": true,
    "checksum": "4e6c57f12e2a17efab747e97e8ff01d06e4b815a",
    "dest": "/home/markh/Git/rhcsa_sim/buffer/node1-id_rsa.pub",
    "md5sum": "3fd17b73762f4fdd24f5ced7ac19df65",
    "remote_checksum": "4e6c57f12e2a17efab747e97e8ff01d06e4b815a",
    "remote_md5sum": null
}
node2 | CHANGED => {
    "changed": true,
    "checksum": "cc65884815dc72fee361db97d4d230a0c7d6703f",
    "dest": "/home/markh/Git/rhcsa_sim/buffer/node2-id_rsa.pub",
    "md5sum": "147be802934a57382302c211089e39fb",
    "remote_checksum": "cc65884815dc72fee361db97d4d230a0c7d6703f",
    "remote_md5sum": null
}
```
copy SSH key from node1 to authorized_keys dir on node2\
```
% ansible -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory server -m authorized_key -a "user='vagrant' state='present' key='{{ lookup('file','buffer/node1-id_rsa.pub') }}'" --limit=node2 -b
node2 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": true,
    "comment": null,
    "exclusive": false,
    "follow": false,
    "key": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDha6Ia/KWeXLKOGBm2dpwz01IRoxV5PqGxSdnfbvGrZoxMD6tYohqjfypdMetCRaOEyxgYPzvm4ZUqXVQRefSfHR/GGq9gOCDvIHsx8kEfKNUKF4BDCktjSD1rGI2giC279SsPzFFFj0bwm2drcwuA+peWlX3aFV7Bmc+AxzlegVtToBulUAonHNjzSO6MKvHFOuCRJ/JfTdEEXryTFmpGuRRm/Vg6zH04YPZV9+X6HfPEKMOihiuks9e63QIBnos3Ogun32z2cmq0Tb+KQDFE7PpVs6gnHz5+Uku5YyUe/P8q64jOQYYirXzsH6utrpvrYdlMmYrxJf2vEeNlsFhuim0scdOsq9gzLw/qFAuAupIUC5CYvOf0ZmU7E9vbeXi6WyF0WMGU0YD1/m5MvYwTc70Dg5F81H9eNVKfPh7R9nq+t1Ix7njWKyibinvP+2lPjv+ieTRdgy4QeO2xsoFSP0herekBdn5fY7XNbpIGswujnVkRmv6NXqfb/Veq4pQXjGoL3wTJepCbDGVBbcN7FHC95jRBPb1DfQgv0xi7edw2Cn19HnFVLjmkTYRElMQUVRAWVrmXAFzL+kA7SGjan4HzeOJM6w1OSQrIBRzjQamsaXJaou8z7XbqGa2ooNsJF/gSVLR6Ppw04SBMJC+hVijjG4eTDjHuc9tD1XeAZQ== root@node1",
    "key_options": null,
    "keyfile": "/home/vagrant/.ssh/authorized_keys",
    "manage_dir": true,
    "path": null,
    "state": "present",
    "user": "vagrant",
    "validate_certs": true
}
```
copy SSH key from node2 to authorized_keys dir on node1\
```
% ansible -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory server -m authorized_key -a "user='vagrant' state='present' key='{{ lookup('file','buffer/node2-id_rsa.pub') }}'" --limit=node1 -b
```
verify key-based login works from node1 to node2\
```
% ansible -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory server -m shell -a "ssh -o StrictHostKeyChecking=no vagrant@node2 'uname -a'" --limit=node1
node1 | CHANGED | rc=0 >>
Linux node2 4.18.0-80.el8.x86_64 #1 SMP Tue Jun 4 09:19:46 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

#### access VM and basic connectivity
ssh to VM node1\
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
```
[vagrant@node1 ~]$ ssh-copy-id vagrant@node2
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
[vagrant@node2 ~]$ exit
logout
Connection to node2 closed.
```
