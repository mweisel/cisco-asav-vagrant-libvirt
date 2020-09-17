A procedure for creating a Cisco ASAv Vagrant box for the [libvirt](https://libvirt.org) provider.

## Prerequisites

  * [Git](https://git-scm.com)
  * [Python](https://www.python.org)
  * [Ansible](https://docs.ansible.com/ansible/latest/index.html)
  * [libvirt](https://libvirt.org) with client tools
  * [QEMU](https://www.qemu.org)
  * [Vagrant](https://www.vagrantup.com) <= 2.2.9
  * [vagrant-libvirt](https://github.com/vagrant-libvirt/vagrant-libvirt)

## Steps

0. Verify the prerequisite tools are installed.

```
which git python ansible libvirtd virsh qemu-system-x86_64 vagrant
vagrant plugin list
```

1. Install the `genisoimage` tool.

> Arch Linux
```
sudo pacman -S cdrtools
```

> Ubuntu 18.04
```
sudo apt update && sudo apt install genisoimage
```

2. Clone this GitHub repo and _cd_ into the directory.

```
git clone https://github.com/mweisel/cisco-asav-vagrant-libvirt
cd cisco-asav-vagrant-libvirt
```

3. Log in and download the [Cisco Adaptive Security Virtual Appliance qcow2 package for the Cisco ASAv Virtual Firewall](https://software.cisco.com/download/home/286119613/type).

4. Copy (and rename) the disk image file to the `/var/lib/libvirt/images` directory.

```
sudo cp $HOME/Downloads/asav992-61.qcow2 /var/lib/libvirt/images/cisco-asav.qcow2
```

5. Modify the file ownership and permissions. Note the owner will differ between Linux distributions.

> Arch Linux
```
sudo chown nobody:kvm /var/lib/libvirt/images/cisco-asav.qcow2
sudo chmod u+x /var/lib/libvirt/images/cisco-asav.qcow2
```

> Ubuntu 18.04
```
sudo chown libvirt-qemu:kvm /var/lib/libvirt/images/cisco-asav.qcow2
sudo chmod u+x /var/lib/libvirt/images/cisco-asav.qcow2
```

6. Create the `day0.iso` file. The file provides the initial configuration for the Cisco ASAv. It will be mounted and read on first boot.

```
cd files
genisoimage -r -o day0.iso day0-config
```

7. Copy the `day0.iso` file to the `/var/lib/libvirt/images` directory.

```
sudo cp day0.iso /var/lib/libvirt/images/
```

8. Modify the file ownership and permissions.

> Arch Linux
```
sudo chown nobody:kvm /var/lib/libvirt/images/day0.iso
sudo chmod u+x /var/lib/libvirt/images/day0.iso
```

> Ubuntu 18.04
```
sudo chown libvirt-qemu:kvm /var/lib/libvirt/images/day0.iso
sudo chmod u+x /var/lib/libvirt/images/day0.iso
```

9. Start the `vagrant-libvirt` network (if not already started).

```
virsh -c qemu:///system net-list
virsh -c qemu:///system net-start vagrant-libvirt
```

10. Run the Ansible playbook. 

```
cd ..
ansible-playbook main.yml
```

11. Add the Vagrant box. 

```
vagrant box add --provider libvirt --name cisco-asav-9.9.2 ./cisco-asav.box
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details
