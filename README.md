Ansible Postgresql
=========

Requirements
------------
no requirements

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:
```yaml

```

## How is installed in default

Restic bin    /opt/restic/                           <restic_download_path>
                          resitc_version/
                                         0.7
                                         0.8
                                         0.9
                                         1.1
                                         1.2/restic

resitc bin    /usr/local/bin/resitc --> /opt/restic/resitc_version/1.2/resitc

restic script /etc/restic/                           <restic_configuration_path>
                         script
                         config

License
-------

MIT

## Devel
for devel use
https://github.com/ansible-community/molecule-libvirt

requirements for testings
KVM host
```
libvirt-python
```
# ansible-restic
